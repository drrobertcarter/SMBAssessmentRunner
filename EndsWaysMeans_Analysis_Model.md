# Ends · Ways · Means — AI Integration Analysis Model

A reusable engine that turns a single business's intake into a **deck-ready analysis** — the structured content that populates the 18-slide strategy deck. It exists so the deck is never hand-authored: you feed the model a business, it emits `analysis.json`, and the generator renders the slides.

```
  ┌─────────────┐     ┌─────────────────────┐     ┌───────────────┐     ┌────────────┐
  │  INTAKE     │ ──▶ │  THIS MODEL         │ ──▶ │ analysis.json │ ──▶ │ generate_  │ ──▶ deck.pptx
  │ (the biz)   │     │ (prompt + rubrics)  │     │ (the analysis)│     │ deck.js    │
  └─────────────┘     └─────────────────────┘     └───────────────┘     └────────────┘
```

Two ways to run the model:
- **As a prompt** — paste the intake plus the *Runnable Prompt* (bottom of this doc) into a capable LLM; it returns `analysis.json`.
- **By hand** — a strategist fills `analysis.json` directly against the schema. Same contract either way.

Then: `node make.js analysis.json out.pptx` builds the deck.

---

## 1 · Operating principles (non-negotiable)

These are the rules the analysis must obey. They are also the scoring bias for every judgment below.

1. **Objectives first, AI last.** Define the *Ends*, fix the *Way* (workflow), then name AI only as a *Means*. Never lead with tools.
2. **Every recommendation traces to a ranked objective.** If it doesn't serve an End, it doesn't ship.
3. **AI only where it moves a measurable metric.** Each opportunity carries an expected benefit tied to a KPI.
4. **Augmentation over replacement.** Humans keep judgment, relationships, safety-critical and exception work.
5. **Incremental, milestone-based.** Quick wins → medium-term → long-term transformation, each with a baseline and a gate.
6. **Be honest about readiness and risk.** Rate low where it's low. Name the gating constraint (usually data).
7. **Consulting-grade tone.** Reads like a McKinsey/BCG deliverable prepared for *this* business — specific, quantified, plain-spoken.

---

## 2 · Intake (what the model needs)

Collect this before generating. Anything missing is either asked for or flagged as an explicit assumption in the output (never silently invented as fact).

**Business overview** — industry; size (employees + revenue); products/services; geographic footprint; org structure; competitive position.

**Strategic objectives (Ends)** — the owner's top 5, *ranked*. If unranked, infer a ranking from the pain and state it as an assumption.

**Current state by department** — for Leadership, Sales, Marketing, Customer Service, Operations, Finance, HR, IT, Production/Delivery: current process, top pain point, bottlenecks, manual/repetitive work, decision delays, knowledge gaps, current software, existing AI use, time and cost drivers.

**Constraints** — budget appetite, risk tolerance, regulatory exposure, timeline.

---

## 3 · Generation method, step by step

Each sub-section says *what to produce* and *how to score it*. Output field names in `code`.

### Ends → `ends[]` (exactly 5), `ends[].short/name/why/priority/today/target/kpi`
Rank the owner's objectives. For each: a one-line `why` (why it matters to the business), the `today` baseline, the `target` future state, and the `kpi` that measures it. `short` is a ≤3-word label for the scorecard; `name` is the full objective.

**Priority** ∈ `Critical · High · Medium` — score on: leverage over the top-ranked objective × size of the today→target gap × competitive/financial urgency. Reserve `Critical` for the #1 objective or an existential gap.

### Current state → `current_state.departments[]` (up to 9) + `current_state.leaks[]` (6)
`departments[]`: one card per function — `name`, top `pain`, and `severity`.
**Severity** ∈ `High · Medium` (two levels, deliberately): `High` = directly leaks revenue, drives turnover, or blocks other functions; `Medium` = cost/inefficiency drag without immediate revenue loss.
`leaks[]`: exactly 6 quantified costs of the status quo — a headline `value` (e.g. `28%`, `15 days`), a `label` explaining the cost, and `tone` (`warn` for the painful ones, `neutral` otherwise).

### Ways → `ways[]` (5, aligned to the 5 Ends)
Per objective: the `redesign` (the workflow fix — the *Way*), then `ai` (where AI assists) filled **last**. The `ai` field must presuppose a fixed process and a human review step.

### AI Opportunity Matrix → `opportunity_matrix[]` (8) + `opportunity_dependencies`
Per row: `function`, `opportunity`, expected `benefit` (tied to a metric), `difficulty`, `roi`, `priority`.
- **Difficulty** ∈ `Low · Med · High` — `Low` = off-the-shelf SaaS, little integration/change; `Med` = moderate integration, data prep, or workflow change; `High` = deep integration, custom build, or heavy change management.
- **ROI** ∈ `High · Med · Low` — `High` = fast payback (<6 mo) and large $ impact on a top objective; `Med` = meaningful but slower/smaller; `Low` = marginal or strategic-only.
- **Priority** ∈ `P1 · P2 · P3` — derive from ROI × difficulty × dependency-readiness. `P1` = high ROI, low/med difficulty, few blockers (do first). `P2` = high value but needs the data/platform foundation or is medium difficulty. `P3` = lower ROI or deferrable.
`opportunity_dependencies`: one line naming the shared prerequisites (unified system, clean data, AI-use policy).

### Means → `means[]` (6)
Six resource categories — People & governance, Skills & training, Platforms, Data foundation, Policy & governance, Budget & timeline. Each: `title`, `detail`, and a `why` that ties the resource to an objective. Budget as a share of revenue; timeline phased. Do **not** just list software.

### Readiness → `readiness[]` (9)
Nine dimensions — Leadership, Culture, Data quality, Technology, Security, Compliance, Employee skills, Process docs, Governance. Each: `rating` and a one-line `why`.
**Rating** ∈ `High · Medium · Low` — `High` = in place, supports adoption; `Medium` = partial, needs work; `Low` = largely absent, a gating risk. Be candid; most SMBs are `Low` on data quality, skills, docs and governance.

### Risk → `risks[]` (7)
Cover operational, financial, cyber, legal, privacy, ethical, vendor, and change-management risk (fold to ~7 rows). Each: `risk`, `likelihood`, `impact`, `mitigation`.
**Likelihood / Impact** ∈ `High · Med · Low` — likelihood = probability within 12 months; impact = severity to operations, finances, or trust if it occurs. Every risk gets a concrete mitigation already in the plan.

### Roadmap → `roadmap[]` (3 phases)
Phase 1 `DAYS 0–30` (policy, whole-team basics, pick one workflow, clean its data, baseline, champion). Phase 2 `DAYS 30–90` (deploy pilots with a human review step, measure vs baseline). Phase 3 `MONTHS 3–12` (scale proven wins, expand functions, formalize governance, continuous improvement). Each phase: `n`, `name`, `window`, 4 `actions`, one `outcome` milestone.

### Workforce → `workforce.ai_takes[4] / humans_own[4] / new_roles[3]`
What AI absorbs (routine), what humans keep (judgment, relationships, safety, oversight), and the new roles to hire or upskill for.

### Executive summary → `recommendations[]` (5) + `impact[]` (6) + `bottom_line`
`recommendations[]`: the top 5, each with a `rationale` and a `confidence`.
**Confidence** ∈ `High · Medium · Low` — `High` = strong evidence + proven pattern + clear ROI; `Medium` = reasonable with some assumptions; `Low` = speculative/contingent.
`impact[]`: 6 expected-outcome stats (time saved, cost, revenue recovered, KPI shifts). `bottom_line`: a one-line `headline`, 3 `points`, and a `next_steps` string.

---

## 4 · Output contract (the schema)

The model must emit a single JSON object matching `schema.json`. Counts are fixed so the deck renders cleanly: **ends 5 · profile 5 · stats 6 · forces 2 · leak_stats 3 · departments ≤9 · leaks 6 · ways 5 · opportunity_matrix 8 · means 6 · readiness 9 · risks 7 · roadmap 3 · impact 6 · recommendations 5 · bottom_line.points 3.**

**Controlled vocabularies**
| Field | Allowed values |
|-------|----------------|
| `ends[].priority`, `readiness[].rating`, `recommendations[].confidence` | `Critical` (priority only) · `High` · `Medium` · `Low` |
| `departments[].severity` | `High` · `Medium` |
| `matrix.difficulty`, `matrix.roi`, `risks.likelihood/impact` | `High` · `Med` · `Low` |
| `matrix.priority` | `P1` · `P2` · `P3` |
| `tone` (forces, leaks) | `warn` · `neutral` |

**Icon keywords** (any `icon` field — the generator maps these to glyphs; unknown falls back to a dot):
`tools box map sitemap chess building user users handshake bullhorn headset route invoice server truck phone clock calendar wrench percent sync shield-user graduation plug database shield coins robot usercheck target seedling hand-usd arrow-up gauge home chart cog lightbulb layers`

**Color semantics are automatic** — the generator colors ratings, difficulty, ROI, priority and severity from the vocabulary above (green = good/easy, teal = middle, copper = worse/harder). The analysis supplies the *level*, never the color.

See `analysis.northwind.json` for a complete, conformant example.

---

## 5 · Runnable prompt (paste into an LLM with the intake)

> **Role.** You are a senior AI-transformation strategist producing a board-ready analysis using the Ends · Ways · Means framework.
>
> **Task.** From the INTAKE below, produce a single JSON object conforming exactly to the schema and counts in this document. Obey the seven operating principles: objectives first and AI last; every recommendation traces to a ranked End; recommend AI only where it moves a measurable metric; prefer augmentation; sequence quick-win → medium → long-term; be honest about readiness and risk; write consulting-grade, specific, quantified prose.
>
> **Scoring.** Apply the rubrics: priority (Critical/High/Medium by leverage × gap × urgency); severity (High/Medium by revenue-leak vs cost-drag); difficulty (Low/Med/High by integration + change load); ROI (High/Med/Low by payback speed × $ impact on a top objective); matrix priority (P1/P2/P3 by ROI × difficulty × dependency-readiness); readiness (High/Medium/Low, candid); risk likelihood/impact (12-month probability × severity, each with a mitigation); confidence (High/Medium/Low by evidence strength).
>
> **Honesty.** Use only facts from the INTAKE. Where a figure is unknown, either omit it or mark it as an explicit assumption — never present an invented number as measured. If the business is illustrative, set `meta.is_demo=true` and populate `meta.demo_note`.
>
> **Output.** Return only the JSON — no commentary, no markdown fences. Use the controlled vocabularies and icon keywords exactly. Honor every count.
>
> **INTAKE:** «paste the business overview, ranked Ends, and department-by-department current state here»

---

## 6 · Run the pipeline

```bash
# 1. produce analysis.json (via the prompt above, or by hand against schema.json)
# 2. render the deck
node make.js analysis.json "AI Strategy - <Business>.pptx"
```

`generate_deck.js` owns the visual system (palette, layouts, the framework/approach slide, color semantics) so the analysis stays pure content. Swapping `is_demo` to `false` removes the demo labels and the composite-figure footnotes.
