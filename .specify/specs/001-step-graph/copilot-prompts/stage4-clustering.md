# Stage 4: Clustering Prompt Template

**Purpose**: Use this prompt in GitHub Copilot Chat to identify semantically similar steps across patents for clustering, considering both textual similarity and transformation patterns.

**When to use**: Stage 4 of the pipeline, after atomicization is complete.

---

## Copilot Chat Prompt

```
Analyze these process steps and identify groups of semantically similar steps that should be clustered together.

**Steps to analyze**:

{paste 10-30 step texts here, numbered}

Example format:
1. "Heat lithium carbonate to 800°C"
2. "Heat mixture to 95°C"
3. "Mix powder A and B"
4. "Grind cobalt oxide"
5. "Heat to 100°C for 2 hours"
...

**Clustering criteria**:
- **Semantic similarity**: Steps describe the same type of action (e.g., all heating steps)
- **Similar parameters**: Values don't need to match exactly, but should be comparable (e.g., 95°C and 100°C are similar, but 95°C and 800°C are not)
- **Transformation similarity**: Steps with similar energy/matter/information form conversions
- **Same domain**: Steps from the same process type (e.g., all battery manufacturing)
- **Threshold**: Only cluster if similarity > 0.85 (on 0-1 scale)

**Return JSON**:

{
  "clusters": [
    {
      "cluster_id": "Generate a descriptive ID (e.g., 'heating-moderate-temp')",
      "similarity_score": 0.0-1.0,
      "member_steps": [1, 2, 5],
      "representative_text": "One-sentence canonical description",
      "rationale": "Why these steps are similar (consider transformations)"
    },
    {
      "cluster_id": "mixing-dry-powders",
      "similarity_score": 0.88,
      "member_steps": [3, 4],
      "representative_text": "Mixing or grinding dry powder materials",
      "rationale": "Both involve mechanical processing of solid powders"
    }
  ],
  "singletons": [6, 7],
  "rationale_for_singletons": "Steps 6 and 7 are unique, no similar steps found"
}

**Guidelines**:
- **Don't over-cluster**: Only group truly similar steps (>0.85 similarity)
- **Representative text**: Should generalize across all members without losing specificity
- **Cluster ID**: Use descriptive names (e.g., "heating-high-temp", "electrochemical-charging")
- **Consider transformations**: Steps with similar from_form→to_form patterns should cluster
- **Singletons are OK**: If a step is unique, don't force it into a cluster

**Context** (include 1-2 examples):

Example 1 - Similar heating steps:
Input:
1. "Heat to 95°C"
2. "Heat mixture to 100°C for 20 minutes"
3. "Heat lithium carbonate to 800°C"

Output:
{
  "clusters": [
    {
      "cluster_id": "heating-moderate-temp",
      "similarity_score": 0.92,
      "member_steps": [1, 2],
      "representative_text": "Heating liquid or mixture to moderate temperature (90-110°C)",
      "rationale": "Both involve heating to similar temperatures (95-100°C) with thermal_ambient→thermal_elevated transformation"
    }
  ],
  "singletons": [3],
  "rationale_for_singletons": "Step 3 is high-temperature heating (800°C), fundamentally different transformation magnitude from moderate-temp steps"
}

Example 2 - Electrochemical processes:
Input:
1. "Charge at 0.5C to 4.2V"
2. "Charge battery at 1C"
3. "Discharge at 1C to 2.5V"

Output:
{
  "clusters": [
    {
      "cluster_id": "electrochemical-charging",
      "similarity_score": 0.88,
      "member_steps": [1, 2],
      "representative_text": "Charging battery at constant current to target voltage",
      "rationale": "Both are charging operations with similar current rates (0.5-1C) and electrical→chemical transformation"
    }
  ],
  "singletons": [3],
  "rationale_for_singletons": "Step 3 is discharging (opposite transformation: chemical→electrical), not similar to charging"
}

Now analyze these steps: {paste steps again}
```

---

## Usage Workflow

1. **Export atomicized steps** to a text file:
   ```bash
   python -m step_graph export \
     --run-id $RUN_ID \
     --stage atomicized \
     --format txt \
     --output steps_for_clustering.txt
   ```

2. **Open `steps_for_clustering.txt` in VS Code**

3. **Select 10-30 steps** to analyze in one batch (don't do all steps at once - too much context)

4. **Open Copilot Chat** (`Ctrl+Shift+I` or `Cmd+Shift+I`)

5. **Paste the prompt** above with selected steps

6. **Review Copilot's clusters**:
   - Check similarity scores (should be >0.85)
   - Validate representative text makes sense
   - Verify transformation patterns align (if available)
   - Ensure no over-clustering (dissimilar steps forced together)

7. **Manually adjust** if needed:
   - Remove steps from clusters if similarity is questionable
   - Merge similar clusters if Copilot split them unnecessarily
   - Create new clusters for missed patterns

8. **Create `clusters.jsonl`** with approved clusters (see [JSONL Format](#jsonl-format))

---

## JSONL Format for Clusters

**File**: `.artifacts/{run_id}/stage4/clusters.jsonl`

```jsonl
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
  "similarity_score": 0.92
}
```

**Note**: `member_step_ids` must be the actual UUIDs from your JSONL files. Cross-reference step numbers from Copilot's response with UUIDs.

---

## Clustering Strategies

### Strategy 1: Process by Domain

Group steps by patent domain before clustering:

1. **Battery charging/discharging steps** → Cluster separately
2. **Thermal processing steps** → Cluster separately
3. **Mechanical mixing/grinding steps** → Cluster separately

**Why**: Different domains may use similar words for different actions.

### Strategy 2: Process by Transformation Pattern

Use transformation_tags from Stage 2 to pre-filter:

1. **High energy transformation** (`energy.magnitude > 0.7`) → Cluster together
2. **Phase change transformations** (`matter.from_form` → `matter.to_form` with state change) → Cluster together
3. **Data acquisition transformations** (`information.to_form` contains "data") → Cluster together

**Why**: Steps with similar transformation patterns are more likely to cluster meaningfully.

### Strategy 3: Iterative Refinement

1. **First pass**: Cluster obvious patterns (e.g., all "heat to X°C" steps)
2. **Second pass**: Review singletons - any that could join existing clusters?
3. **Third pass**: Merge over-split clusters if they're actually similar

---

## Common Clustering Patterns

### Pattern 1: Temperature-Based Heating

**Cluster criteria**:
- Same action type (heating)
- Temperature range within ±20°C
- Similar matter type (liquid vs. solid)
- Similar transformation magnitude

**Example cluster**:
- "Heat to 95°C"
- "Heat mixture to 100°C"
- "Heat to 90°C for 30 minutes"

**Representative text**: "Heating liquid or mixture to moderate temperature (90-110°C)"

**Transformation similarity**: All involve `thermal_ambient → thermal_elevated` with magnitude ~0.6-0.7

---

### Pattern 2: Electrochemical Cycling

**Cluster criteria**:
- Same operation (charge vs. discharge)
- C-rate within ±0.5C
- Similar voltage range
- Same energy transformation direction

**Example cluster**:
- "Charge at 0.5C to 4.2V"
- "Charge at 0.3C to 4.1V"
- "Charge battery at 1C"

**Representative text**: "Charging battery at constant current (0.3-1C) to cutoff voltage"

**Transformation similarity**: All involve `electrical → chemical_potential` with magnitude ~0.8-0.9

---

### Pattern 3: Mechanical Mixing

**Cluster criteria**:
- Same mechanical action (mixing, grinding, stirring)
- Similar RPM or speed (if specified)
- Same matter type (powder vs. liquid)
- Similar matter transformation

**Example cluster**:
- "Mix powder A and B"
- "Grind cobalt oxide"
- "Mix lithium carbonate and nickel oxide"

**Representative text**: "Mixing or grinding dry powder materials for homogeneity"

**Transformation similarity**: All involve `separated_materials → homogeneous_mixture` with magnitude ~0.7-0.8

---

## Similarity Score Guidelines

**How to estimate similarity** (0-1 scale):

| Similarity | Criteria | Example |
|------------|----------|---------|
| **0.95-1.0** | Nearly identical (only numeric parameters differ) | "Heat to 95°C" vs. "Heat to 100°C" |
| **0.85-0.94** | Same action, similar context, similar transformations (should cluster) | "Charge at 0.5C to 4.2V" vs. "Charge at 0.3C to 4.1V" |
| **0.70-0.84** | Similar action, different context or transformation magnitude (borderline) | "Heat to 95°C" vs. "Heat to 200°C" |
| **0.50-0.69** | Related but distinct (don't cluster) | "Heat" vs. "Cool" (opposite transformation directions) |
| **0.00-0.49** | Completely different | "Heat" vs. "Charge" |

**Rule of thumb**: If you hesitate whether two steps should cluster, they probably shouldn't (similarity < 0.85).

---

## Transformation-Aware Clustering

**Key insight**: Steps with similar transformation patterns are semantically similar even if wording differs.

### Example: Identifying Similar Energy Conversions

**Steps**:
1. "Charge battery at 0.5C" (electrical → chemical)
2. "Apply voltage to electrolyze water" (electrical → chemical)
3. "Heat resistor" (electrical → thermal)

**Clustering decision**:
- Steps 1 and 2 should cluster (both electrical→chemical transformations)
- Step 3 is singleton (electrical→thermal is different transformation)

### Example: Identifying Similar Matter Transformations

**Steps**:
1. "Melt lithium at 180°C" (solid → liquid)
2. "Dissolve salt in water" (solid → liquid/dissolved)
3. "Evaporate solvent" (liquid → gas)

**Clustering decision**:
- Steps 1 and 2 might cluster (both involve solid→liquid/dispersed transformation)
- Step 3 is singleton (liquid→gas is different transformation type)

---

## Validation Checklist

After clustering 50-100 steps, review:

- ✅ Do cluster members truly describe similar actions AND transformations?
- ✅ Are similarity scores >0.85 for all clusters?
- ✅ Does representative text generalize without losing meaning?
- ✅ Do transformation patterns align within clusters (similar from_form→to_form)?
- ✅ Are singletons actually unique (couldn't fit any cluster)?

**If not**: Refine clusters manually. Clustering is inherently subjective - your domain expertise matters.

---

## Troubleshooting

### Issue: Copilot over-clusters (groups dissimilar steps)

**Example**: Groups "Heat to 95°C" and "Heat to 800°C" together

**Solution**: Override. These have different transformation magnitudes (moderate vs. extreme thermal elevation). Create separate clusters.

### Issue: Copilot under-clusters (misses obvious similarities)

**Example**: Doesn't group "Mix powder A and B" with "Mix cobalt oxide and lithium carbonate"

**Solution**: Manually merge into one cluster. Both are powder mixing with similar matter transformations.

### Issue: Transformation data not considered

**Example**: Copilot only looks at text, ignores transformation_tags

**Solution**: Explicitly mention transformation patterns in your prompt: "Consider that step 1 has electrical→chemical transformation while step 3 has electrical→thermal"

---

## Advanced: Multi-Pass Clustering

For larger datasets (50+ patents), consider multi-pass approach:

### Pass 1: Coarse Clustering (0.90+ threshold)

Group only highly similar steps with nearly identical transformations:
- All "charge at 0.5C to 4.2V" steps
- All "heat to 95°C" steps

**Result**: 5-10 tight, high-confidence clusters + many singletons

### Pass 2: Moderate Clustering (0.85-0.89 threshold)

Review singletons from Pass 1:
- Can any join existing clusters based on transformation similarity?
- Can 2-3 singletons form new clusters?

**Result**: Absorb some singletons, create 2-5 more clusters

### Pass 3: Singleton Review

Final review of remaining singletons:
- Are they truly unique transformations?
- Document why they don't cluster

**Result**: Final cluster set + justified singletons

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Full functionality restored with transformation awareness)  
**Related**: [quickstart.md](../quickstart.md), [data-model.md](../data-model.md), [stage2-transformation-encoding.md](./stage2-transformation-encoding.md)
