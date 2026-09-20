# DESFlix -- System Architecture

Status: conceptual blueprint, no implementation. See `../ASSUMPTIONS_AND_RISK.md`
for the assumptions (`A1`-`A9`) this architecture is built on, and for the
feasibility caveats on the P2P-rendering and tokenomics subsystems in
particular -- this document describes what those subsystems would need to do,
not a claim that the approach is proven.

## 1. Context diagram -- who and what DESFlix talks to

```mermaid
flowchart LR
    subgraph People
        Creator[Creator / Writer]
        Viewer[Viewer]
        NodeOp[Compute Node Operator]
        Advertiser[Advertiser / Brand]
        Curator[Human Curator / Reviewer]
    end

    DESFlix((DESFlix Platform))

    Creator -->|scripts, storyboards, voice/style refs| DESFlix
    DESFlix -->|render jobs| NodeOp
    NodeOp -->|rendered assets + proofs| DESFlix
    DESFlix -->|catalog, streams| Viewer
    Viewer -->|ratings, downvotes, watch signal| DESFlix
    DESFlix -->|token payouts| Viewer
    DESFlix -->|token payouts| NodeOp
    DESFlix -->|token payouts, royalties| Creator
    Advertiser -->|placement bids, brand assets| DESFlix
    DESFlix -->|placement inventory, performance reports| Advertiser
    Curator -->|review decisions| DESFlix
    DESFlix -->|review queue| Curator

    ExtCloud[(Centralized GPU Cloud\nburst capacity)]
    DESFlix <-->|overflow render jobs| ExtCloud

    PaymentRail[(Fiat on/off-ramp\n+ KYC/AML provider)]
    DESFlix <-->|cash-out, compliance checks| PaymentRail
```

## 2. Container diagram -- major subsystems

```mermaid
flowchart TB
    subgraph ClientLayer[Client Layer]
        WebApp[Web/TV/Mobile Apps]
        CreatorStudio[Creator Studio]
        NodeClient[Node Operator Client]
    end

    subgraph EdgeLayer[Edge / Delivery]
        CDN[CDN + Adaptive Streaming]
        API[API Gateway]
    end

    subgraph CoreServices[Core Platform Services]
        Catalog[Catalog & Metadata Service]
        Ingest[Creator Ingest Service]
        Gate[Quality Gate Pipeline]
        Scheduler[Render Job Scheduler]
        Rating[Rating & Leaderboard Service]
        Wallet[Wallet / Ledger Service]
        AdEngine[Placement & Ad Engine]
        TrustSafety[Trust & Safety / Moderation]
        Identity[Identity & Reputation Service]
    end

    subgraph ComputeLayer[Distributed Render Layer]
        NodeRegistry[Node Registry & Staking]
        JobVerifier[Redundant-Run Verifier]
        P2PNodes[(P2P Render Nodes)]
        CloudBurst[(Centralized Cloud Burst)]
    end

    subgraph DataLayer[Data Layer]
        MetaDB[(Metadata Store)]
        Ledger[(Token Ledger)]
        AssetStore[(Object Storage: video/audio/models)]
        EventLog[(Event Log / Analytics)]
    end

    WebApp --> API
    CreatorStudio --> API
    NodeClient --> API
    API --> Catalog
    API --> Ingest
    API --> Rating
    API --> Wallet
    API --> AdEngine

    Ingest --> Gate
    Gate --> Scheduler
    Scheduler --> NodeRegistry
    NodeRegistry --> P2PNodes
    Scheduler --> CloudBurst
    P2PNodes --> JobVerifier
    JobVerifier --> AssetStore
    JobVerifier --> Gate

    Gate --> TrustSafety
    Gate --> Catalog
    Catalog --> CDN
    CDN --> WebApp

    Rating --> Identity
    Rating --> Wallet
    Rating --> EventLog
    Wallet --> Ledger
    AdEngine --> Catalog
    AdEngine --> Wallet

    Catalog --> MetaDB
    Ingest --> AssetStore
```

## 3. Subsystem responsibilities

| Subsystem | Responsibility | Key risk flagged in ASSUMPTIONS_AND_RISK.md |
|---|---|---|
| Creator Ingest Service | Intake scripts, storyboards, voice/style refs, casting/character models; versioned project workspace | -- |
| Quality Gate Pipeline | Multi-stage review: automated compliance checks -> automated technical QC -> human curator review -> publish decision. See `content-quality-gates.md`. | Automated creative-quality gating is low-confidence without human review (sec. 2.2, sec. 6) |
| Render Job Scheduler | Breaks a project into renderable jobs (per-scene/per-shot), assigns to P2P nodes or cloud burst based on SLA, sensitivity, and cost | -- |
| Node Registry & Staking | Onboards compute contributors, tracks stake/reputation, assigns jobs, slashes for bad output | P2P viability unproven for this workload (sec. 2.3) |
| Redundant-Run Verifier | Runs sensitive jobs on 2+ independent nodes and compares output (perceptual hash / statistical agreement) before accepting | Verification cost directly offsets any P2P cost advantage -- see `p2p-rendering.md` |
| Rating & Leaderboard Service | Collects ratings/downvotes, computes leaderboard, feeds Wallet for payout | Pay-per-rating signal integrity is the platform's single biggest open risk (sec. 3) |
| Identity & Reputation Service | Proof-of-personhood, account reputation, Sybil-resistance signal shared by Rating and Node Registry | Anti-Sybil infra is expensive and imperfect (sec. 3, H1) |
| Wallet / Ledger Service | Token balances, payouts to raters/creators/node operators, advertiser billing, fiat on/off-ramp integration | Legal/regulatory questions open (sec. 5) |
| Placement & Ad Engine | Matches advertiser campaigns to trending content, triggers dynamic asset re-render, tracks placement performance | Depends on re-render pipeline (A8) |
| Trust & Safety / Moderation | Content policy enforcement, appeals, takedown, abuse response | -- |
| CDN + Adaptive Streaming | Standard video delivery; not differentiated from any other streaming platform | -- |

## 4. Deployment topology (conceptual, not infra-as-code)

```mermaid
flowchart LR
    subgraph Region[Per-Region Deployment]
        RegionalCDN[Regional CDN PoPs]
        RegionalAPI[Regional API Gateway]
    end
    subgraph Central[Central / Home Region]
        CoreCluster[Core Services Cluster]
        Ledger[(Token Ledger - single source of truth)]
        Gate[Quality Gate Pipeline]
    end
    subgraph Distributed[Globally Distributed]
        Nodes[(P2P Render Nodes)]
        Cloud[(Cloud Burst GPU Pools\nmulti-region)]
    end

    RegionalAPI --> CoreCluster
    RegionalCDN --> CoreCluster
    CoreCluster --> Ledger
    CoreCluster --> Gate
    Gate --> Nodes
    Gate --> Cloud
```

Notes:
- The token ledger is drawn as a single source of truth, not "the blockchain
  handles consistency" -- whether it's a permissioned ledger, an
  application-managed database, or an actual public chain is `[DECISION]`
  D1 in `tokenomics.md`, deliberately not resolved by this diagram.
- Cloud burst capacity exists in the architecture from day one, not as a
  fallback bolted on later -- per the steelman counterargument in
  `ASSUMPTIONS_AND_RISK.md` sec. 3, the platform should be able to run its core
  content-quality value proposition even if the P2P layer underperforms.

## 5. Phased rollout (addresses the steelman counterargument directly)

The riskiest, least-proven pieces (P2P rendering, token payouts) are
sequenced *after* the parts of the product that are independently
defensible, so a P2P or tokenomics failure doesn't take down the whole
platform.

```mermaid
flowchart TB
    P1["Phase 1: Curated AI content platform\nCentral cloud rendering, human quality gate,\nfiat creator payouts, ordinary engagement leaderboard\n(no token, no P2P)"]
    P2["Phase 2: Token-based rating incentive\nPilot pay-to-rate on closed cohort,\nvalidate against H1/H2/H3 (ASSUMPTIONS_AND_RISK.md sec. 3)"]
    P3["Phase 3: P2P render network\nOpt-in node operators, redundant-run verification,\ncloud burst remains primary path until proven"]
    P4["Phase 4: Dynamic product placement\nRe-render pipeline + advertiser marketplace"]
    P1 --> P2 --> P3 --> P4
```

Each phase has an explicit go/no-go gate tied to the falsification criteria
in `ASSUMPTIONS_AND_RISK.md` sec. 4 -- a phase does not proceed on schedule, it
proceeds on evidence.
