# AI Use Disclosure — PA1

_Generative AI is allowed on PA1 as a tutor, reviewer, debugging partner and
helper with tooling. It is not allowed to be the author of your work. See the PA1
brief, "AI, Authorship and You". Fill in every section. "None" is an acceptable
answer only if it is true._

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [X] Yes, as described below.

## 2. Statement (the syllabus form)

> The author acknowledges the utilization of Claude Opus 5.5, a language model
> developed by Anthropic, in the preparation of this assignment. Claude
> was employed in the following manner(s) within this assignment: finding libraries,
> refining the structure of server.py, reviewing the code, help with run()

## 3. Log

_One row per substantive interaction. Summaries are fine; you do not need to paste
whole conversations, but keep them in case you are asked._

| # | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
|---|---|---|---|---|
| 1 |What libraries should I be using |I used most of the libraries that it gave me |I did not use the struct library |I looked Googled the libraries to see what they are about |
| 2 |How should I structure server.py? Should I use any additional helper functions? |I did not use helpers I just used the format that was given |I did not reject the recommendations made |It made sense to structure it this way |

## 4. Where AI-produced material remains

_List each file and function (or document section) that still contains code or
text an AI tool generated, even after you edited it. If none, say none._

server.py: handle_session(), main(). client.py: run()

## 5. One thing the AI got wrong — or how you checked it

_Describe one answer, suggestion or piece of code that was wrong, misleading or
did not fit the specification, and how you caught it. If you found nothing wrong,
say so and describe how you checked (a test you ran, the part of the
specification you compared it with). If you did not use AI, describe one thing
you got wrong yourself and how you caught it._

The AI told me that I need the struct library, but it was not necessary. You can read the 2 byte payload in the PUT without it. 
