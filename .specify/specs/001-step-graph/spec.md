# Feature Specification: Patent Step Graph Extraction and Visualization

**Feature ID**: 001-step-graph  
**Status**: Draft  
**Version**: 0.3.0  
**Last Updated**: 2025-10-28  
**Language**: English (Canonical)  
**Chinese Counterpart**: `spec.zh-CN.md`

---

## Executive Summary

Build a modular Python pipeline integrating LLM capabilities to extract, characterize, atomize, and cluster technical "Steps" (atomic process operations) from patent JSON extracts, and construct a multi-stage Neo4j knowledge graph for visualization, cross-document aggregation, and reusability analysis.

The system processes patent texts through four progressive stages, each persisting results to Neo4j for incremental visualization and human verification, culminating in a cross-patent unified Step graph.

---

## Goal and Scope

### Primary Objective

Extract structured Step entities and their relationships from patent-derived JSON files, encode them along three characterization dimensions (energy, matter, information), atomize composite steps into indivisible units, cluster similar steps across documents, and visualize the resulting knowledge graph at each stage in Neo4j.

### In Scope

- **Stage 1 (Identification)**: Extract Steps with inputs/outputs, devices, operation modes, reaction/operation conditions, environments; identify serial/parallel relationships; write to Neo4j.
- **Stage 2 (Encoding)**: Characterize Steps along three dimensions (energy transformation, matter transformation, information transformation) using tensor-like representations; augment graph.
- **Stage 3 (Atomicization)**: Split composite Steps into atomic, indivisible units under LLM+evidence constraints; preserve provenance.
- **Stage 4 (Clustering)**: Aggregate similar Steps across patents via clustering/canonicalization; create cross-document unified graph.
- **Multi-stage Neo4j Visualization**: Each stage writes incremental results tagged with \`run_id\` and \`stage\` for side-by-side comparison and debugging.
- **CLI-first LLM Integration**: Python wrappers invoking LLM CLIs (e.g., \`openai\`, \`anthropic\`) with logging, caching, and versioning.
- **Modular VS Code Terminal Workflow**: Each stage runnable independently with user confirmation before persisting artifacts.

### Out of Scope (for v1)

- Patent text extraction from PDFs (assume pre-extracted JSON input).
- Real-time collaborative editing of graph.
- Advanced GDS algorithms (e.g., PageRank, community detection) — deferred to future iterations.
- Multi-language UI (CLI and docs in English/Chinese only).
- Deployment automation (CI/CD for graph updates).

---

## Reference to Constitution

This specification is governed by the project constitution at \`memory/constitution.en.md\` (canonical) and \`memory/constitution.zh-CN.md\` (Chinese counterpart). Key principles:

- **Two-lane input contract**: Lane A (semi-structured JSON), Lane B (raw text fallback).
- **Stage-wise Neo4j visualization**: mandatory persistence and visualization after each of the four stages.
- **LLM orchestration**: CLI + GitHub Copilot + Codex with unified logging/caching.
- **Robust parsing**: handle 0/empty/"null" as unknown; dual-track raw/normalized parameters.
- **Graph modeling**: nodes (Patent, Step, Parameter, Dimension, Cluster), relationships (HAS_STEP, NEXT, HAS_PARAM, DERIVED_FROM, COMPOSED_OF, IN_CLUSTER, SAME_AS), constraints and indexes per Neo4j best practices.

Refer to \`docs/data-profile.md\` for detailed input schema patterns and normalization heuristics.

---

## User Stories

### Epic 1: Stage 1 — Step Identification

**US-1.1**: As a researcher, I want to extract structured Steps from \`technical_process\` fields in patent JSON files, so I can see an ordered sequence of operations for each patent.

- **Acceptance Criteria**:
  - Split \`technical_process\` by \`->\` delimiter; assign \`order_index\`.
  - Create \`Patent\` and \`Step\` nodes in Neo4j with properties: \`patent_id\`, \`step.id\`, \`stage: "identification"\`, \`text_raw\`, \`run_id\`.
  - Establish \`[:HAS_STEP]\` and \`[:NEXT]\` relationships.
  - Visualization in Neo4j shows patent-internal step sequences.

**US-1.2**: As a researcher, I want to extract parameters (temperature, pressure, time, RPM, etc.) and link them to Steps, so I can analyze operating conditions.

- **Acceptance Criteria**:
  - Parse \`technical_parameters\` object; create \`Parameter\` nodes with \`param_name\`, \`raw_value\`, \`raw_unit\`, \`has_value\`.
  - Link parameters to Steps using LLM-inferred associations when explicit links are not present (Q4=B). Implementations should fall back to evidence-based linking (see below) when confidence is low.
  - Establish \`[:HAS_PARAM]\` relationships with \`source_field\`, \`evidence\` (text snippet), and \`link_confidence\` (0-1).
  - Handle 0/empty/"null" as unknown per constitution.

Rationale: LLM-inferred linking improves coverage where the dataset lacks explicit parameter→step associations; all inferred links must include a confidence score and the textual evidence used.

**US-1.3**: As a developer, I want the Stage 1 pipeline to run as a CLI command and persist results to Neo4j, so I can inspect and confirm outputs before proceeding.

- **Acceptance Criteria**:
  - CLI: \`python -m step_graph identify --input <json_dir> --output <artifacts_dir> --run-id <uuid> --neo4j-uri <uri>\`
  - Logs: structured JSON with input hash, prompt version, model, timing.
  - Cache: keyed by \`{input_hash, prompt_version, model}\`.
  - User confirmation step before final Neo4j commit.

### Epic 2: Stage 2 — Step Encoding/Characterization

**US-2.1**: As a researcher, I want each Step characterized along three dimensions (energy, matter, information transformations) with tensor-like representations, so I can compare steps semantically.

- **Acceptance Criteria**:
  - LLM + schema-constrained extraction produces transformation encodings (e.g., \`energy_from_form→to_form\`, \`matter_transformation\`, \`info_transformation\`).
  - Create optional \`Dimension\` nodes (e.g., \`energy:electrical->thermal\`, \`matter:liquid_heating\`).
  - Augment \`Step\` nodes with \`encoding_vector\` or \`transformation_tags\`.
  - Establish \`[:HAS_DIMENSION]\` relationships.
  - Neo4j visualization shows Steps color-coded by dominant transformation type.

**US-2.2**: As a developer, I want Stage 2 outputs tagged with \`stage: "encoding"\` and linked to Stage 1 Steps, so I can compare identification vs. encoding results.

- **Acceptance Criteria**:
  - Stage 2 updates existing \`Step\` nodes or creates new property fields.
  - All nodes/relationships tagged with same \`run_id\` as Stage 1.
  - Cypher query template provided to filter by \`stage\`.

### Epic 3: Stage 3 — Step Atomicization

**US-3.1**: As a researcher, I want composite Steps split into atomic, indivisible units while preserving provenance, so I can achieve consistent granularity for clustering.

- **Acceptance Criteria**:
  - LLM identifies multi-dimensional or compound Steps; splits them into atomic units.
  - New \`Step\` nodes tagged \`stage: "atomicized"\`.
  - \`[:COMPOSED_OF]\` relationships link atomized Steps to original composite Steps.
  - \`[:DERIVED_FROM]\` relationships trace back to \`Patent\`.
  - Preserve \`order_index\` and serial/parallel relationships via \`[:NEXT]\`.

**US-3.2**: As a developer, I want atomicization logged with split rationale, so I can audit and refine splitting rules.

- **Acceptance Criteria**:
  - \`[:COMPOSED_OF]\` includes \`split_reason\` property.
  - Logs record before/after Step counts and LLM prompt used.

### Epic 4: Stage 4 — Step Clustering/Canonicalization

**US-4.1**: As a researcher, I want similar Steps across patents clustered into reusable canonical groups, so I can identify common process patterns.

- **Acceptance Criteria**:
  - LLM + similarity scoring identifies equivalent Steps.
  - Create \`Cluster\` or \`CanonicalStep\` nodes with \`cluster_id\`, \`cluster_method\`, \`run_id\`.
  - Establish \`[:IN_CLUSTER]\` or \`[:SAME_AS]\` relationships with optional \`similarity_score\` property.
  - Neo4j visualization shows cross-patent unified graph with clusters highlighted.

**US-4.2**: As a developer, I want clustering thresholds and methods configurable via CLI flags, so I can experiment with different strategies.

- **Acceptance Criteria**:
  - CLI: \`python -m step_graph cluster --method <llm|embedding|hybrid> --threshold <float> --run-id <uuid>\`
  - Log clustering parameters and metrics (e.g., cluster count, avg similarity).

### Epic 5: Multi-Stage Visualization and Rollback

**US-5.1**: As a researcher, I want to visualize all four stages side-by-side in Neo4j Browser, so I can compare and debug pipeline outputs.

- **Acceptance Criteria**:
  - All nodes/relationships tagged with \`run_id\` and \`stage\`.
  - Cypher templates provided: \`MATCH (s:Step {run_id: $runId, stage: $stage}) RETURN s\`.
  - Documentation includes sample queries for each stage.

**US-5.2**: As a developer, I want to rollback a specific run by \`run_id\`, so I can clean up failed or experimental runs.

- **Acceptance Criteria**:
  - CLI: \`python -m step_graph rollback --run-id <uuid>\`
  - Executes: \`MATCH (n {run_id: $runId}) DETACH DELETE n\`.
  - Logs count of deleted nodes/relationships.

---

## Functional Requirements

### FR-1: Input Parsing (Two-Lane)

- **FR-1.1**: Support Lane A (semi-structured JSON) with fields: \`technical_process\`, \`technical_parameters\`, \`innovation_object\`, \`application_field\`, etc. (see \`docs/data-profile.md\`).
- **FR-1.2**: Support Lane B (raw text fallback) via LLM-based extraction under schema constraints.

Note (Lane B trigger):

- Default behavior: prefer Lane A when available. Lane B is invoked when the user passes `--force-lane-b` (Q3=C) or when pre-processing explicitly flags the input as unsalvageable for Lane A. Implementations SHOULD log when Lane B is used.

Rationale: explicit `--force-lane-b` avoids accidental fallback and keeps behavior deterministic.
- **FR-1.3**: Normalize parameters: map 0/empty/"null" to unknown; interpret \`value:"1"/"0"\` as boolean \`has_value\`.
- **FR-1.4**: Split list-like fields (e.g., \`application_field\`) by Chinese/English separators; tag with \`lang\`.

### FR-2: Step Identification (Stage 1)

- **FR-2.1**: Split \`technical_process\` by \`->\` into ordered Steps; assign \`id\`, \`order_index\`, \`text_raw\`, \`source_field\`, \`run_id\`.

Note (Step ID strategy):

- Chosen strategy: UUID v4 for Step IDs (Q2=A). Each Step created in any stage receives a new UUID. Implementations MUST persist a mapping table that links IDs across stages (e.g., original_step_id -> atomicized_step_ids) so provenance and cross-stage traceability are preserved.

Rationale: UUIDs avoid accidental collisions and simplify idempotency; mapping preserves lineage between versions/stages.
- **FR-2.2**: Extract parameters from \`technical_parameters\`; create dual-track properties (raw + normalized).
- **FR-2.3**: Write \`Patent\`, \`Step\`, \`Parameter\` nodes to Neo4j; establish \`[:HAS_STEP]\`, \`[:NEXT]\`, \`[:HAS_PARAM]\`, \`[:DERIVED_FROM]\`.
- **FR-2.4**: Tag all Stage 1 nodes with \`stage: "identification"\`.

### FR-3: Step Encoding (Stage 2)

- **FR-3.1**: LLM characterizes Steps along three dimensions (energy, matter, information transformations); outputs structured JSON conforming to schema.
- **FR-3.2**: Create/update \`Step\` nodes with \`encoding_vector\`, \`transformation_tags\`.
- **FR-3.3**: Optionally create \`Dimension\` nodes; establish \`[:HAS_DIMENSION]\`.
- **FR-3.4**: Tag augmented nodes/relationships with \`stage: "encoding"\`.

Note (Dimension encoding schema):

- Chosen schema: structured object per-step (Q1=B). Each Step encoding MUST include a structured object for the three dimensions. Example:

```json
{
  "dimensions": {
    "energy": {"type": "mechanical->thermal", "magnitude": 0.73},
    "matter": {"type": "mass_transfer", "magnitude": 0.12},
    "information": {"type": "control_signal", "magnitude": 0.05}
  },
  "encoding_vector": [/* optional embedding vector */]
}
```

Rationale: `magnitude` is a normalized float (0-1) and must be produced by a consistent scoring/aggregation method; design and validation of the magnitude scoring rules is part of Stage 2 implementation.

### FR-4: Step Atomicization (Stage 3)

- **FR-4.1**: LLM identifies composite Steps; splits under evidence constraints.

Note (Atomicization criteria):

- Atomicization decision rules combine heuristics (Q5 = A+B+C combined with C as last-resort authority):
  1. Prefer automatic rules: split when the Step description clearly contains multiple dimension transformations (single-dimension preferred) or multiple explicit verbs/actions (A/B).
  2. Use pattern-based rules (verb+object heuristics) and LLM suggestions to propose splits.
  3. If automatic heuristics disagree or confidence is low, escalate to human-in-the-loop review; human judgement (C) is the final authority and should be used sparingly.

Rationale: automatic heuristics minimize manual work while enabling a clear escalation path when ambiguity is high.
- **FR-4.2**: Create new \`Step\` nodes tagged \`stage: "atomicized"\`; link via \`[:COMPOSED_OF]\` with \`split_reason\`.
- **FR-4.3**: Preserve serial/parallel order via \`[:NEXT]\` relationships.

### FR-5: Step Clustering (Stage 4)

- **FR-5.1**: Compute similarity between atomicized Steps (LLM-based or embedding-based). Default threshold = 0.85 (configurable via CLI `--threshold`).
- **FR-5.2**: Create \`Cluster\` nodes; establish \`[:IN_CLUSTER]\` or \`[:SAME_AS]\` with \`similarity_score\`.
- **FR-5.3**: Tag with \`stage: "clustered"\`.

### FR-6: Neo4j Graph Model

- **FR-6.1**: Implement uniqueness constraints: \`patent_id\`, \`step.id\`, \`cluster_id\`.
- **FR-6.2**: Create indexes on: \`run_id\`, \`stage\`, \`order_index\`, \`param_name\`, \`cluster_method\`.
- **FR-6.3**: Use \`MERGE\` for idempotent writes; batch 500–10K nodes/relationships per transaction.
- **FR-6.4**: Parameterized queries only (prevent Cypher injection).
- **FR-6.5**: Avoid >1KB strings as direct properties; use references or chunking.

### FR-7: LLM Integration

- **FR-7.1**: CLI-first invocation (e.g., \`openai\`, \`anthropic\` CLIs or local wrappers).
- **FR-7.2**: Structured JSON logs: \`{run_id, stage, agent, model, prompt_id, input_hash, output_summary, cache_hit, cost_estimate, timestamp}\`.
- **FR-7.3**: Cache keyed by \`{input_hash, prompt_version, model}\`; store as JSONL.

Cache policy (invalidation & TTL):

- Chosen policy (assistant decision per Q8): version-based invalidation with optional TTL. Cache entries are invalidated when `prompt_version` changes. Additionally, cache entries older than 30 days SHOULD be expired. A manual `--clear-cache` CLI flag MUST be provided to force invalidation. Implementations MAY use size-based LRU eviction as an optimization.

Rationale: version-based invalidation ensures reproducibility when prompts change; TTL limits stale entries while still benefiting from caching.
- **FR-7.4**: Schema-constrained generation; validation via regex/unit/range checks; fallback to "unknown + evidence" on failure.

### FR-8: Modular CLI Workflow

- **FR-8.1**: Each stage exposes CLI subcommand: \`identify\`, \`encode\`, \`atomize\`, \`cluster\`.
- **FR-8.2**: Flags: \`--input\`, \`--output\`, \`--run-id\`, \`--neo4j-uri\`, \`--model\`, \`--temperature\`, \`--cache\`.
- **FR-8.3**: User confirmation before final Neo4j write (optional \`--yes\` to skip).

Confirmation UX:

- Chosen UX (Q7=A): CLI pause that prints a concise summary (counts of patents, steps created/updated, parameters linked, estimated LLM calls and cost) and prompts `[Y/n]` to proceed. Include a `--preview` flag to save the summary to `<output_dir>/<run_id>/preview.txt` for offline review.

Rationale: simple, scriptable, and familiar for terminal-first workflows.
- **FR-8.4**: Artifacts persisted to \`<output_dir>/<run_id>/<stage>/\` with versioned outputs.

### FR-9: Rollback and Versioning

- **FR-9.1**: CLI: \`rollback --run-id <uuid>\`.
- **FR-9.2**: Log deletion counts and provide undo warning.

---

## Non-Functional Requirements

### NFR-1: Performance

- **NFR-1.1**: Stage 1 processes 100 patents (avg 10 steps each) in <5 min on standard laptop (excluding LLM latency).
- **NFR-1.2**: Neo4j batch writes: 500–10K nodes/relationships per transaction; monitor memory.
- **NFR-1.3**: Indexed queries return results in <1s for typical filters (\`run_id\`, \`stage\`).

### NFR-2: Reliability

- **NFR-2.1**: Idempotent writes (MERGE); re-running same \`run_id\` updates rather than duplicates.
- **NFR-2.2**: Graceful LLM failure handling: log error, skip step, continue with partial results (user decides to retry or abort).

Error handling policy (partial failures):

- Chosen policy (Q9=A): If Stage 1 processing of a patent fails (malformed input or unrecoverable error), skip that patent entirely and continue processing others. Stages 2–4 operate on the successfully processed patents only. For transient LLM or network errors, implement retries (3 attempts with exponential backoff) before marking the step as failed and skipping.

Rationale: skipping failed patents avoids blocking batch runs while preserving visibility into failures for later remediation.

### NFR-3: Maintainability

- **NFR-3.1**: Modular Python packages: \`parsers/\`, \`llm/\`, \`neo4j/\`, \`cli/\`.
- **NFR-3.2**: Unit tests for parsers and normalizers (>80% coverage for core logic).
- **NFR-3.3**: Integration tests for end-to-end pipeline with mock LLM and test Neo4j instance.

### NFR-4: Security

- **NFR-4.1**: Parameterized Cypher queries only (no string interpolation).
- **NFR-4.2**: Credentials via environment variables or \`.env\` file (not hardcoded).
- **NFR-4.3**: Input validation: reject malformed JSON, out-of-range parameters.

### NFR-5: Observability

- **NFR-5.1**: Structured JSONL logs for all stages and LLM calls.
- **NFR-5.2**: Log rotation and retention policy (e.g., 30 days).
- **NFR-5.3**: Optional Prometheus metrics export (node/relationship counts, LLM latency).

---

## Data Model (Neo4j)

### Node Labels and Properties

- **Patent**
  - Properties: \`patent_id\` (unique, string), \`source_file\` (string), \`lang\` (string), \`filing_date\` (date), \`run_id\` (string)
  - Constraint: \`CREATE CONSTRAINT patent_id_unique FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE\`
  - Index: \`CREATE INDEX patent_run_id_idx FOR (p:Patent) ON (p.run_id)\`

- **Step**
  - Properties: \`id\` (unique, string), \`stage\` (enum: identification/encoding/atomicized/clustered), \`order_index\` (int), \`text_raw\` (string), \`source_field\` (string), \`run_id\` (string), \`encoding_vector\` (optional list), \`transformation_tags\` (optional object)
  - Constraint: \`CREATE CONSTRAINT step_id_unique FOR (s:Step) REQUIRE s.id IS UNIQUE\`
  - Indexes:
    - \`CREATE INDEX step_run_id_idx FOR (s:Step) ON (s.run_id)\`
    - \`CREATE INDEX step_stage_idx FOR (s:Step) ON (s.stage)\`
    - \`CREATE INDEX step_order_idx FOR (s:Step) ON (s.order_index)\`

- **Parameter**
  - Properties: \`param_name\` (string), \`raw_value\` (string), \`raw_unit\` (string), \`normalized_value\` (float/null), \`unit_std\` (string), \`normalization_version\` (string), \`confidence\` (float 0-1), \`has_value\` (boolean), \`run_id\` (string)
  - Index: \`CREATE INDEX param_name_idx FOR (p:Parameter) ON (p.param_name)\`

- **Dimension** (optional)
  - Properties: \`dimension_name\` (string), \`taxonomy_version\` (string), \`run_id\` (string)

- **Cluster**
  - Properties: \`cluster_id\` (unique, string), \`cluster_method\` (string), \`run_id\` (string)
  - Constraint: \`CREATE CONSTRAINT cluster_id_unique FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE\`
  - Index: \`CREATE INDEX cluster_method_idx FOR (c:Cluster) ON (c.cluster_method)\`

### Relationship Types and Properties

- **[:HAS_STEP]**: Patent → Step
  - Properties: \`order\` (int, optional)

- **[:NEXT]**: Step → Step (intra-document sequence)
  - Properties: \`order_delta\` (int, optional), \`parallel\` (boolean, default false)

- **[:HAS_PARAM]**: Step → Parameter
  - Properties: \`source_field\` (string), \`evidence\` (string, snippet from text)

- **[:DERIVED_FROM]**: Step → Patent
  - Properties: \`source_field\` (string, e.g., "technical_process")

- **[:COMPOSED_OF]**: Step → Step (atomicized → original composite)
  - Properties: \`split_reason\` (string)

- **[:HAS_DIMENSION]**: Step → Dimension (optional)

- **[:IN_CLUSTER]**: Step → Cluster

- **[:SAME_AS]**: Step ↔ Step (equivalence/canonicalization)
  - Properties: \`similarity_score\` (float 0-1)

---

## I/O Contracts (Principle-Level)

### Stage 1: Identification

- **Input**: Directory of JSON files conforming to Lane A schema (see \`docs/data-profile.md\`).
- **Output**: 
  - \`<output_dir>/<run_id>/stage1/steps.jsonl\` (one Step per line with metadata)
  - \`<output_dir>/<run_id>/stage1/parameters.jsonl\`
  - Neo4j: Patent, Step, Parameter nodes with relationships

### Stage 2: Encoding

- **Input**: Stage 1 artifacts + Neo4j Step nodes
- **Output**:
  - \`<output_dir>/<run_id>/stage2/encodings.jsonl\` (Step ID + dimension vectors)
  - Neo4j: Updated Step nodes with encoding properties; optional Dimension nodes

### Stage 3: Atomicization

- **Input**: Stage 2 artifacts + Neo4j Step nodes
- **Output**:
  - \`<output_dir>/<run_id>/stage3/atomized_steps.jsonl\`
  - Neo4j: New atomicized Step nodes + COMPOSED_OF relationships

### Stage 4: Clustering

- **Input**: Stage 3 artifacts + Neo4j atomicized Step nodes
- **Output**:
  - \`<output_dir>/<run_id>/stage4/clusters.jsonl\` (Cluster ID + member Step IDs + similarity matrix)
  - Neo4j: Cluster nodes + IN_CLUSTER/SAME_AS relationships

Note (Cross-stage source of truth):

- Chosen coordination mode (Q10=C): dual-write. Stages MUST write both the JSONL artifacts (for reproducibility, auditing and re-run) and the Neo4j graph (for querying and visualization). The system treats Neo4j as the primary queryable graph and JSONL artifacts as the reproducible provenance/log store; either can be used to rebuild the other if needed.

Rationale: dual-write provides fast exploration via Neo4j while keeping an auditable, re-playable JSONL record.

---

## Success Criteria

1. **End-to-End Pipeline**: Complete run from raw JSON to Stage 4 Neo4j visualization without manual intervention (excluding confirmation steps).
2. **Stage-wise Visualization**: All four stage visualizations accessible in Neo4j Browser with clear differentiation by \`run_id\` and \`stage\`.
3. **Parameter Extraction**: >90% of explicit parameters in \`technical_parameters\` correctly parsed and linked to Steps.
4. **Atomicization Quality**: Manual review of 20-patent sample shows >85% of composite Steps correctly split.
5. **Clustering Coherence**: Manual review confirms >80% of clustered Steps are semantically similar (domain expert validation).
6. **Performance**: Stage 1 processes 100 patents in <5 min (excluding LLM latency).
7. **Rollback**: Successful rollback of a \`run_id\` with zero residual nodes/relationships.

---

## Open Questions

1. **LLM Model Selection**: Which model(s) for each stage (GPT-4, Claude 3, domain-specific fine-tuned)?
2. **Multi-language Support**: How to handle mixed Chinese/English text in \`technical_process\`?
3. **Parameter Normalization Rules Versioning**: Who approves updates to \`normalization_rules_version\`?
4. **Visualization Tool**: Neo4j Browser sufficient or need custom UI (Bloom, GraphXR)?

Notes: The following Open Questions were resolved by stakeholder clarifications and encoded into this spec:

- Encoding Dimension Taxonomy (was #2) — resolved: structured dimension object with `magnitude` field (see FR-3 note).
- Atomicization Guardrails (was #3) — resolved: combined heuristics + human-in-the-loop escalation (see FR-4 note).
- Clustering Method (was #4) — resolved: default threshold and method configurable; default threshold set to 0.85 (see FR-5).

---

## Assumptions and Constraints

### Assumptions

- Users have access to a Neo4j instance (local or cloud).
- Input JSON files are well-formed and conform to Lane A schema (see \`docs/data-profile.md\`).
- LLM APIs (OpenAI, Anthropic) are accessible and rate-limited appropriately.
- VS Code with GitHub Copilot or Codex available for interactive development.

### Constraints

- Python 3.12+ required.
- Neo4j 5.x required for constraint syntax and indexing features.
- LLM quota limits may throttle processing; caching mitigates repeat calls.
- Initial focus: battery/electrochemistry domain patents (can generalize later).

---

## References

- **Constitution (EN)**: \`memory/constitution.en.md\`
- **Constitution (ZH)**: \`memory/constitution.md\`
- **Data Profile (EN)**: \`docs/data-profile.md\`
- **Data Profile (ZH)**: \`docs/data-profile.zh-CN.md\`
- **Neo4j Data Modeling**: https://neo4j.com/docs/getting-started/data-modeling/
- **Cypher Manual**: https://neo4j.com/docs/cypher-manual/current/
- **Neo4j Constraints**: https://neo4j.com/docs/cypher-manual/current/constraints/
- **GraphAcademy**: https://graphacademy.neo4j.com/courses/modeling-fundamentals/

---

## Appendix: Sample Cypher Queries

### Query Steps by Stage

\`\`\`cypher
// Stage 1: Identification
MATCH (p:Patent)-[:HAS_STEP]->(s:Step {stage: "identification"})
WHERE s.run_id = $runId
RETURN p.patent_id, s.id, s.order_index, s.text_raw
ORDER BY s.order_index

// Stage 2: Encoding
MATCH (s:Step {stage: "encoding", run_id: $runId})-[:HAS_DIMENSION]->(d:Dimension)
RETURN s.id, s.transformation_tags, collect(d.dimension_name) AS dimensions

// Stage 3: Atomicization
MATCH (s_atom:Step {stage: "atomicized", run_id: $runId})-[:COMPOSED_OF]->(s_orig:Step)
RETURN s_atom.id, s_orig.id, s_atom.text_raw

// Stage 4: Clustering
MATCH (s:Step)-[:IN_CLUSTER]->(c:Cluster {run_id: $runId})
RETURN c.cluster_id, c.cluster_method, collect(s.id) AS members
\`\`\`

### Rollback by Run ID

\`\`\`cypher
MATCH (n {run_id: $runId})
DETACH DELETE n
\`\`\`

### Count Nodes by Stage

\`\`\`cypher
MATCH (s:Step {run_id: $runId})
RETURN s.stage, count(s) AS step_count
ORDER BY s.stage
\`\`\`

---

**End of Specification**
