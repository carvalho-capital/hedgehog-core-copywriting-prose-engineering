---
name: prose-engineered-copy
description: Write marketing copy or prose (article, Meta ad, landing page, sales page/email, X post) and gate every draft on a code check for AI tells, readability and format limits, revising until it passes.
---

# Prose-engineered copy

Method from skyf0xx/hedgehog-core-copywriting-prose-engineering (MIT). Format
craft notes are adapted from Rob Palmer's copywriting skills (CC BY 4.0) and
Sergey Bulaev's x-post-writer (MIT).

The core idea: the model that writes a draft should not be the one that
decides it is done. A script decides. You draft, run the `check-copy` gate,
fix what it flags, and re-run until it exits 0. The user only ever sees copy
that already passed.

## 1. Brief (ask before drafting)

Get these four things. Ask only for what the user's message and any source
material do not already answer. Never invent an audience, a proof point, a
statistic or a named source.

- **What** is being written (the concrete piece).
- **Type**, one of: `prose` (article, essay, docs; the default), `ad` (Meta
  ad), `landing-page` (bridge or pre-sell page), `direct-response` (sales
  page, VSL, email sequence), `tweet` (single X post; note `premium` if the
  account has X Premium). If the piece could be two types, ask.
- **Audience**.
- **Register** (voice and tone).

Write the brief to `brief.md` with the type on its own first line:

```
type: tweet
premium: false
What: ...
Audience: ...
Register: ...
```

Show the brief back in plain words, including which format contract it will
be checked against, and get a yes. Once drafting starts the brief is fixed:
if the draft wants to become a different type, raise it with the user instead
of switching.

## 2. Set up the gate (once per session)

The gate is a Node script built on retext, write-good and flesch/flesch-kincaid.
It needs Node 18+ and network access for the first install.

```bash
GATE_DIR="${TMPDIR:-/tmp}/prose-gate"
if [ ! -d "$GATE_DIR/repo" ]; then
  git clone --depth 1 https://github.com/skyf0xx/hedgehog-core-copywriting-prose-engineering.git "$GATE_DIR/repo"
fi
GATE="$GATE_DIR/repo/workspace/scripts/check-copy"
(cd "$GATE" && npm install --no-audit --no-fund --ignore-scripts)
```

This installs third-party code, so tell the user once that you are about to
do it. Last reviewed at commit `a1e1625` (2026-11-15).

If Node, git or the network is unavailable, use the fallback in section 6.

## 3. Draft into the format's shape

Write the draft to `draft.md`. The format contract shapes the draft, so
write toward it from the start (see section 5).

- `ad`: use the sections `## Primary text`, `## Headline`, `## Description`.
- `landing-page`: open with a single `# ` headline; end with a clear call to
  action.
- `direct-response`: a CTA, at least one specific number, and proof
  (testimonial, named source or cited result) in the middle.
- `tweet`: the post text only, no markdown.

## 4. The loop

```bash
node "$GATE/index.mjs" draft.md --brief brief.md
```

It prints a JSON report and exits 0 (`pass: true`) or 1 (`pass: false`).
Exit 2 means bad input (empty file, unknown format).

- `pass: false`: fix what each error names, then re-run.
- `pass: true` with warnings: weigh each warning against the brief's
  register. Fix the ones that make the copy better. Do not chase a warning to
  zero if the sentence reads worse for it.
- `pass: true`, no warnings: done.

Cap the loop at 6 runs. If it still fails, stop and tell the user the
specific tension (for example, a technical register that needs words the
gate bans) instead of iterating blindly.

Present the final copy with the last report's key metrics (`format`, word
count, Flesch Reading Ease, Flesch-Kincaid grade, remaining warnings). Save
the finished piece where the user wants it, named after the piece.

### Rules that keep the gate honest

- Never present a draft that has not been run through the gate. "It reads
  clean to me" is not a pass.
- Fix the cause, not the match. Deleting "delve" from a sentence that is
  still a hedge stack games the gate; rewrite the sentence.
- Never drop `--brief` or swap `--format` to get a pass. Running a tweet
  through `prose` hides the character limit instead of meeting it.
- Do not invent per-draft exceptions. If a flagged word is a genuine term of
  art (a "robust" estimator in a stats paper), tell the user the brief's
  register conflicts with the general-audience contract and let them decide.
- Never fabricate proof to satisfy a "needs a number or source" rule. Ask the
  user for a real one.

## 5. What the gate checks

These run on every draft (errors block, warnings do not):

**AI tells (errors unless noted)**
- Banned words: delve, tapestry, boundaries, elevate, unleash, robust,
  seamless, leverage, moreover, furthermore, crucial, pivotal, garner,
  bolster, underscore, showcase, foster, enhance, align, landscape,
  intricate, meticulous, testament, vibrant, interplay, navigate, myriad,
  synergy, holistic, multifaceted, nuanced, comprehensive, cutting-edge,
  transformative, groundbreaking, innovative, game-changer, harness,
  revolutionize, utilize, facilitate, empower, streamline, unlock, realm,
  resonate, compelling, paramount, surpass, mosaic, ecosystem, symphony,
  labyrinth, beacon, cornerstone, bedrock, cacophony, kaleidoscope, odyssey,
  dynamic.
- Banned phrases: "it's important/worth noting that", "at its core", "when it
  comes to", "in the realm of", "plays a vital role in", "in today's
  fast-paced world", "unlock the potential of", "in conclusion", "in
  summary", "to summarize", "I hope this helps", "studies show", "experts
  agree", "research suggests", "industry reports", "the future looks
  bright", "only time will tell".
- Every em dash (use a comma, period or parentheses).
- Negation formulas: "it's not just X, it's Y", "X, not Y" as the emphatic
  last word of a sentence.
- Emphatic fragments: short standalone sentences like "Nothing else
  matters." or "Every time." that restate the previous claim.
- Hedge stacks: could/may/might + potentially/possibly/eventually/ultimately.
- Copula avoidance: "serves as", "stands as", "functions as", "marks a",
  "represents a" instead of "is".
- Warnings: more than one "X, Y, and Z" triplet; Title Case headers.

**Prose quality (warnings unless noted)**
- Repeated words ("the the"): error.
- Passive voice, weasel/intensifier words, wordy phrases and clichés.
- Flesch Reading Ease below 50 or Flesch-Kincaid grade above 10.
- Low burstiness: sentence-length standard deviation under 3 words (4+
  sentences). Mix short and long.
- Nominalizations (-tion, -ment, -ance, -ence) above 3% of words.
- Any paragraph over 100 words.
- Not checked, but do it anyway: a paragraph listing several items reads
  better as a list.

**Format contracts (add to the above, never relax them)**

| Type | Errors | Warnings |
| --- | --- | --- |
| `ad` | missing primary text; primary text over 2,200 chars; headline over 40; description over 30; no CTA in primary text; headline that is a button label ("Learn more", "Shop now"); unqualified claims (see below) | primary text running past the 125-char "See more" fold mid-thought; headline over 27 chars; no number, named source or timeframe |
| `landing-page` | no `# ` headline; no call to action | over 500 words; headline that wraps past ~3 mobile lines; no number, named source or timeframe; any paragraph over 60 words |
| `direct-response` | no CTA verb; no specific number; vague benefits ("next level", "unlock your potential", "best-in-class", "world-class", "state-of-the-art", "industry-leading", "achieve your goals", "reach new heights"); unqualified claims | no testimonial, named source or cited result past 150 words |
| `tweet` | over 280 chars (25,000 with Premium), counted the way X counts: any URL is 23, emoji and CJK are 2; engagement bait ("RT if you", "like if you", "follow for more", "tag a friend", "reply YES", "drop a X below"); all-caps opening line | more than one hashtag; external link in the body (put it in a reply); long first line before the break; "Unpopular opinion:" with no named opposing view |

Unqualified claims (ad and direct-response): "guaranteed you'll/to",
"100% risk-free", "zero risk", "results are typical", "make money fast",
"get rich quick", "no effort required", "cures your...", "lose N lbs".
Qualify them or cut them; ad networks suspend accounts over these.

### Craft notes per format

- **Ad**: the first 125 characters are the whole ad for most impressions, so
  land a complete hook or an open loop before the fold. The headline is a
  second hook carrying a specific benefit, not a button label. Match the
  hook to the reader's awareness level, and back each claim with a number,
  name or timeframe. Offer 2 or 3 variations with different hooks (curiosity,
  contrarian, social proof).
- **Landing page**: three short paragraphs. The discovery (authority and the
  root cause), the proof (results with numbers), the call to action (urgency
  and where the click leads). Match the message of the ad that sends traffic
  here. Keep paragraphs short for phones.
- **Direct response**: specific headline, an opening line that earns the
  second line, open loops, quantified pain, a "so what?" chain from feature
  to outcome, real testimonials, a clear single CTA.
- **Tweet**: front-load the claim in the first line. One idea per post. No
  hashtag stacks, no bait, links go in a reply.

## 6. Fallback when the gate cannot run

If Node, git or network access is missing, do not pretend to pass. Check the
draft yourself against every rule in section 5, counting characters and
paragraph lengths with whatever tool you have (a short Python script is
fine), and fix what you find. Then present it labelled clearly as **"not
gate-checked: manual pass only"**, and say what blocked the gate so the user
can fix it.
