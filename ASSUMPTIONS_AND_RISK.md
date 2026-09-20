# DESFlix — Assumptions, Evidence, and Risk

This document exists because the rest of the blueprint (architecture, dataflows,
tokenomics, quality gates, mockups) describes a *design*, not a proven system. A
design document that only enthuses about the idea is not useful; this file is
where the idea gets stress-tested before any of it is built. Read this before
docs/*.md — several documents downstream reference the open questions raised
here and pick a specific answer, flagged `[DECISION]`.

Scope note: this is a product/systems analysis, not legal advice. Anywhere
this file discusses securities, money-transmission, or gambling law, treat
it as "questions to put in front of counsel," not as a legal conclusion.

---

## 1. Explicit assumptions

Numbered so later docs can reference them (`A1`, `A2`, ...). Each has a
confidence rating for "this is a reasonable assumption to design against,"
not for "this will definitely happen."

| # | Assumption | Confidence |
|---|---|---|
| A1 | "AI-generated" here means text-to-video / diffusion / neural-rendering pipelines producing full episodes and films from a creator's script, storyboard, voice, and style references — not merely AI-assisted VFX on live-action footage. | High (this is the plain reading of "bring their designs to life") |
| A2 | DESFlix is a greenfield product with no existing user base, catalog, or capital committed yet — this blueprint is pre-seed / pre-funding planning. | High (stated by user: "new project," "new repo," no code yet) |
| A3 | "P2P AI rendering" means the platform can offload inference/rendering compute to a distributed network of contributor-operated nodes (viewers, enthusiasts, render-farm operators), as an alternative or supplement to centrally-owned GPU capacity. | High (explicit in the request) |
| A4 | The cryptocurrency is a platform-native token used to (a) reward raters, (b) pay P2P compute contributors, and (c) settle product-placement/advertiser spend. Whether it is a blockchain token, a permissioned ledger, or an off-chain points system redeemable for cash is **not yet decided** — see Decision D1 in `docs/tokenomics.md`. | Medium (mechanism unspecified by user, only the *outcome* — "get paid for rating" — is specified) |
| A5 | "Quality gates like Netflix" means DESFlix wants an editorial bar comparable to a premium streamer's curated catalog, not literally Netflix's internal tooling (which is not public). | High |
| A6 | "No shorts" means no content under some minimum runtime threshold (a full episode/film, not TikTok/Reels-style clips), and probably also no bait-y micro-content used purely for feed engagement. Exact threshold is undefined — flagged as an open parameter. | Medium |
| A7 | The leaderboard's purpose is twofold: signal viewer demand back to creators/the platform ("make more of this"), and provide social proof to advertisers evaluating placement spend. | Medium (inferred from "so people know they want to see more of it" + "companies can pay for product placement if something becomes popular") |
| A8 | Product placement is inserted post-hoc / dynamically into AI-rendered content (swap a branded prop, sign, or background asset) rather than requiring a full re-render or re-shoot per deal. | Medium-High (this is the only version of "pay for product placement" that is meaningfully differentiated by an AI-rendering pipeline; a traditional baked-in placement would work identically to how film/TV already does it, which wouldn't need this platform's specific architecture) |
| A9 | Target regulatory environment is unspecified. This blueprint assumes a US/EU-facing consumer product by default because no jurisdiction was given, and flags every place that assumption changes the risk profile. | Low (pure default, stated explicitly so it can be corrected) |

If any of A1–A9 is wrong, the affected downstream document is named so the
correction is localized instead of invalidating the whole blueprint.

---

## 2. Fact, inference, and speculation — kept separate

Mixing these is the main way blueprints go wrong: a speculative claim
("volunteer compute will be cheap") quietly gets treated with the same
confidence as a fact ("diffusion models can generate video today") a few
paragraphs later. They're separated here on purpose.

### 2.1 Facts (independently verifiable, not specific to DESFlix)

- Text-to-video and image/video diffusion models capable of producing
  short-to-medium AI-generated video clips are a real, deployed technology
  class as of this writing. *Confidence: high.* (Public, widely reported
  technology category; exact current SOTA quality is a moving target I
  cannot verify live in this session.)
- Volunteer/distributed compute networks are an established pattern with a
  multi-decade track record for **batch-tolerant, easily-verified** workloads
  — e.g., BOINC-family scientific computing, Folding@home. *Confidence: high
  that the pattern exists and works for that workload class.*
- Token-incentivized distributed compute/rendering marketplaces exist as a
  real project category (e.g., render-farm token networks, decentralized
  transcoding networks, decentralized compute marketplaces). *Confidence:
  high that the category exists; low confidence on any specific project's
  current scale, pricing, or reliability — I cannot verify live metrics in
  this session and won't invent numbers.*
- Token-curation / pay-for-engagement social platforms (rewarding users in a
  native token for upvotes, ratings, or content curation) have been tried
  multiple times since roughly the mid-2010s, and multiple independent
  implementations have publicly documented problems with vote manipulation,
  bot/Sybil farming, and reward concentration among a small set of large
  holders. *Confidence: high that this failure pattern is real and recurring
  across independent projects; I'm not citing specific current statistics
  because I can't verify them live, but the pattern is well enough
  documented in tech press and postmortems over ~8+ years to treat as
  established, not speculative.*
- In distributed/untrusted computing generally, a worker node can return an
  incorrect or fabricated result for a job it was paid to run, and the
  requester cannot detect this without either redundant computation,
  cryptographic proof of correct execution, or trusted hardware. This is a
  structural property of the trust model, not a DESFlix-specific risk.
  *Confidence: high — this is why verifiable-computation research exists.*
- Premium streaming platforms curate their catalog primarily through
  editorial/human greenlighting *before* production plus data-driven renewal
  decisions *after* release, not through a public, automated, per-title
  "quality gate" that content must pass. *Confidence: moderate-high based on
  well-documented industry reporting; internal tooling specifics are not
  public and I'm not claiming to know them.*

### 2.2 Inferences (reasoned from the facts above, not directly observed)

- If rating content is directly and individually paid, and that same rating
  feeds a public leaderboard that plausibly affects creator payouts, catalog
  placement, or ad/placement value, then rational actors are incentivized to
  create or automate additional identities to farm rating rewards — *unless*
  the marginal cost of a credible additional identity exceeds the marginal
  reward per rating. This follows from basic incentive-compatibility
  reasoning applied to the stated design; it does not require citing a
  specific historical failure, though §2.1 shows the historical failure
  pattern is also observed.
- A purely automated "no AI slop" filter can plausibly catch objective,
  checkable defects (broken audio sync, unresolved artifacts, banned/unsafe
  content, duplicate/plagiarized assets, undisclosed-AI-origin violations),
  but subjective creative quality ("is this a good story, well acted, well
  paced") is not the kind of property current automated classifiers reliably
  judge at a human-editorial standard. So a gate that is genuinely
  "Netflix-grade" almost certainly requires a human review stage, not only
  automated slop-detection — otherwise "quality gate" silently narrows to
  mean "compliance gate," which is a materially smaller claim than "no AI
  slop... just like Netflix."
- Because generative pipelines can re-render a scene with a swapped asset
  far more cheaply than reshooting live-action footage, per-viewer or
  per-campaign dynamic product placement is a technically coherent,
  differentiated capability for this specific architecture — this is not
  true for a normal video platform showing pre-baked film, which is why the
  A8 assumption is plausible rather than arbitrary.

### 2.3 Speculation (plausible, but currently unverified either way)

- That a token-based rating-and-leaderboard system will produce a *better*
  demand/quality signal than the engagement metrics incumbents already use
  (completion rate, rewatch rate, unprompted return visits). No evidence
  either direction; this needs a pilot, not a guess.
- That volunteer/P2P compute can be cost-competitive with centralized GPU
  cloud capacity specifically for **latency-sensitive, IP-sensitive,
  production-quality generative video rendering** — as opposed to the
  batch-tolerant, easily-verified workloads where P2P/volunteer compute has
  an actual track record (§2.1). Generative output is non-deterministic
  across runs and hardware, which makes it far more expensive to verify
  cheaply than, say, transcoding a known source file (checksum/perceptual
  hash) or scientific batch jobs (deterministic recomputation). This is the
  single largest unproven technical assumption in the whole blueprint.
- That regulators would classify the proposed token as a utility/rewards
  instrument rather than a security or money-transmission instrument.
  Genuinely unknown, jurisdiction-dependent, and outside what I can
  responsibly conclude here — see §5.
- That "no AI slop" can be reduced to a rubric precise enough to be applied
  consistently at scale without creators learning to game the rubric's
  specific checks instead of actually improving quality (Goodhart's Law).

**Do not treat §2.3 items as settled** anywhere else in this blueprint — where
downstream docs rely on one of them, they say so explicitly and name the
validation step needed before committing engineering budget to it.

---

## 3. The central open question, treated as competing hypotheses

Stated plainly: *does paying users directly for the specific rating that
becomes the public quality/demand signal produce a trustworthy signal, or
does it collapse into Sybil/bot noise?* This is the single riskiest
mechanism-design choice in the product, so it gets a full hypothesis
treatment rather than being asserted.

**H1 — "It works with strong anti-Sybil infrastructure."**
Proof-of-personhood, stake-to-rate, reputation decay, and ML fraud detection
keep the cost of a fraudulent rating above its reward, so genuine signal
dominates.
*Evidence for:* identity-gated platforms (KYC'd trading/prediction markets)
show materially lower manipulation than open, anonymous ones — a real,
observable contrast.
*Evidence against:* proof-of-personhood systems have themselves reported
large-scale farming of their own sign-up process; SIM-farm-driven airdrop
and reward farming is a recurring, independently-reported problem across
many unrelated crypto reward programs.
*Verdict:* plausible, but only with anti-Sybil infrastructure that is
itself expensive to build and maintain — most startups underinvest here
precisely because it doesn't look like the "core product."

**H2 — "Paying for the exact signal you want corrupts the signal, structurally."**
Goodhart's/Campbell's Law: once a rating is also a paycheck, it stops
measuring taste and starts measuring "what maximizes payout," and every
patch to the reward algorithm gets gamed in turn.
*Evidence for:* multiple independent token-curation platforms since ~2016
have documented this exact failure mode converging toward whale/bot-heavy
distributions, under different specific implementations — independent
replications of the same failure across unrelated teams is stronger
evidence than one project's postmortem.
*Evidence against:* none of those platforms had the specific anti-Sybil
stack proposed in H1 fully deployed at launch, so H2 is not proven immune
to a sufficiently well-funded defense — absence of a counterexample is not
proof that no counterexample can exist.
*Verdict:* the strongest-supported hypothesis by historical base rate, but
not proof that no implementation can beat it.

**H3 — "Decouple payment from the public ranking signal."**
Pay a flat, capped, identity-gated stipend for the *act* of rating (like a
research panel), while computing the public leaderboard from a separate,
not-directly-paid signal (completion rate, rewatch rate, deliberate
return visits) — the same category of signal incumbents already trust.
*Evidence for:* no enduring large consumer content platform today ranks
content by a number it pays users, per-unit, to produce. After roughly a
decade of both incumbent streaming platforms and token-curation
experiments coexisting, the absence of a durable counterexample is *weak*
but real evidence — weak because absence of evidence isn't evidence of
absence, real because it's a long enough window that someone would likely
have shown it working if it reliably did.
*Verdict:* the option best supported by the overall evidence, and the
one this blueprint's `docs/tokenomics.md` designs toward — but this is a
recommendation, not a resolution imposed on the user's stated design; the
alternative (pay directly for the rating that ranks content) is documented
alongside it as `[DECISION]` D2, explicitly left open.

**Strongest counterargument to the whole DESFlix concept, steelmanned:**
An AI-generation platform with a human curation bar is arguably *not*
differentiated by the P2P-compute or crypto-payout layers at all — a
centrally-hosted platform with the same quality bar, ordinary cloud
rendering, and ordinary fiat creator payouts could deliver the same
"no-slop, no-shorts, writers bring designs to life" value proposition with
far less engineering and regulatory risk. The P2P and token layers add the
platform's two largest unproven-feasibility risks (§2.3) and its only
open legal risk (§5) without a demonstrated benefit over the simpler
alternative. If the user's actual goal is "a curated AI-content streamer,"
the P2P/token layers should be treated as a *phase 2 bet*, not phase 1
infrastructure — this is reflected in `docs/architecture.md`'s phased
rollout, not decided away here.

---

## 4. What would falsify or strengthen each major claim

Concrete, checkable — not rhetorical.

| Claim | Would be falsified/weakened by | Would be strengthened by |
|---|---|---|
| "Pay-to-rate produces trustworthy signal" (H1 over H2) | A closed pilot cohort shows rating distributions diverge sharply between a paid group and an unpaid control group once obvious bots are filtered | Paid and unpaid cohorts converge on similar rankings after fraud-filtering, across more than one content genre |
| "Automated quality gate can hit Netflix-grade curation" | Automated rubric scored against a human-expert-labeled test set shows precision/recall no better than raw engagement metrics | Rubric matches human-panel judgments at a rate comparable to inter-rater agreement between two human reviewers |
| "P2P compute is viable for production rendering" | Pilot P2P render jobs show output verification (redundant-run agreement) failing above an acceptable threshold, or latency/cost worse than centralized cloud after accounting for redundancy overhead | Pilot shows P2P cost-per-render below centralized cloud even after 2–3x redundant-run verification overhead, at acceptable latency |
| "Dynamic product placement is viable" | Re-rendering a scene with a swapped asset introduces visible artifacts, continuity breaks, or unacceptable render cost at scale | A pilot swap-render is visually indistinguishable from the baseline and cheap enough to run per-campaign |
| "Token avoids securities/money-transmission classification" | Counsel concludes the token's investment-return characteristics (assumptions A4, D1) trigger securities treatment, or that per-user payouts trigger money-transmitter licensing | Counsel concludes a specific implementation (e.g., non-transferable points redeemable only for platform perks) sits outside that regulatory perimeter |

## 5. Legal and regulatory questions for counsel (not conclusions)

Flagging these because the user's brief explicitly includes a crypto payout
mechanism; not answering them here because that would be presenting
unverified legal conclusions as fact, which the response-style constraints
this session operates under specifically forbid.

- Does paying users a tradable/liquid token for rating content, where the
  token's value may appreciate with platform success, create an investment
  contract under a Howey-style test in the relevant jurisdiction(s)?
- Does routing per-user crypto payouts at platform scale trigger
  money-transmitter licensing obligations, and in which jurisdictions?
- Does "rate content, get paid, climb a leaderboard" risk classification as
  a game of chance / sweepstakes mechanic in jurisdictions with gambling
  regulation, depending on how reward variance is structured?
- What KYC/AML obligations attach to (a) paying out compute contributors,
  (b) paying out raters, (c) accepting advertiser/placement spend, once
  real money enters or leaves the token?
- What content-liability exposure exists for AI-generated series depicting
  real people, trademarked products (outside paid placement), or
  copyrighted training-adjacent style, in the platform's target markets?

**None of these are answered in this blueprint.** Treat every architecture
and tokenomics decision downstream that touches money movement as
provisional pending counsel's input — this is called out again at the
specific decision points in `docs/tokenomics.md`.

---

## 6. Confidence summary (feasibility, not desirability)

| Subsystem | Confidence it is *technically achievable* as described | Basis |
|---|---|---|
| AI generation of full episodes/films from creator input | High | Established technology category (§2.1); quality bar vs. human-made content is the open variable, not existence |
| Automated compliance/safety gate (banned content, plagiarism, technical QC) | High | Automatable, checkable properties |
| Automated *creative-quality* "no slop" gate matching Netflix-level judgment | Low, without a human review layer | §2.2 inference; no known automated system does this reliably today |
| P2P rendering competitive with centralized cloud for this workload | Low–Medium, unproven | §2.3; closest precedent (BOINC/transcoding) has easier verification properties than generative rendering |
| Pay-per-rating token producing trustworthy signal without decoupling (H2 risk) | Low, absent heavy anti-Sybil investment | §3, repeated independent historical failure pattern |
| Dynamic product placement enabled by re-render pipeline | Medium-High | §2.2 inference, technically coherent, not yet piloted |
| Token avoiding securities/money-transmission exposure | Unknown | §5 — genuinely requires counsel, not estimable here |

Everything in this table should be re-scored once the first pilot data
exists; treat this as the pre-pilot baseline, not a permanent verdict.
