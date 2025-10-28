# Stage 4 Output Format: Clustered Steps

**Purpose**: Document the JSONL output format for Stage 4 (Clustering) with semantically similar steps grouped into clusters.

**File location**: `.artifacts/{run_id}/stage4/clusters.jsonl`

**Key Concept**: This stage identifies groups of semantically similar steps across patents based on action similarity, parameter ranges, and transformation patterns.

---

## JSONL Format Overview

Each line represents one cluster of similar steps:

```jsonl
{"cluster_id": "heating-moderate-temp", "run_id": "550e8400-e29b-41d4-a716-446655440000", "cluster_method": "manual-copilot", "similarity_threshold": 0.85, "member_step_ids": ["a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0"], "member_count": 3, "representative_text": "Heating liquid or mixture to moderate temperature (90-110°C)", "similarity_score": 0.92, "timestamp": "2024-10-28T16:00:00Z"}
{"cluster_id": "electrochemical-charging", "run_id": "550e8400-e29b-41d4-a716-446655440000", "cluster_method": "manual-copilot", "similarity_threshold": 0.85, "member_step_ids": ["d6ee4b1h-0db0-6e7f-c34c-3d1d37e8g0h0", "e7ff5c2i-1ec1-7f8g-d45d-4e2e48f9h1i1"], "member_count": 2, "representative_text": "Charging battery at constant current (0.3-1C) to cutoff voltage", "similarity_score": 0.88, "timestamp": "2024-10-28T16:05:00Z"}
```

---

## Cluster Object Structure

### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `cluster_id` | string | Descriptive identifier for cluster (human-readable) | `"heating-moderate-temp"`, `"electrochemical-charging"` |
| `run_id` | string (UUID v4) | Identifier for this processing run | `"550e8400-e29b-41d4-a716-446655440000"` |
| `cluster_method` | string | How clustering was performed | `"manual-copilot"`, `"transformation-based"`, `"hybrid"` |
| `similarity_threshold` | number (0-1) | Minimum similarity score for membership | `0.85`, `0.90` |
| `member_step_ids` | array of UUIDs | List of `step_id` values in this cluster | `["a3bb189e-...", "b4cc290f-..."]` |
| `member_count` | integer | Number of steps in cluster | `3`, `5`, `12` |
| `representative_text` | string | Canonical description generalizing all members | `"Heating liquid to moderate temperature (90-110°C)"` |
| `similarity_score` | number (0-1) | Average similarity among cluster members | `0.92`, `0.88` |
| `timestamp` | string (ISO 8601) | When cluster was created | `"2024-10-28T16:00:00Z"` |

### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `transformation_signature` | object | Representative transformation pattern for cluster | See below |
| `parameter_ranges` | object | Parameter value ranges across cluster members | See below |
| `domain` | string | Process domain/category | `"battery_manufacturing"`, `"chemical_synthesis"` |
| `rationale` | string | Explanation of why steps cluster | `"All involve heating to similar temperatures with thermal transformation"` |

---

## Complete Example: Moderate-Temperature Heating Cluster

```json
{
  "cluster_id": "heating-moderate-temp",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "cluster_method": "manual-copilot",
  "similarity_threshold": 0.85,
  "member_step_ids": [
    "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
    "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
    "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0"
  ],
  "member_count": 3,
  "representative_text": "Heating liquid or mixture to moderate temperature (90-110°C)",
  "similarity_score": 0.92,
  "transformation_signature": {
    "energy": {
      "from_form": "electrical",
      "to_form": "thermal",
      "magnitude_range": [0.75, 0.90]
    },
    "matter": {
      "from_form": "liquid_ambient_temp",
      "to_form": "liquid_elevated_temp",
      "magnitude_range": [0.5, 0.7]
    },
    "information": {
      "from_form": "temperature_sensor",
      "to_form": "control_feedback",
      "magnitude_range": [0.3, 0.5]
    }
  },
  "parameter_ranges": {
    "temperature": {
      "min": "90°C",
      "max": "110°C",
      "normalized_min": 90.0,
      "normalized_max": 110.0,
      "unit": "celsius"
    },
    "duration": {
      "min": "20 minutes",
      "max": "45 minutes",
      "normalized_min": 20.0,
      "normalized_max": 45.0,
      "unit": "minutes"
    }
  },
  "domain": "chemical_synthesis",
  "rationale": "All steps involve heating liquid mixtures to similar moderate temperatures (90-110°C) using electrical heating with similar energy→thermal transformation patterns",
  "timestamp": "2024-10-28T16:00:00Z"
}
```

---

## Transformation Signature Object

**Purpose**: Characterize the typical transformation pattern across cluster members.

### Structure

```json
"transformation_signature": {
  "energy": {
    "from_form": "electrical",
    "to_form": "thermal",
    "magnitude_range": [0.75, 0.90]
  },
  "matter": {
    "from_form": "liquid_ambient_temp",
    "to_form": "liquid_elevated_temp",
    "magnitude_range": [0.5, 0.7]
  },
  "information": {
    "from_form": "temperature_sensor",
    "to_form": "control_feedback",
    "magnitude_range": [0.3, 0.5]
  }
}
```

### Field Specifications

| Field | Type | Description |
|-------|------|-------------|
| `from_form` | string | Representative initial form (most common across members) |
| `to_form` | string | Representative final form (most common across members) |
| `magnitude_range` | array [min, max] | Range of magnitude values across cluster members |

**Note**: Transformation signature is **optional** but highly valuable for understanding cluster characteristics. Include if transformation data is available from Stage 2.

---

## Parameter Ranges Object

**Purpose**: Show parameter value distributions across cluster members.

### Structure

```json
"parameter_ranges": {
  "temperature": {
    "min": "90°C",
    "max": "110°C",
    "normalized_min": 90.0,
    "normalized_max": 110.0,
    "unit": "celsius"
  },
  "duration": {
    "min": "20 minutes",
    "max": "45 minutes",
    "normalized_min": 20.0,
    "normalized_max": 45.0,
    "unit": "minutes"
  }
}
```

**Use case**: Helps identify parameter tolerances and validate cluster coherence (wide ranges may indicate over-clustering).

---

## Cluster Methods

| Method | Description | When to use |
|--------|-------------|-------------|
| `manual-copilot` | Human curator with Copilot Chat assistance | **MVP default** - manual curation with Copilot suggestions |
| `transformation-based` | Clustered by similar transformation patterns (from_form→to_form) | When transformation data is high quality |
| `textual-similarity` | Clustered by text embedding similarity (not used in MVP) | Future: automated clustering |
| `hybrid` | Combination of methods | Future: multi-criteria clustering |

---

## Common Cluster Patterns

### Pattern 1: Temperature-Based Heating Clusters

**Cluster ID**: `heating-moderate-temp`, `heating-high-temp`, `heating-cryogenic`

**Similarity criteria**:
- Same action type: heating
- Temperature range: ±20°C
- Similar matter type: liquid vs. solid
- Similar transformation: electrical→thermal

**Example clusters**:
- **Moderate-temp**: 90-110°C (solvent evaporation, gentle reactions)
- **High-temp**: 200-400°C (calcination, sintering)
- **Very high-temp**: 700-1000°C (high-temperature synthesis)

**Representative transformation**:
```json
{
  "energy": {"from_form": "electrical", "to_form": "thermal", "magnitude_range": [0.75, 0.90]},
  "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude_range": [0.5, 0.7]}
}
```

---

### Pattern 2: Electrochemical Cycling Clusters

**Cluster ID**: `electrochemical-charging`, `electrochemical-discharging`

**Similarity criteria**:
- Same operation: charge vs. discharge
- C-rate range: ±0.5C
- Similar voltage range
- Same energy transformation direction

**Example clusters**:
- **Charging (slow)**: 0.1-0.5C to 4.2V
- **Charging (fast)**: 1-2C to 4.2V
- **Discharging**: 0.5-1C to 2.5V cutoff

**Representative transformation**:
```json
{
  "energy": {"from_form": "electrical", "to_form": "chemical_potential", "magnitude_range": [0.85, 0.95]},
  "matter": {"from_form": "lithium_ions_dispersed", "to_form": "lithium_ions_intercalated", "magnitude_range": [0.6, 0.8]},
  "information": {"from_form": "voltage_measurement", "to_form": "battery_state_data", "magnitude_range": [0.4, 0.6]}
}
```

---

### Pattern 3: Mechanical Mixing Clusters

**Cluster ID**: `mixing-dry-powders`, `mixing-liquids`, `grinding`

**Similarity criteria**:
- Same mechanical action: mixing, stirring, grinding
- Similar RPM range (if specified)
- Same matter type: powder vs. liquid
- Similar matter transformation: separated→mixed

**Example clusters**:
- **Dry powder mixing**: 100-500 rpm, solid powders
- **Liquid mixing**: 500-2000 rpm, liquid solutions
- **Grinding**: 5000-10000 rpm, particle size reduction

**Representative transformation**:
```json
{
  "energy": {"from_form": "electrical", "to_form": "mechanical", "magnitude_range": [0.6, 0.8]},
  "matter": {"from_form": "separated_powders", "to_form": "homogeneous_mixture", "magnitude_range": [0.7, 0.9]},
  "information": {"from_form": "none", "to_form": "none", "magnitude_range": [0.0, 0.2]}
}
```

---

### Pattern 4: Measurement Clusters

**Cluster ID**: `spectrophotometry`, `ph-measurement`, `temperature-measurement`

**Similarity criteria**:
- Same measurement type: optical, electrochemical, thermal
- Similar information transformation
- Minimal energy/matter changes

**Example clusters**:
- **Spectrophotometry**: Absorbance at various wavelengths
- **pH measurement**: Electrochemical pH sensing
- **Temperature measurement**: Thermocouple, RTD, IR

**Representative transformation**:
```json
{
  "energy": {"from_form": "electromagnetic", "to_form": "none", "magnitude_range": [0.1, 0.3]},
  "matter": {"from_form": "sample", "to_form": "sample", "magnitude_range": [0.0, 0.1]},
  "information": {"from_form": "optical_signal", "to_form": "concentration_data", "magnitude_range": [0.7, 0.9]}
}
```

---

## Validation Rules

### Cluster Validation

✅ **Valid cluster**:
- `similarity_score` ≥ `similarity_threshold`
- `member_count` ≥ 2 (at least 2 steps to form a cluster)
- All `member_step_ids` exist in Stage 3 output
- `representative_text` generalizes all members without being too vague
- Transformation signatures align (similar from_form→to_form patterns)

⚠️ **Questionable cluster** (flag for review):
- `similarity_score` < 0.85 (borderline similarity)
- `member_count` > 20 (potential over-clustering)
- Wide parameter ranges (e.g., 50-500°C suggests over-generalization)
- Transformation signatures have wide magnitude ranges (>0.5 spread)

❌ **Invalid cluster**:
- `similarity_score` < `similarity_threshold` (violates clustering rule)
- `member_count` = 1 (singleton, not a cluster - should be excluded)
- `member_step_ids` contains duplicates
- Missing required fields

### Singleton Handling

**Singletons** (steps that don't cluster with any others) are **not** included in `clusters.jsonl`. They remain in Stage 3 output but are not part of clustering results.

**Optional**: Create a separate file `.artifacts/{run_id}/stage4/singletons.jsonl` listing singleton step IDs with rationale:

```jsonl
{"step_id": "f8gg6d3j-2gd2-8g9h-e56e-5f3f59g0i2j2", "run_id": "550e8400-e29b-41d4-a716-446655440000", "rationale": "Unique cryogenic process at 4K - no similar steps found in dataset", "timestamp": "2024-10-28T16:10:00Z"}
```

---

## Usage Workflow

### Creating Stage 4 JSONL

1. **Load Stage 3 JSONL** (`.artifacts/{run_id}/stage3/steps_atomicized.jsonl`)

2. **Extract atomic steps only** (exclude composite parents, use only atomic steps or sub-steps)

3. **Batch clustering** (10-30 steps at a time):
   - Use Copilot Chat with clustering prompt (see `copilot-prompts/stage4-clustering.md`)
   - Paste 10-30 step texts with numbering
   - Review Copilot's cluster suggestions

4. **Validate clusters**:
   - Check similarity scores (>0.85)
   - Verify representative text makes sense
   - Ensure transformation patterns align (if available)
   - Adjust membership if needed

5. **Create cluster objects**:
   - Generate descriptive `cluster_id` (e.g., `heating-moderate-temp`)
   - Map step numbers from Copilot to actual `step_id` UUIDs
   - Calculate `member_count`
   - Set `cluster_method` to `manual-copilot`

6. **Export to Stage 4 JSONL**: `.artifacts/{run_id}/stage4/clusters.jsonl`

### Reading Stage 4 JSONL

**Python example**:

```python
import json

# Load clusters
clusters = []
with open('.artifacts/{run_id}/stage4/clusters.jsonl', 'r') as f:
    for line in f:
        cluster = json.loads(line)
        clusters.append(cluster)

# Find largest clusters
clusters_sorted = sorted(clusters, key=lambda c: c['member_count'], reverse=True)
print(f"Largest cluster: {clusters_sorted[0]['cluster_id']} with {clusters_sorted[0]['member_count']} members")

# Filter by domain
battery_clusters = [c for c in clusters if c.get('domain') == 'battery_manufacturing']

# Filter by transformation pattern
electrical_to_thermal = [
    c for c in clusters
    if c.get('transformation_signature', {}).get('energy', {}).get('from_form') == 'electrical'
    and c.get('transformation_signature', {}).get('energy', {}).get('to_form') == 'thermal'
]
```

**Command-line example** (using `jq`):

```bash
# Count total clusters
cat clusters.jsonl | wc -l

# Extract all cluster IDs
cat clusters.jsonl | jq -r '.cluster_id'

# Find clusters with >5 members
cat clusters.jsonl | jq 'select(.member_count > 5)'

# Get average similarity score
cat clusters.jsonl | jq '.similarity_score' | awk '{sum+=$1} END {print sum/NR}'
```

---

## Complete JSONL File Example

**File**: `.artifacts/550e8400-e29b-41d4-a716-446655440000/stage4/clusters.jsonl`

```jsonl
{"cluster_id": "heating-moderate-temp", "run_id": "550e8400-e29b-41d4-a716-446655440000", "cluster_method": "manual-copilot", "similarity_threshold": 0.85, "member_step_ids": ["a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0"], "member_count": 3, "representative_text": "Heating liquid or mixture to moderate temperature (90-110°C)", "similarity_score": 0.92, "transformation_signature": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude_range": [0.75, 0.90]}, "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude_range": [0.5, 0.7]}, "information": {"from_form": "temperature_sensor", "to_form": "control_feedback", "magnitude_range": [0.3, 0.5]}}, "parameter_ranges": {"temperature": {"min": "90°C", "max": "110°C", "normalized_min": 90.0, "normalized_max": 110.0, "unit": "celsius"}}, "domain": "chemical_synthesis", "rationale": "All involve heating to similar temperatures with electrical→thermal transformation", "timestamp": "2024-10-28T16:00:00Z"}
{"cluster_id": "electrochemical-charging", "run_id": "550e8400-e29b-41d4-a716-446655440000", "cluster_method": "manual-copilot", "similarity_threshold": 0.85, "member_step_ids": ["d6ee4b1h-0db0-6e7f-c34c-3d1d37e8g0h0", "e7ff5c2i-1ec1-7f8g-d45d-4e2e48f9h1i1"], "member_count": 2, "representative_text": "Charging battery at constant current (0.3-1C) to cutoff voltage", "similarity_score": 0.88, "transformation_signature": {"energy": {"from_form": "electrical", "to_form": "chemical_potential", "magnitude_range": [0.85, 0.95]}, "matter": {"from_form": "lithium_ions_dispersed", "to_form": "lithium_ions_intercalated", "magnitude_range": [0.6, 0.8]}, "information": {"from_form": "voltage_measurement", "to_form": "battery_state_data", "magnitude_range": [0.4, 0.6]}}, "parameter_ranges": {"c_rate": {"min": "0.3C", "max": "1C", "normalized_min": 0.3, "normalized_max": 1.0, "unit": "C"}, "voltage": {"min": "4.1V", "max": "4.2V", "normalized_min": 4.1, "normalized_max": 4.2, "unit": "volts"}}, "domain": "battery_manufacturing", "rationale": "Both are charging operations with similar C-rates and electrical→chemical_potential transformation", "timestamp": "2024-10-28T16:05:00Z"}
{"cluster_id": "mixing-dry-powders", "run_id": "550e8400-e29b-41d4-a716-446655440000", "cluster_method": "manual-copilot", "similarity_threshold": 0.85, "member_step_ids": ["f8gg6d3j-2gd2-8g9h-e56e-5f3f59g0i2j2", "g9hh7e4k-3he3-9h0i-f67f-6g4g60h1j3k3"], "member_count": 2, "representative_text": "Mixing or grinding dry powder materials for homogeneity", "similarity_score": 0.90, "transformation_signature": {"energy": {"from_form": "electrical", "to_form": "mechanical", "magnitude_range": [0.6, 0.8]}, "matter": {"from_form": "separated_powders", "to_form": "homogeneous_mixture", "magnitude_range": [0.7, 0.9]}, "information": {"from_form": "none", "to_form": "none", "magnitude_range": [0.0, 0.2]}}, "domain": "powder_processing", "rationale": "Both involve mechanical mixing of solid powders with electrical→mechanical transformation", "timestamp": "2024-10-28T16:08:00Z"}
```

---

## Troubleshooting

### Issue: Copilot over-clusters dissimilar steps

**Problem**: Groups "Heat to 95°C" and "Heat to 800°C" together

**Solution**:
- Override Copilot's decision
- Create separate clusters: `heating-moderate-temp` and `heating-high-temp`
- Use transformation magnitude as discriminator (moderate vs. extreme thermal elevation)

### Issue: Copilot under-clusters obvious similarities

**Problem**: Doesn't group "Mix powder A and B" with "Mix cobalt oxide and lithium carbonate"

**Solution**:
- Manually merge into one cluster
- Add `rationale` explaining why they're similar (both powder mixing with matter transformation)

### Issue: Transformation data not available

**Problem**: Stage 2 encoding was skipped or incomplete

**Solution**:
- Cluster based on textual similarity and parameter ranges only
- Omit `transformation_signature` field (optional)
- Consider adding transformation encoding later to refine clusters

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Transformation-aware clustering)  
**Related**: [stage3-output-format.md](./stage3-output-format.md), [copilot-prompts/stage4-clustering.md](../copilot-prompts/stage4-clustering.md), [plan.md](../plan.md)
