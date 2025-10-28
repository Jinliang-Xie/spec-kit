# Project Constitution (Canonical English)

Note: This is the canonical source of truth. A Chinese companion lives at `memory/constitution.md`.

## Goal and Scope

Integrate an LLM into a Python pipeline to identify and atomize process "Steps" from patent-related JSON text, encode/unify them, cluster/canonicalize, and build a Neo4j Step Graph visualization.

Pipeline stages:
- Stage 1: Identification
- Stage 2: Encoding / Unified Characterization
- Stage 3: Atomicization (Splitting & Refinement)
- Stage 4: Clustering / Canonicalization

Each stage must be runnable and inspectable from a VS Code terminal; on user confirmation, artifacts are persisted for the next stage and for auditability.

### Stage-wise Visualization (Required)

- Every stage must write to Neo4j and produce corresponding visualizations for multi-stage comparison, verification, and debugging.
- Visualization requirements:
  - Stage 1 (Identification): Patent nodes + Step nodes (initial) + HAS_STEP/NEXT relationships; each Step tagged with `stage: "identification"`, `order_index`, `text_raw`.
  - Stage 2 (Encoding): build on Stage 1; augment Step nodes with `encoding_vector`/`dimension_tags`; optionally add Dimension nodes and HAS_DIMENSION relationships.
  - Stage 3 (Atomicization): create new atomized Step nodes (tagged `stage: "atomicized"`); link via COMPOSED_OF to original Steps; retain DERIVED_FROM to Patent.
  - Stage 4 (Clustering): add Cluster or CanonicalStep nodes; link via IN_CLUSTER or SAME_AS to atomized Steps; optionally include similarity weights.
- Version isolation: tag all nodes and relationships with `run_id` to support side-by-side multi-version comparison in the same database; provide Cypher query templates to filter visualizations by `run_id`.
- Reference: Neo4j official docs at https://neo4j.com/docs/ for Cypher, data modeling, and visualization best practices.

## Input Contract (Two-Lane)

- Lane A (Semi-structured/curated JSON): Prioritize the curated domain JSON present in this repo. Typical fields include:
  - `technical_process` (string, with steps joined by "->")
  - `technical_parameters` (nested object containing `nominal_capacity`, `nominal_voltage`, `charge_rate`, `discharge_rate`, `power_density`, `energy_density`, `internal_resistance`, etc., with subfields like `<name>`, `unit`, `value` ("1"/"0" flag), and `actions` free-text)
  - Additional context: `innovation_object`, `innovation_component`, `technical_effect`, `application_field`, `battery_type`, `electrochemical_battery_type`, `separator`, `sustainability`, `safety`, etc.
  - Data characteristics: numeric `0` and empty `unit` may signal "unknown"; `value` is a string flag ("1"/"0"); some fields may contain the literal string "null"; content is primarily Chinese with units/symbols.
- Lane B (Raw patent text / generic JSON): Retain support for raw/generic forms from patent sources. Follow an extract→validate→structure approach; if no structured fields exist, steps and parameters are extracted from text.

Principle: The parser must be robust to "0/empty/\"null\"/Chinese-English mix/comma-like list separators" and map ambiguous values to "unknown" rather than numeric zero.

Reference: See the detailed dataset profile at `docs/data-profile.md` for observed fields, anomalies, and normalization heuristics (English canonical; Chinese counterpart at `docs/data-profile.zh-CN.md`).

## Step Identification and Atomicization

- Source priority: prefer explicit `technical_process`; then enrich with `technical_effect`, `application_field`, etc. Lane B extracts from body text (LLM + patterns) under schema constraints.
- Atomicization rules (Lane A):
  - Split on "->" as the first pass to get an ordered sequence; preserve order with `order_index`.
  - Further refine to remove duplicates and separate conjunctive content; keep key signals (materials, ratios, temperature, time, voltage/current).
  - For references/loops/conditions, retain as annotations without forced expansion.
  - Minimal Step fields: `id`, `order_index`, `text_raw`, optional `tokens`, `source_field`, `source_doc`, and `provenance`.
- Lane B atomicization: combine patterning/templating and LLM under schema/guardrails.

## Parameter Modeling and Normalization

- First-class Parameters: create `Parameter` nodes and link via `HAS_PARAM` to Step or Step groups; keep `actions` free-text as provenance/explanation.
- Dual-track (Raw/Normalized):
  - Raw: `raw_text`, `raw_numeric`, `raw_unit`, `source_field`, `source_path`, `evidence`.
  - Normalized: `normalized_value`, `unit_std` (SI or industry-common), `normalization_rules_version`, `confidence`.
- Interpretations: treat 0/empty/"0"/"null" as "unknown" unless clearly a mathematical zero; map `value:"1"/"0"` to boolean `has_value`.
- Language and lists: split `application_field` and similar fields by Chinese/English separators; retain a `lang` tag.
- Parameter Dictionary v0 (evolvable): `nominal_capacity (mAh/Ah)`, `nominal_voltage (V)`, `charge_rate (C)`, `discharge_rate (C)`, `power_density (W/kg)`, `energy_density (Wh/kg)`, `internal_resistance (mΩ)`, `cycle_life (cycles)`, `self_discharge_rate (%)`, `operating_temperature_min/max (°C)`, `separator_thickness (mm)`, `porosity (%)`, `dimensions (length,width,height + unit)`, etc.

## Minimal Graph Model (Neo4j)

### Nodes and Labels

- `Patent`: patent document node; properties at minimum: `patent_id` (unique), `source_file`, `lang`.
- `Step`: step node; properties at minimum: `id` (unique), `stage` ("identification"/"encoding"/"atomicized"/"clustered"), `order_index`, `text_raw`, `run_id`.
- `Parameter`: parameter node; properties: `param_name`, `raw_value`, `raw_unit`, `normalized_value`, `unit_std`, `normalization_version`, `confidence`, `has_value` (boolean).
- `Dimension` (optional): dimension/category node; properties: `dimension_name`, `taxonomy_version`.
- `Cluster`: cluster node; properties: `cluster_id`, `cluster_method`, `run_id`.

### Relationship Types

- `[:HAS_STEP]`: Patent → Step, denoting patent contains step; optional property `order` (sequence number).
- `[:NEXT]`: Step → Step, intra-document step sequence; optional property `order_delta` (gap).
- `[:HAS_PARAM]`: Step → Parameter, step associates parameter; optional properties `source_field`, `evidence` (provenance snippet).
- `[:DERIVED_FROM]`: Step → Patent, post-atomicization/clustering traceability to source; property `source_field` (e.g., `technical_process`).
- `[:COMPOSED_OF]`: Step → Step, new atomized Step points to original split Step; optional property `split_reason`.
- `[:HAS_DIMENSION]` (optional): Step → Dimension, step tagged with dimension.
- `[:IN_CLUSTER]`: Step → Cluster, step belongs to cluster.
- `[:SAME_AS]`: Step ↔ Step, equivalence/canonicalization relation (synonymous steps); optional property `similarity_score`.

### Constraints and Indexes

- Uniqueness Constraints:
  - `CREATE CONSTRAINT patent_id_unique FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE`
  - `CREATE CONSTRAINT step_id_unique FOR (s:Step) REQUIRE s.id IS UNIQUE`
  - `CREATE CONSTRAINT cluster_id_unique FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE`
- Indexes:
  - Index high-frequency query fields: `run_id`, `stage`, `order_index`, `param_name`, `cluster_method`.
  - Example: `CREATE INDEX step_run_id_idx FOR (s:Step) ON (s.run_id)`
  - Example: `CREATE INDEX step_stage_idx FOR (s:Step) ON (s.stage)`
- Property Existence Constraints (Enterprise): optionally enforce non-null for critical properties (e.g., `Step.stage`, `Patent.patent_id`).

### Write Principles (per Neo4j Best Practices)

- Idempotent writes: use `MERGE` instead of `CREATE`; merge nodes by unique identifiers (e.g., `patent_id`, `step.id`) to avoid duplicates.
- Batch transactions: batch 500–10,000 nodes/relationships per transaction; use `UNWIND` + parameterized Cypher; monitor memory and transaction logs.
- Parameterized queries: all Cypher queries must use parameters (`$param`) to prevent Cypher injection and improve query cache hits.
- Version isolation: tag all nodes and relationships with `run_id` (UUID or timestamp); support multi-version side-by-side in the same database.
- Rollback support: log node/relationship counts and timestamps per `run_id`; provide Cypher template to delete a specific run:
  ```cypher
  MATCH (n {run_id: $run_id})
  DETACH DELETE n
  ```
- Property guidelines: avoid large strings (>1KB) as direct node properties; consider external storage + references or chunked storage.

### Reference and Learning Resources

- Neo4j Data Modeling Fundamentals: https://neo4j.com/docs/getting-started/data-modeling/
- Cypher Manual (Cypher 5): https://neo4j.com/docs/cypher-manual/current/
- Constraints and Indexes: https://neo4j.com/docs/cypher-manual/current/constraints/
- GraphAcademy Data Modeling Course: https://graphacademy.neo4j.com/courses/modeling-fundamentals/

## LLM Invocation and Traceability

- Python-first, CLI invocation preferred (official CLIs or local wrappers). Version all prompts/models/params.
- Logging and cache: structured JSON logs with input digest, model+hyperparams, output summary, timing, cost estimate; cache keys include input hash and prompt version.
- Anti-hallucination: schema-constrained generation; regex/unit/range validations for critical fields; on failure, fallback to "unknown + evidence".

### Orchestration: Stepwise CLI + GitHub Copilot + Codex

- Stepwise CLI is the production path: every stage exposes a stable CLI subcommand (text/JSON I/O) with flags like `--model`, `--temperature`, `--cache`, `--run-id`. LLM outputs only become valid pipeline inputs after being produced via CLI, persisted, and logged.
- GitHub Copilot for interactive authoring: used in VS Code to draft/refactor code and prototype prompts. Before commit, stabilize prompts/commands under the agent directories (see `AGENTS.md`), and verify outputs via the corresponding CLI.
- Codex CLI for scripted/batch calls: store common tasks under `.codex/commands/*` (or equivalent), pass parameters via `$ARGUMENTS`; share the same logging/cache contract as the pipeline CLI (`run_id`, `input_hash`, `prompt_version`, `agent`, `model`, etc.).
- Unified telemetry and cache: regardless of origin (pipeline CLI, Copilot-triggered, or Codex CLI), write JSONL logs and cache entries with keys at least `{input_hash, prompt_version, model}`; minimum log fields: `run_id`, `stage`, `agent`, `model`, `prompt_id`, `cache_hit`, `cost_estimate`.
- Gatekeeping: IDE-generated results never flow directly into the next stage; they must pass through the stage’s CLI for validation and persistence.
- Directories and conventions: place agent/assistant prompts and workflows per `AGENTS.md` (e.g., `.github/prompts/`, `.codex/commands/`) to enable cross-tool reuse and packaging.

## Governance and Language Policy

- Bilingual docs: English is canonical; Chinese is a companion for communication. Keep versions in sync and note any deltas.
- Change control: modifications to input contract, parameter dictionary, or graph model require "sample evidence + impact assessment + rollback plan".
- Quality gates:
  - Build: scripts/deps are reproducible and documented.
  - Type/format: all artifacts validated by JSON Schema and unit/range checks.
  - Tests: minimal coverage must include "step splitting + 2 parameter normalizations + idempotent graph writes".

Version: 0.2.0 (draft) | Amendment: calibrated using latest samples from `IG_start/data/raw_patent_data` for the two-lane input contract, 0/empty/"null" interpretation rules, Parameter Dictionary v0, and atomicization rules.
