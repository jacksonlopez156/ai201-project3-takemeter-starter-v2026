# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy on the held-out test set is at least 0.65 across all three seeds.

A classifier that gets fewer than 2 in 3 posts right on a three-class problem is not meaningfully better than an informed guesser. With three roughly balanced labels a random guesser sits around 0.33, and a pattern-matcher that just memorises the most common label hits about 0.38–0.42 on this dataset. I set the bar at 0.65 because that is the point where the model would plausibly save a reader time sorting posts — below that, the error rate is high enough that you'd be correcting it constantly.

**Why this target:** The running forum has three labels roughly balanced at about 33% each (analysis ≈ 35%, hot_take ≈ 35%, reaction ≈ 30%), so a majority-class guesser scores around 0.35. 0.65 is roughly double that, which is the minimum I'd call useful. The target has to hold across all three seeds, which is stricter than it looks — a lucky seed can push accuracy to 0.70 even on a weak model.

---

## 2. Every label's F1 score on the held-out test set is at least 0.55 across all three seeds.

Per-label F1 catches cases where overall accuracy looks fine but one label is being systematically ignored. `reaction` is my smallest label at about 60–65 examples out of 200, which puts roughly 9–10 posts in the test split. A model that assigns zero `reaction` predictions would still score about 0.70 overall accuracy on this dataset, so criterion 1 alone is insufficient.

**Why this target:** I set 0.55 rather than 0.60 because `reaction` has the fewest examples and its test split will only have about 9 posts, meaning a single misclassification swings F1 by about 0.10. The `hot_take`/`reaction` boundary is the hardest in this taxonomy (both are unsupported claims; one is personal, one is general), so I expect `reaction` to be the weakest label. 0.55 is the floor below which the label is effectively unused.

---

## 3. The trained model beats the zero-shot baseline on overall accuracy across all three seeds.

Fine-tuning on 200 labeled examples should outperform a general model reading only the label definitions. If it doesn't, the training data added no signal — which would be a real finding about the quality of my labels or the difficulty of the task, but not a success.

**Why this target:** This is a relative criterion rather than an absolute number. The baseline is the honest opponent: it has the same label definitions and the same posts, just no training examples. If the fine-tuned model can't beat it on the same held-out posts, the 200-example training set did not improve on the prior. I expect the gap to be at least 0.05 in accuracy because fine-tuning on even a small, consistent dataset tends to pull the model toward the specific vocabulary and phrasing of the community.

---

## 4. The precision of the `analysis` label on the held-out test set is at least 0.70 across all three seeds.

`analysis` is the most distinctive label in this taxonomy — it requires a specific, checkable fact. If the model calls a post `analysis`, it should be right at least 70% of the time, because false positives on `analysis` are the most misleading error (a reader seeking fact-backed posts and getting unsupported opinions). I care about precision more than recall here because the cost of a false `analysis` label is higher than missing one genuine analysis post.

**Why this target:** `analysis` has the clearest boundary — a post either cites a checkable fact or it doesn't — so the model should learn it more reliably than the fuzzier `hot_take`/`reaction` boundary. I expect precision to be achievable at 0.70 because the presence of numbers, dates, and statistics is a strong lexical signal that a bag-of-words approximation could exploit. 0.70 is high enough to be useful without demanding perfection on a 200-example dataset.

---

## 5. Inter-annotator agreement between my labels and the staff set is at least 0.70 (21 of 30 posts matching) under the staff taxonomy.

Consistency is the prerequisite for a trainable signal. If I can't apply a written taxonomy reliably enough to agree with the authors 70% of the time, the 200 labels I produced in unit 5 are too noisy to train on. The staff taxonomy uses the same three labels (`analysis`, `hot_take`, `reaction`) with explicit decision rules, so 70% agreement is a realistic and honest floor — the staff note that a rate in the 70s is normal for these posts.

**Why this target:** The staff taxonomy document itself says "a rate in the 70s is normal and expected." I set 0.70 (21/30) as the floor rather than something higher because several of the 30 posts were specifically chosen for their ambiguity. Expecting 90%+ agreement on deliberately ambiguous posts would mean I was not reading the decision rules carefully, just defaulting to majority-class guesses. 0.70 is the minimum that would make me confident my unit 5 labels were applied consistently.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
