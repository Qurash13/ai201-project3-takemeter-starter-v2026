# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts **Hacker News comments** into four kinds: `evidenced`
(a point backed by something a reader could check), `opinion` (a position
with no checkable support), `quip` (a joke or bare reaction), and `question`
(a real request for information). HN is a good fit because one thread mixes
all four: someone posts benchmark numbers, someone replies "Google is dead",
someone makes a pelican joke, someone asks which version it was. The useful
distinction for a reader is the first one, and whether a small fine-tuned
model can find the comments that bring evidence.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `evidenced`

**Definition:** Makes a point and supports it with at least one specific a
reader could check without asking the commenter: a number, date, link, named
source, named product/version/event, or a concrete technical fact.

**Example 1:**
> \>most movies/TV are still 24Hz aren't they? — They are and that's part of
> the reason why it matters. 144/24 = 6, 120/24 = 5, 60/24 = 2.5. TVs
> compensate by showing you every 2nd image longer causing a slight but
> noticeable judder effect...

**Example 2:**
> Yup, and the "industry standard" wholesale discount publishers give to e.g.
> bookstores is usually 55%

### `opinion`

**Definition:** States a position, judgment or prediction in earnest, with no
checkable specific offered in support, however long or confident it is.

**Example 1:**
> Google is dead, it's impossible to find new sites.

**Example 2:**
> The Copenhagen interpretation doesn't even deserve to sit with the other
> interpretations because it's not well defined. It's protoscience, like
> alchemy.

### `quip`

**Definition:** A joke, sarcastic jab, or bare reaction (thanks, congrats,
agreement, eye-roll) that argues no position.

**Example 1:**
> No pelicans riding bicycles on this one, eh?

**Example 2:**
> rnicrosoft.com sure didn't need unicode tricks to fool people lol

### `question`

**Definition:** A comment whose main purpose is to ask for information or
clarification the commenter does not already have.

**Example 1:**
> What field did you go into?

**Example 2:**
> Link to an explanation of the C++26 change? I can't find it.

### The hardest boundary

**Which two labels:** `evidenced` vs. `opinion`

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->

If the comment names at least one specific that a stranger could look up or
verify without asking the commenter (a number, a date, a link, a named
source/product/version/event, or a concrete technical fact stated as fact),
**and** that specific supports the comment's point, it's `evidenced`. This
holds even if the tone is heated or sarcastic, and even if the number is
unsourced. Otherwise it's `opinion`, no matter how long, confident or
well-reasoned it is. Vague appeals ("research has shown", "go look at any
video") don't count. A first-hand report counts only if it names something
checkable ("Disney+ won't stream above 720p to Linux"), not if it's a feeling
("in my experience it's worse").

**Two secondary rules:**
- `quip` vs. `opinion`: if the comment's main move is a joke, sarcasm, or a
  bare reaction, it's `quip`. If it states a position in earnest, even in one
  sentence ("Google is dead..."), it's `opinion`.
- `question` vs. anything else: a rhetorical question that's really an
  assertion is labelled by what it asserts. *"I'm going to guess you didn't
  see Dhurandhar or Ne Zha 2 (the highest grossing films in India and
  China)?"* is `evidenced`. A question is `question` only when the commenter
  genuinely wants an answer.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Hacker News comments, pulled 2026-10-26
through the public HN Algolia API (`search_by_date`, comments only). That gave
816 comments of 3–140 words from about 30 hours of activity across many
threads. Each row's `note` keeps the HN comment id (`hn:<id>`) so it can be
traced. 250 comments were taken in random order. I dropped 8: link-only
"Related:" shares, a garbled fragment, a bare article quote, and one with no
content. A second, **targeted** pass added 44 more from the rest of the pool
(21 `question`, 16 `quip`, 7 `evidenced`), because `question` was thinnest
at 21. That pass picked short or question-mark comments on purpose, so those
two labels skew shorter than a random sample would.

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->
Claude pre-labelled every row using my definitions. The `note` column says
so on each one. For the 20 `cold` rows, Claude labelled 20 random rows again
without seeing the first labels. It matched itself 20/20. That only shows
the rules are consistent for one labeller. It's not a second person.
**I reviewed and approved every label in the dataset.**

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `opinion` | 134 | 46.9% |
| `evidenced` | 63 | 22.0% |
| `quip` | 47 | 16.4% |
| `question` | 42 | 14.7% |
| **Total** | 286 | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Research has shown there isn't any ambiguity here. Stuff like
> this isn't good for adults, and is even worse for kids. So yeah, obvious
> child abuse."
>
> *Could have been:* `evidenced` or `opinion`
>
> *I chose, because:* `opinion`. It appeals to research but names none, so
> a reader has nothing to check. That's exactly the vague appeal the decision
> rule excludes.

**2.**
> *The post:* "It didn't work so well for elementary school students in Iran."
>
> *Could have been:* `quip` or `evidenced`
>
> *I chose, because:* `evidenced`. It reads as a dark one-liner, but it
> points at a specific, checkable event to rebut the parent comment. The rule
> says tone doesn't matter if a checkable specific supports the point.

**3.**
> *The post:* "The server doesn't respond right now (2026-10-25T05:30:00Z).
> What happened? The wallet is at $2,016.73"
>
> *Could have been:* `question` or `evidenced`
>
> *I chose, because:* `question`. It has a timestamp and a dollar figure,
> but they aren't backing an argument. They're context for a real request
> for information.

**Also paused on:** "Something like 80% of workers are full time." I labelled
it `evidenced`: it's an unsourced number, but a stranger could check it.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** <!-- epochs, learning rate, batch size, seed -->
3 epochs, learning rate 2e-5, batch size 16, max length 128, seed 42.
Trained on **CPU** (codespace, torch 2.14.1+cpu, Python 3.13). The run took
about 5 minutes.

**Anything I changed from the defaults, and why:** Nothing. I wanted a clean
first run.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->
Train 200, val 43, test 43. Test has 10 `evidenced`, 20 `opinion`, 7 `quip`
and 6 `question`. `quip` and `question` are under 8 in test, so their scores
will jump between seeds.

**First look:** The model mostly guessed `opinion`. That's my biggest label.
I'll dig into why next unit.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked Claude (Claude Code) to read a community and
  propose candidate distinctions. I first picked r/mildlyinteresting, then
  r/nyu.

- *What I changed:* I choose Hacker News instead as the community. I didnt like what I found on reddit I also dropped
  "quotes and rebuts a parent" as a label, because it describes structure,
  not substance.

**Moment 2**

- *What I asked for:* I asked Claude to help me design the taxonomy 





<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.326 | 0.465 | +0.140 |
| Macro F1 | 0.273 | 0.159 | -0.114 |
| F1 — `evidenced` | 0.368 | 0.000 | -0.368 |
| F1 — `opinion` | 0.091 | 0.635 | +0.544 |
| F1 — `question` | 0.632 | 0.000 | -0.632 |
| F1 — `quip` | 0.000 | 0.000 | +0.000 |

Both models were scored on the same 43 posts in `test_split.csv`. The
baseline is `facebook/bart-large-mnli`, zero-shot, using my
`label_definitions.txt`. The trained model is the seed-42 run from unit 5.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I think my trained model will only beat the baseline on `opinion`, because
in unit 5 it mostly guessed `opinion`. I think the baseline will do better on
`question`, since questions are easy to spot from the definition alone.
(Written before running `baseline.py`.)

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

My prediction was right. My trained model only "wins" on accuracy because it
calls almost everything `opinion`, and `opinion` is about half the test set.
On macro F1 the baseline is better: it actually finds `question` (0.63) and
some `evidenced` (0.37), and my model finds neither. So in this run my
fine-tuning added nothing useful. It learned "guess the biggest label".

**Definitions vs. bare label names:** I also ran the baseline with no
definitions, just the label names. It scored **0.395** accuracy and 0.363
macro F1. That's better than with my definitions (0.326 / 0.273). My long
definitions seem to confuse the zero-shot model, mostly on `opinion`. The
version reported in the table above is the one with definitions.


---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.465 | 0.465 | 0.465 | MISSED |
| 2. Macro F1 | ≥ 0.50 | 0.159 | 0.159 | 0.159 | MISSED |
| 3. F1 — `evidenced` | ≥ 0.60 | 0.000 | 0.000 | 0.000 | MISSED |
| 4. Lowest per-label F1 | ≥ 0.30 | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Confident third minus unsure third (accuracy) | ≥ 0.15 | 0.571 | 0.500 | 0.786 | MET |

Device: **CPU** (codespace, torch 2.14.1+cpu). 3 epochs, lr 2e-5, batch 16.
I ran each seed in its own process, with the same notebook code. Running
all three in one process ran this codespace out of memory at seed 7.

The spread is 0.000 on every measure. That isn't a stable model, it's a
stuck one. It predicted `opinion` for every test post on all three seeds.
The split is stratified, so every seed's test set has 10/20/7/6 posts, and
"always `opinion`" scores the same each time.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42 (seeds 7 and 2024 are identical):

| true \ predicted | evidenced | opinion | quip | question |
|---|---|---|---|---|
| **evidenced** | 0 | 10 | 0 | 0 |
| **opinion** | 0 | 20 | 0 | 0 |
| **quip** | 0 | 7 | 0 | 0 |
| **question** | 0 | 6 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

All 10 real `evidenced` posts were called `opinion`, and 0 went the other
way. The model never learned the `evidenced`/`opinion` boundary, or any
boundary. Every column except `opinion` is empty.


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 | MISSED | 0.465 on all three seeds |
| 2 | Macro F1 | ≥ 0.50 | MISSED | 0.159 on all three seeds |
| 3 | F1 — `evidenced` | ≥ 0.60 | MISSED | 0.000 on all three seeds |
| 4 | Every label F1 | ≥ 0.30 | MISSED | Three labels at 0.000 on every seed |
| 5 | Confidence gap | ≥ 0.15 | MET | 0.571, 0.500, 0.786, all above 0.15. But see below: it's a hollow pass |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criteria 1–4: one cause. The model didn't train enough, so it collapsed
to my biggest label.**

How I know:
- **It predicts one label for everything.** The matrix has one column.
  That's not a confused boundary. Nothing was learned.
- **The loss barely moved.** Validation loss ended at 1.11. Pure guessing
  over 4 labels is about 1.39. With 200 training posts and batch 16, 3
  epochs is only 39 update steps at a small learning rate.
- **The labels aren't the problem.** My agreement check was 30/30, so I can
  apply a written rule the way its authors did. That's not full proof about
  my own 286, but it argues against messy labelling as the main cause.
- **The signal is there.** The zero-shot baseline, using only my
  definitions, got `question` F1 0.63 and `evidenced` F1 0.37. So these
  labels can be told apart from the text. My model just didn't get there.

So this is a training problem, not a labelling problem. Secondary cause:
`quip` (7) and `question` (6) are thin in test, so even a working model
would have jumpy scores on them.

**Criterion 5: MET, but hollow.** Every prediction is `opinion`, so
"confidence" only measures how opinion-like a post looks. The most confident
third is mostly real `opinion` posts, so it scores well. It passes the
number I set, but it doesn't show the model's confidence is useful.


---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

I had no disagreements. I labelled all 30 myself under the staff taxonomy
and matched staff on every post, including all 9 that staff marked as hard.

The hard ones I had to think about most:
- **sp_005** (carbon plates, then two numbers): I went `analysis`. The
  second sentence's numbers carry the argument (rule 1).
- **sp_014** ("Times are down about 3%... so obviously the shoes"): I went
  `hot_take`. The 3% is decoration on a conclusion already reached (rule 1).
- **sp_020** ("Every injury thread is the same injury"): I went `hot_take`.
  It could be checked in theory, but no count is given.
- **sp_028** ("Third DNF this year"): I went `reaction`. The number isn't used
  to argue anything (rule 3).

**What the pattern in my disagreements tells me:**

With no disagreements, there's no (a), (b) or (c) pile to sort. I think the
staff rules are just very tight: each hard post had one rule that clearly
decided it. This set also has three labels split 10/10/10 with clean
examples. That's easier than my HN data, where `opinion` vs. `evidenced` and
`opinion` vs. `quip` are much blurrier. So 100% here doesn't prove my own
286 labels are consistent. It only shows I can follow a written rule the way
its authors did.



---

## The Improvement

**What I changed:** One thing: learning rate **2e-5 → 5e-5**. Same data,
same labels, same 3 epochs, same three seeds, still on CPU.

**Which diagnosis pointed at it:** My diagnosis said the model undertrained:
it predicted only `opinion`, and the loss barely dropped. A higher learning
rate lets the same 39 steps actually move the model. I picked this over more
epochs because 10 epochs would have taken about 90 minutes on this CPU.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.605 | 0.628 | 0.558 | MISSED |
| 2. Macro F1 | ≥ 0.50 | 0.503 | 0.574 | 0.323 | MISSED |
| 3. F1 — `evidenced` | ≥ 0.60 | 0.286 | 0.375 | 0.625 | MISSED |
| 4. Lowest per-label F1 | ≥ 0.30 | 0.250 | 0.375 | 0.000 | MISSED |
| 5. Confident third minus unsure third (accuracy) | ≥ 0.15 | 0.286 | -0.071 | 0.571 | MISSED |

Confusion matrix after, seed 42:

| true \ predicted | evidenced | opinion | quip | question |
|---|---|---|---|---|
| **evidenced** | 2 | 8 | 0 | 0 |
| **opinion** | 1 | 18 | 0 | 1 |
| **quip** | 0 | 5 | 1 | 1 |
| **question** | 1 | 0 | 0 | 5 |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->

Yes, a lot, but not enough to meet any criterion. Accuracy went from 0.465
on every seed to 0.56–0.63. Macro F1 went from 0.159 to 0.32–0.57. The model
now predicts all four labels, and `question` reached 0.77 F1 on two seeds.
Seeds 42 and 7 now beat the baseline on macro F1 (0.503 and 0.574 vs. 0.273).

But every criterion is still MISSED. Criterion 5 went from MET to MISSED:
once the model started making real guesses, seed 7's confident predictions
were slightly *worse* than its unsure ones. The spread also grew, from 0.000
to 0.070 in accuracy and 0.251 in macro F1. Seed 2024 never predicted `quip`
or `question` at all. So the fix worked on the cause I named, but the result
depends heavily on the split.


---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

- **1. Accuracy (0.56–0.63, target 0.65):** Close. I'd train for more
  epochs at the new learning rate. I stopped because each run on CPU takes
  about 30 minutes, and the course allows one change.
- **2. Macro F1 (seed 2024 at 0.323):** One bad seed sinks it. Seed 2024
  missed `quip` and `question` entirely. I'd add about 30 more examples of
  each, so the model sees them in every split.
- **3. `evidenced` F1 (0.29–0.63):** The model still sends most `evidenced`
  posts to `opinion` (8 of 10 on seed 42). I'd add more `evidenced` examples
  and check whether my rule about "a concrete technical fact" was applied the
  same way across all 63.
- **4. Lowest per-label F1 (0.000 on seed 2024):** Same cause as 2. Only 6–7
  test posts per thin label, so a seed can miss them all.
- **5. Confidence gap (-0.071 on seed 7):** With 14 posts per third, a few
  posts swing this. More test data would make it measurable. I stopped
  because more data is a second change.

I stopped because I'd made my one allowed change and measured it.


**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

I meant `evidenced` to catch any comment that backs its point with something
checkable, but the model mostly learned "long and serious = `opinion`", and
sent 8 of 10 `evidenced` posts there. It learned `question` well because
questions have an obvious surface cue (a `?`), but evidence in HN comments
is often a link, a number or a technical detail buried in a normal-looking
sentence, and 43 examples weren't enough to teach that.


<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
