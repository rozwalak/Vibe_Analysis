# Agentic Scientific Discovery with Codex

This repository is a template for using Codex as a scientific collaborator rather than only as a code generator. It gives the agent a scientific question, an evidence-driven plan, durable project memory, and explicit rules for handling data, uncertainty, reproducibility, and reporting.

Our example concerns population genomics and remote-family discovery for a bacteriophage. Replace that scientific content with your own project, but retain the separation of responsibilities among the files. That separation is what helps an agent remain scientifically directed over a long investigation.

## How the repository works

The core documents answer different questions:

| File | Question it answers | How Codex uses it |
| --- | --- | --- |
| `SCIENTIFIC_BRIEF.md` | What are we trying to discover, and why? | Establishes hypotheses, scientific questions, available evidence, scope, non-goals, and success criteria. |
| `ANALYSIS_PLAN.md` | What evidence and analyses could answer those questions? | Provides phases, controls, outputs, decision gates, validation criteria, and stop conditions. |
| `AGENTS.md` | How must the agent behave while doing the work? | Supplies persistent operating rules: scientific validity first, source data read-only, explicit assumptions, durable evidence, and reporting obligations. Codex reads this automatically when working in the directory. |
| `STATUS.md` | What is currently known, uncertain, blocked, or next? | Acts as concise scientific working memory across sessions. It should reflect evidence, not merely task completion. |
| `DECISIONS.md` | Why were consequential methodological choices made? | Preserves choices about thresholds, exclusions, models, statistical methods, and changes of direction. |
| `README.md` | How should a person adopt and operate this template? | Provides the human-facing entry point. |

Together they create a feedback loop:

```text
Scientific question
        ↓
Evidence and analysis plan
        ↓
Codex inspects data and performs the smallest adequate analysis
        ↓
Durable tables, figures, diagnostics, and report conclusions
        ↓
Status and decisions updated from the evidence
        ↓
Next question or revised analysis
```

This is deliberately not a prompt that tells an agent to execute a fixed pipeline from beginning to end. The decision gates in the plan require the evidence to determine what happens next. Unexpected findings, missing metadata, confounding, or failed assumptions should change the investigation.

## What to change for a new project

Start by copying the repository, then edit the documents in this order.

### 1. Rewrite `SCIENTIFIC_BRIEF.md`

Replace all example-specific content, including:

- the system, organism, cohort, treatment, or phenomenon being studied;
- the central hypothesis and primary scientific questions;
- input datasets, metadata, identifiers, units, and known sample counts;
- the scientific scope and explicit non-goals;
- expected confounders and plausible alternative explanations;
- the evidence required for each claim;
- meaningful deliverables and success criteria.

Write questions that can be answered or narrowed by evidence. Distinguish exploratory aims from confirmatory tests, and do not encode the hoped-for conclusion as an assumption.

### 2. Adapt `ANALYSIS_PLAN.md`

Replace the phage-specific work packages with the smallest analyses capable of discriminating among the hypotheses. For each phase, define:

- its scientific objective;
- required inputs and metadata;
- data-quality checks, units, missingness checks, and sample counts;
- baselines and controls;
- confounders and dependency structure, such as repeated measures or batches;
- methods and configurable thresholds;
- durable tables, figures, diagnostics, and report sections;
- a decision gate stating when to proceed, revise, or stop;
- independent validation for important or unexpected findings where practical.

Avoid turning the plan into a speculative catalogue of every analysis that might be possible. A shorter plan with explicit evidence requirements and decision gates is usually more useful than a large pipeline that the available data cannot support.

### 3. Specialise `AGENTS.md` cautiously

Keep the general scientific safeguards, but change project-specific details such as:

- required documents and their reading order;
- source-data and sensitive-data locations;
- expected output directories;
- domain-specific statistical standards;
- compute environment, scheduler, or approved workflow conventions;
- the primary scientific report path;
- project-specific completion criteria.

Instructions closer to a working subdirectory can be placed in another `AGENTS.md`; Codex applies the nearest relevant instructions. Use that mechanism for genuinely different subprojects, not to weaken repository-wide safety or scientific standards.

### 4. Initialise `STATUS.md` and `DECISIONS.md`

Before the first analysis, populate `STATUS.md` with the current question, confirmed inputs, known data-quality issues, and the next highest-value analysis. Leave unsupported finding sections empty rather than filling them with expectations.

Add entries to `DECISIONS.md` only for choices that affect interpretation or reproducibility. A useful entry records the date, decision, evidence or rationale, alternatives considered, and consequences. Examples include changing an inclusion threshold, excluding a batch, selecting a statistical model, or abandoning an analysis after a failed assumption.

## What to leave alone

Preserve the following principles unless the scientific domain genuinely requires a stricter version:

- Scientific validity and correct interpretation take priority over code elegance.
- Source data are read-only; derived outputs go to separate locations.
- Original sample identifiers and provenance are preserved.
- Joins, missingness, units, sample counts, exclusions, and unmatched records are checked explicitly.
- Observations, interpretations, hypotheses, and unresolved uncertainty remain distinct.
- Effect sizes and uncertainty accompany significance tests.
- Repeated measures, batches, cohorts, leakage, multiple testing, and pseudoreplication are addressed where relevant.
- Simple, interpretable baselines precede complex or AI-based methods.
- Important claims point to durable tables, figures, or diagnostics.
- Expensive or stochastic analyses are reproducible, resumable, versioned, and seeded where applicable.
- Negative and inconclusive results are reported.
- The agent stops at genuine decision gates instead of silently inventing metadata, resolving ambiguity, or forcing a result.
- Working code is not treated as scientific completion.

Also leave `LICENSE` and its attribution unchanged unless the copyright holder intentionally chooses different licensing terms.

## Recommended directory layout

The documents describe a useful separation between immutable inputs, temporary work, reproducible analysis, durable evidence, and interpretation:

```text
.
├── AGENTS.md
├── SCIENTIFIC_BRIEF.md
├── ANALYSIS_PLAN.md
├── STATUS.md
├── DECISIONS.md
├── inputs/                 # source data and metadata; treat as read-only
├── work/                   # caches, logs, and regenerable intermediates
├── scripts/                # reproducible analysis and plotting code
├── notebooks/              # exploration, not the sole record of key results
├── results/
│   ├── tables/             # durable numerical evidence
│   └── figures/            # final, reproducible figures
└── reports/
    └── scientific_report.md
```

This layout is a convention, not a requirement. If an existing project already has a coherent structure, update the documents to match it rather than reorganising data merely to resemble this example. Never place large source datasets in Git. Read-only symbolic links can be appropriate on shared or HPC systems.

## Running an investigation with Codex

Open Codex in the repository root after customising the brief and plan. A useful first request is:

> Read `SCIENTIFIC_BRIEF.md`, `ANALYSIS_PLAN.md`, `STATUS.md`, and `DECISIONS.md`, then audit the repository and inputs according to `AGENTS.md`. Do not modify source data. Report blockers and complete only the smallest analysis needed to pass the first decision gate. Update the scientific report, status, and decisions with evidence.

For later sessions, ask Codex to continue from `STATUS.md` and verify that its stated next step still follows from the available evidence. Requests should be framed around scientific outcomes, for example:

- “Determine whether the apparent group difference survives adjustment for batch and repeated sampling.”
- “Validate this unexpected association using an independent calculation and update the report.”
- “Compare the complex model with the predefined baseline and quantify when it adds information.”
- “Review the evidence behind each substantive conclusion and identify unsupported claims.”

Codex should first inspect the repository, data structure, existing results, environment, and uncommitted changes. It should then perform the smallest adequate analysis, save evidence under `results/`, update the scientific interpretation promptly, and record only consequential decisions.

## Human responsibilities

Agentic work does not remove the need for scientific oversight. A domain expert remains responsible for:

- confirming that the hypothesis and measurements are scientifically meaningful;
- providing correct metadata, units, study design, and data-access constraints;
- reviewing exclusions, thresholds, labels, and causal language;
- approving use of sensitive data and costly compute;
- challenging alternative explanations and checking external validity;
- deciding whether evidence is strong enough for publication or operational use.

The agent can make analysis faster and more systematic, but it cannot recover study-design information that was never recorded or turn biased data into a causal experiment.

## Signs the workflow is working

A healthy project accumulates concise, connected evidence: every major conclusion in the scientific report links to a reproducible result; `STATUS.md` makes the remaining uncertainty obvious; `DECISIONS.md` explains why consequential choices were made; and the analysis plan changes when observed data invalidate its assumptions.

Warning signs include many scripts but no scientific conclusions, undocumented thresholds, findings that exist only in notebooks or chat, complex AI models without baselines, ignored negative results, or a status file that reports implementation progress without saying what has been learned.

The goal is not autonomous execution for its own sake. The goal is a traceable collaboration in which an agent helps turn well-posed questions and carefully governed data into defensible scientific knowledge.
