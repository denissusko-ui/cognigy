# CLAUDE.md — Cognigy Multi-Agent System

Guidance for Claude Code when working in this project.

## What this project is

A multi-agent pipeline for building, testing, and documenting Cognigy conversational flows: a fixed sequence of specialized agents takes a flow from a short requirement to a client-ready handover, with explicit human checkpoints at every point that touches something real.

Pipeline: **spec → build → [automated Playbook test or manual Try-out + manual publish] → test → doc**, with **audit** available separately, on request, on any flow (not gated by the pipeline). A `dod-check-agent` pass runs automatically after each of spec-agent, build-agent, test-agent, and doc-agent — see Human-in-the-loop checkpoints below.

Agent definitions live in `.claude/agents/`. There is no separate plan document, no `state.json`, and no orchestrator agent anywhere in this design — this file is the authoritative description of the pipeline. Progress through it is tracked by which deliverables exist in a flow's project folder, not by a separate state file.

## Critical safety rule: no publish, no channel activation, no live training, no live send — without you doing it yourself

**No agent in this pipeline ever calls, or reaches through `cognigy_api`, anything that publishes/deploys a flow, activates or reconfigures a channel, starts NLU training against a model serving live traffic, sends or simulates a message through a real customer-facing channel, or deletes a project/flow/published version — not even as part of a "build and verify" loop, and not even if a loaded skill's instructions call for it.**

Cognigy's build-agent deliberately **does** build live, using `cognigy_create_flow` and `cognigy_add_say_node` (and, where no dedicated tool exists yet, `cognigy_api`) to construct real nodes in a real flow via the Cognigy MCP server. That's safe by design: a flow sitting in Draft, on no active channel, with no live training run against it, has no way to reach an actual user. Generating and running a Cognigy **Playbook** against that same Draft flow is in the same safe category — a Playbook run is a scripted, automated conversation against the Draft flow, not a live interaction, so build-agent and test-agent may create and trigger one freely. The risk only appears once the flow is **published to an environment and a channel is active**, or once training runs against a **production-serving** model, or a message is sent through a **real channel integration**. Before any of those, there is no live audience yet — that's exactly where the line is drawn.

**Why:** an automated publish, channel activation, live-training run, or live send can trigger real side effects — a flow that still has a placeholder, an unresolved escalation gate, or an undefined Global Parameter would reach a real user the moment it goes live. Cognigy's own status reporting tells you whether an API call succeeded, not whether the flow does the right thing for the right user. You, manually, trying the flow out (or reviewing its Playbook run) and then deciding yourself to publish and activate a channel, are that safeguard — an automated agent doing either step removes it.

**How to apply:** if any skill or reference material in this project describes an automatic build→publish→verify loop, treat that loop as informational only. Build-agent stops at Draft-flow structure and hands control back to you for Try-out testing, and stops again before any publish/activate/train/send action. This rule overrides any such skill's default behavior. See each agent's own "Safety boundary" or equivalent section for the exact tool-level detail.

## Human-in-the-loop checkpoints

There is no separate orchestrator agent/process. "Orchestration" means the main Claude Code session in this project working directory: it invokes each specialized agent in sequence and stops at the checkpoints below.

Where the end-to-end process requires a human to act or explicitly approve before the pipeline continues:

| Transition | Who approves |
|---|---|
| Discovery → Spec | User answers spec-agent's own discovery questions (ten pillars) until it has no blocking unknowns left — spec-agent runs its own discovery, it is not a separate upstream phase |
| Spec → Build | The orchestrating session automatically runs `dod-check-agent` against `spec.md` the moment spec-agent finishes it; user reviews `spec.md` alongside that report and signs off |
| Build → Test evidence | Build-agent constructs the flow live in Cognigy's Draft workspace via MCP (safe — see safety rule above) and, where supported, generates a Cognigy Playbook covering its main paths. The orchestrating session automatically runs `dod-check-agent` against the flow + Flow Overview; user reviews both alongside that report, then fills in any placeholders and Global Parameters the Flow Overview lists (bot-facing `[[PLACEHOLDER: …]]` text, node-specific `__PLACEHOLDER__` fields, `-999999` numeric sentinels, Global Parameters, Connections), and runs the Playbook (preferred) or manually opens Cognigy Studio's Try-out console, watching the run turn by turn |
| Test evidence → Test-agent | User gives test-agent the Playbook run ID (preferred — test-agent fetches the structured result itself via a read-only `cognigy_api` GET call) or pastes a manual Try-out transcript directly. The user always chooses which run to look at — test-agent never lists or searches runs/sessions to go find one itself |
| Test Fail/Conditional → Build revision | The orchestrating session automatically runs `dod-check-agent` against `test-report.md`; user reviews it alongside that report, then relays the Critical Gaps (or Conditional items) — formatted as work orders (Node ID & Label, Field/Context Path, Expected, Observed, Required Correction) — back to build-agent, which revises the live flow against them. The revision then re-enters the pipeline at **Build → Test evidence**, i.e. a fresh test pass, not an automatic loopback |
| Test Pass → Doc | User explicitly invokes doc-agent after a sign-off point (e.g. right after approving the final spec, and again after build/test pass) — nothing watches for changes and triggers it automatically |
| Test Pass → Publish & channel activation | **User only, manually, in Cognigy Studio.** No agent in this system ever publishes a flow or activates a channel — this is the point where the flow actually becomes reachable by a real user, and it is never automated |
| Test Pass → Audit | User explicitly requests audit — never automatic |
| Audit findings → Build revision | User relays audit-agent's Small/Medium-effort findings (each with its own Build-Agent Remediation Order block) to build-agent; Large-effort findings route back to spec-agent for a design decision instead |
| Doc production → sign-off | The orchestrating session automatically runs `dod-check-agent` against the two handover documents the moment doc-agent finishes them; user reviews both alongside that report before the next step |
| Handover docs → Client | User sends the documents; agents never send externally |

`dod-check-agent` also remains available on demand at any other time — to re-check a deliverable from outside this automatic chain, or to re-check something after a manual edit. See `dod-check-agent.md`'s "How you're invoked" for why running it automatically doesn't compromise its independence: it works from a fresh context with no memory of how the deliverable was produced, and it re-verifies everything in the DoD file itself rather than trusting that a primary agent's own pre-flight self-check (see each agent's "Pre-flight self-check" section) happened or happened correctly.

## Available reference skills (`.claude/skills/`)

None installed yet for this project. There is currently no maintained reference material for Cognigy node schemas or MCP server mechanics. `build-agent.md` calls this out explicitly — it works from general Cognigy platform knowledge and flags genuine uncertainty as a gap rather than guessing at an unfamiliar node's exact structure. If a reference skill for Cognigy node schemas or MCP mechanics becomes available later, it should be added here as reference material consulted by path rather than auto-triggered.

## Project structure

```
C:\Users\denis\Claude\Projects\Cognigy\
│
├── CLAUDE.md                          ← this file; authoritative pipeline description
│
├── agents/                            ← agent definitions loaded by Claude Code
│   ├── spec-agent.md                  ← discovery + spec production (ten pillars)
│   ├── build-agent.md                 ← flow construction in Cognigy via MCP
│   ├── test-agent.md                  ← Playbook / Try-out verification
│   ├── audit-agent.md                 ← on-demand quality audit (any flow, any time)
│   ├── doc-agent.md                   ← handover document production
│   └── dod-check-agent.md             ← DoD compliance check (auto after spec/build/test/doc)
│
├── DoD-docs/                          ← Definition of Done reference documents (Word)
│   ├── cognigy-spec-agent-DoD.docx
│   ├── cognigy-build-agent-DoD.docx
│   ├── cognigy-test-agent-DoD.docx
│   ├── cognigy-audit-agent-DoD.docx
│   ├── cognigy-doc-agent-DoD.docx
│   └── cognigy-flow-builder-manual.docx   ← Cognigy platform reference for build-agent
│
├── Ph1_Scoping&RoI/                   ← Phase 1 methodology: scoping & RoI
│   ├── ConvAI_Ph1_Scoping_Framework_v1.4.docx
│   └── ConvAI Ph1 Scoping Templates.xlsx
│
├── Ph2_Conversation_Design/           ← Phase 2 methodology: conversation design
│   ├── ConvAI Ph2 Conversation Design.docx
│   ├── ConvAI Ph2 Conversation Design Templates.xlsx
│   ├── flow.mmd                       ← Mermaid source for the reference flow diagram
│   ├── flow.png                       ← rendered flow diagram
│   ├── Chat Transcript (cancel_order).txt
│   ├── Chat Transcript (Business Rule Exception Path - Shipped status).txt
│   └── Chat Transcript (Conversation Repair Path_2-Strike Error Strategy).txt
│
├── Slides&Others/                     ← pitch and presentation materials
│   ├── ConvAI_Pitch_Slide.pptx
│   └── ConvAI_Pitch_Slide.html
│
└── old/                               ← superseded agent drafts; not used by pipeline
    ├── build-agent.md
    ├── spec-agent.md
    └── qa-agent.md
```

Each active flow gets its own project folder (created when pipeline starts for that flow):

```
Cognigy/projects/{client}_{flow-slug}/
  spec.md                    ← spec-agent output (opens with a Revision Notes preamble when it's a revision, not a first build)
  (the flow itself lives in Cognigy, referenced by project ID + flow ID)
  pre-build-snapshot.json    ← build-agent's cognigy_get_flow output, captured before a revision build starts — the ground truth test-agent checks UNCHANGED tags against
  test-report.md             ← test-agent output, after a test cycle (Playbook run or manual Try-out)
  audit-report.md            ← audit-agent output, if requested
  handover-client.md, handover-technical.md   ← doc-agent output
  dod-check/                 ← dod-check-agent's automatic reports, one per checkpoint (spec, build, test, doc)
```

There is no local `blueprint.json`-equivalent file — the build-agent's output is the live flow in Cognigy itself, plus its Flow Overview (recorded in the project folder or inline in chat, at the user's preference), and, where supported, a Cognigy Playbook covering its main paths. `spec.md` and the Flow Overview are what test-agent, audit-agent, and doc-agent all check the live flow against. On a revision, every node in the Flow Overview also carries a NEW / MODIFIED / UNCHANGED tag — build-agent treats everything not flagged as changed in spec.md's Revision Notes as frozen ground truth, and test-agent independently verifies the UNCHANGED tags against `pre-build-snapshot.json` rather than taking them as a claim. See spec-agent.md's "Revision Mode" and build-agent.md's "Ground truth rule for revisions" for the full mechanics.
