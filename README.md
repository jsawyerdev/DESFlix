# DESFlix

**D**istributed **E**ntertainment **S**ervice — a blueprint for a curated,
AI-generated streaming platform where writers/creators turn scripts and
designs into full episodes/films via generative AI, rendered partly on a
distributed (P2P) compute network, published only after passing an
editorial quality bar, and ranked by a viewer leaderboard tied to a
token-based rating-reward system. Companies can buy dynamic product
placement in content that's trending.

**Status: blueprint only. No code in this repo, by design.** This is the
pre-implementation design phase: architecture, dataflows, data model,
economic model, and UI mockups, plus an explicit feasibility/risk analysis.
See `ASSUMPTIONS_AND_RISK.md` before anything else — several documents
below make specific design choices in response to risks identified there.

## Read order

1. **[`ASSUMPTIONS_AND_RISK.md`](./ASSUMPTIONS_AND_RISK.md)** — explicit
   assumptions (`A1`-`A9`), fact/inference/speculation breakdown, competing
   hypotheses on the platform's riskiest mechanism-design question
   (pay-to-rate incentive integrity), falsification criteria, legal/
   regulatory open questions, and a per-subsystem feasibility confidence
   table. Read this first — everything else builds on it.
2. **[`docs/architecture.md`](./docs/architecture.md)** — context diagram,
   container diagram, subsystem responsibilities, deployment topology, and
   a phased rollout that sequences the riskiest subsystems (P2P rendering,
   token payouts) after the independently-defensible core product.
3. **[`docs/dataflows.md`](./docs/dataflows.md)** — four end-to-end sequence
   diagrams: creator upload -> render -> publish; watch -> rate -> payout ->
   leaderboard; compute contribution -> verification -> node payout;
   advertiser placement -> dynamic render -> performance report. Each
   includes the failure modes specific to that flow.
4. **[`docs/content-quality-gates.md`](./docs/content-quality-gates.md)** —
   the "no AI slop, no shorts, quality gates like Netflix" system: a
   five-stage pipeline (automated compliance -> technical QC -> format
   rules -> community jury -> professional curator), with an explicit
   argument for why the last stage is mandatory rather than optional.
5. **[`docs/p2p-rendering.md`](./docs/p2p-rendering.md)** — the distributed
   render network: node lifecycle, tiered verification model (and its real
   cost), job routing by IP sensitivity, node economics, and collusion
   resistance. Flagged throughout as the least-proven subsystem.
6. **[`docs/tokenomics.md`](./docs/tokenomics.md)** — the crypto payment
   system: an open ledger-mechanism decision (`D1`), and a specific
   recommendation (`D2`) for decoupling "get paid for rating" from "the
   number that drives the public leaderboard," with the brief's literal
   pay-per-rating-drives-leaderboard design documented alongside it as an
   explicit alternative, not silently overridden.
7. **[`docs/advertising-product-placement.md`](./docs/advertising-product-placement.md)**
   — the "companies pay for placement when something's popular" system:
   planned vs. reactive placement, the scoped-re-render mechanism that
   makes reactive placement technically distinctive, marketplace flow, and
   disclosure requirements.
8. **[`docs/data-model.md`](./docs/data-model.md)** — conceptual entity
   relationships tying every subsystem's vocabulary together (no schema/
   code), so "a rating," "a render job," "a placement campaign" mean the
   same thing across every document above.
9. **[`mockups/`](./mockups/)** — static HTML/CSS UI mockups: home/discovery,
   watch + rate player, leaderboard, creator studio, node operator
   dashboard, and wallet. Open `mockups/index.html` in a browser.

## What this blueprint deliberately does NOT claim

- That the P2P rendering network is cost-effective versus centralized cloud
  GPU — this is explicitly unproven (`ASSUMPTIONS_AND_RISK.md` §2.3, §6)
  and gated behind a pilot in the phased rollout.
- That paying users directly for the exact rating that drives a public
  leaderboard is a safe design — the dominant historical pattern in
  comparable systems is Sybil/bot degradation (`ASSUMPTIONS_AND_RISK.md`
  §3), which is why `docs/tokenomics.md` recommends decoupling payout from
  ranking signal rather than building the brief's literal mechanism as
  stated.
- That the token avoids securities or money-transmission regulatory
  exposure — genuinely unresolved, flagged for counsel
  (`ASSUMPTIONS_AND_RISK.md` §5), not guessed at.
- That an automated system can reliably judge creative quality at a
  "Netflix-grade" bar — low confidence; the quality-gate pipeline keeps a
  mandatory human curator stage specifically because of this
  (`docs/content-quality-gates.md` §2).

## Key open parameters (not yet decided, intentionally)

- Runtime floor for "no shorts" (episode/film minimums).
- Ledger mechanism: public token vs. permissioned ledger vs. off-chain
  points (`docs/tokenomics.md` `D1`).
- Whether to build the brief's literal pay-per-rating leaderboard design or
  the decoupled recommendation (`docs/tokenomics.md` `D2` vs `D2-alt`).
- Target launch jurisdiction(s) (`ASSUMPTIONS_AND_RISK.md` `A9`) — changes
  the regulatory risk profile materially.

## Assumptions summary

Full detail in `ASSUMPTIONS_AND_RISK.md` §1; headline items:

- "AI-generated" = full generative pipeline (script -> voice/style refs ->
  rendered video), not AI-assisted VFX on live footage.
- This is pre-seed planning: no existing users, catalog, or capital
  assumed.
- The cryptocurrency's exact mechanism (public chain vs. permissioned
  ledger vs. points) is open — only the user-facing outcome ("get paid for
  rating") is treated as fixed by the brief.
- Target jurisdiction defaults to a generic US/EU-facing consumer product
  in the absence of a stated target market — flagged as the weakest
  assumption in the set, easiest to correct.
