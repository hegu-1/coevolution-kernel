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
| **Anthropic Skills** (`anthropics/skills` + [agentskills.io spec](https://agentskills.io/specification)) | `SKILL.md` plug-in format for Claude; horizontal task-completion patterns (PDF / docx / frontend-design / etc.) | Spec defines *what a skill is*, not *when to invoke which / how to detect drift / how to preserve judgment*. The cognitive-meta layer above them is unaddressed |
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
| **Agent Skills standardization** — Anthropic publishes spec + reference repo; prior art window for the *cognitive-meta layer above* horizontal skills narrows fast | **2026-Q2 (active now)** |

Four of these are 2026 events, one is already active. The window for establishing reference vocabulary is small.

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

## The delegation boundary

The failure modes above share an assumption: the user is *present*. But a long-horizon personal kernel also has to survive the user being *absent* — and the logical endpoint of human-AI co-evolution is that some decisions must eventually route *through* the agent when the human can't make them in time.

This is the most dangerous moment for judgment integrity. If the agent gradually absorbs authority — deciding a little more on its own each time the user leans on it — the boundary moves silently. The no-silent-drift principle has to extend to authority itself: **the day the agent decides for you must be authored, not drifted.**

The **capability-token-protocol (CTP)** is the mechanism:

- The secret that authorizes a boundary crossing (a passphrase, or a rotating challenge referent) **never enters any channel the agent can read** — not its memory, not its logs, not its repos, not the conversation. Asking the agent to "just store the passphrase" destroys it.
- It lives behind an **air gap**, in the user's sovereign vault, next to a verifier the agent's reasoning layer cannot see.
- When the user issues a boundary-crossing intent, the verifier checks the secret and mints a **scoped capability token** — `{ scope, action, expiry, nonce, log_ref }` — carrying no secret, only *what is authorized, how wide, how long, how to audit it*.
- The agent receives only the token. Inside scope it acts; without a valid token, or out of scope, it **refuses and surfaces** rather than degrading into a plausible guess. Every exercise is logged against `log_ref`.
- Two directions matter. An **authorization** token (user → agent: "you may cross here"), and a **challenge** (user → agent: a liveness/continuity proof that a substituted or drifted model would fail) — the user's tool to verify the *agent*, not only the agent verifying the user.

The recursive guarantee: because the secret can only live where the agent cannot read, the agent is *structurally* unable to forge or leak it. "The agent waits until you hand it the key" stops being a promise and becomes a property of the architecture — verifiable by the fact that when the vault is physically detached, the air gap is real, not asserted.

**Analog**: object-capability security applied to human→agent delegation — authority is an unforgeable, attenuable, revocable token, never an ambient permission.

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

### Relation to Anthropic's Agent Skills spec

Each `SKILL.md` in the reference implementation validates 100% against the official [Agent Skills spec](https://agentskills.io/specification) — same `name` / `description` / `metadata` / `compatibility` / `license` schema as `anthropics/skills`.

The differentiation is **layer**, not spec:
- `anthropics/skills` operates on **task-completion** — *how to do X* (process PDFs, generate UI, send Slack messages).
- This kernel + `claude-cognition-skills` operate on **cognitive-meta** — *how to read what kind of movement is happening* before deciding what task to invoke.

The two stack rather than compete. Task skills are the *what*; the cognition kernel is the *how-do-I-know-I'm-on-the-right-path*.

---

## Applied at scale

The same kernel — memory as the container of continuity, provenance on every write, judgment preserved against silent drift — is **scale-invariant**. Scaled up to a company-wide AI OS, it lands as the two layers that actually decide who wins: **memory** (accumulated, provenance-tracked cognition) and **governance** (auditable autonomy). Everything else — compute, models, ingestion, action, orchestration — commoditizes.

→ [`enterprise-ai-os-architecture`](https://github.com/hegu-1/enterprise-ai-os-architecture) — the enterprise-scale reference, derived bottom-up from an n=1 system. One architecture, three scales (personal / startup / enterprise); the kernel is the invariant.

---

## Vocabulary anchors

The following terms are deliberately surfaced here to establish prior art for the substrate this thesis describes. As the cognitive-meta layer matures through 2026, these names should remain referenceable.

### Layer & substrate
- **personal-ai-os** — the user-side OS stack for human-AI co-evolution
- **personal-ai-os-kernel** — the provenance-enforced layer at its core
- **provenance-enforced-agent** — an agent that emits "why this changed" with every mutation
- **judgment-aware-agent** — an agent that protects user judgment integrity during translation
- **drift-aware-agent** — an agent that detects and surfaces its own drift over time

### Mechanism & protocol
- **calibration-loop-protocol** — external feedback → tagged at source → schema delta → human ratifies → integrated with audit trail
- **schema-coexistence-spec** — how core (stable judgment) and edge (evolving capability) schemas coexist
- **continuity** — preserving cognitive thread across sessions, tools, and agents
- **capability-token-protocol** — how a user authorizes the agent to cross a boundary (or eventually decide on their behalf) without the agent ever holding the secret: authorization is minted as a scoped, expiring, logged capability token on the far side of an air gap, so delegation is *authored, not drifted*

### Counterpoint to AGI

A specific naming gap this thesis flags:

- **AYI** — **A**rtificial **Y**ou-**I**ntelligence — the per-individual analog of AGI. Where AGI optimizes for universal capability ("works for anyone"), AYI optimizes for fidelity to one specific person's cognitive thread ("works for you, specifically, over time"). The G→Y substitution is intentional: not a different scale of intelligence, a different *target* of intelligence.
- **个智** (gè zhì) — Chinese: the personal counterpart to **通智** (tōng zhì = AGI). 通智 is the same intelligence shared across everyone; 个智 is intelligence that converges to one person's specific continuity.

> AGI is for everyone. AYI is for you.
> 通智服务所有人，个智只围着你一个人转。

This pair is not a competing roadmap to AGI — it's a complementary objective that AGI labs structurally cannot optimize for (their incentive is generalization). The personal-ai-os substrate is what makes AYI tractable: continuity, provenance, and judgment integrity are prerequisites for an AI to be *yours* rather than *general*.

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
