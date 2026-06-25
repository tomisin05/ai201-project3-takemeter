# TakeMeter — Planning

A fine-tuned text classifier that evaluates the _type_ of take in a World Cup
discussion community. This document is the working design log. Milestone 1
(community + label taxonomy) is complete; later milestones append below.

---

## Milestone 1 — Community & Label Taxonomy

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

## Milestone 2

## Milestone 3

## Milestone 4

## Milestone 5

## Milestone 6
