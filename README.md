# Cognigy ConvAI — From Brief to Deployed

A structured methodology and AI-assisted delivery system for building Conversational AI projects on **Cognigy.AI** — from business case to a tested, handover-ready flow.

---

## What this is

Three phases. One system. Human approval at every handoff.

| Phase | What it produces | Who decides |
|---|---|---|
| **1 — Scoping & RoI** | Scope & Feasibility Report (intent matrix, ROI model, KPIs) | Business Analyst + client sign-off |
| **2 — Conversation Design** | Dialogue scripts, flowchart, variable dictionary, handoff spec | Conversation Architect + client workshop sign-off |
| **3 — Build & Delivery** | Live Cognigy flow (Draft) + test report + handover docs | AI agents (build/test/doc) + you (publish & channels) |

No agent in this system ever publishes a flow, activates a channel, or sends a message through a live integration. That step is always manual — by design.

---

## Repository structure

```
cognigy/
│
├── CLAUDE.md                        ← pipeline rules & agent instructions for Claude Code
├── .gitignore
│
├── agents/                          ← AI agent definitions (used in Claude Code sessions)
│   ├── spec-agent.md                ← discovery + flow specification (10 pillars)
│   ├── build-agent.md               ← builds the flow live in Cognigy via MCP
│   ├── test-agent.md                ← evaluates Playbook runs / manual Try-out transcripts
│   ├── audit-agent.md               ← on-demand quality audit (any flow, any time)
│   ├── doc-agent.md                 ← produces client & technical handover documents
│   └── dod-check-agent.md           ← automated DoD compliance check after each agent
│
├── DoD-docs/                        ← Definition of Done documents (per agent)
│   ├── cognigy-spec-agent-DoD.docx
│   ├── cognigy-build-agent-DoD.docx
│   ├── cognigy-test-agent-DoD.docx
│   ├── cognigy-audit-agent-DoD.docx
│   ├── cognigy-doc-agent-DoD.docx
│   └── cognigy-flow-builder-manual.docx
│
├── Ph1_Scoping&RoI/                 ← Phase 1 methodology & templates
│   ├── ConvAI_Ph1_Scoping_Framework_v1.4.docx
│   └── ConvAI Ph1 Scoping Templates.xlsx
│
├── Ph2_Conversation_Design/         ← Phase 2 methodology, templates & example scripts
│   ├── ConvAI Ph2 Conversation Design.docx
│   ├── ConvAI Ph2 Conversation Design Templates.xlsx
│   ├── flow.mmd                     ← Mermaid source (import into Lucidchart)
│   ├── flow.png                     ← rendered cancel_order flow diagram
│   ├── Chat Transcript (cancel_order).txt
│   ├── Chat Transcript (Business Rule Exception Path - Shipped status).txt
│   └── Chat Transcript (Conversation Repair Path_2-Strike Error Strategy).txt
│
├── Slides&Others/                   ← pitch decks & overview documents
│   ├── ConvAI_Pitch_Slide.pptx
│   ├── ConvAI_Pitch_Slide.html
│   └── ConvAI_Prozessuebersicht_DE.docx   ← German process overview
│
└── old/                             ← superseded agent drafts (not used)
```

Per-flow project folders (created when a real build starts) follow this pattern:

```
projects/{client}_{flow-slug}/
  spec.md
  pre-build-snapshot.json
  test-report.md
  audit-report.md
  handover-client.md
  handover-technical.md
  dod-check/
```

---

## How the build pipeline works (Phase 3)

The pipeline runs inside **Claude Code** using the agent definitions in `agents/`. There is no orchestrator agent — the session itself sequences the agents and stops at each human checkpoint.

```
spec-agent → dod-check → [you review & approve]
    ↓
build-agent → dod-check → [you fill placeholders, run Playbook or Try-out]
    ↓
test-agent → dod-check → [you review; loop back to build if gaps found]
    ↓
doc-agent → dod-check → [you review handover docs]
    ↓
[you publish manually in Cognigy Studio]
```

`audit-agent` is available on demand at any point — not gated by the pipeline.

---

## Key design principle

> *Not consulting that improvises — a structured system with AI-assisted delivery at every step.*

Every phase has a defined input, a defined output, and a human sign-off before the next phase begins. The AI does the build work; the human controls what goes live.

---

## Author

Denis Šuško · October 2026
