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
Every row was pre-labelled by Claude against the definitions above. The
`note` column says so on each row. **Still to do by me:** label 20 rows cold
(marked `cold`), then read and correct every pre-label.

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

**Base model:**

**Settings:** <!-- epochs, learning rate, batch size, seed -->

**Anything I changed from the defaults, and why:**

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->



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
- *What came back:* Reddit blocked every automated read (403). Claude
  switched to Hacker News through its public API, read 48 comments, and
  proposed four distinctions with real quotes (see `notes/milestone1.md`).
- *What I changed:* I accepted Hacker News as the community. I also dropped
  "quotes and rebuts a parent" as a label, because it describes structure,
  not substance.

**Moment 2**

- *What I asked for:* I asked Claude to design the taxonomy and pre-label
  the dataset.
- *What came back:* Three labels planned from the first reading. Once it
  read the actual data, it added `question` because genuine questions were
  common and fit none of the others. It also caught and fixed a bug in its
  own labelling script that had swapped `quip` and `question`.
- *What I changed:* <!-- fill in after your check pass: which labels you
  overturned, and how many -->

**Pre-labelling disclosure:** All 286 rows were pre-labelled by Claude, and
the `note` column marks every one. <!-- Update once done: "I labelled 20
rows cold first (marked `cold`), then read and corrected all pre-labels,
changing N of them." -->

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

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

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
