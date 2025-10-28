# Implementation Plan: Patent Step Graph Extraction and Visualization

**Branch**: `001-step-graph` | **Date**: 2025-10-28 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `.specify/specs/001-step-graph/spec.md`

---

## Summary

Build a modular Python pipeline that extracts, characterizes, atomizes, and clusters technical "Steps" from patent JSON data, persisting results to JSONL artifacts with **optional** Neo4j visualization. The system uses **human-in-the-loop Copilot/Codex-driven workflow** for LLM interactions (avoiding programmatic API calls), supporting both semi-structured JSON (Lane A) and raw text fallback (Lane B).

**Key Technical Approach**:
- **Stage 1 (Identification)**: Parse patent JSON to extract Steps, Parameters, and relationships; write to JSONL + (optional) Neo4j
- **Stage 2 (Encoding)**: Characterize Steps along three **transformation-focused** dimensions (energy form conversion, matter form conversion, information form conversion) using Copilot Chat assistance
- **Stage 3 (Atomicization)**: Split composite Steps into atomic units with Copilot suggestions; track provenance in JSONL
- **Stage 4 (Clustering)**: Aggregate similar Steps across patents using manual curation + Copilot analysis

**Neo4j Visualization**: Optional at each stage (skip if network/capacity issues arise). JSONL artifacts are source of truth and can be imported to Neo4j later for validation/exploration.

---

## Technical Context

**Language/Version**: Python 3.12+ (current environment: Python 3.12.10)  
**Primary Dependencies**: 
- **Neo4j Python Driver** (5.x): Graph database integration (optional - for visualization)
- **Click** (8.3.x): CLI framework
- **Pydantic** (v2, optional): Data validation if needed
- **GitHub Copilot / Codex**: LLM assistance via IDE integration (no API calls)

**Storage**: 
- **JSONL files**: Primary source of truth for all stages (`<output_dir>/<run_id>/<stage>/`)
- **Neo4j 5.x** (optional): Graph visualization when available; can be rebuilt from JSONL
- **No LLM cache needed**: Copilot/Codex handles context internally

**Testing**: 
- **pytest**: Basic unit testing for parsers and utilities
- **Manual validation**: Human review of Stage 2-4 outputs with domain expertise
- **Coverage target**: >60% for core parsing logic (prioritize quality over automation)

**Target Platform**: 
- **Linux/macOS/Windows**: Cross-platform CLI
- **VS Code Terminal**: Primary development workflow
- **Docker**: Optional Neo4j containerization

**Project Type**: Single Python project with modular CLI subcommands

**Performance Goals**: 
- Stage 1 processes 10-50 patents manually with human review (~30-40 min per patent)
- JSONL file writes are instant (<1s per stage)
- Neo4j imports (when used) handle 100-1000 nodes per batch efficiently
- Quality over speed: Manual curation ensures >85% accuracy

**Constraints**: 
- **No LLM API quota**: Using Copilot/Codex eliminates quota management
- **Human time**: Manual curation is bottleneck (~30-40 min/patent for Stages 2-4)
- **Neo4j optional**: System must work without Neo4j; JSONL is authoritative
- **Offline capability**: Stage 1 works offline; Stages 2-4 require Copilot (online IDE)

**Scale/Scope**: 
- **Initial dataset**: 10-20 patents (battery/electrochemistry domain) for MVP validation
- **Growth target**: 50-100 patents with refined workflow
- **Graph size estimate**: 500-5000 nodes (if using Neo4j visualization)
- **Concurrent runs**: Multiple `run_id` versions maintained in separate JSONL directories

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

✅ **Two-lane input contract**: Lane A (semi-structured JSON) supported; Lane B (raw text) designed with fallback  
✅ **Stage-wise artifact generation**: All four stages write to JSONL; Neo4j visualization is optional  
✅ **Copilot/Codex-driven workflow**: Human-in-the-loop LLM interaction via IDE tools (no API calls)  
✅ **Transformation-focused dimensions**: Energy/matter/information characterized by **form conversion** (A→B transitions)  
✅ **Robust parsing**: 0/empty/"null" handled as unknown; dual-track raw/normalized parameters  
✅ **Graph modeling**: Nodes (Patent, Step, Parameter, Cluster) with JSONL-first, Neo4j-optional approach  
✅ **Modular architecture**: `parsers/`, `neo4j/`, `cli/`, `utils/` package structure (no `llm/` module)  
✅ **Parameterized queries**: All Cypher queries use `$param` syntax when Neo4j is used  
✅ **Versioning/rollback**: `run_id` tagging in JSONL filenames enables multi-version tracking  
✅ **Provenance tracking**: All transformations preserve `source_field`, `evidence`, `split_reason` in JSONL  

⚠️ **Neo4j dependency**: Visualization requires Neo4j availability; system gracefully degrades to JSONL-only mode  
⚠️ **Manual curation overhead**: Human-in-the-loop workflow limits throughput to 10-50 patents (trade-off for quality)  
⚠️ **Domain specificity**: Initial focus on battery/electrochemistry patents; generalization requires taxonomy expansion

---

## Project Structure

### Documentation (this feature)

```text
.specify/specs/001-step-graph/
├── spec.md                      # Feature specification (English canonical)
├── spec.zh-CN.md                # Feature specification (Chinese counterpart)
├── plan.md                      # This file - implementation plan
├── research.md                  # Phase 0: technology research and validation
├── data-model.md                # Phase 1: JSONL schema and Neo4j graph model (optional)
├── quickstart.md                # Phase 1: setup and human-in-the-loop workflow guide
├── copilot-prompts/             # Phase 1: Copilot Chat templates for Stages 2-4
│   ├── stage2-transformation-encoding.md  # Energy/matter/information form conversion prompts
│   ├── stage3-atomization.md              # Composite step identification prompts
│   ├── stage4-clustering.md               # Similarity analysis prompts
│   └── transformation-examples.md         # Few-shot examples for battery domain
├── contracts/                   # Phase 1: JSONL format documentation (markdown)
│   ├── lane-a-format.md         # Input patent JSON structure
│   ├── stage1-output-format.md  # JSONL format for identification stage
│   ├── stage2-output-format.md  # JSONL format for encoding stage
│   ├── stage3-output-format.md  # JSONL format for atomicization stage
│   └── stage4-output-format.md  # JSONL format for clustering stage
└── tasks.md                     # Phase 2: implementation task breakdown (NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
step_graph/                      # Main Python package
├── __init__.py
├── __main__.py                  # CLI entry point (python -m step_graph)
│
├── parsers/                     # Input parsing (Lane A/B)
│   ├── __init__.py
│   ├── lane_a.py                # Semi-structured JSON parser
│   ├── lane_b.py                # Raw text parser (future)
│   ├── normalizers.py           # Parameter normalization (0/null/empty handling)
│   └── schemas.py               # Pydantic models for validation (optional)
│
├── neo4j/                       # Neo4j operations (optional - for visualization)
│   ├── __init__.py
│   ├── client.py                # Neo4j driver wrapper
│   ├── models.py                # Graph node/relationship definitions
│   ├── importers.py             # JSONL → Neo4j batch import (MERGE-based)
│   └── queries.py               # Cypher query templates for visualization
│
├── cli/                         # CLI commands
│   ├── __init__.py
│   ├── identify.py              # `step_graph identify` - Stage 1 (automated parsing)
│   ├── import_neo4j.py          # `step_graph import-neo4j` - Import JSONL to Neo4j (optional)
│   ├── export.py                # `step_graph export` - Export JSONL to CSV/TXT
│   └── rollback.py              # `step_graph rollback` - Delete data by run_id
│
└── utils/                       # Shared utilities
    ├── __init__.py
    ├── ids.py                   # UUID generation
    ├── config.py                # Configuration management (.env support)
    └── artifacts.py             # JSONL file read/write operations

tests/                           # Test suite (simplified)
├── conftest.py                  # pytest fixtures
├── unit/                        # Unit tests
│   ├── test_parsers.py
│   ├── test_normalizers.py
│   ├── test_llm_cache.py
│   ├── test_llm_prompts.py
│   └── test_neo4j_writers.py
├── integration/                 # Integration tests
│   ├── test_stage1_identify.py
│   ├── test_stage2_encode.py
│   ├── test_stage3_atomize.py
│   ├── test_stage4_cluster.py
│   └── test_end_to_end.py
└── fixtures/                    # Test data
    ├── sample_patents.json      # Sample Lane A input
    ├── mock_llm_responses.json  # Cached LLM outputs for testing
    └── expected_neo4j_state.cypher # Expected graph state snapshots

pyproject.toml                   # Project configuration (Poetry/Hatch)
.env.example                     # Example environment variables
README.md                        # Project README with setup instructions
```

**Structure Decision**: 
Selected **single Python project** structure with modular packages for parsers, LLM integration, Neo4j operations, and CLI commands. This design:
- Separates concerns by layer (parsing → LLM → graph → CLI)
- Enables independent testing of each module
- Supports future expansion (e.g., Lane B parser, additional stages)
- Aligns with constitution's requirement for CLI-first architecture
- Facilitates packaging as a distributable CLI tool (`pip install step-graph`)

---

## Complexity Tracking

> **No constitution violations requiring justification**

The implementation adheres to constitution principles with necessary adjustments for human-in-the-loop workflow:
- Modular architecture with clear separation (parsers, neo4j optional, cli, utils)
- CLI-first design with human-mediated Copilot interaction for Stages 2-4
- JSONL-based artifact persistence (source of truth)
- Neo4j visualization is optional; graceful degradation when unavailable
- Transformation-focused dimension taxonomy aligns with domain requirements
- No complex LLM API orchestration needed (Copilot handles this)

---

## Phase 0: Research and Validation

**Objective**: Validate technical feasibility of key components before detailed design.

### Research Tasks

1. **Neo4j Driver and Optional Visualization**
   - Validate Neo4j Python driver (5.x) for batch imports when available
   - Test graceful degradation when Neo4j is unavailable (network/capacity issues)
   - Document JSONL→Neo4j import workflow for later visualization
   - Measure batch import performance with 100-1000 nodes
   - **Deliverable**: `research.md` section on optional Neo4j setup

2. **Copilot/Codex Workflow Validation**
   - Test GitHub Copilot Chat for transformation-focused dimension characterization
   - Prototype effective prompts for energy/matter/information form conversions
   - Validate Copilot Edits for batch processing similar steps
   - Document best practices for human-in-the-loop curation workflow
   - **Deliverable**: `research.md` section on Copilot workflow + `copilot-prompts/` templates

3. **Input Data Profiling (Focused on 5-10 Sample Patents)**
   - Analyze 5-10 representative patents from `IG_start/data/raw_patent_data/`
   - Document observed `technical_process` and `technical_parameters` field patterns
   - Validate 0/null/empty handling assumptions
   - Identify common transformation types in battery/electrochemistry domain
   - **Deliverable**: Updated section in `research.md` with sample analysis

4. **Dimension Taxonomy Design (Transformation-Focused)**
   - Research domain-specific **transformation** classifications for three dimensions:
     - **Energy**: Energy **form conversions** (thermal→mechanical, electrical→chemical, etc.)
     - **Matter**: Matter **form conversions** (solid→liquid, liquid→gas, mixture→compound, etc.)
     - **Information**: Information **form conversions** (measurement→data, data→control signal, etc.)
   - Prototype JSON schema for dimension encoding with transformation types + magnitude
   - Validate transformation magnitude scoring (0-1 scale: degree of form change)
   - **Deliverable**: `research.md` section on transformation taxonomy + Copilot prompt templates

### Research Deliverable: `research.md`

Structure:
```markdown
# Research: Patent Step Graph Extraction

## Neo4j Setup and Configuration
[Optional visualization setup, batch import patterns]

## Copilot/Codex Workflow Validation
[Testing Copilot Chat for dimension characterization, effective prompts]

## Input Data Validation
[Sample analysis from IG_start/data/, edge case handling]

## Transformation-Focused Dimension Taxonomy
[Energy/matter/information form conversions, magnitude scoring for transformation degree]

## Risks and Mitigation
[Identified risks: Neo4j unavailability, manual curation throughput]

## Version History
[Research iteration log]
```

---

## Phase 1: Detailed Design

**Objective**: Create comprehensive technical specifications for implementation.

### Design Deliverables

#### 1. Data Model (`data-model.md`)

**Contents**:
- **JSONL Artifact Schema (Source of Truth)**
  - Stage 1-4 JSONL file formats with complete field specifications
  - UUID generation strategy for Steps, Parameters, Clusters
  - Provenance tracking fields (`source_field`, `transformation_evidence`)
  - Cross-stage mapping patterns

- **Neo4j Graph Schema (Optional Visualization)**
  - Node labels with essential properties for graph queries
  - Relationship types for visualization
  - Minimal constraints (uniqueness only)
  - Import scripts for JSONL→Neo4j when visualization is available
  - Graceful degradation notes when Neo4j is unavailable

- **Parameter Normalization Rules (Simplified)**
  - Focus on top 5-10 most common parameters (temperature, time, voltage, current, capacity)
  - Basic unit conversion (e.g., °F→°C, mAh→Ah)
  - 0/null/empty handling: Map to "unknown" with flag
  - Manual normalization workflow with Copilot assistance

- **Transformation-Focused Dimension Schema**
  - **Energy**: Form conversion taxonomy (thermal↔mechanical, electrical↔chemical, etc.)
  - **Matter**: Form conversion taxonomy (solid↔liquid, liquid↔gas, mixture→compound, etc.)
  - **Information**: Form conversion taxonomy (raw measurement→processed data, data→control signal, etc.)
  - Magnitude scoring: Degree of transformation (0=no change, 1=complete form conversion)
  - JSON structure for transformation objects with `from_form`, `to_form`, `magnitude`
  - Example characterizations for battery/electrochemistry steps

**Example Section** (Transformation-Focused):
```markdown
### Energy Transformation: Electrical → Chemical (Charging)

**Step example**: "Charge lithium battery at 0.5C to 4.2V"

**Transformation characterization**:
```json
{
  "energy": {
    "from_form": "electrical",
    "to_form": "chemical",
    "magnitude": 0.9,
    "rationale": "Electrical energy is converted to chemical potential in lithium ions"
  },
  "matter": {
    "from_form": "lithium_ion_dispersed",
    "to_form": "lithium_ion_intercalated",
    "magnitude": 0.7,
    "rationale": "Lithium ions migrate and intercalate into electrode structure"
  },
  "information": {
    "from_form": "voltage_measurement",
    "to_form": "charge_state_data",
    "magnitude": 0.5,
    "rationale": "Voltage readings are transformed into battery state-of-charge information"
  }
}
```

**Magnitude interpretation**:
- 0.9 = Near-complete energy form conversion (electrical fully converted to chemical storage)
- 0.7 = Significant matter rearrangement (majority of Li+ ions change position/bonding)
- 0.5 = Moderate information transformation (voltage → derived state metric)
```

#### 2. Quickstart Guide (`quickstart.md`)

**Contents**:
- **Prerequisites**
  - Python 3.12+ installation
  - Neo4j 5.x setup (optional - for visualization only)
  - VS Code with GitHub Copilot or Codex enabled
  - No API keys needed (Copilot handles LLM)

- **Installation**
  - Clone repository
  - Install minimal dependencies: `pip install neo4j click` (no Poetry, no API SDKs)
  - Configure `.env` with Neo4j credentials only (if using visualization)

- **Basic Workflow (Human-in-the-Loop)**
  - Stage 1 (Automated): `python -m step_graph identify --input patents/ --output artifacts/ --run-id $(uuidgen) [--neo4j-uri bolt://localhost:7687]`
  - Stage 2 (Copilot-Assisted): Open JSONL in VS Code, use Copilot Chat to characterize transformations, edit JSONL manually
  - Stage 3 (Copilot-Assisted): Use Copilot Chat to identify composite steps, create atomic sub-steps in JSONL
  - Stage 4 (Manual Clustering): Export steps to text, use Copilot Chat to suggest clusters, create clusters.jsonl manually
  - Optional: Import JSONL to Neo4j for visualization/validation

- **Neo4j Visualization (Optional)**
  - When to skip: Network issues, capacity limits, local-only workflow
  - How to import later: `python -m step_graph import-neo4j --jsonl artifacts/{run_id}/ --neo4j-uri bolt://localhost:7687`
  - Example queries by stage (for when Neo4j is available)

- **Troubleshooting**
  - Working without Neo4j (JSONL-only mode)
  - Copilot prompt refinement tips
  - JSONL validation and syntax checks

#### 3. Copilot Prompt Templates (`copilot-prompts/`)

**Purpose**: Reproducible Copilot Chat prompts for Stages 2-4 (human-in-the-loop workflow)

**Files to create**:

**`stage2-transformation-encoding.md`**: Template for characterizing energy/matter/information transformations
```markdown
# Stage 2: Transformation Encoding Prompt

Use this prompt in Copilot Chat to characterize process steps:

"Analyze this step and characterize the transformations along three dimensions:

Step: '{paste step text}'

Return JSON:
{
  "energy": {
    "from_form": "initial energy form",
    "to_form": "final energy form",
    "magnitude": 0.0-1.0,
    "rationale": "explanation"
  },
  "matter": {...},
  "information": {...}
}

Focus on FORM CONVERSIONS (A→B transitions), not just presence.
Magnitude = degree of transformation (0=no change, 1=complete conversion).
"
```

**`stage3-atomization.md`**: Template for identifying composite steps

**`stage4-clustering.md`**: Template for similarity analysis

**`transformation-examples.md`**: Few-shot examples of common battery/electrochemistry transformations

#### 4. JSONL Format Documentation (`contracts/`)

**Purpose**: Define JSONL artifact schemas (simpler than formal JSON Schema)

**Files to create (markdown format)**:

**`lane-a-format.md`**: Input patent JSON structure with examples

**`stage1-output-format.md`**: JSONL format for Stage 1 (identification)
```markdown
### Stage 1 Output: Identified Steps and Parameters

**File**: `.artifacts/{run_id}/stage1/steps.jsonl`

**Format** (one JSON object per line):
```json
{
  "patent_id": "CN-100342579-C",
  "step_id": "uuid-generated",
  "run_id": "run-uuid",
  "order_index": 1,
  "text_raw": "Heat to 95°C",
  "stage": "identification",
  "parameters": [
    {
      "id": "param-uuid",
      "name": "temperature",
      "value": "95°C",
      "normalized_value": 95.0,
      "unit": "celsius"
    }
  ]
}
```

**`stage2-output-format.md`**: JSONL format for Stage 2 (transformation encoding)
- Same as Stage 1 plus `transformation_tags` field with energy/matter/information conversions

**`stage3-output-format.md`**: JSONL format for Stage 3 (atomicization)
- Atomic sub-steps with `parent_id` field for provenance

**`stage4-output-format.md`**: JSONL format for Stage 4 (clustering)
- Cluster definitions with `member_step_ids` array

---

## Phase 2: Implementation Task Breakdown

**Note**: This phase is executed by the `/speckit.tasks` command, NOT by `/speckit.plan`. The plan provides the foundation, but task decomposition happens separately.

**Expected `tasks.md` structure** (created by `/speckit.tasks`):
- Tasks organized by Epic (Stage 1-4 + Infrastructure)
- Dependencies clearly marked
- Parallel execution opportunities flagged
- Test-driven development (TDD) structure where applicable
- File paths specified for each task
- Checkpoint verification after each Epic

---

## Architecture Decision Records (ADRs)

### ADR-001: UUID v4 for Step IDs Across All Stages

**Context**: Need persistent identifiers for Steps that work across stages while avoiding collisions.

**Decision**: Use UUID v4 for all Step IDs. Each stage creates new UUIDs; maintain a mapping table (in Neo4j or JSONL) linking original_step_id → atomicized_step_ids.

**Rationale**:
- Avoids sequential ID collisions across concurrent runs
- Simplifies idempotency (MERGE operations)
- Mapping table preserves provenance without ID reuse

**Consequences**:
- Additional storage for mapping table (negligible)
- Must implement mapping table persistence
- Cross-stage queries require JOIN-like operations on mapping

**Alternatives Rejected**:
- Reuse IDs across stages: Loses stage differentiation, complicates rollback
- Composite IDs (stage + UUID): More complex, harder to query

---

### ADR-002: Dual-Write to JSONL + Neo4j for Source of Truth

**Context**: Need reproducibility (JSONL artifacts) AND queryability (Neo4j graph).

**Decision**: Dual-write pattern—stages write to both JSONL artifacts AND Neo4j. Either can reconstruct the other if needed.

**Rationale**:
- JSONL provides audit trail, re-run capability, offline analysis
- Neo4j provides fast exploration, graph algorithms, visualization
- Redundancy acceptable given storage is cheap vs. re-processing cost

**Consequences**:
- Slight write overhead (mitigated by async writes)
- Must ensure consistency between JSONL and Neo4j (transaction boundaries)
- JSONL files can grow large (use compression, rotation)

**Alternatives Rejected**:
- Neo4j-only: Loses reproducibility, harder to debug LLM outputs
- JSONL-only: No graph algorithms, poor query performance

---

### ADR-003: CLI-First Architecture with Explicit `--force-lane-b` Flag

**Context**: Need deterministic behavior for Lane A/B fallback to avoid accidental use of raw text parsing.

**Decision**: Default to Lane A (semi-structured JSON). Lane B invoked ONLY via explicit `--force-lane-b` flag or when preprocessor marks input unsalvageable.

**Rationale**:
- Explicit user intent prevents silent fallback surprises
- Logs when Lane B is used for auditing
- Aligns with constitution's "robust parsing" principle

**Consequences**:
- Users must explicitly request Lane B
- Preprocessing step may need to flag inputs as "Lane A incompatible"
- Clear separation of parser codepaths

**Alternatives Rejected**:
- Automatic fallback: Risk of silent data quality degradation
- Lane B as default: Contradicts constitution's Lane A priority

---

### ADR-004: Version-Based Cache Invalidation + 30-Day TTL

**Context**: Need cache to reduce LLM costs, but ensure reproducibility when prompts change.

**Decision**: Hybrid strategy—invalidate cache when `prompt_version` changes (version-based) AND expire entries older than 30 days (TTL-based). Manual `--clear-cache` flag available.

**Rationale**:
- Version-based: Ensures prompt changes always get fresh results
- TTL: Limits stale entries accumulating over time
- Manual flag: Escape hatch for debugging or forced refresh

**Consequences**:
- Cache entries must track `created_at` timestamp
- Prompt versioning discipline required (increment on any change)
- Storage usage bounded by TTL (automatic cleanup)

**Alternatives Rejected**:
- Version-only: Cache grows unbounded
- TTL-only: Stale results if prompts change within TTL window
- No cache: Prohibitive costs for iterative development

---

### ADR-005: Three-Tier Atomicization Decision (Automatic → LLM → Human)

**Context**: Need balance between automation (speed) and accuracy (quality) for splitting composite steps.

**Decision**: Three-tier escalation:
1. **Automatic heuristics**: Split if step contains multiple dimensions OR multiple explicit action verbs
2. **LLM suggestions**: If heuristics ambiguous, invoke LLM with pattern-based rules
3. **Human-in-the-loop**: If LLM confidence <0.7, flag for manual review (used sparingly)

**Rationale**:
- Automatic tier: Fast, deterministic, covers 70%+ of cases
- LLM tier: Handles nuanced cases without manual work
- Human tier: Final authority for edge cases, prevents error propagation

**Consequences**:
- Must implement confidence scoring in LLM outputs
- Manual review queue mechanism needed (future iteration)
- Clear documentation of escalation thresholds

**Alternatives Rejected**:
- LLM-only: Expensive, slower, over-reliance on external API
- Human-only: Not scalable to 10K+ patents
- Automatic-only: Accuracy suffers on complex steps

---

## Phase 3: Implementation Roadmap

**Note**: Detailed tasks generated by `/speckit.tasks` command. This section provides high-level sequencing only.

### Milestone 1: Infrastructure Foundation (Week 1-2)
- Neo4j client with constraint/index creation
- LLM client with caching and logging
- Input parser (Lane A) with normalization
- CLI framework with shared options

### Milestone 2: Stage 1 Implementation (Week 3-4)
- Step identification from `technical_process`
- Parameter extraction and normalization
- Neo4j write operations with idempotency
- JSONL artifact persistence
- Unit + integration tests

### Milestone 3: Stage 2-3 Implementation (Week 5-6)
- Dimension encoding with LLM
- Atomicization with three-tier decision logic
- UUID mapping table implementation
- Cross-stage provenance tracking

### Milestone 4: Stage 4 + End-to-End Validation (Week 7-8)
- Clustering with similarity scoring
- Visualization query templates
- Rollback functionality
- End-to-end pipeline testing (100-patent sample)

---

## Open Questions and Decisions Needed

1. **LLM Model Selection**: Which specific models for each stage?
   - **Recommendation**: GPT-4-turbo for Stages 2-4 (schema adherence), GPT-3.5-turbo for Stage 1 (cost optimization)
   - **Decision needed**: Budget approval for estimated $X per 100 patents

2. **Multi-language Handling**: How to process mixed Chinese/English text in `technical_process`?
   - **Recommendation**: Preserve original language; rely on LLM's multilingual capabilities; add `lang` tag to Steps
   - **Decision needed**: Validate with sample patents containing mixed text

3. **Visualization Tool**: Neo4j Browser sufficient or need custom UI?
   - **Recommendation**: Start with Neo4j Browser + Cypher query templates; defer custom UI to post-v1
   - **Decision needed**: User acceptance testing with researchers

4. **Parameter Dictionary Governance**: Who maintains and approves updates to parameter normalization rules?
   - **Recommendation**: Designated domain expert(s); versioned in `data-model.md`; changelog tracked
   - **Decision needed**: Assign ownership and approval process

---

## Risk Assessment and Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| LLM API rate limits during batch processing | High | High | Implement exponential backoff; batch job scheduling; cache aggressively |
| Neo4j memory exhaustion on large batches | Medium | High | Profile batch sizes; implement adaptive batching; monitor heap usage |
| Low LLM accuracy on dimension encoding | Medium | Medium | Iterate prompts with validation set; collect expert feedback; version prompts |
| 0/null/empty handling edge cases not covered | Medium | Medium | Comprehensive test suite from `data-profile.md`; log unknown cases for review |
| UUID mapping table grows too large | Low | Low | Archive old mappings; compress JSONL; use Neo4j native storage |
| Cross-language text parsing errors | Medium | Low | Validate with multilingual sample set; add language detection fallback |

---

## Success Metrics (from Spec)

1. ✅ **End-to-End Pipeline**: Complete run from JSON to Stage 4 visualization (excluding confirmations)
2. ✅ **Stage-wise Visualization**: All four stages accessible in Neo4j Browser with `run_id`/`stage` filters
3. ✅ **Parameter Extraction**: >90% of explicit parameters correctly parsed and linked
4. ✅ **Atomicization Quality**: >85% of composite steps correctly split (20-patent manual review)
5. ✅ **Clustering Coherence**: >80% of clusters semantically similar (domain expert validation)
6. ✅ **Performance**: Stage 1 processes 100 patents in <5 min (excluding LLM latency)
7. ✅ **Rollback**: Zero residual nodes/relationships after rollback by `run_id`

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0.0 | 2025-10-28 | Initial plan based on spec v0.3.0 and constitution v0.2.0 | AI Assistant |

---

**Next Steps**:
1. Review and approve this plan
2. Execute Phase 0 research tasks → produce `research.md`
3. Execute Phase 1 design tasks → produce `data-model.md`, `quickstart.md`, `contracts/`
4. Run `/speckit.tasks` to generate detailed task breakdown → produce `tasks.md`
5. Begin implementation with Milestone 1
