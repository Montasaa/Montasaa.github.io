# Agent Atlas: product and architecture plan

Status: Deliverable 1 (visual foundation) built. Needs your approval before Deliverable 2.
Prototype: `atlas/index.html` (one self-contained file, no build step).

---

## 1. What the brief is actually asking for

You asked for six things: an architecture map, an agent workspace, an infrastructure map, a workflow designer, a cost manager and a planning environment. Underneath, they all need the same thing: **one typed model of your AI environment that can be read in several ways and changed safely.**

If each of the six becomes its own screen, you end up with six dashboards that drift apart. Atlas uses **one graph, several lenses and drafts on top**:

| Your question | Answered by |
|---|---|
| What do I own, and how does it connect? | Architecture lens |
| Where does it run? What's on the VPS? | Infrastructure lens (same nodes, grouped by host) |
| What am I paying for, and what depends on it? | Economics lens plus access-path analysis |
| How does a task move through the system? | Flows lens (animated signal along real relations) |
| What if I add, remove or move X? | Drafts, which are change-sets compared against reality |
| What should I build next? | Findings engine (rules over the graph, ranked) |

The key product decision is that **the thinking-tool part lives in drafts plus consequence analysis, not in the canvas.** Dragging boxes around is not what makes this a planning tool. The planning comes from Atlas telling you, after a change, which agents can't run any more, what cost moved, and which new risks appeared.

## 2. What the references contribute (and what they don't)

| Reference | What I took | What I left |
|---|---|---|
| Micro-personalization engine | Left-to-right causal flow (input → engine → output), a feedback band beneath, and a thin signal line as the only glow | Stacked cards and marketing badges |
| Survey flow tree | Branching routes drawn as quiet connectors with orange junction points; the idea that a path is the product | Tree layout (your system is a graph, not a tree) |
| DeerFlow tiles | Dot-matrix numerals (the Doto face) for figures such as cost; precise small caps labels; black-on-black depth | A wall of 12 metric tiles. Most of those numbers would be invented for you today |

The tone is warm near-black (#0a0908), hairline lines, one signal colour (#ff7a1a). Orange is reserved for **meaning**: the selected item, its dependency chain, proposed changes, and a task in motion. The page has no decorative glow.

## 3. Product architecture

```
┌──────────────────────── Interfaces ────────────────────────┐
│  Atlas web UI   ·   (later) Atlas MCP server   ·   REST     │
└──────────────┬────────────────────────────────┬────────────┘
               │ read / propose                  │
┌──────────────▼──────────── Core ───────────────▼────────────┐
│  Domain graph        Scenario engine      Analysis engine   │
│  (entities,          (drafts = ops on     (access paths,    │
│   relations,          a parent; diff)      impact, findings, │
│   provenance)                              cost roll-up)     │
│                 Event log (every change, who/what made it)   │
└──────────────▲────────────────────────────────▲─────────────┘
               │ proposed changes               │
┌──────────────┴─────── Sources (adapters) ─────┴─────────────┐
│ Manual · Hermes controller · VPS discovery agent · Billing   │
└──────────────────────────────────────────────────────────────┘
```

- **Atlas owns the model.** Hermes is one adapter among several. If Hermes is replaced, Atlas still works.
- Sources never write straight into reality. They submit **proposed changes**, which you approve or which auto-apply under rules you set, such as "health status auto-applies; new agents need approval".

## 4. Information architecture

- **Top bar**: lens (Architecture · Infrastructure · Economics · Flows), search (`/`), scenario (Current, drafts, new, duplicate), Compare, Add, Data.
- **Left: Brief.** What you're looking at (reality or a draft), the single *Build next* recommendation, inventory by kind (click to isolate), known monthly cost, and ranked findings (click to spotlight).
- **Centre: Map.** Pan, zoom, semantic zoom (labels simplify when zoomed out), and a dependency-chain focus when you select something.
- **Right: Inspector.** Shaped per kind. An agent shows its *operating context* (work from → agent → hands to; runs in → thinks with → paid via; tools, memory, review; a verdict). An access plan shows what it unlocks and its blast radius. A harness shows what models it can actually reach and through which plan.
- **Overlays**: draft banner, economics strip, flow timeline, data sources.

## 5. Domain model (implemented in the prototype)

**Entity kinds (11):** person, interface, agent, harness, model (family), access (subscription or API plan), host, tool, mcp, memory, project.

**Relation types (13), each with legal endpoint kinds:**

| Relation | From → To | Meaning |
|---|---|---|
| reaches | person → interface | you use this interface |
| routes_to | interface → agent/harness | entry point for work |
| delegates_to | agent → agent | orchestration |
| review_by | agent → agent | independent review |
| runs_in | agent → harness | execution runtime |
| uses_model | agent → model (`fallback` flag) | intelligence used |
| grants | access → model | a plan gives you a model family |
| authorizes | access → harness | a plan can be used inside a harness |
| hosted_on | agent/harness/interface/mcp/memory/tool/project → host | location |
| uses_tool / connects_mcp / uses_memory / tracks_in | agent/harness → capability | what it can touch |

Illegal links are refused with an explanation. For example, if you try to link an agent directly to a subscription, Atlas says: *"Agents don't pay directly. Give the agent a harness and a model; Atlas finds the access plan that covers both."*

**The core rule is the access path.** An agent can run only if some access plan both *grants* its model and *authorizes* its harness. This is why "subscription ≠ model ≠ harness" matters. Claude models are available through two of your plans (Claude and Google/Antigravity), but only one of them works inside Claude Code. Removing a plan therefore breaks specific agents, not "everything that uses Claude".

**Every fact carries provenance:** `stated` (you said it), `documented` (vendor docs), `inferred` (Atlas guessed, shown dotted), `proposed` (a draft), `recorded` (you edited it). Unknown values stay `null` and render as "unknown". Nothing is filled in with a guess.

**Scenarios are change-sets.** A draft is `{parent, ops[]}` with ops `add | link | remove | unlink | set`. Reality is never copied. Diffing, forking ("What if I remove X?") and later *applying* a draft all follow from this.

**Positions are view state.** Lens, pins, focus and zoom never touch the graph. Open **Data** in the prototype to see the exact JSON the map is drawn from.

## 6. Deliverables

| # | Deliverable | Goal | What works at the end | What stays conceptual | Acceptance |
|---|---|---|---|---|---|
| **1** | **Visual foundation** *(built)* | Make the idea visible; approve the direction | Graph model, 4 lenses, drafts and compare, what-if fork, access-path analysis, findings, agent operating context, animated workflows, create/connect/disconnect/move | Persistence beyond this browser, real data, auth, discovery | You can judge the visual language, the navigation, density and the interaction philosophy |
| 2 | Real inventory | Replace the seed with your actual environment | Guided capture (tiers, costs, where each harness runs, which agents exist), import/export JSON, entity editing in reality with audit trail | Automation | Every question in your brief's "When I open the application" list answers from your own data or says "unknown" |
| 3 | Persistent backend and event log | Stop living in localStorage | Small service (SQLite/Postgres) holding graph, drafts and events; REST API; apply-draft-to-reality | Multi-user | Two browsers see the same workspace; every change is attributable |
| 4 | Agent and workflow design depth | Design agents as first-class objects | Agent templates, skills, permissions, approval gates, workflow editor (not just playback), fallbacks | Running the workflows | You can design a new workflow and Atlas flags a step no agent can perform |
| 5 | Economics v2 | Cost *and* capacity | Usage import, quota windows (5-hour/weekly), utilization, cost per workflow, overlap scoring | Forecasting | "What does adding X change?" answered with $, quota and dependents |
| 6 | Live sync | Atlas mirrors reality | Atlas MCP server + adapters (Hermes, VPS discovery, billing); proposed-change inbox; health states on the map | Autonomous remediation | Installing a tool on the VPS shows up as a proposed change within minutes |
| 7 | Controller mode | Atlas as control plane | Hermes (or any agent) can read Atlas and act on approved drafts | — | A draft applied in Atlas results in a real configuration change, with audit |

Dependencies: 2 needs your input. 3 is required before 6. 5 needs 3 plus a usage source. 7 needs 6 and trust rules.

## 7. Path to live automation

1. **Adapter contract.** Each source emits `ProposedChange {source, observedAt, ops[], evidence}`, which uses the same op vocabulary as drafts. There is one pipeline, whether the change comes from you, Hermes or a script.
2. **Atlas MCP server.** It exposes `atlas.query`, `atlas.propose`, `atlas.impact(draft)` and `atlas.findings`. Any MCP-capable harness can then ask "what would break if…" before acting. This is the least-coupled way to let Hermes participate.
3. **VPS discovery agent.** A small daemon that lists containers, systemd services, listening ports and known agent configs, then proposes host contents and health.
4. **State model.** Each entity gains `observed` state (up/down/degraded, last seen) separate from `declared` state. Drift between the two is itself a finding.
5. **Trust rules.** Per source and per op type: auto-apply, needs approval, or ignore.

## 8. What you may be overlooking

1. **Access terms are volatile, and the volatility belongs in the model.** Anthropic blocked Claude plans in third-party harnesses in April 2026 and restored them with conditions in May (reported: [TechCrunch](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/), [VentureBeat](https://venturebeat.com/technology/anthropic-reinstates-openclaw-and-third-party-agent-usage-on-claude-subscriptions-with-a-catch)). ChatGPT-in-OpenCode relies on community OAuth plugins that OpenAI doesn't endorse. Atlas marks such paths with a ⚠ and counts them as risk.
2. **For flat plans, quota is the scarce resource, not dollars.** OpenCode Go has dollar-denominated caps per 5 h, week and month ([opencode.ai/docs/go](https://opencode.ai/docs/go/)). Antigravity quota scales by tier. Economics needs a capacity dimension, not just a price.
3. **Product vs. installation.** "Claude Code" installed locally and headless on the VPS are two deployments with different reachability. The prototype handles this crudely (agents can override location). Deliverable 2 should add a `deployment` entity.
4. **Credentials and secrets.** Every access path is really a credential stored somewhere. Expiry, rotation and "which host holds which key" are architecture, not ops trivia.
5. **Permissions and approval gates.** It matters which agents can push to GitHub, spend money or message people. Right now nothing expresses trust boundaries.
6. **Observability.** Nothing reports health today. This is the first live source worth building, because it also turns the VPS single point of failure from a guess into a signal.
7. **Memory backup and ownership.** Hermes memory is private to Hermes. If the VPS dies, that memory goes with it.
8. **Evaluation.** Whether the Reviewer actually catches anything, or which model is better for the Coder, requires outcome data per workflow run. That feedback loop is the thing that eventually tells you what to build next.
9. **History.** You'll want "what did my architecture look like in March?" The event log (Deliverable 3) gives you this for free if it's designed in from the start.

## 9. Research notes behind the seed data

| Item | Role in Atlas | Confidence |
|---|---|---|
| Claude subscription | Access: grants Claude models, authorizes Claude Code | Documented |
| ChatGPT subscription | Access: grants GPT models, authorizes Codex ([OpenAI Help](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)) | Documented |
| Google AI subscription | Access: grants Gemini (and some Claude models inside Antigravity), authorizes Antigravity ([docs](https://antigravity.google/docs/plans/)) | Documented; Claude-in-Antigravity limits vary by tier |
| OpenCode Go | Access: open-weight models via one OpenAI-compatible endpoint; $10/mo list price ([opencode.ai/go](https://opencode.ai/go)) | Documented; your actual price not confirmed |
| Claude Code, Codex, OpenCode, Antigravity | Harnesses | Documented |
| OMP / Oh My Pi | Harness; fork of pi by Mario Zechner, 60+ providers incl. OAuth and plan sign-in ([repo](https://github.com/can1357/oh-my-pi)) | Documented; your sign-in not recorded |
| Hermes Agent | Harness/runtime with memory, skills, messaging gateway and MCP ([docs](https://hermes-agent.nousresearch.com/docs/)); the likely controller | Documented |
| VPS, local machines | Hosts | Stated by you; provider/specs unknown |
| Terminal, GitHub | Interface, tool | Inferred; confirm |

Nothing about *where* each harness runs, which tiers you pay for, or which agents you already have was stated. Those values are left unknown in **Current**. **Draft A** is my proposal and is clearly marked as one.
