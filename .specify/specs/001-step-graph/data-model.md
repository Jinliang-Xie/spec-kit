# Data Model: Patent Step Graph (Simplified for MVP)

**Feature**: 001-step-graph  
**Last Updated**: 2025-10-28  
**Version**: 1.0.0 (Minimal Viable)

---

## Overview

This document defines the **minimal viable data model** for the patent step graph extraction pipeline. Keep it simple - focus on what's essential for 10-20 patents.

**Design Principles**:
1. **JSONL as source of truth**: Neo4j is derivative, can be rebuilt from JSONL
2. **Simple schema**: 4 node types, 4 relationship types - no more
3. **UUID v4 for IDs**: Generate once in Stage 1, persist everywhere
4. **Human-readable artifacts**: Prioritize debuggability over performance

---

## Neo4j Graph Schema

### Node Types

#### 1. Patent

Represents a single patent document.

**Essential Properties**:

| Property | Type | Example |
|----------|------|---------|
| `patent_id` | String | `"CN-100342579-C"` |
| `run_id` | String (UUID) | `"550e8400-e29b-..."` |
| `title` | String | `"锂电池制造方法"` |
| `raw_json` | String | Original JSON (for reference) |

**Constraints**:
```cypher
CREATE CONSTRAINT patent_id_unique IF NOT EXISTS
FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE;
```

---

#### 2. Step

Represents a single process step.

**Essential Properties**:

| Property | Type | Stage Added | Example |
|----------|------|-------------|---------|
| `id` | String (UUID) | 1 | `"a3bb189e-8bf9-..."` |
| `run_id` | String (UUID) | 1 | `"550e8400-e29b-..."` |
| `stage` | String | 1 | `"identification"`, `"encoding"`, `"atomicized"`, `"clustered"` |
| `text_raw` | String | 1 | `"Heat to 95°C"` |
| `order_index` | Integer | 1 | `1`, `2`, `3` |
| `transformation_tags` | JSON Object | 2 | See [Transformation Schema](#transformation-schema-stage-2) |

**Optional Properties** (add only if needed):

| Property | Type | Stage | Description |
|----------|------|-------|-------------|
| `is_composite` | Boolean | 3 | `true` if step was split |
| `parent_id` | String | 3 | UUID of parent composite step |

**Constraints**:
```cypher
CREATE CONSTRAINT step_id_unique IF NOT EXISTS
FOR (s:Step) REQUIRE s.id IS UNIQUE;

CREATE INDEX step_run_id_idx IF NOT EXISTS
FOR (s:Step) ON (s.run_id);
```

---

#### 3. Parameter

Represents a technical parameter (temperature, time, voltage, etc.).

**Essential Properties**:

| Property | Type | Example |
|----------|------|---------|
| `id` | String (UUID) | `"f4a8b3c2-1d5e-..."` |
| `run_id` | String (UUID) | `"550e8400-e29b-..."` |
| `name` | String | `"temperature"` |
| `value` | String | `"95°C"` |
| `normalized_value` | Float (optional) | `95.0` |
| `unit` | String (optional) | `"celsius"` |

**Note**: Start simple - just store `name` and `value`. Add normalization later if you find it useful.

**Constraints**:
```cypher
CREATE CONSTRAINT parameter_id_unique IF NOT EXISTS
FOR (p:Parameter) REQUIRE p.id IS UNIQUE;
```

---

#### 4. Cluster

Represents a group of similar steps (Stage 4).

**Essential Properties**:

| Property | Type | Example |
|----------|------|---------|
| `cluster_id` | String (UUID) | `"c7d8e9f0-1a2b-..."` |
| `run_id` | String (UUID) | `"550e8400-e29b-..."` |
| `cluster_method` | String | `"manual-copilot"` |
| `member_count` | Integer | `5` |

**Constraints**:
```cypher
CREATE CONSTRAINT cluster_id_unique IF NOT EXISTS
FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE;
```

---

### Relationship Types

#### 1. HAS_STEP

`(Patent)-[:HAS_STEP]->(Step)`

**Properties**: None needed for MVP.

---

#### 2. HAS_PARAMETER

`(Step)-[:HAS_PARAMETER]->(Parameter)`

**Properties**: None needed for MVP.

---

#### 3. COMPOSED_OF

`(CompositeStep:Step)-[:COMPOSED_OF]->(AtomicStep:Step)`

**Properties**:

| Property | Type | Example |
|----------|------|---------|
| `order` | Integer | `1`, `2`, `3` |

---

#### 4. IN_CLUSTER

`(Step)-[:IN_CLUSTER]->(Cluster)`

**Properties**: None needed for MVP.

---

## Transformation Schema (Stage 2)

**Introduced in**: Stage 2

Each step is characterized along **3 dimensions**: Energy, Matter, Information. The key innovation is capturing **transformations** (from_form → to_form) rather than just involvement.

### JSON Structure

```json
{
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "thermal",
      "magnitude": 0.85,
      "rationale": "Electrical heating element converts electricity to thermal energy"
    },
    "matter": {
      "from_form": "liquid_ambient_temp",
      "to_form": "liquid_elevated_temp",
      "magnitude": 0.6,
      "rationale": "Liquid mixture heated from room temperature to 95°C"
    },
    "information": {
      "from_form": "temperature_sensor_signal",
      "to_form": "control_feedback",
      "magnitude": 0.4,
      "rationale": "Temperature measurements provide feedback for heating control"
    }
  }
}
```

### Field Details

#### `from_form` and `to_form`

**Purpose**: Capture the actual **conversion** or **transformation** that occurs during this step.

**Common Energy Forms**:
- `electrical`, `thermal`, `mechanical`, `chemical_potential`, `electromagnetic`, `acoustic`, `gravitational`, `none`

**Common Matter Forms**:
- `solid_crystalline`, `solid_amorphous`, `liquid_pure`, `liquid_solution`, `gas`, `dispersed_particles`, `intercalated`, `separated_phases`, `homogeneous_mixture`

**Common Information Forms**:
- `voltage_signal`, `current_signal`, `temperature_reading`, `optical_signal`, `digital_data`, `control_command`, `user_observation`, `calculated_state`

**Note**: Use descriptive forms that capture the actual state change. If no transformation occurs, use `from_form = to_form` with low magnitude.

#### `magnitude`

**Scale** (0.0 to 1.0) - Represents **degree of transformation**, not just involvement:

- `0.0-0.2`: Minimal transformation (state mostly preserved)
- `0.3-0.5`: Partial transformation (some conversion)
- `0.6-0.8`: Significant transformation (substantial change)
- `0.9-1.0`: Near-complete transformation (nearly full conversion)

**Critical**: `magnitude` measures **how much transformation occurs**, not just how much that dimension is involved.

**Example**: A step "maintain temperature at 95°C" might have:
- High energy involvement (heating element running)
- But LOW energy transformation magnitude (just compensating for heat loss, not actively heating)

#### `rationale`

**Description**: 1-sentence explanation of the transformation (why these forms, why this magnitude).

**Required**: Yes (helps with reproducibility and validation).

**Good rationale**: "Electrical energy from charger converted to chemical potential energy stored in lithium-ion battery with ~90% efficiency"

**Poor rationale**: "Energy involved" (too vague, doesn't explain transformation)

### Complete Example

**Step**: `"Charge battery at 0.5C to 4.2V"`

**Characterization** (transformation-focused):

```json
{
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "chemical_potential",
      "magnitude": 0.9,
      "rationale": "Electrical energy from charger converted to chemical potential energy stored in lithium-ion battery with ~90% efficiency"
    },
    "matter": {
      "from_form": "lithium_ions_dispersed",
      "to_form": "lithium_ions_intercalated",
      "magnitude": 0.7,
      "rationale": "Lithium ions migrate from cathode through electrolyte and intercalate into graphite anode structure"
    },
    "information": {
      "from_form": "voltage_measurement",
      "to_form": "battery_state_data",
      "magnitude": 0.5,
      "rationale": "Voltage measurements transformed into state-of-charge (SOC) estimate via battery management system"
    }
  }
}
```

### Common Transformation Patterns

#### Energy Transformations

| From → To | Example Steps | Typical Magnitude |
|-----------|---------------|-------------------|
| `electrical → thermal` | Heating with electric heater | 0.7-0.9 |
| `electrical → chemical_potential` | Battery charging | 0.85-0.95 |
| `chemical_potential → electrical` | Battery discharging | 0.85-0.95 |
| `electrical → mechanical` | Motor-driven mixing | 0.6-0.8 |
| `thermal → thermal` (maintain) | Temperature maintenance | 0.1-0.3 |

#### Matter Transformations

| From → To | Example Steps | Typical Magnitude |
|-----------|---------------|-------------------|
| `separated_powders → homogeneous_mixture` | Powder mixing | 0.7-0.9 |
| `liquid_ambient_temp → liquid_elevated_temp` | Heating liquid | 0.5-0.7 |
| `solid_crystalline → liquid_solution` | Dissolution | 0.8-0.95 |
| `lithium_ions_dispersed → lithium_ions_intercalated` | Battery charging | 0.6-0.8 |
| `liquid → gas` | Evaporation | 0.85-0.95 |

#### Information Transformations

| From → To | Example Steps | Typical Magnitude |
|-----------|---------------|-------------------|
| `voltage_measurement → battery_state_data` | SOC calculation | 0.6-0.8 |
| `temperature_sensor → control_feedback` | PID control | 0.4-0.6 |
| `optical_signal → concentration_data` | Spectrophotometry | 0.7-0.9 |
| `none → none` | No information processing | 0.0-0.1 |

---

## Parameter Normalization (Optional)

**Status**: Implement only if you find it useful during curation.

### Common Parameters (Top 5-10)

Based on analysis of `IG_start/data/` patents:

| Parameter Name | Common Values | Normalization Rule |
|----------------|---------------|-------------------|
| `temperature` | `"95°C"`, `"100°C"`, `"200°F"` | Convert to Celsius, store as float |
| `time` | `"30 min"`, `"1.5 hours"` | Convert to minutes, store as float |
| `voltage` | `"3.7V"`, `"4.2V"` | Store as float (volts) |
| `current` | `"0.5C"`, `"1C"` | Store as float (C-rate) |
| `capacity` | `"2000 mAh"` | Store as float (mAh) |

**Example Normalization**:

```json
{
  "name": "temperature",
  "value": "95°C",
  "normalized_value": 95.0,
  "unit": "celsius"
}
```

**Implementation**: Do this manually in JSONL for now. If you process 50+ patents and find normalization tedious, write a simple Python function.

---

## JSONL Artifact Schema

**Philosophy**: JSONL files are the **source of truth**. Neo4j is just a view for querying.

### Stage 1: Identification

**File**: `.artifacts/{run_id}/stage1/steps.jsonl`

```jsonl
{
  "patent_id": "CN-100342579-C",
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "order_index": 1,
  "text_raw": "Heat to 95°C",
  "stage": "identification",
  "parameters": [
    {
      "id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d",
      "name": "temperature",
      "value": "95°C"
    }
  ]
}
```

**Keep it simple**: One JSON object per line, human-readable.

---

### Stage 2: Encoding

**File**: `.artifacts/{run_id}/stage2/steps-encoded.jsonl`

**Format**: Same as Stage 1, but with `transformation_tags` added:

```jsonl
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "text_raw": "Heat to 95°C",
  "stage": "encoding",
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "thermal",
      "magnitude": 0.85,
      "rationale": "Electrical heating element converts to thermal energy"
    },
    "matter": {
      "from_form": "liquid_ambient_temp",
      "to_form": "liquid_elevated_temp",
      "magnitude": 0.6,
      "rationale": "Liquid heated from room temperature to 95°C"
    },
    "information": {
      "from_form": "temperature_sensor",
      "to_form": "control_feedback",
      "magnitude": 0.4,
      "rationale": "Temperature monitoring provides feedback for heating control"
    }
  }
}
```

**Workflow**: Edit this file manually in VS Code with Copilot Chat assistance (see `copilot-prompts/stage2-transformation-encoding.md`).

---

### Stage 3: Atomicization

**File**: `.artifacts/{run_id}/stage3/steps-atomicized.jsonl`

**Format**: For composite steps, add atomic sub-steps as new entries:

```jsonl
# Original composite step
{
  "step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "text_raw": "Mix powder A and B, then heat to 95°C",
  "stage": "atomicized",
  "is_composite": true
}

# Atomic sub-steps
{
  "step_id": "new-uuid-1",
  "text_raw": "Mix powder A and B",
  "stage": "atomicized",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "order": 1
}
{
  "step_id": "new-uuid-2",
  "text_raw": "Heat mixture to 95°C",
  "stage": "atomicized",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "order": 2
}
```

---

### Stage 4: Clustering

**File**: `.artifacts/{run_id}/stage4/clusters.jsonl`

```jsonl
{
  "cluster_id": "c7d8e9f0-1a2b-3c4d-5e6f-7a8b9c0d1e2f",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "cluster_method": "manual-copilot",
  "member_step_ids": [
    "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
    "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
    "..."
  ],
  "member_count": 5
}
```

**Keep it simple**: Just list member step IDs. Neo4j will create `IN_CLUSTER` relationships when you import.

---

## Neo4j Initialization Script

**Minimal setup** (copy-paste into Neo4j Browser):

```cypher
-- Uniqueness constraints
CREATE CONSTRAINT patent_id_unique IF NOT EXISTS
FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE;

CREATE CONSTRAINT step_id_unique IF NOT EXISTS
FOR (s:Step) REQUIRE s.id IS UNIQUE;

CREATE CONSTRAINT parameter_id_unique IF NOT EXISTS
FOR (p:Parameter) REQUIRE p.id IS UNIQUE;

CREATE CONSTRAINT cluster_id_unique IF NOT EXISTS
FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE;

-- Performance indexes (only the essentials)
CREATE INDEX step_run_id_idx IF NOT EXISTS
FOR (s:Step) ON (s.run_id);

CREATE INDEX step_stage_idx IF NOT EXISTS
FOR (s:Step) ON (s.stage);
```

**That's it!** Don't over-index. For 10-20 patents (~500 steps), these constraints are sufficient.

---

## Common Queries

### View all steps from one patent

```cypher
MATCH (p:Patent {patent_id: "CN-100342579-C"})-[:HAS_STEP]->(s:Step)
RETURN s.order_index, s.text_raw, s.transformation_tags
ORDER BY s.order_index
```

### Find composite steps and their sub-steps

```cypher
MATCH (composite:Step {is_composite: true})-[:COMPOSED_OF]->(atomic:Step)
WHERE composite.run_id = "{your_run_id}"
RETURN composite.text_raw, collect(atomic.text_raw) AS atomic_steps
```

### View clusters

```cypher
MATCH (s:Step)-[:IN_CLUSTER]->(c:Cluster)
WHERE c.run_id = "{your_run_id}"
RETURN c.cluster_id, collect(s.text_raw) AS steps
```

### Find high-energy transformation steps

```cypher
MATCH (s:Step)
WHERE s.transformation_tags.energy.magnitude > 0.8
  AND s.run_id = "{your_run_id}"
RETURN s.text_raw, s.transformation_tags.energy
ORDER BY s.transformation_tags.energy.magnitude DESC
```

### Find electrical→chemical transformations (e.g., battery charging)

```cypher
MATCH (s:Step)
WHERE s.transformation_tags.energy.from_form = "electrical"
  AND s.transformation_tags.energy.to_form = "chemical_potential"
  AND s.run_id = "{your_run_id}"
RETURN s.text_raw, s.transformation_tags.energy.magnitude
ORDER BY s.transformation_tags.energy.magnitude DESC
```

---

## Workflow Summary

1. **Stage 1**: Run CLI tool → Generates `steps.jsonl` with UUIDs → Import to Neo4j (optional)
2. **Stage 2**: Edit `steps.jsonl` manually (add `transformation_tags` with Copilot Chat) → Re-import to Neo4j (optional)
3. **Stage 3**: Create new atomic step entries in JSONL (with Copilot suggestions) → Re-import to Neo4j (optional)
4. **Stage 4**: Create `clusters.jsonl` manually (group similar steps with Copilot) → Import to Neo4j (optional)

**Key principle**: JSONL first, Neo4j optional. You can defer Neo4j import until you want visualization/querying.

---

## Validation Checklist

Before scaling beyond 5 patents, validate:

- ✅ Can you parse `technical_process` correctly? (Check 10 steps manually)
- ✅ Do transformation characterizations make sense? (Show to domain expert)
- ✅ Are from_form→to_form conversions accurate? (Verify 10 examples)
- ✅ Are composite steps split logically? (Review 5 examples)
- ✅ Do clusters group truly similar steps with similar transformations? (Spot-check 3 clusters)

**Success criteria**: If 85%+ of steps are correctly processed with transformation tags and <30 min/patent curation time, you're ready to scale.

---

## Next Steps

1. **Process 5 patents** end-to-end
2. **Refine dimension taxonomy** based on what you see
3. **Document effective Copilot prompts** in `copilot-prompts/`
4. **Scale to 20 patents** once workflow is smooth

---

## Resources

- **Implementation plan**: [plan.md](./plan.md)
- **Research findings**: [research.md](./research.md)
- **User guide**: [quickstart.md](./quickstart.md)
- **Copilot prompts**: `copilot-prompts/` directory
- **Neo4j docs**: https://neo4j.com/docs/cypher-manual/current/

---

**Last Updated**: 2025-10-28  
**Version**: 1.0.0 (Minimal Viable MVP)  
**Philosophy**: Simple schema, human-readable artifacts, quality over complexity
