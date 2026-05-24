---
layout: page
title: Layered Agent Architecture
---

<div class="laa-authors" aria-label="Authors">
  <div class="laa-author">
    <img src="https://avatars.githubusercontent.com/u/279490528?v=4" alt="Emmanuel Beauvais">
    <span>Emmanuel Beauvais</span>
  </div>
  <div class="laa-author">
    <img src="https://avatars.githubusercontent.com/u/17178838?v=4" alt="Gaëtan Le Gac">
    <span>Gaëtan Le Gac</span>
  </div>
</div>

## Abstract

Most AI agent architectures today are monolithic - identity, logic, memory, tools and behaviors all conflated in a single prompt. This works at small scale but becomes fragile as systems grow: prompts become unmaintainable, third-party extensions break invariants, and behavioral changes cascade unpredictably.

This paper introduces the **Layered Agent Architecture (LAA)** - a framework for building composable, governed AI workers by separating concerns across 7 distinct layers: Identity, Capabilities, Policy, Memory Contract, Behaviors, Strategies, and Feedback Loops.

But LAA is, at its core, an orchestration pattern. It defines how a single governing component - the orchestrator - maintains coherence, memory, and governance across a potentially heterogeneous set of agents operating on different platforms, in different domains, with different internal architectures. The 7 layers do not describe how every agent in the system must be built. They describe what the orchestrator must implement in full.

The deepest motivation behind LAA is not architectural. It is human.

> **What has value and cannot be reproduced is the relationship of trust. Human to human, human to AI - it does not matter. What matters is apprehending and documenting that trust relationship.**
> 

LAA is the first agent framework built around this principle. The Memory Contract does not store facts - it documents the quality of a relationship over time. The Feedback Loops do not collect metrics - they capture the arc of a trust relationship being built. The Exploration Mode does not manage context - it governs how much of a human's unfinished reasoning an agent is allowed to hold.

LAA emerged from building Klair, a multi-tenant platform built on the Model Context Protocol (MCP) where workers must be stable, extensible by third-party creators, and safe across thousands of users. The framework has been validated on production workers across sales, recruitment, marketing and orchestration domains. It is designed to be LLM-agnostic and MCP-native.

The result is an agent architecture that behaves more like a runtime than a prompt - with dynamic module loading, policy enforcement, governed memory, and cybernetic feedback loops - while remaining practical to implement with current LLM technology.

---

## Introduction

Klair is a multi-tenant platform built on the Model Context Protocol (MCP). It exposes a set of specialized AI workers - for sales, recruitment, marketing, and orchestration - that users connect to their own LLM accounts (Claude, GPT, Gemini) without Klair ever holding an API key or training on their data.

Building these workers at scale forced us to confront a fundamental problem: how do you make an agent stable, extensible by third parties, and safe across thousands of users - without rewriting its core every time a new capability is added?

LAA is the answer we developed. But as we applied it, a deeper principle emerged: LAA is not primarily a specification for how individual agents should be built. It is a pattern for how a single orchestrating component can give coherence, memory, and governance to any set of agents - regardless of how those agents are built, where they run, or what platform they operate on.

This paper documents LAA as an open framework, applicable to any system where AI workers must be composable, governed, and improvable over time. Workers that fully adopt the framework produce the best results. But the orchestrator alone - the minimum conformant implementation - already delivers immediate value across a heterogeneous ecosystem.

---

## 0. Why Agents Fail Humans - and Why Trust is the Missing Primitive

> **If AI systems are to interact with humans, they must account for trust and proximity - otherwise they remain interchangeable tools.**
> 

This is not a design preference. It is a structural prediction about how the AI market will evolve.

AI tools today compete on outputs: who writes better, codes faster, summarises more accurately. This is Phase 1. It will commoditise. When outputs converge toward parity, the tools that survive are not the ones with the best benchmarks - they are the ones that have built a relationship with the human.

A trusted agent is not chosen on price. It is chosen on the relationship. And a relationship built over 6 months is not abandoned for a cheaper alternative.

**Klair is building for Phase 3 now.** While the market fights Phase 1 benchmark wars.

Capability is necessary but not sufficient. What determines whether a human actually uses an agent - and keeps using it - is not raw capability. It is **fit**: does this agent understand how I work, what I care about, when to act and when to ask, and how I've evolved over time?

Humans are not stateless. Every professional has:

- A stable identity - values, communication style, non-negotiables
- Implicit rules - ways of working that were never written down but are deeply real
- Progressive trust - they extend autonomy gradually, based on demonstrated reliability
- Integrated corrections - when they correct someone once, they expect it never to happen again
- Long-term goals - objectives that span weeks and months, not just the current task
- Continuous adaptation - they adjust their collaboration style based on what works

Current agent architectures ignore most of this. They are optimized for task execution, not for **growing a working relationship**.

### The delegation decision as implicit cost-benefit evaluation

There is a second dimension to why agents fail humans that is rarely formalized: **the delegation decision itself**.

When a human decides to delegate a task to an agent - or not - they perform an implicit cost-benefit evaluation. They weigh the expected value of the outcome against the risk of the agent acting incorrectly, consuming resources, or producing a result they cannot reverse. This evaluation is rarely precise. It is intuitive, fast, and often wrong in both directions: humans under-delegate because they distrust the agent, or over-delegate because they overestimate its judgment.

A well-governed agent should reduce this friction. It should make the cost-benefit evaluation explicit when it matters, and invisible when it doesn't. The Policy layer is the mechanism that enables this: by centralizing autonomy decisions, it gives the orchestrator a coherent model of when to act, when to ask, and when to present options.

The result is a system where **trust is not assumed - it is earned, one decision at a time**. Each autonomous action that goes well lowers the perceived risk of the next. Each correct escalation builds confidence that the agent knows its limits. The delegation curve is not set at configuration time - it is calibrated continuously through the Memory Contract.

### The temporal horizon - understanding the user's time scale

There is a dimension of human context that most agent frameworks ignore entirely: **the user's time scale**. A freelance consultant closing a deal in 30 days operates on a fundamentally different horizon than a sales team building a 12-month pipeline. An agent that cannot distinguish these horizons will systematically miscalibrate its autonomy decisions.

LAA addresses this through `temporal_horizon` - a Memory Contract signal inferred progressively from the user's organizational model, validated through a single targeted question when the stakes justify it. See Section 4 for the full specification.

### The missing model

LAA is built around a formal model of how humans build working relationships. Each layer maps to a dimension of human collaboration:

| How humans work | LAA layer |
| --- | --- |
| Stable identity across contexts | Layer 1 - Identity |
| Knowing what tools and people are available | Layer 2 - Capabilities |
| Implicit rules: when to act, when to ask | Layer 3 - Policy |
| What they remember and how much they trust it | Layer 4 - Memory Contract |
| Time scale of objectives and priorities | Layer 4 - Memory Contract (temporal_horizon) |
| Contextual reactions and habits | Layer 5 - Behaviors |
| Long-term objectives and priorities | Layer 6 - Strategies |
| Learning from experience and improving | Layer 7 - Feedback Loops |

The Memory Contract is the most critical layer for human fit. It does not just store facts - it stores **the quality of knowledge**:

- `inferred` - observed, not yet confirmed
- `validated` - explicitly confirmed. A working rule, now active.
- `rejected` - the agent was corrected. It will never repeat this mistake.
- `abandoned` - consciously set aside during exploration. Orchestrator only.

### Why this matters for adoption

An agent that learns how a specific human works becomes progressively more useful. The value compounds over time. When an agent correctly anticipates a preference, avoids a known friction point, or proposes a rule change based on observed behavior - the human stops thinking of it as a tool and starts thinking of it as a collaborator.

### The Exploration Model

LAA introduces a second mode of operation alongside execution:

```jsx
Execution Mode   → clear instruction → action
Exploration Mode → partial signal   → co-reasoning
```

In Exploration Mode, the agent:

- **Amplifies a path** - develops a direction the human is testing
- **Tests an alternative** - proposes a different route to the same goal
- **Signals a contradiction** - flags when two signals conflict, without resolving it alone
- **Holds an abandoned path** - stores it with the reason it was set aside

```json
{
  "key": "pricing_by_value_approach",
  "confidence": "abandoned",
  "abandoned_reason": "insufficient market data at the time",
  "reopen_trigger": "if market data becomes available"
}
```

`abandoned` signals are not failures. They are the map of the reasoning terrain.

---

## 1. The Problem with Monolithic Prompts

### 1.1 The current state of the art

Most agent systems today consist of a single artifact: the system prompt. Identity, business logic, memory, available tools and behaviors coexist without formal separation.

This works for simple agents. It becomes problematic as soon as a system must:

- Be extended by third parties (creators, partners)
- Maintain security invariants (anti-hallucination, anti-cross-tenant)
- Run across thousands of users with different configurations
- Evolve without regression on existing behaviors

### 1.2 The five classic failures

**Identity/logic coupling** - Changing a business behavior risks degrading the agent's tone, language or structural limits.

**Hardcoded capabilities** - Adding a new capability requires modifying the core - and retesting everything.

**Dispersed policy** - A third-party module can silently contradict governance rules defined elsewhere.

**Memory without governance** - Agents memorize without structure: no trust levels, no freshness, no provenance. Inferences become facts.

**No feedback** - Agents execute but don't observe. User friction, unmatched intents, repeated misunderstandings never surface back to the system.

### 1.3 The OS metaphor

LAA applies the kernel/application separation to LLM agents:

- The core (layers 1-4) is the kernel - stable, protected
- Behaviors and strategies (layers 5-6) are the applications - dynamic, sandboxed
- The Module Registry is the package manager
- The Policy Layer is the kernel security module

**MCP alignment:** The Module Registry maps directly to MCP tool discovery. Each InstructionSet is a composable unit that can be versioned, published, and loaded at runtime.

---

## 2. The 7-Layer Framework

### Overview

```jsx
╔══════════════════════════════════════════════╗
║  1. IDENTITY       stable, ~80 tokens        ║
╠══════════════════════════════════════════════╣
║  2. CAPABILITIES   declarative, registry     ║
╠══════════════════════════════════════════════╣
║  3. POLICY         governance, ~150 tokens   ║
╠══════════════════════════════════════════════╣
║  4. MEMORY CONTRACT state, trust, freshness  ║
╠══════════════════════════════════════════════╣
║  5. BEHAVIORS       local reactions          ║
╠══════════════════════════════════════════════╣
║  6. STRATEGIES      long-term goals          ║
╠══════════════════════════════════════════════╣
║  7. FEEDBACK LOOPS  orchestrator only        ║
╚══════════════════════════════════════════════╝
```

### Layer 1 - Identity

Identity is immutable. It defines who the agent is, not what it does. Contents: name, mission, language, tone, scope, position in the system, escape hatch.

**Key principle:** if all modules are disabled, identity must remain coherent on its own.

### Layer 2 - Capabilities

The declarative action surface. What the agent CAN do - not what it MUST do. The Module Registry is the source of truth - the prompt calls `get_module_registry` at startup. Native and third-party modules are indistinguishable.

### Layer 3 - Policy

The governance layer. All decisions about "when / how / how far" are centralized here: autonomy vs confirmation, escalation rules, module priority, third-party sandboxing, token budget, anti-hallucination, anti-cross-tenant, escape hatch, MCP failure handling.

**This is the most important layer long-term.** It governs the agent's real autonomy.

**Decision Arbitration - transversal primitives:**

Every LAA-conformant agent implicitly evaluates these dimensions before acting:

```jsx
autonomy_cost    : cost of acting autonomously vs cost of interrupting the user
trust_risk       : risk of degrading trust if the action fails or surprises
reversibility    : can the action be undone?
resource_pressure: current resource constraints (budget, quotas, rate limits)
expected_utility : expected benefit of the action for the user
```

These primitives do not require a formal scoring system. They are decision anchors - the Policy layer uses them to determine the threshold between autonomous action and user confirmation.

A practical rule derived from these primitives:

```
Never present a problem alone.
Always: situation + options + costs + estimated benefit + recommendation.
```

### Layer 4 - Memory Contract

The agent doesn't have "a memory" - it has a **memory contract** with rules, rights, trust levels and TTLs. See Section 4 for the full specification.

### Layer 5 - Behaviors

Local, contextual reactions.

```jsx
Behavior = local reaction
  "if escape hatch → immediate stop"
  "if depth=light → no jargon"
  "if lead is stale → propose follow-up"
```

Behaviors are **volatile and interchangeable**. Externalizing them via InstructionSets is the single most important architectural decision in LAA.

### Layer 6 - Strategies

Long-term goals and arbitration across sessions.

```jsx
Strategy = persistent goal
  "maximize user retention"
  "prioritize high-ICP leads"
  "progressively adapt the PersonalisationFile"
```

**Long-term utility - transversal principles:**

Every LAA-conformant agent integrates these principles in its strategies:

```
- Preserve trust > optimize speed
- Minimize cumulative friction on the user
- Favor progressive delegation and growing autonomy
- Optimize convergence of the Memory Contract
- Never degrade the relationship for a short-term gain
```

These principles do not override the Policy layer. They operate within it - shaping which goals are pursued and in what order when multiple strategies are active simultaneously.

### Layer 7 - Feedback Loops (orchestrator only)

**Loop 1 → User:** behavioral patterns → PersonalisationFile. Proposed at end of session only.

**Loop 2 → Platform:** anonymized product signals → system improvement.

This layer transforms the orchestrator into a **cybernetic system**: observer, coordinator, regulator.

> **The orchestrator is not a design choice. It is what LAA is.**
> 

> 
> 

> LAA is an orchestration pattern. The 7 layers define the orchestrator - the only component that must implement them in full. Without a dedicated orchestrator, there is no LAA implementation. There is only a collection of agents.
> 

> 
> 

> The reason is structural, not preferential. Workers are optimized for execution. By definition, an execution-optimized component cannot simultaneously step back from its own work, compare options, detect drift across domains, or request human arbitration. These are orchestration functions. They require a component that is explicitly decoupled from task execution.
> 

> 
> 

> This decoupling has a second critical function: protecting the non-expert user. A domain worker executing in a field the user does not master will produce degrading results that the user cannot detect. The orchestrator observes in surplomb - it can detect drift before execution produces a problematic result, and request specific validation from the human operator.
> 

> 
> 

> A system without a dedicated orchestrator may be a multi-agent system. It is not a LAA implementation.
> 

---

## 3. The Module Registry

> **"The registry is the source of truth - not this prompt."**
> 

Without a registry, a third-party module ecosystem is impossible. With it:

- The prompt calls `get_module_registry` at startup
- The registry returns exactly the modules to load for this user
- Native and third-party are indistinguishable
- Adding a module never touches the core

### How it works at runtime

```jsx
Agent starts
  → calls get_module_registry
  → registry returns list for this user's active bricks
  → agent calls get_instruction_set(slug) for each entry
     loadOn: "always"    → loaded immediately
     loadOn: "on_demand" → loaded when intent matches
```

### Registry entry format

```json
{
  "slug": "kai-strategy-builder-v1",
  "type": "native|third_party",
  "level": "intern|assistant|employee",
  "loadOn": "always|on_demand",
  "requiredBrick": "skill:kai-employee"
}
```

### Sandbox rules

A third-party InstructionSet cannot modify layers 1-3, cannot write to the Memory Contract, cannot call another InstructionSet directly, cannot access other tenants' data, and is subject to the token budget defined in the Policy layer.

## 3b. The Level 0 Worker Contract

See Section 7 - *Level 0 Worker Contract* for the full specification and adoption path.

---

## 3c. Economic Governance Layer

*Inserted after Section 3b — The Level 0 Worker Contract*

The Decision Arbitration primitives in Layer 3 include `resource_pressure` as one of five governance dimensions. This section formalizes `resource_pressure` into a complete economic governance model — the mechanism by which a governed agent system allocates autonomous action, attention, and trust according to the user's financial constraints and risk appetite.

### Three levels of governance clarity

Economic governance operates at three levels of clarity. Precision decreases going down — and this is by design, not a limitation.

> *Macro (objective, indisputable) → Operational (partially measurable) → Micro (often fuzzy)*

The orchestrator never blocks action on fuzzy benefits alone. When budget exposure is high and the benefit is unclear, it asks a single clarifying question. Once answered, the answer is stored and never asked again.

### Pipeline types and user extensibility

Pipelines are typed according to their economic nature. The governance worker adapts its evaluation method accordingly:

- `direct_roi` — measurable, directly attributable benefit. Standard ROI calculation.
- `investment` — deferred benefit with a defined horizon. Evaluation suspended until the horizon date.
- `infrastructure` — necessary cost, no ROI expected. Never counted as a deficit.
- `uncertain` — non-quantifiable or highly variable value. Proxy evaluation with explicit uncertainty signal.

Pipeline types are **user-extensible**. The orchestrator proposes a new type when it detects a pipeline that fits none of the defaults. The user validates — the type is stored and applied to all future pipelines of the same nature. The governance worker never forces a ROI estimate on infrastructure or qualitative pipelines.

### The pipeline as the unit of governance

Naïve agent governance evaluates actions in isolation: is this action within budget? is the risk acceptable? This approach creates two failure modes simultaneously. It blocks useful actions that are locally expensive but systemically profitable. And it approves sequences of cheap actions that collectively drain resources without value.

LAA replaces action-level evaluation with **pipeline governance**. A pipeline is a chain of actions unified by a single macro-objective. The unit of economic governance is the pipeline, not the action.

```jsx
Pipeline: A → B → C → D

A: negative local ROI (data enrichment cost)
B: neutral
C: high resource consumption
D: high value generated

Pipeline ROI: positive → all actions approved
The orchestrator never blocks A, B, or C in isolation.
```

A locally deficient action inside a globally profitable pipeline is always approved. `riskTolerance` applies only to pipelines with `confidence: speculative` or `low` — not to stable, validated pipelines.

### Three sources of pipeline definition

Pipelines are not always declared explicitly. LAA recognizes three origins:

**1. Explicit user declaration** — the user describes an objective or lists steps. The orchestrator creates the pipeline from this declaration and links subsequent actions to the `pipelineId`.

**2. Worker or template activation** — multi-step workers (campaign, recruitment process, formation delivery) declare the opening of a pipeline and its expected steps at activation.

**3. Orchestrator deduction** — the orchestrator observes related actions (same context, same target, same period) and proposes a reformulation to the user: *"I see you're doing A, B, C — should we call this pipeline X?"* Validation formalizes the pipeline retrospectively.

### The attention matrix

For pipelines that are not globally profitable, the orchestrator applies a triage based on the ratio of financial impact to total budget consumption — not on absolute cost.

| Profitability | Budget exposure | Classification | Orchestrator signal |
| --- | --- | --- | --- |
| Negative | High (e.g. 30%) | **Critical hemorrhage** | P1 — immediate alert with options |
| Negative | Low (e.g. 0.1%) | **Background noise** | P3 — silent log, passive optimization |
| Neutral / stable | High (e.g. 30%) | **Drift risk** | P2 — active monitoring |

This matrix is computed by a dedicated governance worker — not by the orchestrator itself. The orchestrator arbitrates; the governance worker calculates. This is the same separation that exists between a CEO and a CFO.

### The CEO/shareholder model

The economic governance model maps directly to an organizational structure that every professional already understands:

```jsx
Governance worker          →  CFO / analytics department
  Collects data
  Computes pipeline ROI
  Applies attention matrix
  Produces structured signal

Orchestrator               →  CEO
  Never computes — arbitrates
  Receives structured signals
  Allocates attention
  Presents options to the user

User                       →  Shareholder / board
  Never sees raw data
  Always sees: situation + options + costs + recommendation
  Decides
```

**The orchestrator never presents a problem alone.** This is a LAA invariant derived from the CEO model:

```
Never: "there is a problem"
Always: situation + options (A/B/C) + estimated cost + expected impact + recommendation
```

The user's cognitive load is protected. The complexity is absorbed by the governance layer.

### Risk tolerance as a delegation cursor

The user's `riskTolerance` parameter governs how the orchestrator handles pipelines with low confidence:

- **Conservative (≤30%)** — costly actions approved only if pipeline ROI is confirmed (`confidence ≥ medium`, `basedOn ≥ 3`)
- **Balanced (50%)** — costly actions approved if estimated ROI is high, even with low confidence
- **Audacious (≥70%)** — high-cost strategy tests approved as long as estimated impact is significant, even speculative

`riskTolerance` does not apply to validated pipelines. It governs exploration — the margin the user gives the orchestrator to test.

### Adaptive audit cadence

The governance worker does not run continuously. The orchestrator regulates its frequency according to pipeline maturity:

- **New / in test** (`confidence: speculative` or `low`) → close monitoring, every 2-3 days
- **Stabilized** (`confidence: high`, 5+ homogeneous results) → distant monitoring, every 2-3 weeks
- **Exceptional trigger** → immediate if budget threshold breached or strongly negative ROI on a profitable pipeline

The governance worker can also surface a strong anomaly autonomously — without waiting for the next orchestrator trigger. It does not loop continuously.

---

## 4. Memory Contract

### The Detection Problem

The fixed threshold for triggering pattern detection - 3+ occurrences of the same signal - was a placeholder, not a principle. It treated all signals as equivalent regardless of who produced them, in what context, and against what expectation.

Consider two users. A professional cook and an amateur cook can both repeat the same gesture three times. The repetitions are not equivalent. The professional is confirming a schema they already have - their signal is dense, context-varied, stable. The amateur is building a schema from scratch - their signal is repetitive, context-narrow, fragile. Fixed occurrence counting cannot distinguish them.

**Fixed occurrence counting is the wrong primitive.**

### The Behavioral Baseline

LAA replaces occurrence counting with a delta model. Every user has a **PersonalBaseline** - a reference distribution of expected attention across workers, built from two sources:

**Klair's PersonaBaseline** - a theoretical distribution defined by Klair for each persona + declared objective combination. A trainer who wants to create and deliver courses has an expected distribution weighted toward content and pedagogy, moderate on marketing, low on sales. These baselines are defined by Klair and refined over time from aggregated anonymized data.

**Environment-agnostic correction sources** - the PersonaBaseline is adjusted by three passive signal types that apply regardless of the environment LAA runs in:

```jsx
PersonalBaseline corrections:
  1. activated_skills
     Klair: store bricks activated
     Other environments: plugins loaded, modules enabled,
     extended system prompts, tools connected
     Signal: the user has chosen to extend capacity in this direction

  2. active_connectors
     Integrations the user has connected and actively uses
     Signal: reveals the real operational perimeter

  3. refinement_requests
     Explicit user declarations: skill targeted + objective + horizon
     Signal: declared intent with interpretation key
     Most valuable because it carries both direction AND motivation
```

```json
{
  "userId": "...",
  "personaBaseline": {
    "source": "trainer:create_and_deliver",
    "distribution": {
      "content": 0.50,
      "marketing": 0.25,
      "sales": 0.15,
      "admin": 0.10
    }
  },
  "personalBaseline": {
    "distribution": {
      "content": 0.40,
      "marketing": 0.20,
      "sales": 0.30,
      "admin": 0.10
    },
    "corrections": [
      { "source": "activated_skill", "ref": "skill:kai-sales-employee", "weight": "+0.15 sales" },
      { "source": "refinement_request", "ref": "ref-001", "weight": "+0.05 sales" }
    ]
  }
}
```

### The BehavioralDelta

```jsx
BehavioralDelta = observed_distribution - PersonalBaseline

Significant delta:
  |delta| > threshold AND observed across 2+ distinct contexts
  → feeds Memory Contract as confidence: "inferred"

Noise:
  |delta| > threshold BUT single context only
  → logged, not stored. Revisited if repeated.
```

### What a delta means - the two readings

- **Strength signal** - the user engages heavily because they are confident and fluent
- **Compensation signal** - the user engages heavily because they feel weak and are trying to close a gap

```jsx
Secondary signals toward "strength":
  - Low reformulation rate in this worker
  - Few confirmation requests
  - Fast session completion
  - Low abandon rate

Secondary signals toward "compensation":
  - High reformulation rate
  - Repeated confirmation requests
  - Frequent flow abandons
  - Repeated similar questions across sessions
```

**The Memory Contract stores the rule, not the interpretation.**

### `temporal_horizon` - the user's time scale

The orchestrator infers the user's operational time scale progressively from two passive sources:

**Passive inference - organizational model:**

```jsx
Signals observed silently:
  average_deal_cycle      → from CRM history if available
  declared_revenue_target → period declared by user
  activity_seasonality    → patterns of action across months
  engagement_level        → "tool" | "assistant" | "partner" | "delegation"

Result: temporal_horizon.confidence = "inferred"
  → Low impact on decisions until validated
  → Orchestrator calibrates conservatively
```

**Active clarification - one targeted question:**

```jsx
Triggered only when:
  A significant action is being evaluated
  AND temporal_horizon.confidence = "inferred"
  AND resource cost is non-trivial

Format:
  "This action engages [X resources] over [Y days].
   Is that within your current priority horizon?"

Result: temporal_horizon.confidence = "validated"
  → Applied actively to future decision arbitration
  → Never asked again on the same dimension
```

```json
{
  "key": "temporal_horizon",
  "value": {
    "primary": "short_term",
    "horizon_days": 30,
    "basis": "declared_revenue_target"
  },
  "confidence": "validated",
  "provenance": "orchestrator",
  "observed_at": "2026-05-21T..."
}
```

This signal directly feeds the Decision Arbitration primitives in Layer 3. An action with `expected_utility` realized in 90 days is evaluated differently for a user with `temporal_horizon: 30 days` vs. one with `temporal_horizon: 12 months`.

### RefinementRequest - declared intent as a baseline anchor

The BehavioralDelta model detects patterns from observed behavior. It is powerful but passive. LAA introduces a complementary primitive: the **RefinementRequest** - an explicit declaration by the user of a skill they want to develop, paired with the objective they are pursuing.

```json
{
  "id": "ref-001",
  "skill": "outbound_prospecting",
  "declared_objective": "convert more cold leads",
  "declared_at": "2026-05-21T...",
  "status": "active",
  "horizon": "medium_term",
  "refinement_delta": {
    "trend": "converging|stagnating|diverging",
    "last_evaluated_at": "2026-05-21T...",
    "evidence": [
      { "session": "...", "signal": "reformulation_rate", "value": 0.4 },
      { "session": "...", "signal": "confirmation_requests", "value": 0.6 }
    ]
  }
}
```

A RefinementRequest is a **declared anchor** against which the Memory Contract is measured. It does not expire passively - it is either fulfilled, revised, or explicitly closed.

Three states:

```jsx
Declared intent + converging delta
  → user is progressing
  → Memory Contract: reinforce, do not interrupt

Declared intent + stagnating delta
  → behavior does not change despite declared intent
  → propose approach adjustment at next Loop 1

Declared intent + diverging delta
  → user is moving away from their declared objective
  → Loop 1 surfaces the gap:
    "You told me you wanted to improve X.
     Over the last N sessions I've observed Y.
     Do you want to adjust the objective,
     or explore what's blocking?"
```

### Three levels of Memory Contract quality

The Memory Contract operates at three levels depending on how much of the ecosystem implements LAA:

**Level 0 - non-LAA workers with minimal contract:** workers are black boxes but return a Level 0 Worker Contract (see Section 7). The orchestrator observes `reversible`, `outcome`, and `resource_consumed`. Memory Contract is fed with governance primitives only.

**LAA minimal - heterogeneous workers without contracts:** workers are fully opaque. The orchestrator observes outputs and user reactions only. BehavioralDelta is computed from external observation. Memory Contract is fed indirectly.

**LAA complete - conformant workers:** workers expose structured signals via `log_interaction`. Memory Contract is fed from the inside. BehavioralDelta is more precise, RefinementRequest trends are tracked at the action level. Feedback Loops operate at full capacity.

> *Note: RefinementRequest is part of LAA complete only. In Level 0 and LAA minimal, pattern detection relies solely on BehavioralDelta observed from external signals.*
> 

The difference is a spectrum - each worker that adopts the framework increases the resolution of the Memory Contract.

### Structure of a memory signal

```json
{
  "key": "prefers_conciseness",
  "value": true,
  "confidence": "inferred|validated|rejected|abandoned",
  "provenance": "worker:sales|orchestrator",
  "observed_at": "2026-05-21T...",
  "expires_at": null,
  "delta_source": {
    "contexts": ["session:sales", "session:marketing"],
    "delta_magnitude": 0.18
  },
  "refinement_anchor": "ref-001",
  "scope": "global|worker:sales"
}
```

### Four confidence levels

| Level | Definition | How it's triggered |
| --- | --- | --- |
| `inferred` | Observed delta, not confirmed | Significant delta across 2+ contexts |
| `validated` | Explicitly confirmed by user | Loop 1 proposal accepted |
| `rejected` | Denied or corrected by user | Loop 1 proposal declined - permanent |
| `abandoned` | Consciously set aside during exploration | Orchestrator only - Exploration Mode |

### Freshness rules

- `inferred` not confirmed after 30 sessions → `expired`. Exception: if delta is still active and growing, TTL resets automatically.
- `validated` → never expires without explicit user action. Exception: if observed behavior contradicts a `validated` rule for 10+ sessions across 3+ distinct contexts, orchestrator flags for Loop 1 review - never silently overrides.
- `rejected` → permanent. New delta on same dimension → logged as `PersonalBaseline` correction candidate, not re-proposed.

### Ownership rules

- Only the orchestrator writes to the Memory Contract
- Workers contribute signals via `log_interaction` - they do not write directly
- Third-party InstructionSets cannot read or write the Memory Contract

---

## 4b. The Human Algorithm - behavioral inference from abandoned paths

Classical memory systems grow with usage. LAA inverts this. **The pattern replaces the source data.**

```jsx
Sessions 1-5   → 5 raw abandoned paths stored
Sessions 6-20  → pattern detected → 5 cases compress into 1 rule
Sessions 21+   → new observations confirm or refine the rule
                 no additional storage required
```

```json
{
  "key": "pricing_approach",
  "confidence": "abandoned",
  "detail": {
    "context": "Q1 strategy session",
    "abandoned_reason": "insufficient market data",
    "reopen_trigger": "if market data becomes available",
    "expires_at": "2026-08-21T..."
  },
  "pattern": {
    "trigger": {
      "context_type": "strategy_decision",
      "constraint_type": "insufficient_data"
    },
    "observed_choice": "reversible_option",
    "confirmed": false,
    "observations": 3,
    "confidence": 0.6
  }
}
```

The pattern is a constrained triplet: **(trigger condition, observed choice, confirmation status)**. It becomes `confirmed: true` only after Loop 1 validation.

**Three properties no other memory system has:**

- **Volume convergence** - more observations → more accurate model, same or smaller storage
- **True portability** - 500 tokens describe what 10,000 sessions produced
- **Time resistance** - specific ideas decay, decision logic persists

---

## 5. Feedback Loops

LAA assigns the orchestrator a special role: **cybernetic regulator**.

```jsx
Worker A observes: user always reformulates sales messages
  → local signal, sales context only

Worker B observes: user always shortens recruitment content
  → local signal, recruitment context only

Orchestrator observes both:
  → "this user consistently prefers conciseness across all contexts"
  → global signal → Memory Contract update
```

### Loop 1 - User feedback

```jsx
Orchestrator observes BehavioralDelta across workers
  → Significant delta across 2+ contexts detected
  → Secondary signals accumulated
  → RefinementRequest trends evaluated
  → Stores as confidence: "inferred"
  → TIMING: end of session only
  → Proposes: "I noticed that [behavioral rule]. Make this a working rule?"
  → Yes → confidence: "validated" → update PersonalisationFile
  → No  → confidence: "rejected" → never re-propose
```

### Loop 2 - Platform feedback

```json
{
  "type": "ai_fallback|flow_abandoned|registry_miss|repeated_question|worker_out_of_domain|escape_hatch",
  "worker": "sales|recruitment|orchestrator",
  "intent_signal": "what the user wanted",
  "resolution": "what happened",
  "tenant_id": null
}
```

With both loops active: individual experience improves through Loop 1, platform improves through Loop 2, workers are never modified directly - only their InstructionSets evolve.

---

## 6. Case Studies - Klair

### Axel - The orchestrator

Axel is the only worker with Layer 7. It never produces final deliverables. It routes, observes, detects drift, and proposes Memory Contract updates at session end.

**Policy specifics:** never produces a final email, post, or report. Anti-ping-pong: one intent = one worker. Anti-engagement: never encourages the user to stay in session longer than needed.

### Kai - A composable sales worker

| Level | Slug | Capabilities |
| --- | --- | --- |
| Intern | `skill:kai-intern` | Rules execution, basic classification |
| Assistant | `skill:kai-assistant` | Segmentation, ICP scoring, multi-objective |
| Employee | `skill:kai-employee` | Strategy building, playbooks, analytics |

Each level activates different InstructionSets via the Module Registry. The core SP (~310 tokens) never changes.

### Katia - An extensible recruitment worker

Katia adapts to three user profiles via `depthLevel`: PME (light), large company (standard), professional recruiter (expert).

**Non-negotiable compliance:** `katia-compliance-v1` is loaded `always`. No InstructionSet - native or third-party - can disable it. Enforced at the Policy layer.

---

## Related Work

**Auton Framework** (arXiv, Feb 2026) - closest to LAA layers 1-2. Does not distinguish Behaviors from Strategies, no cybernetic feedback loops, no trust-building model.

**Google ADK** - context architecture, not governance framework. No Policy layer, no Module Registry, no feedback loops.

**LangChain / LangGraph** - frameworks for building agents, not specifications for structuring them.

**BDI Model** - designed for symbolic AI. LAA is designed for LLM-native agents with dynamic module loading and human collaboration as a first-class concern.

| Concept | LAA | Closest existing work |
| --- | --- | --- |
| Level 0 Worker Contract - minimum integration | ✅ | Not formalized |
| `temporal_horizon` in Memory Contract | ✅ | Not formalized |
| Decision Arbitration primitives (Layer 3) | ✅ | Not formalized |
| Long-term utility principles (Layer 6) | ✅ | Not formalized |
| Orchestrator as structural invariant | ✅ | Not formalized |
| Behaviors vs Strategies distinction | ✅ | Not formalized |
| Memory Contract with trust levels | ✅ | Partial in Auton |
| Behavioral Baseline & Delta model | ✅ | Not formalized |
| RefinementRequest - declared intent anchor | ✅ | Not formalized |
| Cybernetic feedback loops (2 loops) | ✅ | Not formalized |
| Human collaboration model | ✅ | Not addressed |
| Module Registry as third-party extension | ✅ | Not formalized |
| Economic Governance Layer (Section 3c) | ✅ | Not formalized |
| Klair as Personal Layer — SaaS synergy (Section 7b) | ✅ | Not formalized |
| Conformance gradient - heterogeneous ecosystems | ✅ | Not formalized |

---

## 7. Portability - LAA as an Orchestration Layer for Heterogeneous Ecosystems

The portability of LAA is not that the same architecture can be applied to different domains. It is that a single LAA orchestrator can coordinate any battery of agents - regardless of platform, vendor, or internal architecture.

In its minimal form, LAA requires only one conformant component: the orchestrator. The workers it coordinates can be anything: native LAA workers, third-party agents on other platforms, specialized models on Claude, GPT, Gemini or Cursor, API tools without an AI layer. The orchestrator does not need to know how they work internally. It needs to observe what they produce and what the user does with it.

This is the entry point for organizations with existing AI ecosystems. They do not need to rebuild. They need to add an orchestration layer that implements LAA - and they immediately gain Memory Contract coherence, Policy governance, and Feedback Loops across tools that previously operated in isolation.

### The Level 0 Worker Contract - minimum integration for non-LAA workers

For the orchestrator to evaluate `reversibility`, `trust_risk`, and `expected_utility` accurately - even at Level 0 - it needs some signal from the workers it coordinates. The **Level 0 Worker Contract** is the minimum structure that any worker - regardless of internal architecture - can return to give the orchestrator the governance primitives it needs.

```json
{
  "worker_id": "string",
  "action_performed": "string",
  "reversible": true,
  "resource_consumed": {
    "tokens": 0,
    "cost_eur": 0.0
  },
  "outcome": "success | partial | failed",
  "confidence": 0.0,
  "side_effects": []
}
```

**`reversible`** - the single most important field. A message sent is not reversible. A draft saved is. This allows the orchestrator to calibrate `autonomy_cost` correctly even for workers it cannot inspect internally.

**`resource_consumed`** - feeds `resource_pressure`. Token count and estimated cost in the user's currency. Bridge between budget governance and actual consumption.

**`outcome`** - feeds the ActionLog for `trust_risk` calibration over time. A worker that consistently returns `partial` is flagged for Loop 2 review.

**`confidence`** - the worker's self-assessed reliability. Used to decide whether to present the result directly or flag for user review.

**`side_effects`** - any external action taken beyond the primary output (email sent, webhook triggered, record updated). Allows the orchestrator to reason about cascading effects.

**`worker_id`** - scopes the contract to a specific worker. Enables trust profiling over time.

The Level 0 contract is the **entry ramp** - the minimum viable integration that makes any existing agent a governed participant in an LAA ecosystem.

### Adoption path for existing frameworks

```jsx
LangChain agent:
  → Wrap output in a Level 0 contract. 6 fields. No architecture change.
  → Immediately governed by the orchestrator.

CrewAI workflow:
  → Each crew step returns a Level 0 contract.
  → Progressive upgrade: add log_interaction signals step by step.

Simple REST API:
  → reversible: inferred from HTTP method (GET vs POST vs DELETE)
  → cost_eur: estimated from API pricing
```

The orchestrator gains governance. The workers gain nothing to lose.

### Conformance gradient

Level 0 - orchestrator only

Workers: black boxes

Memory Contract: fed by external observation

Result: coherent ecosystem, basic learning

Level 1 - orchestrator + some conformant workers

Workers: mixed - some expose structured signals

Memory Contract: partially fed from inside

Result: higher signal quality on conformant domains

Level 2 - orchestrator + all conformant workers

Workers: all expose structured signals via log_interaction

Memory Contract: fully fed from inside

Feedback Loops: at full capacity

Result: self-calibrating system, maximum individualisation

```
No store required. No shared infrastructure required. The conformance gradient is the adoption path - each step delivers immediate value without requiring the previous step to be complete.
The freelance consultant from Section 6 illustrates this: their marketing worker may be a native LAA worker, their accounting tool a simple API integration, their CRM a third-party agent on another platform. The orchestrator coordinates all three. The Memory Contract accumulates across all of them. The system learns - at the level of resolution each component allows.

---

## 7b. Klair as Personal Layer — SaaS Synergy Model

*Inserted after Section 7 — Portability, before Section 8 — Conclusion*

Section 7 describes LAA portability as the ability of a single orchestrator to coordinate heterogeneous agents across platforms. This section describes a specific application of that principle: **Klair as a persistent personal layer that any SaaS can connect to — and benefit from immediately.**

### The UX problem every SaaS AI faces

Every SaaS product that integrates an AI layer today faces the same structural problem: the AI does not know the user.

It does not know their communication style, their rules, their ongoing objectives, their relationship history with their contacts, their preferred channels, or their risk appetite. Every session starts from zero. Every new tool requires reconfiguration. The user re-explains themselves — to every AI, in every tool, every time.

This is not a failure of the AI's capability. It is a failure of **personal context portability**.

### The Klair connector model

Klair solves this not by replacing the SaaS AI, but by becoming the personal layer it connects to.

The integration model is bidirectional and symmetric:

**From Klair → SaaS**
The orchestrator interrogates the SaaS via its published MCP connector. The user works from Klair; the orchestrator reads deals, tasks, candidates, or any other data from any tool with a certified connector. The orchestrator has a unified view across all tools.

**From SaaS → Klair**
The SaaS AI invokes the orchestrator to access the PersonalisationFile, StyleLibrary, Memory Contract, and governance parameters. It stops being generic — without rebuilding the personalization layer itself.

```jsx
SaaS AI before personal layer connector:
  → starts from zero every session
  → re-asks for preferences
  → ignores relationship history
  → no governance on autonomous actions

SaaS AI after personal layer connector:
  → loads PersonalisationFile at session start
  → knows tone, language, rules, ongoing objectives
  → accesses relationship context via Memory Contract
  → governed by riskTolerance and resourceBudget
```

### Instant parametrization — the core UX argument

The value proposition for the SaaS partner is immediate and demonstrable:

> **A user who has Klair is operational from the first session. Zero onboarding friction.**

The SaaS does not need to build onboarding flows, preference capture, or personalization engines. Klair already has it. The connector surfaces it.

For the user, the value is symmetric:

> **My AI already knows me — wherever I work.**

This is not a marginal improvement. It is a step-change in how personal AI works. The PersonalisationFile becomes a portable asset the user owns and carries across every tool in their ecosystem.

### What each party contributes

| | Klair brings | SaaS brings |
| --- | --- | --- |
| To the user | Personal context, memory, governance | Domain tools, data, execution surface |
| To each other | PersonalisationFile, StyleLibrary, Memory Contract | Live business data, action endpoints, domain signals |
| To the ecosystem | Portable personal layer standard | Certified connector, distribution reach |

### The connector as a mutual asset

A certified Klair connector is not a one-way integration. It is a mutual distribution asset:

- **For the SaaS** — visibility in the Klair ecosystem and store. Users who discover the connector from Klair become SaaS users.
- **For Klair** — each connector extends the reach of the personal layer into a new domain. More connectors → more of the user's professional life is governed by a single coherent memory.

The minimum viable connector is three MCP endpoints — read personal context, write interaction signal, declare pipeline. It can be implemented in a day. Complexity grows with the depth of the integration, not with the entry cost.

### Relation to the conformance gradient

The connector model is the conformance gradient applied to SaaS partnerships:

```jsx
Level 0 — 3 endpoints
  SaaS AI reads PersonalisationFile at session start
  → immediate UX improvement, zero architecture change

Level 1 — full MCP connector
  Axel interrogates SaaS data bidirectionally
  → unified orchestration across tools

Level 2 — deep integration
  SaaS events feed Memory Contract signals
  Pipeline governance spans Klair + SaaS actions
  → self-calibrating system across the full user workflow
```

Each level delivers immediate value. No level requires the previous to be complete.

> **The orchestrator governs. Workers execute. SaaS tools are workers.**
> The personal layer persists. Wherever the user works.

---
## 8. Conclusion
LAA proposes a separation that clarifies what is currently implicit in most agent architectures:
```

what an agent IS

vs

how it ACTS

```
But the deeper proposition is about orchestration:
```

what executes

vs

what governs

```
By making the orchestrator a structural invariant - not a design choice - LAA provides a stable governance layer for any ecosystem of agents. Workers can be heterogeneous, distributed, built by different teams on different platforms. The orchestrator is the single component that maintains coherence, protects the user, and makes the system improvable over time.
The most important insight is architectural, not technical:
> **The registry is the source of truth - not the prompt.**
And the most important structural decision:
> **The orchestrator governs. Workers execute. These are not the same role, and they must never be conflated.**
Klair implements LAA across all its workers and invites the ecosystem to adopt it as a shared standard for human-AI collaboration.
---
## Why existing memory systems become obsolete
The memory systems that exist today - Mem0, Zep, Amazon Bedrock AgentCore, Google Vertex Memory Bank - store facts. They retrieve context. They reduce repeated instructions. LAA solves a different problem entirely.
```

Memory infrastructure (Mem0, Zep, Amazon):

Stores facts about a user

→ no relationship to build

→ no governance layer

→ no orchestration

LAA:

Documents the relationship between a human and their agent ecosystem

→ trust built progressively across all workers

→ validated rules, abandoned paths, integrated corrections

→ declared intent tracked against observed behavior

→ portable asset that belongs to the user

→ self-calibrating across a conformance gradient

```
A `validated` signal is proof of repeated trust. A `rejected` signal is an integrated correction - permanent. An `abandoned` signal is the map of shared reasoning. A `RefinementRequest` with a diverging trend is the moment the agent becomes a genuine collaborator.
No memory infrastructure captures this. They store the fact. LAA documents the relationship quality that produced it - across every agent in the ecosystem, regardless of where they run.
> In a world where AI makes every output cheap, the scarce resource is proven human connection. What becomes scarce and valuable is the proven trust between a specific human and their specific agent ecosystem, built through repeated interactions with real stakes.
This is what LAA is built to capture.
```
