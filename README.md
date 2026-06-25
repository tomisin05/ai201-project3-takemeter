# TakeMeter

A text classifier that sorts **r/worldcup** posts by _what kind of take they
are_ — not what they're about, but the **rhetorical mode** of the contribution.
Every post is one of four types: reasoned `analysis`, a bare `hot_take`, a
`prediction`, or a practical `logistics` question.

The project compares two classifiers on the same task: a **zero-shot LLM
baseline** (Groq `llama-3.3-70b-versatile`) and a **fine-tuned DistilBERT**. The
zero-shot baseline is already deployment-grade (**90%** accuracy). The fine-tuned
model tells a two-part story: a first pass at the default 3 epochs **collapsed**
onto the majority classes (45% accuracy, macro-F1 0.28), and raising training to
**15 epochs recovered the minority classes and lifted it to 61% accuracy
(macro-F1 0.63)** — a large, diagnosable improvement that still trails the
baseline. The most useful output is not a single accuracy number but a clear map
of _which boundary each model can and cannot learn at this data scale_.

> 🎥 **Demo video:** https://drive.google.com/file/d/1yK56LO76YB6Cn3TTjgRuEiwSVunzcB2P/view?usp=sharing
> 📋 **Design notes / working log:** [PLANNING.md](PLANNING.md) (Milestones 1–6)

---

## 1. The task and why this community

r/worldcup during the live 2026 tournament is text-heavy and varied: the same
surface topic ("this team / this match") is expressed as four structurally
different kinds of post. The _subject_ barely varies but the _mode_ does, so the
classifier has to learn the **shape of the contribution**, not topic keywords —
a harder and more interesting signal than "is this about football."

### Labels

| Label        | Definition                                                                                                                                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `analysis`   | Argues a point about the football using **specific, load-bearing evidence** — match events, tactics, formations, player roles, stats, history. Includes starting-XI / formation posts. The reasoning _is_ the substance. |
| `hot_take`   | A bold subjective **opinion, grievance, or reaction** asserted with little or only cherry-picked/decorative evidence.                                                                                                    |
| `prediction` | A **forward-looking forecast** of an outcome — who wins a match/group/tournament, a knockout result, a scoreline.                                                                                                        |
| `logistics`  | A **practical or community-operations** question/help post — tickets, travel, stadiums, kits, broadcasts, schedules, standings/qualification math.                                                                       |

Full definitions, two real examples per label, and the hard annotation edge-case
rules are in [PLANNING.md](PLANNING.md) (Milestones 1 and 3).

---

## 2. Dataset

- **206 labeled examples**, all real public r/worldcup self-text posts.
- Collected via two read-only Reddit data APIs (**Arctic Shift** for recent 2026
  posts, **PullPush** for older + targeted keyword queries) from a deduped pool
  of 1,343 candidates; each candidate was read individually and labeled, dropping
  image-only/news/meme/megathread/spam/off-topic posts.
- **Annotation:** each post was pre-labeled by **GPT, Gemini, and Claude**, then
  given a human **"My Final Label"** (the ground truth). See the AI usage section.
- Source file: [`data/worldcup_labels.csv`](data/worldcup_labels.csv) (text +
  the three model pre-labels + final label). Clean training file:
  [`data/worldcup_takes.csv`](data/worldcup_takes.csv) (`text`, `label`).

### Label distribution (final, n=206)

| Label        | Count | Share |
| ------------ | ----: | ----: |
| `logistics`  |    64 | 31.1% |
| `hot_take`   |    62 | 30.1% |
| `analysis`   |    44 | 21.4% |
| `prediction` |    36 | 17.5% |

No class exceeds 70%. `analysis` is the genuinely rare class in this community
(~10% on the first labeling pass); a targeted tactical-vocabulary sweep plus
light down-sampling of the majority classes brought it to ~21% without padding
the boundary with ambiguous posts (see PLANNING.md M3).

**Split:** the notebook does a stratified 70/15/15 split (`random_state=42`) →
**144 train / 31 validation / 31 test**. Test-set counts: logistics 10,
hot_take 9, analysis 7, prediction 5.

---

## 3. Models

|              | Baseline                                                 | Fine-tuned                                          |
| ------------ | -------------------------------------------------------- | --------------------------------------------------- |
| Model        | `llama-3.3-70b-versatile` (Groq)                         | `distilbert-base-uncased`                           |
| Method       | Zero-shot prompt with label definitions + 1 example each | Supervised fine-tuning, fresh 4-way head            |
| Key settings | `temperature=0`, `max_tokens=20`                         | **15 epochs**, lr 2e-5, batch 16, weight decay 0.01 |
| Trained on   | nothing (in-context only)                                | 144 examples                                        |

**Hyperparameter change (and why).** The first fine-tune used the notebook
default of **3 epochs** and collapsed onto the two majority classes (accuracy
0.452, macro-F1 0.28). With ~144 examples ÷ batch 16 ≈ 9 steps/epoch, 3 epochs is
only ~27 optimizer steps — far too few for a randomly-initialized 4-way head, so
it fell back on the class prior. I raised `num_train_epochs` to **15** (~135
steps); every other setting kept at default. This single change recovered the two
minority classes and lifted accuracy 0.452 → 0.613. Both runs are committed:

- `Copy_of_ai201_project3_takemeter_starter_clean.ipynb` — first pass, 3 epochs
- `Copy_of_ai201_project3_takemeter_starter_clean (1).ipynb` — **15 epochs, the
  run reported below**

---

## 4. Evaluation Report

### 4.1 Headline results (locked test set, n=31)

| Metric       | Zero-shot baseline | Fine-tuned (3 ep, first pass) | Fine-tuned (15 ep, final) |
| ------------ | -----------------: | ----------------------------: | ------------------------: |
| **Accuracy** |          **0.903** |                         0.452 |                     0.613 |
| **Macro-F1** |           **0.90** |                          0.28 |                      0.63 |

Going from 3 → 15 epochs lifted the fine-tuned model **+0.16 accuracy / +0.35
macro-F1** and, more importantly, made it predict all four classes (the 3-epoch
run never predicted `analysis` or `prediction` at all). The final fine-tuned
model still trails the baseline by ~0.29 accuracy. (Random guessing on 4 classes
≈ 0.25.) All numbers below refer to the **15-epoch** model unless stated.

### 4.2 Per-class metrics

**Baseline (Groq, zero-shot):**

| Label         | Precision | Recall |       F1 | Support |
| ------------- | --------: | -----: | -------: | ------: |
| `analysis`    |      0.86 |   0.86 |     0.86 |       7 |
| `hot_take`    |      0.80 |   0.89 |     0.84 |       9 |
| `logistics`   |      1.00 |   1.00 |     1.00 |      10 |
| `prediction`  |      1.00 |   0.80 |     0.89 |       5 |
| **macro avg** |      0.91 |   0.89 | **0.90** |      31 |

**Fine-tuned DistilBERT (15 epochs, final):**

| Label         | Precision | Recall |       F1 | Support |
| ------------- | --------: | -----: | -------: | ------: |
| `analysis`    |      0.71 |   0.71 |     0.71 |       7 |
| `hot_take`    |      0.38 |   0.33 |     0.35 |       9 |
| `logistics`   |      0.67 |   0.60 |     0.63 |      10 |
| `prediction`  |      0.71 |   1.00 |     0.83 |       5 |
| **macro avg** |      0.62 |   0.66 | **0.63** |      31 |

**The flip.** Compared with the 3-epoch run, the two _minority_ classes recovered
dramatically (`analysis` F1 0.00 → 0.71, `prediction` F1 0.00 → **0.83**, now the
model's best class), while `hot_take` — previously a catch-all that absorbed
everything — **dropped to the worst class** (F1 0.53 → 0.35). Longer training let
the model carve out the classes with distinctive vocabulary (tactical terms for
`analysis`, "will win / champions" for `prediction`) and left `hot_take`, which is
defined by what it _isn't_, as the confused remainder.

For reference, the 3-epoch first pass scored `analysis` 0.00 / `hot_take` 0.53 /
`logistics` 0.60 / `prediction` 0.00 (macro-F1 0.28) — it predicted only
`hot_take` and `logistics`.

### 4.3 Confusion matrix — fine-tuned model (15 epochs)

Rows = true label, columns = predicted. (Also committed as
[`confusion_matrix.png`](confusion_matrix.png).)

| true ↓ / pred → | analysis | hot_take | logistics | prediction |
| --------------- | -------: | -------: | --------: | ---------: |
| **analysis**    |    **5** |        1 |         0 |          1 |
| **hot_take**    |        2 |    **3** |         3 |          1 |
| **logistics**   |        0 |        4 |     **6** |          0 |
| **prediction**  |        0 |        0 |         0 |      **5** |

All four classes are now predicted (no empty columns). The diagonal is much
stronger — `prediction` is perfect (5/5) and `analysis` is solid (5/7). The error
mass has moved to **`hot_take`**: its row scatters into every other class
(→`analysis` 2, →`logistics` 3, →`prediction` 1, only 3/9 correct), and
`logistics` leaks _into_ it (4/10). So the hard boundary is no longer "minority
classes ignored" but **`hot_take` ↔ everything**.

> First-pass (3-epoch) matrix for contrast: the `analysis` and `prediction`
> columns were entirely zero — every post was forced into `hot_take` or
> `logistics`.

### 4.4 Failure analysis

**AI-surfaced patterns (then verified by re-reading the cases).** I gave the
12 misclassifications to an LLM and asked for common themes; I then re-read every
example to confirm or discard each pattern. What held up:

1. **`hot_take` is the new failure center.** It's involved in **10 of 12** errors
   — 6 true-`hot_take` posts scattered out (into all three other classes) and
   4 other-class posts pulled in. `hot_take` is a residual category ("an opinion
   with no load-bearing evidence, not a forecast, not practical"), so once the
   model learned the three _positive_ classes, it had no clean signal left for it.
2. **The model is now confident — including when wrong.** Misclassification
   confidences are **0.71–1.00** (several at exactly 1.00), versus ~0.29 in the
   3-epoch run. 15 epochs produced sharp, decisive boundaries; the cost is
   _confident_ errors instead of hedged ones.
3. **The residual errors land on the exact M1 edge cases.** `hot_take`↔`analysis`
   and `analysis`↔`prediction` — the two boundaries the spec flagged as hardest —
   account for the most damaging confident mistakes (see deep dives below). These
   are _evidence-quality_ and _forecast-vs-reasoning_ distinctions, not surface
   topic.
4. **`hot_take` leaks along whatever surface cue dominates a post** — a pattern I
   checked and _kept_: hot-takes that contain football reasoning go to `analysis`
   (#9, #11), hot-takes that name games/teams/crowds go to `logistics` (#10, #12).
   So the model isn't random about `hot_take`; it's defaulting to the _content_
   signal and missing the _rhetorical-mode_ signal the label encodes.

**Is this a labeling problem or a data/training problem?** Still
data/training, but now sharper. The boundaries that fail are precisely the
documented hard edge cases (M1 #1–#3) — i.e. labeled consistently by a written
rule, but genuinely hard distinctions. `hot_take` is also the most _heterogeneous_
class (opinions/grievances/reactions on any topic), so it needs **more** examples,
not fewer, to be learned — 43 training rows isn't enough to teach "reasoned-sounding
but not load-bearing." Leakage and pipeline bugs were ruled out in M4/M5 (disjoint
stratified split; baseline scores 0.90 on the same test set).

**Three failures analyzed in depth (15-epoch model):**

1. **`hot_take` → `analysis` (conf 1.00): "You trust VAR or no? I'm starting to
   believe it's corrupt… Ghana's robbed penalty…"** _Which boundary:_
   hot*take↔analysis — this is **literally M1 edge case #1** ("referees decide
   games"). \_Why hard:* the post cites specific incidents (Ghana penalty,
   Argentina penalty), which _looks_ like evidence, but the evidence is
   cherry-picked toward a pre-decided "corrupt" conclusion → `hot_take` by my
   rule. The model has no way to judge whether cited evidence is _load-bearing_
   vs _decorative_ from 31 analysis examples, so it confidently (1.00) treats "has
   specific references" as analysis. _Fix:_ more `hot_take` examples that cite
   evidence-shaped-but-cherry-picked claims, to teach the distinction explicitly.

2. **`analysis` → `prediction` (conf 1.00): "Brazil have the most difficult group
   and likely the hardest path to the finals \[Elo breakdown]."** _Which
   boundary:_ analysis↔prediction — **M1 edge case #2**. _Why hard:_ the
   reasoning (Elo ratings, group strength) _is_ the substance, so I labeled it
   `analysis`, but the forward-facing framing ("hardest path to the finals")
   triggers the model's now-strong `prediction` detector. The model over-weights
   future-tense/outcome language and under-weights "is the reasoning the point?"

3. **`hot_take` → `logistics` (conf 0.95): "What's with Canadians when it comes to
   engaging with the game? … the lamest crowd I've ever experienced."** _Which
   boundary:_ hot*take↔logistics — **M1 edge case #3** (venting vs asking).
   \_Why hard:* it's a grievance (hot*take) but is phrased as a question about
   match-attendance experience, and "crowd / match / stadium" nouns are strong
   `logistics` cues. The model reads the topic, not the venting intent. \_Labeling
   problem?* No — consistent with the venting-vs-asking rule; the model just can't
   recover intent from surface nouns at this data scale.

### 4.5 Sample classifications (fine-tuned model)

Posts run through the **15-epoch** fine-tuned model, with predicted label and
confidence (4 correct, 2 wrong — covering all four classes):

| Post (truncated) | Predicted | Confidence | True | ✓/✗ |
| ---------------- | --------- | ---------: | ---- | :-: |

| "4-4-1-1 is the best formation for England. GK- Pickford LB- Kyle Walker Peters CB- John Stones CB- Harry Maguire RB- Trent Alexander Arnold CM- Jude Bellingham CM- Declan Rice LM- Jadon Sancho RM- Bukayo Saka CAM- Phil Foden ST- Harry Kane
" | `analysis` | 1.00 | `analysis` | ✓ |
| "Desperately searching for a Scotland Morocco Match Scarf from June 19 in Boston. Does anyone have an extra match scarf from the June 19 match in Boston between Scotland and Morocco? It was my mine and my wifeâ€™s first soccer match and it was on our anniversary. I want to frame it along with a FIFA Boston poster, replica tix, and some photos of us from the match. I live in Washington, DC and willing to pay for shipping (plus actually cost). We visited every merch stand in the stadium to try to find one but everyone was sold out
" | `logistics` | 1.00 | `logistics` | ✓ |
| "2 hot takes ronaldo vs messi and round of 32 games. So obviously Ronaldo is one of the best players of all time no hate there but he is nowhere close to Messi IMO please let me know how there is still a debate... and 2nd how is South Africa vs Canada an knock out game in the fooking World Cup!???! !" | `hot_take` | 1.00 | `hot_take` | ✓ |
| "If other countries ever manage to win and if that's Aisa then what country will win?. As I know none of the Asian countries won the World Cup. So if one day if Asian country wins then what country will it be? (And I have this question is it possible for Korea vs Japan in the World Cup final)?" | `prediction` | 0.98 | `prediction`| ✓ |
| "You trust VAR or no? I'm starting to believe it's corrupt. The reason for that is Ghanas robbed penalty, it was a penalty clear as day but VAR did not even look at it!? The penalty Argentina got was less clear because it was atleast a touch on the ball and the argentinian players protested wildly. Ghana players did not even protest but should they have to? Isn't VAR here for this reason...so you don't need to protest? I'm 100% certain if it was the other way around England would've gotten the penalty. Are the big countries being favoured by VAR?" | `analysis` | 1.00 | `hot_take` | ✗ |
| "Brazil have the most difficult group and likely the hardest path to the finals. [Here is a breakdown by Elo rating](https://www.google.com/amp/s/sports.yahoo.com/amphtml/world-cup-draw-groups-rankings-analysis-192900374.html) - Hardest group by Elo rating - Then having to play either Portugal or Uruguay - followed by a likely Germany/Spain matchup - Argentina semis - France final Meanwhile Argentina and France got off easy with light groups and favorable R16 matchups" | `prediction` | 1.00 | `analysis` | ✗ |

**Why a correct one is reasonable:** _"4-4-1-1 is the best formation for England:
GK Pickford, CB Stones, CM Bellingham…"_ → `analysis` at **1.00**. This is a
starting-XI / formation post — pure tactical football content, which is exactly
the `analysis` definition. (Notably, the 3-epoch model got this _wrong_, calling
it `hot_take`; the extra training taught the model the formation/XI genre.) The
contrast with the two wrong rows is the whole lesson: the model is **confident and
right** on classes with distinctive vocabulary (`analysis`, `prediction`,
`logistics`) and **confident but wrong** on the `hot_take` boundary, where the
giveaway is _evidence quality / intent_, not words — the VAR rant cites real
incidents (so it "looks like" analysis) but the evidence is cherry-picked, and the
Elo post reasons toward who advances (so it "looks like" a prediction).

---

## 5. Reflection — what I intended vs. what the model learned

I designed the labels around a **functional distinction** (the rhetorical mode of
a post). The 70B baseline captured that distinction almost perfectly — strong
evidence the taxonomy itself is sound and learnable.

The fine-tuned model's behavior changed sharply with training length, and the
final (15-epoch) model captured **the classes with distinctive vocabulary, but
not the rhetorical-mode distinction the taxonomy is really about.** It learned
`prediction` (the "will win / champions" register) and `analysis` (tactical/stats
vocabulary) well — but it captures them as **topic/word-pattern** classes, not as
the functional categories I defined. The proof is in the residual errors: it
confidently sends an evidence-citing rant to `analysis` (VAR-corrupt post) and a
reasoning-rich bracket post to `prediction`, because it keys on surface signals
(specific references → analysis; future-tense → prediction) rather than the
**load-bearing-evidence** and **reasoning-is-the-point** tests my definitions
encode.

What it most clearly **missed** is `hot_take` — which is exactly the class that
_can't_ be defined by vocabulary, because it's the residual "opinion without
load-bearing evidence" category that can appear on any topic. Once the model
carved out the three positive classes, `hot_take` became the confused remainder
(F1 0.35).

So between the two runs, the story isn't "fine-tuning failed" but "fine-tuning
learns _word patterns_ fast and _functional intent_ slowly": 3 epochs gave a
majority-class prior; 15 epochs gave good topic-pattern classes; neither recovered
the evidence-quality / intent boundary that the 70B baseline gets essentially for
free from pretraining. That gap — not the labels, which the baseline shows are
learnable — is the real limit at ~200 examples.

---

## 6. Spec reflection

**One way the spec helped:** the Milestone 2 evaluation plan committed me to
**macro-F1 + per-class metrics + a confusion matrix**, and explicitly predicted
the `analysis`↔`hot_take` confusion as the hardest boundary. That paid off twice:
in the 3-epoch run it gave me the right lens to diagnose _minority-class collapse_
(per-class F1 = 0, empty confusion columns) instead of hand-waving "it
underperformed"; and in the 15-epoch run, after the minority classes recovered,
the **residual** confident errors landed on exactly the M1 edge cases the spec
anticipated (#1 evidence-quality, #2 forecast-vs-reasoning, #3 venting-vs-asking).
The spec told me where to look both times.

**One way the implementation diverged:** the spec set a success target of
"fine-tune beats the baseline macro-F1 by ≥0.05." Even after raising epochs 3 → 15
(macro-F1 0.28 → 0.63), the fine-tune **still trails** the baseline (0.90), so
that target was not met. I diverged by treating the baseline as the genuinely
deployable artifact and the fine-tune as a _diagnostic_ — at ~200 examples,
honestly reporting "the 70B zero-shot model is the better classifier, and
fine-tuning learns word-patterns but not the evidence-quality boundary" is more
truthful than forcing the fine-tune to look like the winner. (The data pipeline
also diverged: the spec imagined mostly manual collection; I used the Reddit data
APIs plus multi-LLM pre-labeling, disclosed below.)

---

## 7. AI usage

This project used AI tools at several explicit points; in each case a human made
the final decision.

1. **Data collection & pre-labeling.** I directed Claude (via Claude Code) to
   fetch r/worldcup posts through the Arctic Shift / PullPush APIs and produce a
   first-pass label for each. In the annotation spreadsheet, **GPT, Gemini, and
   Claude** each pre-labeled every post; I then set a human **"My Final Label"**
   for all 206 rows, overriding the models wherever they disagreed or got a
   boundary case wrong (the `analysis`↔`hot_take` and `prediction`-vs-`logistics`
   calls especially). The model pre-labels are preserved in
   `data/worldcup_labels.csv` for transparency.

2. **Baseline prompt.** I asked an LLM to draft the Groq zero-shot classification
   prompt from my planning.md label definitions. It produced the structure
   (definitions + one example per label + "output only the label name"); I
   reviewed it, swapped in examples that match my exact edge-case rules, and kept
   the label strings consistent with the CSV. Result: 0/31 unparseable responses.

3. **Failure-pattern analysis (both runs).** I pasted the misclassified test
   examples into an LLM and asked for common themes, then **verified each by
   re-reading the cases**. For the 3-epoch run I **discarded** a "short/exclamatory
   post → hot_take" theme (short logistics/prediction posts also collapsed → the
   real driver was the class prior). For the 15-epoch run I **kept** the
   "`hot_take` leaks along the dominant surface cue" pattern after confirming each
   case, and confirmed the residual errors map onto the M1 edge cases. Only
   verified patterns appear in §4.4.
