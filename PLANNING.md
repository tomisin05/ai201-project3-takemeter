# TakeMeter — Planning

A fine-tuned text classifier that evaluates the _type_ of take in a World Cup
discussion community. This document is the working design log. Milestone 1
(community + label taxonomy) is complete; later milestones append below.

---

## Milestone 1: Choose Your Community and Define Your Labels

### Community: r/worldcup

[r/worldcup](https://www.reddit.com/r/worldcup/) is an active, text-heavy
community organized around the FIFA World Cup. It is especially live right now
(the 2026 tournament is in its group stage), so the front page is a constant
stream of opinions, forecasts, match breakdowns, and practical questions from
fans actually attending or following the matches.

I read over 40 real posts before committing to labels. What jumped out is that posts differ mostly in **what kind of contribution they are**: some
_argue_ about the football with evidence, some _assert_ an opinion with none,
some _forecast_ what will happen, and a large share are _practical questions_
about tickets, travel, kits, and broadcasts . Those four types are recognizable to any regular
and are what I built the taxonomy around.

### Labels

| Label        | One-sentence definition                                                                                                                                                                                |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `analysis`   | Argues a point about the football itself using **specific, verifiable evidence** — match events, tactics, formations, player roles, stats, or historical comparison. The reasoning _is_ the substance. |
| `hot_take`   | A bold subjective opinion or grievance **asserted with little or only cherry-picked/decorative evidence**. The claim might be true, but the post asserts rather than argues.                           |
| `prediction` | A **forward-looking forecast of an outcome** — who wins a match/group/tournament, a knockout result, or a scoreline.                                                                                   |
| `logistics`  | A **practical or community-operations question/help post** about attending or following the tournament — tickets, travel, stadiums, kits, broadcasts, schedules.                                       |

### Examples per label (all real r/worldcup posts, with links)

#### `analysis`

1. **"Qatar 2022 — Group G preview"** — breaks down all four teams by formation
   and role: _"Tite can use two systems for Brazil. The first one is a 4-2-3-1
   with Richarlison at CF and Neymar behind him. The second one is a 4-3-3 with
   Fred starting over Richarlison... [Switzerland] relies more on counterattacks,
   direct play and width."_ Specific, verifiable, reasoning-first.
   → https://www.reddit.com/r/worldcup/comments/ywsse0/
2. **"Mohamed Salah delivers the goods as Egypt come from behind... [Match
   Analysis]"** — minute-by-minute account (goal times, assists), Salah's
   second-half role, Egypt's defensive reorganization after the break, and the
   Group G points table. Conclusions are tied to specific match events.
   → https://www.reddit.com/r/worldcup/comments/1uca23o/

#### `hot_take`

1. **"Football is completely carried by the atmosphere and the crowds."** —
   _"The actual sport itself is mind numbingly boring. If you removed the crowds
   ... this shit would be unwatchable."_ Pure assertion, zero supporting
   evidence.
   → https://www.reddit.com/r/worldcup/comments/1ucbd4e/
2. **"This world cup so far clearly proved: Referees decide games."** — sweeping
   match-fixing claim. It _names_ two matches/referees, but the evidence is
   anecdotal and cherry-picked to fit the conclusion (see edge case #1).
   → https://www.reddit.com/r/worldcup/comments/1ucazym/

#### `prediction`

1. **"France Will Win the World Cup — Here's the Real Reason"** — body in full:
   _"Who do you think will win the world cup? I may be biased but i say France."_
   A forecast with no real reason despite the title.
   → https://www.reddit.com/r/worldcup/comments/1ubtrnv/
2. **"Japan will be the first non European and South American country to win the
   world cup"** — _"their tactics and training are to be admired ... the so
   called '100 year plan' is definitely taking shape."_ A forecast with soft
   reasoning (see edge case #2).
   → https://www.reddit.com/r/worldcup/comments/1ubj5hc/

#### `logistics`

1. **"Getting a player's name on the back of the jersey?"** — _"I recently
   bought a Spain jersey ... just a blank jersey with no name or number. Is there
   anyway that I can get the name and number on myself?"_ Practical help request.
   → https://www.reddit.com/r/worldcup/comments/1ucbk37/
2. **"My Personal World Cup Ticketing Nightmare. Has anyone had the same happen
   to them?"** — detailed FIFA ticketing failure, asking the community for help
   and resale guidance.
   → https://www.reddit.com/r/worldcup/comments/20wsct/

### Hardest edge cases & decision rules

The boundary work is the point of this milestone. Three real posts that sit
between labels, and the rule I'll apply consistently when annotating:

**Edge case #1 — `hot_take` vs `analysis`: "Referees decide games"**
([1ucazym](https://www.reddit.com/r/worldcup/comments/1ucazym/)).
It cites specific matches (Ecuador–Curaçao, Belgium–Iran) and referees, which
_looks_ like analysis. But the cited facts are anecdotal and selected to fit a
pre-decided conspiracy conclusion; strip the opinion framing and no rigorous
argument remains.

- **Decision rule:** evidence must be both **specific and load-bearing** — the
  argument has to stand _on_ it. If the evidence is cherry-picked, vague, or
  decorative (just enough to sound credible), it's `hot_take`. → **`hot_take`**.

**Edge case #2 — `prediction` vs `analysis`: "Japan will win the World Cup"**
([1ubj5hc](https://www.reddit.com/r/worldcup/comments/1ubj5hc/)).
It forecasts an outcome _and_ gestures at reasons ("tactics and training",
"100 year plan").

- **Decision rule:** if the post's **core purpose is forecasting a future
  outcome**, it's `prediction` — even when it offers reasons. It's only
  `analysis` if the reasoning is specific/verifiable and the _reasoning itself_
  is the point. Here the reasons are vague and the headline is the forecast.
  → **`prediction`**.

**Edge case #3 — `logistics` vs `hot_take`: "censorship around valid
discussions involving eliminations"**
([1uc9z04](https://www.reddit.com/r/worldcup/comments/1uc9z04/)).
Opens like a question about the confusing elimination/scoring math (logistics-y)
but is delivered as a grievance about moderation.

- **Decision rule:** if the post is **primarily asking for practical info/help**,
  it's `logistics`; if it's **primarily venting an opinion or complaint**, it's
  `hot_take`. This one is mostly venting. → **`hot_take`**.

---

## Milestone 2: Write Your Spec Before Any Code

### 1. Community

**r/worldcup**, during the live 2026 group stage. It is a good fit for classification because the same surface topic gets expressed as four different kinds of post: evidence-backed argument, bare opinion, forecast, and practical help request. The _subject_ barely varies but the
_rhetorical mode_ does, so a classifier has to learn the shape of the
contribution rather than just spotting topic keywords. That's a harder and more
interesting signal than "is this about football."

### 2. Labels

Defined in full with two real example posts each in Milestone 1.

- **`analysis`** — argues a point about the football using specific, verifiable,
  load-bearing evidence (match events, tactics, stats, history) where the
  reasoning _is_ the substance.
- **`hot_take`** — asserts a bold subjective opinion or grievance with little or
  only cherry-picked/decorative evidence.
- **`prediction`** — forecasts a future outcome (match/group/tournament winner,
  knockout result, scoreline).
- **`logistics`** — a practical or community-operations question/help post
  (tickets, travel, stadiums, kits, broadcasts, schedules).

I deliberately did **not** add a `reaction` label: pure reactions on r/worldcup
are mainly image/link posts with little or no body text, so they would
be a near-empty class for a _text_ classifier.

### 3. Hard edge cases

Three real boundary posts and the decision rules are documented in Milestone 1.
The single hardest boundary is **`analysis` vs `hot_take`**: a post can cite
specific matches/referees and still be a hot take if the evidence is
cherry-picked or decorative rather than load-bearing. **Decision rule applied
consistently during annotation:** evidence must be both _specific and
load-bearing_ — strip the opinion framing, and if no standalone argument
remains, it is `hot_take`. The other recurring boundary is `prediction` vs
`analysis`: if the post's core purpose is forecasting an outcome it is
`prediction` even when it offers reasons; it is only `analysis` when the
specific reasoning is the point. I will keep a running `edge_cases.md` log: every
post that took more than a few seconds to decide gets its text, my label, and
the rule that resolved it, so the rules stay consistent across all 200+ examples
and so I can re-check earlier annotations if a rule shifts.

### 4. Data collection plan

**Source.** reddit.com through two public read-only Reddit data APIs that _are_ reachable:

- **Arctic Shift** —
  `https://arctic-shift.photon-reddit.com/api/posts/search?subreddit=worldcup&sort=desc&limit=N`
  (and `/api/posts/ids?ids=...`). Has the very recent 2026 posts.
- **PullPush** —
  `https://api.pullpush.io/reddit/search/submission/?subreddit=worldcup&size=N&selftext=:not_blank`
  (older posts; lacks the newest 2026 ids).

I filter to text posts (`selftext` non-empty, non-`[removed]`/`[deleted]`),
store `id`, title, selftext, created date, and a rebuilt permalink
(`https://www.reddit.com/<permalink>`), and dedupe by `id`.

**Target.** **≥ 200 labeled examples total**, with a hard floor of **≥ 20% per
label** (≥ 40 each) so no class is starved at train time. The classifier input
is title + selftext concatenated. I split **70 / 15 / 15** into
train / validation / test with **stratified** sampling so each split preserves
the label distribution, and I fix the random seed so the split is reproducible.

**Expected imbalance and the fix.** From reading the sub, `logistics` and
`hot_take` are common and `analysis` is the rarest. My plan for any label still under the 40-example floor after
an initial ~200-post sweep is to pull more candidates likely to contain the rare
class (e.g. PullPush `q=` keyword searches like "preview", "tactics",
"formation", "match thread" for `analysis`), and pull from older time windows
via PullPush rather than only the recent Arctic Shift feed.

### 5. Evaluation metrics

Accuracy alone is wrong here for two reasons: the classes are imbalanced (a model
that ignores `analysis` could still score decently on accuracy), and the classes
are **not equally costly to confuse**. My evaluation reports:

- **Macro-averaged F1** It averages per-class F1 with
  equal weight, so the rare-but-important `analysis` class counts as much as the
  common ones.
- **Per-class precision, recall, and F1.**
- **A 4×4 confusion matrix.** The matrix
  lets me confirm whether real errors match that prediction or reveal an
  unexpected confusion.
- **Macro-F1 of the fine-tuned DistilBERT vs the Groq `llama-3.3-70b-versatile`
  zero-shot baseline**, on the _same_ held-out test set. Beating a strong
  zero-shot LLM is the bar that shows fine-tuning earned its keep.

### 6. Definition of success

Concrete, checkable thresholds on the held-out **test** split:

macro-F1 **≥ 0.75** and fine-tuned model **beats the Groq zero-shot baseline on
macro-F1** by a meaningful margin (target ≥ +0.05).

"Good enough for deployment" in a real community tool means it meets tge two conditions outlined above.

### AI Tool Plan

- **Label stress-testing (do now, before annotating).** I will give an Claude my four label definitions plus the edge-case rules and ask it to
  generate 8–10 synthetic posts deliberately sitting on the `analysis`↔`hot_take`
  and `prediction`↔`analysis` boundaries. If I can't cleanly classify its output
  with my current rules, the definitions are too loose and I tighten them _before_ labeling the 200 real examples.
- **Annotation assistance.** I _will_ use an chat gpt, gemini and claude to **pre-label** each batch with
  one of the four labels, then review and correct every
  pre-label myself — the LLM proposes, I decide.
- **Failure analysis.** After evaluation I'll hand the LLM the list of
  misclassified test posts (text, true label, predicted label) and ask it to
  cluster the errors into patterns (e.g. "decorative-evidence hot takes read as
  analysis," "short low-context posts"). I then **verify each proposed pattern by
  hand** against the actual confusion matrix and the offending posts before it
  goes in the writeup.

---

## Milestone 3: Collect and Annotate Your Dataset

### What was collected

All examples are real, public r/worldcup self-text posts. They were pulled
through the two read-only Reddit data APIs from the M2 plan: **Arctic Shift** for the recent 2026 posts and
**PullPush** for older posts and targeted keyword queries. From a deduped pool of
**1,343** text posts I read 441 candidates individually and labeled the ones that
fit a single label cleanly; image-only posts, news headlines, memes, match/daily
megathreads, spam, off-topic, and posts with no clean single-label fit were
dropped rather than force-fit.

### Final label distribution

| Label        | Count |
| ------------ | ----: |
| `hot_take`   |    62 |
| `logistics`  |    64 |
| `analysis`   |    44 |
| `prediction` |    36 |
| **Total**    |   206 |

No label exceeds 70% (largest is 31.068%), so the imbalance checkpoint passes.

### Difficult cases encountered during annotation

Beyond the three taxonomy edge cases in Milestone 1, these are the recurring ambiguities that came up across 200+ posts. To improve consistency, I ran each post through three LLMs — GPT, Gemini, and Claude — and used their outputs as a first-pass signal. When all three agreed, I accepted the label with minimal review. When they conflicted, I manually reviewed the post against the decision rules and recorded my final label. The cases below are drawn from those conflicts.

M3 case #1 — prediction vs analysis: "Austria vs Algeria … biggest match fixing". GPT and Gemini both labeled this prediction; Claude labeled it analysis. It reads like incentive analysis — laying out exactly why both teams prefer 2nd place to avoid Spain — structurally similar to the M1 "Disgrace of Gijón" post I labeled analysis. The deciding difference is core purpose: the Gijón post's purpose is to explain a historical scenario (analysis); this post's purpose is to forecast that the upcoming match will be a non-contest (prediction). When incentive reasoning is bent toward "here's what will happen," the forecast intent wins → prediction.
M3 case #2 — analysis vs hot_take on the Croatia post. GPT and Gemini labeled it analysis; Claude labeled it hot_take. The post cites Croatia's World Cup placements and Nations League finish as evidence they deserve more respect, which looks like load-bearing reasoning. On manual review, the facts are assembled to defend a pre-decided grievance ("why doesn't anyone respect us?") rather than to argue a tactical or football-substance point. Strip the opinion framing and no standalone football argument remains → hot_take.
M3 case #3 — analysis vs hot_take on the Ronaldo hate post. GPT and Gemini labeled it analysis; Claude labeled it hot_take. The post makes several claims — that Martinez is the real problem, that Ramos isn't better than Ronaldo, that CR7 should play 60 minutes — but none are backed by specific match events or stats. These are asserted, not argued. On manual review I sided with GPT and Gemini: the post engages seriously enough with the football question (who's actually at fault in Portugal's performances?) that it sits closer to analysis than a pure vent → analysis. Close call.
M3 case #4 — hot_take vs analysis on the Qatar post. GPT and Claude labeled it hot_take; Gemini labeled it analysis. The post cites concrete stats (17 goals conceded across two World Cups, specific Asian Cup results, named opponents) to build a claim about Qatar's quality and the integrity of the AFC Asian Cup. On manual review, the evidence is specific and load-bearing — not decorative — which distinguishes it from a pure assertion. The conspiracy-adjacent conclusion about the Asian Cup is speculative, but the football reasoning that leads there is grounded → analysis.
M3 case #5 — logistics vs hot_take vs prediction on broadcast/atmosphere posts. Three LLMs gave three different labels for the "stadium moments are wholesome" post (GPT: prediction, Gemini: logistics, Claude: hot_take). This exposed a recurring ambiguity: posts about TV production, stadium audio, and broadcast decisions sit at the edge of multiple labels. The rule I applied consistently: if the post asks for or shares practical info ("where to stream with English commentary", "what's the cooling-break song?") → logistics; if it vents an opinion or preference ("they should show more of this instead of ads", "instant replay is a joke") → hot_take. The wholesome-stadium post is an opinion about broadcast choices with no ask and no football substance → hot_take.
M3 case #6 — analysis vs hot_take on the Japan national team post. All three LLMs disagreed: GPT labeled it logistics, Gemini prediction, Claude analysis. The post makes verifiable claims (4-1 vs Germany, FIFA rankings) but uses them as a springboard for sweeping assertions like "Japan would beat the USA 7-1" and "Japan is guaranteed top four at 2026." On manual review, the evidence is cherry-picked and the conclusions far outrun what it supports. Core purpose is forecasting Japan's dominance → prediction.
M3 case #7 — analysis vs hot_take on the "top 3 teams outside Europe/South America" post. GPT labeled it prediction, Gemini analysis, Claude hot_take. The post makes team-by-team arguments (Senegal's defensive resilience, Japan's technical quality, Morocco's tactical improvement vs Brazil) with specific, verifiable match references as load-bearing evidence. On manual review, however, the post's framing ("confidence in them beating strongest teams") is ultimately a preference/opinion list rather than a football argument — the reasoning is gesture-level, not rigorous → hot_take.
M3 case #8 — logistics vs analysis on the Bosnia qualification math post. GPT labeled it hot_take; Gemini and Claude both labeled it logistics. The post asks what combination of results would knock Bosnia out, referencing the NYT's 99% probability figure. This is a "help me follow the tournament" question with no football argument and no forecast → logistics, consistent with M3 case #5 in the original taxonomy (standings/qualification math = logistics).

### AI usage disclosure (annotation)

Per the M2 AI Tool Plan, every example was **pre-labeled chat gpt, gemini and claude and
then human-reviewed**; the boundary calls above are some of the reviewed decisions.

## Milestone 4: Run Your Baseline

### Where the baseline struggled

Only ~3 of 31 were missed, and the errors land **exactly where M2 predicted**:

- **`analysis` ↔ `hot_take` is the live boundary.** `hot_take` precision (0.80)
  is the lowest number on the board — the model over-applies `hot_take`, pulling
  in posts that are really reasoned `analysis`. `analysis` itself is at 0.86/0.86,
  so the rare class is _not_ the model's weak point; the _confusion_ between these
  two is.
- **`prediction` recall 0.80** — one forecast was missed.
- **`logistics` is cleanly separable (1.00/1.00).** Practical/operational posts
  have a distinct surface vocabulary, exactly as expected.

**Hypothesis going into the eval milestone:** any model's residual errors will
concentrate on `analysis`↔`hot_take` (and secondarily `prediction`↔`analysis`),
not on `logistics`. I'll confirm this against the confusion matrix.

### Reflection vs. the M2 success criteria

This baseline **already clears every M2 "definition of success" threshold**:
macro-F1 0.90 (≥0.75 ✅), no class below F1 0.84 (≥0.65 floor ✅), `analysis`
recall 0.86 (≥0.70 ✅).

## Milestone 5: Fine-Tune Your Model

### Setup

`distilbert-base-uncased` + a fresh 4-way classification head, fine-tuned on the
144-example training split. **Default hyperparameters, unchanged:** 3 epochs,
learning rate 2e-5, batch size 16, weight decay 0.01, 50 warmup steps, best model
by validation accuracy. Runtime: T4 GPU. (No hyperparameters were altered for this
first pass, so there is nothing to justify here — changes, if any, come in the
retraining note below.)

### Fine-tuned results (test set, n=31)

**Accuracy: 0.452** · **Macro-F1: 0.28**

| Label         | Precision | Recall |   F1 | Support |
| ------------- | --------: | -----: | ---: | ------: |
| `analysis`    |      0.00 |   0.00 | 0.00 |       7 |
| `hot_take`    |      0.38 |   0.89 | 0.53 |       9 |
| `logistics`   |      0.60 |   0.60 | 0.60 |      10 |
| `prediction`  |      0.00 |   0.00 | 0.00 |       5 |
| **macro avg** |      0.25 |   0.37 | 0.28 |      31 |

**Confusion matrix** (rows = true, cols = predicted; reproduces
`confusion_matrix.png`):

| true ↓ / pred → | analysis | hot_take | logistics | prediction |
| --------------- | -------: | -------: | --------: | ---------: |
| **analysis**    |        0 |        6 |         1 |          0 |
| **hot_take**    |        0 |        8 |         1 |          0 |
| **logistics**   |        0 |        4 |         6 |          0 |
| **prediction**  |        0 |        3 |         2 |          0 |

The two right-most columns are **entirely zero**: the model never once predicted
`analysis` or `prediction`. It learned only the two largest training classes
(`hot_take`, `logistics`) and routed everything into them.

### Baseline vs. fine-tuned

| Metric          | Zero-shot baseline | Fine-tuned DistilBERT |
| --------------- | -----------------: | --------------------: |
| Accuracy        |          **0.903** |                 0.452 |
| Macro-F1        |           **0.90** |                  0.28 |
| `analysis` F1   |           **0.86** |                  0.00 |
| `hot_take` F1   |           **0.84** |                  0.53 |
| `logistics` F1  |           **1.00** |                  0.60 |
| `prediction` F1 |           **0.89** |                  0.00 |

The fine-tuned model is **worse across the board** — a −0.45 accuracy regression.

### Retrain — 15 epochs (the change + result)

The 3-epoch run only does ~27 optimizer steps (144 ÷ 16 ≈ 9 steps/epoch × 3),
far too few for a cold 4-way head — so it falls back on the class prior. **The one
change:** `num_train_epochs` 3 → **15** (~135 steps); everything else default.
This recovered the two minority classes and lifted the model from a collapsed
prior to a real classifier.

**Accuracy: 0.613** · **Macro-F1: 0.63** (vs 0.452 / 0.28 at 3 epochs)

| Label         | Precision | Recall |   F1 | Support |
| ------------- | --------: | -----: | ---: | ------: |
| `analysis`    |      0.71 |   0.71 | 0.71 |       7 |
| `hot_take`    |      0.38 |   0.33 | 0.35 |       9 |
| `logistics`   |      0.67 |   0.60 | 0.63 |      10 |
| `prediction`  |      0.71 |   1.00 | 0.83 |       5 |
| **macro avg** |      0.62 |   0.66 | 0.63 |      31 |

**Confusion matrix** (15 epochs; rows = true, cols = predicted):

| true ↓ / pred → | analysis | hot_take | logistics | prediction |
| --------------- | -------: | -------: | --------: | ---------: |
| **analysis**    |    **5** |        1 |         0 |          1 |
| **hot_take**    |        2 |    **3** |         3 |          1 |
| **logistics**   |        0 |        4 |     **6** |          0 |
| **prediction**  |        0 |        0 |         0 |      **5** |

**The flip.** The minority classes recovered hard (`analysis` F1 0→0.71,
`prediction` F1 0→0.83, now the best class), and **`hot_take` became the worst
class** (0.53→0.35). Longer training let the model carve out the classes with
distinctive vocabulary and left `hot_take` — the residual "opinion, no
load-bearing evidence" category — as the confused remainder. Errors are now
**high-confidence (0.71–1.00)** instead of the 3-epoch run's ~0.29: the model is
decisive, including when wrong.

**Where the residual errors land (verified):** the confident mistakes fall on
exactly the M1 edge cases — `hot_take`→`analysis` on an evidence-citing rant ("VAR
is corrupt", conf 1.00 = M1 edge #1), `analysis`→`prediction` on a reasoning-rich
bracket post ("hardest path to the finals", conf 1.00 = M1 edge #2), and
`hot_take`→`logistics` on a venting-as-question post ("lamest crowd", conf 0.95 =
M1 edge #3). The model learned **word/topic patterns** but not the
**evidence-quality / intent** distinction the labels encode.

**Still short of the spec target.** Even at 0.63 macro-F1 the fine-tune trails the
0.90 baseline, so the M2 "beat baseline by ≥0.05" goal is not met. Remaining
levers: class weighting (for `hot_take`), more heterogeneous `hot_take` examples,
and early-stopping on macro-F1 rather than accuracy.

Artifacts: `evaluation_results.json` (final), `data/finetuned_results.json`
(per-class + both runs), `confusion_matrix.png` (15-epoch), notebook
`Copy_of_ai201_project3_takemeter_starter_clean (1).ipynb`.

## Milestone 6
