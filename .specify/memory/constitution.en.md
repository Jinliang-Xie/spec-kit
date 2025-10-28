# Project Constitution

This charter defines the project’s goals and principles. It stays high-level (principles and boundaries) and intentionally avoids prescribing implementation details or specific thresholds. Concrete details (metrics, algorithm choices, field-level schema) live in the Spec, Plan, and ADR documents.

Project theme: Integrate an LLM into a Python workflow to extract and characterize “Steps” (the smallest technically indivisible process units) from patent texts, build a step graph with entities, relations, and parameters, and—via cross-document aggregation—form a reusable, transferable step knowledge graph in Neo4j.

## I. Scope and Staged Deliverables

1) Stage 1 — Step Identification
- Goal: Using LLM plus code, identify and structure steps from pre-extracted patent JSON; capture inputs/outputs, equipment, operation mode, reaction/operation conditions, and environment; infer step sequence and parallel relations at the logical level.
- Deliverable: Write Patent, Step, Step relations, parameters, patent time, and entity–parameter relations into Neo4j; first visualization (per-patent process graph).

2) Stage 2 — Step Encoding (Unified Characterization)
- Goal: Using LLM plus code, consistently map each step across three dimensions:
  - Energy transformation (mechanical, thermal, chemical, electrical, radiation, nuclear—multiple may occur concurrently) with quantitative increases/decreases; can be viewed as an energy-transform tensor.
  - Matter transformation (momentum/heat/mass transfer, chemical reaction, etc.—sub-dimensions may be further refined; multiple may occur concurrently) with quantitative changes; can be viewed as a matter-transform tensor.
  - Information transformation (acquisition, processing—multiple may occur concurrently) with quantitative changes; can be viewed as an information-transform tensor.
- Deliverable: Map dimensions and sub-dimensions into the graph in Neo4j; second visualization (per-patent, after unified characterization).

3) Stage 3 — Step Atomicization (Splitting/Refinement)
- Goal: Under the joint constraints of encoding results and textual evidence, split any step with multi-valued dimensions/sub-dimensions or composite actions/conditions into more basic, uniformly-grained atomic steps that are “indivisible” under current knowledge; preserve and make explicit sequence/parallel relations.
- Deliverable: Atomic Step set and relations in Neo4j (retain provenance to original composite steps); third visualization; provide uniform-granularity input for clustering/canonicalization.

4) Stage 4 — Step Clustering/Canonicalization
- Goal: Using clustering/similarity analysis, merge or mark “SAME_AS” for similar atomic steps to form a reusable, transferable step set with weak dependence on concrete I/O and parameters; fuse identical process nodes.
- Deliverable: Cross-patent aggregated graph in Neo4j; fourth visualization (cross-document summary graph).

Note: The charter does not constrain algorithms or thresholds. Such thresholds and evaluation methods are defined and iterated in the Plan and evaluation documents.

## II. Method and Data Principles

- Modular by design: The workflow must be sufficiently modular to inspect stepwise outputs in the VS Code terminal. With the user’s confirmation, intermediate results are automatically saved into a project-scoped folder for progressive optimization and debugging.
- Truth and provenance: Extract only from the invention’s technical solution and embodiments; exclude background/known art. Record provenance (patent id, paragraph/page, snippet) for every structured item.
- Anti-hallucination: Enforce strict schema validation; treat “unknown/missing” explicitly—do not guess.
- Reproducibility: Record prompt, model, parameter versions; the same input with the same configuration should yield stable output.
- Progressive refinement: First ensure an end-to-end usable pipeline and visualization; then iteratively raise extraction quality, characterization completeness, and aggregation effectiveness.
- LLM integration: Prefer invoking the LLM via CLI from Python (with a unified wrapper, API fallback allowed). Persist prompt, model/params, and I/O snapshots for traceability.
- Parameter policy: Treat temperature, pressure, atmosphere, time, RPM, flow, concentration, etc. as Parameters; maintain explicit links from Steps; preserve units and raw values.

## III. Graph Modeling and Write Principles

- Minimal schema first: Start with only necessary objects and relations to avoid premature complexity.
  - Core nodes: Patent, Step, Dimension (dimension/sub-dimension), Parameter (environment/condition/formulation), Cluster (or CanonicalStep for canonicalization).
  - Core relations: HAS_STEP, PRECEDES (or NEXT), HAS_DIMENSION, HAS_PARAM, DERIVED_FROM/COMPOSED_OF (atomicization provenance), SAME_AS/IN_CLUSTER.
- Stable identifiers: Generate stable, verifiable IDs for core entities like Step (source + text snippet). Use idempotent writes (MERGE) with batch markers.
- Versioning and rollback: Track write batches and graph versions; support rollback and replay by batch/version.

## IV. Quality and Evaluation (High-level)

- Evaluation axes: Extraction correctness/completeness, encoding consistency, aggregation rationality, graph consistency/deduplication, end-to-end usability (visualization and queries).
- Metrics and thresholds: Defined in Plan/Eval docs; tighten progressively with data scale and domain coverage. Must pass quality gates before merge.
- Human review: Maintain a small golden set and sampling review mechanism for regression and anomaly triage.

## V. Engineering and Security (Brief)

- Engineering basics: Unified code style (format/lint), types, clear error layering; small PRs and reviews; auditable deps/config.
- Compliance: Respect patent text usage licenses; inject secrets securely; avoid persisting sensitive data.
- Observability: Baseline logs and minimal metrics to localize failures in extraction/write/aggregation.

## VI. Documentation and Boundaries

- Document split: This charter sets principles/boundaries; Spec states “what to build”; Plan defines technical path, algorithms, and thresholds; ADR records key trade-offs.
- Non-goals (current phase): No productized UI or auth systems; do not hard-code models/algorithms/params in the charter; no field-level details or test cases here.
- Language policy: English is canonical. Provide a Chinese counterpart file named with suffix `.zh-CN.md` for major Markdown artifacts.

— Changes to this charter must go through PRs with rationale and impact; detailed updates should land in Spec/Plan/ADR as appropriate.