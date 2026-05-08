# coevolution-kernel

A position paper on the missing kernel layer between human judgment and self-evolving AI agents.

> Self-evolving agents drift silently.
> Long-term memory accumulates without provenance.
> AI translation flattens judgment into corporate language.
>
> These are not separate problems. They are aspects of the same architectural gap:
> **there is no kernel layer protecting human judgment integrity while AI capability evolves around it.**

---

## TL;DR

If you're building agent systems where:

- agents reflect on or mutate their own prompts
- memory persists across sessions
- humans collaborate with the agent over days or weeks
- decisions made early need to anchor decisions made later

— you'll eventually hit a class of failure modes that look unrelated but share a root cause:
**there is no provenance-aware substrate underneath your agent's evolution.**

This repo is a thesis about what that substrate should be, why it's needed now, and how to build it.

---

## What this addresses

### 1. Silent drift in self-evolving agents

Agents that mutate their own prompts (Reflexion, Voyager, etc.) accumulate silent shifts in objective. The agent thinks it's making progress; the user realizes hours later they've drifted from the original intent.

### 2. Memory bloat without provenance

MemGPT / Letta / mem0 give agents long-term memory but no way to distinguish **current judgment** from **stale opinion**. Old beliefs corrupt new reasoning. Audit ("why does the agent think this?") returns nothing useful.

### 3. Translation flattening

AI assistants translating raw founder thinking into "professional" outputs sand off the sharp edges. The judgment that was the value gets erased. The user reads back what was sent and finds it doesn't sound like them anymore.

### 4. Path quality degradation in long-horizon agents

Claude Code, Cursor agent mode, etc. — long task → agent commits to wrong direction → user has to manually halt and reset. Throughput-completion metrics don't capture this; the task "succeeds" but the path was wrong.

### 5. Multi-tenant judgment leak

Enterprise deployments where one user's memory drift contaminates another user's reasoning, or worse: shared memory pools where the system's collective state silently overrides individual judgment.

---

## What's the missing layer

A **provenance-enforced kernel** sits between:

- the agent (which can self-evolve, mutate, learn)
- the user's stable judgment (which must not be silently overwritten)

### Functions

| Function | What it does |
|---|---|
| **Source-tag every memory update** | Records who proposed it, what evidence, when. No silent writes. |
| **Path-quality detector** | Classifies execution-overrun, abstraction-drift, output-before-structure, premature closure. Surfaces drift before it hardens. |
| **Core / edge schema separation** | Stable judgment lives in core (rare, explicit updates). Capability extensions live in edge (fast, ephemeral, can be discarded). |
| **Calibration loop with provenance** | External feedback → tagged at source → proposed schema delta → human ratifies → integrated with audit trail. |
| **Faithful translator** | Serializes thought across audiences without flattening. Original judgment preserved as anchor; translation is a view, not a replacement. |

---

## How this differs from existing work

| | What it does | What's missing for this thesis |
|---|---|---|
| **Constitutional AI** (Anthropic) | Critique-revise loop with stable principle | Model-level (training time), not runtime user kernel; no per-user provenance |
| **Reflexion** (Shinn et al) | Self-correction via reflection | Single session; no provenance; silent prompt mutation |
| **MemGPT / Letta / mem0** | Long-term agent memory layers | Focus on retrieval, not provenance; no calibration concept |
| **Voyager** (Wang et al) | Skill library + auto skill generation | Optimizes for capability not judgment; no schema protection |
| **Anthropic Skills** | `SKILL.md` plug-in format for Claude | Skills independent; no cognition kernel meta-layer above them |
| **Sakana Transformer²** | Self-adaptive LLM via weight modulation | Weight-level; no provenance; not user-tunable |

The pattern: each component above is necessary but not sufficient. They all assume the user is either absent (autonomous agent settings) or static (training-time alignment). This thesis assumes the user is **present, evolving, and central** — and the kernel exists to protect that user's judgment integrity while AI capability evolves alongside.

---

## Why now (forcing functions)

| Forcing function | Timeline |
|---|---|
| **EU AI Act traceability requirements enforce** — agents must be auditable | **Q3 2026** |
| **Memory drift surfaces in mainstream usage** — users of Claude Memory / GPT Memory accumulate 1+ year of context, drift becomes visible | **Q3-Q4 2026** |
| **Reasoning models exhibit path-quality degradation** (o1 / Claude Extended Thinking / Gemini DeepThink) — longer thinking, more drift | already happening |
| **Creator tooling pushback** — Notion AI / Tana / Reflect users notice their "voice" being homogenized | already starting |

Three of these are 2026 events. The window for establishing reference vocabulary is small.

---

## Three merge paths

How an existing self-evolving agent system could adopt this kernel:

### Path 1: Outer guard

```
Inner: agent self-mutates freely (Hermes-style speed)
Outer: kernel evaluates each mutation against core schema
       — absorbs only if it doesn't break stable judgment
```

**Analog**: immune system — aggressive exploration + selective absorption.

### Path 2: Two-layer schema

```
Core schema: stable judgment, only updated via explicit calibration
Edge schema: evolving capability, can self-improve
New skill grows in edge → validated → promoted to core with provenance
```

**Analog**: stable API + experimental layer in software architecture.

### Path 3: Provenance-enforced self-evolution (most promising)

```
Self-evolution allowed.
Every mutation must emit provenance:
  - why change
  - source signal
  - hypothesis being tested
  - verification path
```

Speed unchanged. Transparency dramatically improved. Silent drift eliminated.

**Analog**: git commit messages required; non-empty pushes only.

---

## Reference implementation

A working prototype of the cognition-skill side of this kernel is at [`claude-cognition-skills`](https://github.com/hegu-1/claude-cognition-skills) — six Claude Code skills that operationalize the principles:

- `nepm-kernel` — read situation before answering (routing layer)
- `trap-detector` — drift detection (path quality monitor)
- `helpful-now-deriver` — minimal correct next move
- `decision-snapshot-builder` — state checkpoint creator
- `founder-thought-translator` — faithful serialization across audiences
- `nepm-orchestrator` — execution planner

The current repo (kernel) extends those principles to the broader agent stack.

---

## Lineage

This thesis sits inside an emerging conversation:

- **Andrej Karpathy** — "Software 3.0 / LLM as kernel" — same kernel framing, no specific product
- **Geoffrey Litt + Ink and Switch** — malleable software, local-first, user owns memory and controls mutation
- **Linus Lee, Maggie Appleton** — Tools for Thought + AI partner — long-horizon co-thinking
- **Anthropic Constitutional AI** — embedded principle anchor at model level (this thesis is the user-level analog)

---

## What this is and is not

This is **not** another agent framework.
This is **not** a memory system to plug in.
This is **not** a productivity assistant.

This is a **position paper + early architecture spec** for a layer that should exist between agents and users in long-horizon AI collaboration. It's deliberately published early, before full implementation, to anchor priority of vocabulary and invite collaboration on the substrate before it gets reframed by larger companies whose incentive structure points at autonomy rather than co-evolution.

---

## Status

Early. Reference architecture, not production framework.
Primary artifact: this README.
Reference implementation: `claude-cognition-skills` (linked above).

If you're working on similar problems — agent provenance, judgment integrity, long-horizon human-AI co-evolution — open an issue or reach out.

---

## One-line summary

Skills for what AI gets wrong when humans and agents are supposed to grow together.

License: MIT.
