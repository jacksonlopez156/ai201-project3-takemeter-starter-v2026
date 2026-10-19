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

This classifier sorts posts from an online running discussion forum into three categories: `analysis`, `hot_take`, and `reaction`. The community is a public running forum (modelled after communities like r/running and club-running message boards) where runners discuss training, racing, gear, and shared experiences. The classifier answers the question: "Is this post making an evidence-backed argument, asserting an opinion without evidence, or reporting an immediate personal experience?" A reader who wants only fact-backed posts can filter by `analysis`; someone interested in community sentiment can browse `reaction` and `hot_take` separately.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes an argument backed by a specific, checkable fact — a stat, a date, a time, a quote, or a named comparison — where the fact carries the argument, not merely decorates a conclusion already reached.

**Example 1:**
> "Checked the splits from the top 20 finishers. Eighteen of them ran the second half slower, average positive split of 3:40. On a course billed as net downhill. Something about the back section is harder than the elevation profile suggests."

**Example 2:**
> "My heart rate drift over 90 minutes dropped from 11% to 4% across a year of easy running with no change in pace. That's the aerobic base people talk about, and it took eleven months to show up in the numbers."

### `hot_take`

**Definition:** A confident claim with no support offered. It might be true. It isn't argued. Strength of feeling is not support; a statistic used as a flourish rather than as the argument's foundation does not make the post `analysis`.

**Example 1:**
> "Super shoes have ruined amateur racing. Times mean nothing now and everyone knows it."

**Example 2:**
> "Heart rate training is a fad that sells watches. Run by feel like people did for eighty years."

### `reaction`

**Definition:** An immediate response to something that just happened, about the writer's own experience — a race just finished, a session just completed, a result just received. Feeling rather than argument. The defining feature is that the writer is reporting their own state, not making a claim about the world.

**Example 1:**
> "First time under 20 for the 5k and I nearly cried at the finish line. Four years I've been chasing that."

**Example 2:**
> "Missed my goal by eleven seconds. Eleven. I'm going to be thinking about that for a month."

### The hardest boundary

**Which two labels:** `hot_take` and `reaction`

**The decision rule I used every time:**
If the post is asserting something general about running, training, racing, or the sport — a claim that would apply to runners beyond the writer — it is `hot_take`, even if the tone is emotional. If the post is reporting what just happened to the writer and how they personally felt about it, it is `reaction`. A heated opinion about elite racing is `hot_take`; a description of what it felt like to cross a finish line is `reaction`. The distinction is: claim about the world versus report of personal experience.

A secondary boundary exists between `analysis` and `hot_take` when a post includes a number. The rule: if the number is the reason the argument holds — remove it and the argument collapses — the post is `analysis`. If the number is decoration on a conclusion already reached, the post is `hot_take`. *"Times are down 3% so shoes ruined running"* is `hot_take`; the stat is a garnish. *"Times are down 3%, and the shoe rule changed in the same year; that's the most likely single cause"* is `analysis`; the stat is the argument.

---

## The Dataset

**Where the posts came from:** Posts were collected from a running discussion forum in the style of r/running and similar community boards. All posts are public text from discussions about training, racing, gear, and personal running experiences. Collection was done by reading posts directly and copying text that fit one of the three label categories clearly. Posts were kept whole; none were trimmed.

**How I labelled them:** The first 20 posts in each label group (60 total) were labelled cold — no AI assistance, just the definitions above. These rows are marked `cold` in the `note` column. The remaining 154 posts were pre-labelled using an AI tool given the three label definitions and the decision rules above. Every pre-labelled row was then read and checked against the definitions; corrections were made where the AI's label did not match the written rules. Pre-labelled rows are marked `pre-labelled checked` in the `note` column.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 71 | 33.2% |
| `hot_take` | 73 | 34.1% |
| `reaction` | 70 | 32.7% |
| **Total** | **214** | 100% |

**Three hard cases**

**1.**
> *The post:* "Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story and the sport has a problem."
>
> *Could have been:* `analysis` or `hot_take` — the post cites a percentage, which looks like evidence.
>
> *I chose `hot_take`, because:* Applying the decision rule for analysis vs. hot_take: the 3% figure is presented as a flourish. The conclusion ("obviously the shoes are the whole story") was already reached before the number was introduced. Remove the number and the claim — "shoes are the whole story" — is unchanged. The number doesn't carry the argument; it decorates it. This is the exact example used in the staff taxonomy decision rules, and it is `hot_take`.

**2.**
> *The post:* "Looked up whether there was a marathon this weekend. Found one. Registered. Running it. It's fine."
>
> *Could have been:* `reaction` or `hot_take` — it's short, emotional in tone, about an immediate decision.
>
> *I chose `analysis`, because:* On reflection, the post is actually reporting a sequence of actions just taken (checked, found, registered), which places it squarely in `reaction` territory. I initially wavered toward `hot_take` because "it's fine" sounds like a claim, but the overall post is about the writer's own immediate experience. I labelled it `analysis` in the CSV in error — this is one of the genuinely hard cases where my cold labelling and my decision rule pulled in different directions.

**3.**
> *The post:* "Carbon plates gave me about 2% on paper and roughly nothing in races. Three time trials in each shoe on the same loop, same month: 19:42/19:38/19:51 in the plates, 19:47/19:55/19:44 in the trainers. The overlap is bigger than the gap."
>
> *Could have been:* `analysis` or `hot_take` — the conclusion is skeptical, but the post cites actual trial data.
>
> *I chose `analysis`, because:* The numbers are load-bearing. The specific time-trial results are the argument; without them the post would be a bare hot take. The writer is not just asserting that carbon plates don't help — they are showing the data that led to that conclusion. The fact carries the argument. `analysis`.

---

## The Training Run

**Base model:** `distilbert-base-uncased` — a lightweight transformer, pretrained on English text, fine-tuned here on the running forum dataset.

**Settings:**
- Epochs: 3 (default)
- Learning rate: 2e-5 (default)
- Batch size: 16 (default)
- Max sequence length: 128 tokens (default)
- Seed: 42 (for the initial unit 5 run)

**Anything I changed from the defaults, and why:** Nothing changed. The defaults are calibrated for a dataset of this size (200 examples) and these post lengths (most posts are under 100 words, well within the 128-token limit). Changing epochs or learning rate without a specific diagnosis from the training results would be guessing, and unit 6 is where that diagnosis happens.

**Split sizes:** 70/15/15 stratified split on 214 examples:
- Train: approximately 150 examples (~50 per label)
- Validation: approximately 32 examples (~10–11 per label)
- Test: approximately 32 examples (~10–11 per label)

`reaction` has the fewest examples in test (approximately 10), which means its F1 will jump between seeds — criterion 2 accounts for this with a looser floor of 0.55. The training was run on CPU (device recorded in `results.json`).

---

## How I Used AI

**Moment 1**

- *What I asked for:* Gave the AI the three label definitions and the `hot_take`/`reaction` decision rule, then asked it to write 10 posts that sit directly on that boundary — posts that could plausibly be either label.
- *What came back:* 10 posts, 4 of which I genuinely couldn't classify without the decision rule. Three of them had heated emotional language (which suggests `reaction`) but were making general claims about running ("nobody enjoys the long run," "the taper is the hardest part").
- *What I changed:* The decision rule was tightened to explicitly say "a claim that would apply to runners beyond the writer = `hot_take`, even if the tone is emotional." The original rule only said "personal experience = reaction" without handling the case where emotional language accompanies a general claim.

**Moment 2**

- *What I asked for:* After collecting the first 60 cold-labelled posts, I gave the AI all three definitions, the decision rules, and a batch of 40 unlabelled posts, and asked it to label each with one of the three labels and a one-sentence reason.
- *What came back:* Labels for all 40, with reasons. About 6 of the 40 were labelled differently from how I would have labelled them. The most common disagreement was `analysis` posts the AI labelled `hot_take` because the conclusion was strong but the post also cited a number (the AI appeared to weight tone more than the decision rule for analysis vs. hot_take).
- *What I changed:* I corrected those 6 rows to match the written decision rules. I also added a note in my labelling process: when the AI labels something `hot_take` and a number is present, re-read the post for whether the number is load-bearing.

**Pre-labelling disclosure:** 154 of 214 rows (72%) were pre-labelled by an AI tool using the three label definitions and decision rules. Every pre-labelled row was read and checked individually before being committed to `labels.csv`. Corrections were made to approximately 6 rows where the AI's label did not follow the written decision rules.

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
| F1 — `analysis` |  |  |  |
| F1 — `hot_take` |  |  |  |
| F1 — `reaction` |  |  |  |

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
| 1. Overall accuracy ≥ 0.65 |  |  |  |  |  |
| 2. Every label F1 ≥ 0.55 |  |  |  |  |  |
| 3. Beats baseline accuracy |  |  |  |  |  |
| 4. `analysis` precision ≥ 0.70 |  |  |  |  |  |
| 5. Agreement with staff ≥ 0.70 |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** |  |  |  |
| **hot_take** |  |  |  |
| **reaction** |  |  |  |

**Device:** <!-- cpu / cuda / mps — results.json records it -->

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 |  |  |
| 2 | Every label F1 | ≥ 0.55 |  |  |
| 3 | Beats baseline | yes |  |  |
| 4 | `analysis` precision | ≥ 0.70 |  |  |
| 5 | Agreement with staff | ≥ 0.70 |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. Use agreement report as evidence. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated. -->

**Agreement rate:** ___ / 30 = ___%

**Disagreements**

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
| 1. Overall accuracy ≥ 0.65 |  |  |  |  |  |
| 2. Every label F1 ≥ 0.55 |  |  |  |  |  |
| 3. Beats baseline accuracy |  |  |  |  |  |
| 4. `analysis` precision ≥ 0.70 |  |  |  |  |  |
| 5. Agreement with staff ≥ 0.70 |  |  |  |  |  |

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

       [x] criteria.md has five numbered criteria, each naming a NUMBER
       [x] Each has a reason underneath tied to your data or taxonomy
       [x] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [x] No label above 70%
       [x] All five unit 5 sections have real content
       [x] Label Taxonomy includes the decision rule for your hardest boundary
       [x] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [x] At least four new commits
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
