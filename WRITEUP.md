# COMP 440 HW3 write-up

Name: Nadezhda Dominguez Salinas

Type each answer after the `>` under its question. An answer can run over several lines, but a blank line
ends it: to start a new paragraph, put a line with just `>` between the two. In a table, fill every cell
of every numbered row. Claude may paste a program's output where a question asks for it, but never an
answer. The homework page ticks a step when every answer in its section is filled in.

## Learning goals

In this homework you will:

- Understand what makes a program an *agent*. You will study a language model that picks the next Wikipedia search query, reads the page that comes back, and decides when to stop.
- Gain hands-on experience with the loop around the model by writing it yourself: its tools, its instructions, and its limit of 12 lookups.
- Compare the agent with two simpler methods on your claims, and measure how effective each one is.
- Check the agent's evidence yourself, and decide whether accuracy alone is a fair measure of it.

For extra credit, in Part 3, you find out why the agent fails on your claims, make one change, and see where Wikipedia, which is written by its volunteer editors, is thin, out of date or wrong.

# Part 0: Set up and check one claim

## 0a. Get your copy running

There is nothing to write for this step except your name, after `Name:` at the top of this file.

## 0b. Ask the model to assess a claim without tools

What verdict did the model give (*supported*, *refuted* or *doesn't say*), and what reason did it give? Quote the lines of the output that show them.

> The verdict it provides is that "doesn't say", the reason was " I cannot confirm from memory who composed "Persian Surgery Dervishes" or find a Wikipedia article that gives that composer's birth date, so I can't verify the claim either way".

## 0c. Give it one search

Which Wikipedia page did it read? Quote the line of the output that shows it.

>"Page: Persian Surgery Dervishes\n\nPersian Surgery Dervishes is a recording of two live solo electric organ concerts, the first held in Los Angeles on 18 April 1971 and the second in Paris on 24 May 1972, by the avant-garde minimalist composer Terry Riley, following his A Rainbow in Curved Air and In C.\n\nSections: Background and release, Music, References, External links"

Did that page settle the claim correctly? If not, why do you think the search led there?

>No it did not because the verdict was "doesn't say" implying that the page did not resolve the claim's verdict. I think the search led there because the first three words of the search matched the title of the page, of "Persian Surgery Dervishes".

## 0d. Run the agent and read its trace

In one or two sentences, how did the agent get from the claim to its verdict? Point to the trace entries by their numbers.

> It first did a search in it's Number 1 trace, and it's query was "Persian Surgery Dervishes composer", then the second trace included a read of the page "Persian Surgery Dervishes" to find the composer Terry Riley, third trace was also a read but to a different page titled "Terry Riley" that stated when they were born, where in the fourth trace was the answer that stated the verdict was supported.

Which lookup would you have done differently, and how? If none, which one surprised you?

> I would changed the first lookup trace of search, specifically to re-word it as a question to get a more related page, "Who is the composer of Persian Surgey Dervishes and when were they born". This is a a very direct question that would hopefully result in a page that holds the answer sooner than in this example.

Describe Wikidata briefly. Why may it be important to this task?

> Wikidata consists of a free database of facts, terefore it stores each fact as three parts, an item, a property, and a value. It is important in this task because from Wikidata we could gather te composers date of birth.

# Part 1: How the agent works

## 1a. Design the loop

Your loop should repeatedly call the model with the message history so far, and ask for its next step. It should check whether the model has given an answer yet. If not, it should make the lookups that step asks for, and add what they return to the history. It should also pay attention to whether the loop has reached its limit: a claim that runs out of lookups cannot be decided. One limit is set for you: 12 lookups for a claim.

The Background and an example outline are under 1a on the homework page, and in [instructions/part1.md](instructions/part1.md#1a-design-the-loop).

Write your outline of the loop in the code block: what it remembers, what it repeats, and every way it stops.

```
Remember: The message history (so far), what is the total lookup limit (12 in this case), and which number of lookup we are currently on (none yet)

while Claim is non empty:
    Call the model with the message history so far, and ask for its next step
    If the model has an answer:
        Reply with that answer
        Stop
    Else:
        If the lookup count is at the limit or one above the limit:
            Reply with "No Verdict"
            Stop
        For each lookup asked for
            Add 1 to the lookup count
        Once every lookup has its result add what they returned to the message history
```

List each thing your loop remembers from one turn to the next, and say what would go wrong without it.

> Each thing that my loop remembers from one turn to the next is the message history (so far), the total lookup limit (this case we are given 12), and the number we are currently on out of the lookup limit. Without the lookup limit then the model would just continously continue to run until it comes up with an answer but this would delay a reply and add complexity if we have alot of claims, so having a limit allows us to constrain the model. Additionally we must have message history incoporated so the model can troubleshoot what it already attempted and work from there, instead of starting from a blank canvas everytime to validate a claim.

Now open `reference_loop.py` and compare it with your loop. Describe one difference in implementation and the resulting effects on agent behavior.

> Comparing both loops I can see that they are kind of similiar in structure except the referene loop looks more concise. One difference in the implementation is that in my loop I put the logic of "for each lookup asked for (by the model)" in the else branch if the model has no answer, while the reference loop does not have the logic of whether or not the model asked for a lookup, it assumes it does, so the resulting effect on the agent is that it would loop over the reply.lookups even if there was not really any lookups asked for by the model.

## 1b. Be the agent

The claim: "Vic & Anthony's Steakhouse is located at the American casino that was combined with the California Club. The casino is owned by Landry's, Inc."

List your lookups, in order. Ask Claude to paste every column but the last from `runs/part1/lookups.jsonl`. Type "Why you chose it" yourself.

| # | Tool | What you asked for | Why you chose it |
|---|---|---|---|
| 1 | search | "Where is Vic & Anthony's steakhouse located and what is the history behind the location" |I chose this as the first lookup to try to get a sense of whether the claim that the resturaunt was insife the American casino was correct.|
| 2 | search | "Is Vic & Anthony's Steakhouse located at the American Casino?" | From the first lookup, none of the results helped validate the claim, so I wanted to come up with a more specific search that moight lead to a clearer answer, so I wanted to ask a very binary Yes or No question to the location. |
| 3 | read | "Landry's" (no section) | I chose the read tool because from the second lookup the first result seemed to match the description of Landry's, Inc company claim, so I wanted to look into this company.|
| 4 | read | "Landry's", section "Golden Nugget casinos" | I chose to keep reading into this section because I was confused on why American casino was not an option, and instead only found Golden Nugget casinos. |
| 5 | wikidata | entity "Vic & Anthony's Steakhouse", property "location and owner history" | My previous lookups were not very good at validating the claim, so I attemped to pivot to wikidata. |
| 6 | search | "Who owns Vic & Anthony's Steakhouse" | I wanted a clear answer on the owner of the resturaunt.|
| 7 | search | "Is it true that Vic & Anthony's Steakhouse is located at American Casino?" | Next, after validating that Landry's, Inc owns this restuaurnt I move to focus on location of it.|
| 8 | read | "List of restaurants in the Las Vegas Valley" (no section) | I did this lookup because I thought the list of restuarants would have a location attatched to Vic & Anthony's Steakhouse.|
| 9 | search | "restuaruants located at the American Casino" |I chose the route of backtracking whether or not the resturaunt was inside American casino.|
| 10 | search | "Is Vic & Anthony's Steakhouse inside of a casino?" |Lastly, I chose this because the other lookups were unclear of the location, so I wanted to start off again of whether it was even inside of a casino to begin with. |

What is your verdict on the claim, and which sentence or sentences from what you read is it based on?

> My verdict is that the claim is refuted by the lookups the model made. Specifically, I read in lookup 7 the first result was "List of restaurants in Las Vegas Valley: Vic & Anthony's Steakhouse", and in another lookup it states Vic & Anthony's is an Americna steakhouse restaurant chain with locations in Downtown Houston, and inside the Golden Nugget Casinos in Las Vegas, Atlantic" , therefore refuting that it is located at the American Casino.

## 1c. Watch the agent work

List the three traces you read closely. At least one should be a claim the agent got wrong (if it got any wrong).

| # | Claim (first few words) | Articles | Did it match the dataset's label? | Did it get there for a good reason? Why or why not |
|---|---|---|---|---|
| 1 | Myles Kennedy and Robert Palmer, share the mutual occupations of musician, singer, songwriter|2 articles | Yes it matched the dataset's label.| Yes it got there for a good reason as its qouted evidence are real and they match each of the two person's background and profession, therefore relevant about the claim that Myles and Robert share mutual oocupations.|
| 2 | The quartets that released One Thousand Years of Trouble and Vector were both a rock band.|3 articles | Yes it matches the dataset label of not supported as the agent's verdict is refuted.  | Yes it got there for good reason because it states the claim is refuted for two reasons of neither band being a quartet and that one is a rock dance band verses the other which is more of a progressive metal band.|
| 3 | This Singapore freestyle was referenced in the song Detail (record producer) is known for producing. They were undefeated since the Athens games with 28 medals.| 4 articles| This does not match because the datasets label is not supported, meanwhile the agent gave no verdict.| No it did not get there for good reason and the agent ran out of steps. The traces for this claim were kind of all over the place going abck and forth of searching song lyrics to then someone with 28 medals.|

Which trace did you find most interesting, and what did the agent do well or badly in it?

> The trace I found the most interesting was in the second claim of the lookup 6 of the agent where it read "Vector(band)". This was interesting because in the claim "Vector" is not a band itself but rather an album, therefore it is interesting on why the agent would chose to read this section. I think it should have pursued a different lookup, especially because an earlier lookup of search "Vector album band released" gave a response that Vector is a studio album by Haken, therefore no further information on this part of the claim was need, aka unecessary lookup on behalf of the agent.

## 1d. Write the agent's instructions

What changed in the agent's traces after you wrote your instructions? Give one claim as an example.

> The main thing that changed in the agent's traces is the amount of entries that it has. For example, I looked into the claim "Scotland holds both this landmark and Cowie Castle. The landmark is located in East Catterline". The agent's trace in the before run has 13 entries: 12 lookups and 1 answer, versus the after run which has only 7 entries: 6 lookups and 1 answer. 

## 1e. Start the full run, and predict

Predict the share of claims each version will get right (its verdict matches the dataset's label), as a percentage. A guess is fine; the reasons are what I grade.

| Version | 2 articles | 3 articles | 4 articles | Why you think so |
|---|---|---|---|---|
| No tools |40 | 20|10 | Having no tools is more dificult at proving a claim, therefore I placed the percentage of claims right with no tools to be less than 50 percent, and it will decrease with the more artices involved as the model is only having what it remembers from its training, so it is more likely to be incorrect than the one search or agent.|
| One search | 50|45 | 30| One search can have some evidence to help prove a claim is right, a claim that has less articles involved, will have a higher claim right for a one search, because a claim that involves 4 articles and only gets one search has less of a chance of collecting evidence to prove claim is correct.|
| Agent | 90| 80|70| I think the agent will do a good job at getting the claim right when it has less articles involved in the claim because it has less things to validate from, so it is more specific, but the more articles involved in the claim might be more difficult to focus on what evidence it needs for claim|

How many lookups will the agent make per claim, on average? Why?

> I predict that the agent will make 8 lookups on average. The reason is because not all claims are very difficult to validate so some might take very few lookups such as 5 or 6 lookups, meanwhile other might take 11,12, or 13 to validate, and for me the in between number is 8 lookups on average that the agent will need.

# Part 2: The agent against the baselines

## 2a. Compare accuracy

Ask Claude to paste the report of your full run, from `runs/part2/report.txt`, into the empty block below.

```
```

How do the results compare with your predictions? Where were you furthest off? Use the numbers from your run, by number of articles.

>

Does the gap between the agent and the one-search baseline grow with the number of articles? What in the claims or the traces explains what you see?

>

The no-tools baseline read nothing. How did it do, and what does that tell you about these claims?

>

## 2b. Check the evidence yourself

Fill in one row for each of the first five claims in your full run, in the order of the claims file, where the agent's verdict is *supported* or *refuted*.

| # | Claim (first few words) | Agent's verdict | Matched the dataset's label? | Does the quoted evidence support that verdict? | Why or why not |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

Of the five, how many matched the dataset's label, and for how many of those does the quoted evidence support the verdict?

>

Is accuracy alone a good measure of this agent? Why or why not?

>

# Part 3 (extra credit): Why the agent fails, and one change

## 3a. Sort the failures

Fill in one row for each of the first eight failures in your full run, rows 1 to 8 of the claims table for runs/part2 with "Result" in "The agent" box set to "does not match the dataset's label (any reason)". If you have fewer than eight, fill a row for each and delete the rest.

| Row | Claim (first few words) | Your label | The entry of the trace that shows it | Is the dataset's label right? (yes, no or can't tell) | Claude's label |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |
| 7 | | | | | |
| 8 | | | | | |

Where did you and Claude disagree, and who do you think is right?

>

Which kind of failure was most common? If no kind repeats, say which failure you will aim at in 3b, and why.

>

## 3b. Make one change

What did you change, and why did you expect it to help?

>

How many of your evaluation claims did the agent match before the change and after it? Which of your failures from 3a now match?

>

Quote one claim the agent matched before the change and not after it, if there is one, and say what you think happened. If there is none, say so.

>

## 3c. Write it up

What does the agent's loop add over one search, what does it cost, and when would you not use an agent for a job like this? Use your numbers from Part 2 and Part 3.

>

What did your claims show you about Wikipedia as a source? Where it was thin or wrong, why might that be, given who writes it?

>

Your numbers rest on 100 claims. Which of your conclusions would you trust on another 100 claims, which might be chance, and what would you measure to find out?

>
