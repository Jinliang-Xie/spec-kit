# Stage 3 Output Format: Atomicized Steps

**Purpose**: Document the JSONL output format for Stage 3 (Atomicization) with composite steps split into atomic sub-steps.

**File location**: `.artifacts/{run_id}/stage3/steps_atomicized.jsonl`

**Key Concept**: This stage identifies composite steps (containing multiple actions) and splits them into atomic sub-steps, preserving provenance via `parent_id` links.

---

## JSONL Format Overview

### Atomic Steps (Unchanged)

Steps determined to be atomic remain unchanged from Stage 2:

```jsonl
{"step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 1, "text_raw": "Add catalyst dropwise over 10 minutes", "extraction_confidence": 0.95, "parameters": [...], "transformation_tags": {...}, "timestamp": "2024-10-28T15:00:00Z"}
```

### Composite Steps (Split)

Composite steps are represented as:
1. **Original composite step** (preserved for provenance)
2. **Atomic sub-steps** (new entries with `parent_id` pointing to original)

```jsonl
{"step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 2, "text_raw": "Heat to 95°C and maintain for 30 minutes", "extraction_confidence": 0.92, "parameters": [...], "transformation_tags": {...}, "is_composite": true, "atomicization_metadata": {"method": "manual", "confidence": 0.95, "rationale": "Two distinct actions: heating (active) and maintaining (passive)"}, "timestamp": "2024-10-28T15:05:00Z"}
{"step_id": "d6ee4b1h-0db0-6e7f-c34c-3d1d37e8g0h0", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "sub_step_order": 1, "text_raw": "Heat to 95°C", "extraction_confidence": 0.95, "parameters": [...], "transformation_tags": {...}, "timestamp": "2024-10-28T15:06:00Z"}
{"step_id": "e7ff5c2i-1ec1-7f8g-d45d-4e2e48f9h1i1", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "sub_step_order": 2, "text_raw": "Maintain temperature for 30 minutes", "extraction_confidence": 0.95, "parameters": [...], "transformation_tags": {...}, "timestamp": "2024-10-28T15:07:00Z"}
```

---

## Atomic Step Structure

**Atomic steps** (steps that cannot be meaningfully split) have the same structure as Stage 2 output:

| Field | Type | Description |
|-------|------|-------------|
| All Stage 2 fields | - | Inherited from Stage 2 without modification |

**Example**:
```json
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 1,
  "text_raw": "Add catalyst dropwise over 10 minutes",
  "extraction_confidence": 0.95,
  "parameters": [
    {
      "parameter_id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d",
      "name": "duration",
      "value": "10 minutes",
      "normalized_value": 10.0,
      "unit": "minutes",
      "association_method": "explicit",
      "link_confidence": 1.0
    }
  ],
  "transformation_tags": {
    "energy": {"from_form": "gravitational", "to_form": "none", "magnitude": 0.1, "rationale": "Minimal gravitational energy for dropwise addition"},
    "matter": {"from_form": "catalyst_separated", "to_form": "catalyst_mixed", "magnitude": 0.8, "rationale": "Catalyst added dropwise integrates into reaction mixture"},
    "information": {"from_form": "visual_observation", "to_form": "addition_rate_control", "magnitude": 0.4, "rationale": "Visual monitoring ensures controlled dropwise addition"}
  },
  "timestamp": "2024-10-28T15:00:00Z"
}
```

---

## Composite Step Structure

**Composite steps** (steps containing multiple actions) add these fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| All Stage 2 fields | - | Yes | Inherited from Stage 2 |
| `is_composite` | boolean | Yes | Always `true` for composite steps |
| `atomicization_metadata` | object | Yes | Metadata about atomicization decision (see below) |

### Atomicization Metadata

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `method` | string | How atomicization was performed | `"manual"`, `"copilot_assisted"` |
| `confidence` | number (0-1) | Confidence in atomicization decision | `0.95`, `0.8` |
| `rationale` | string | Explanation of why step is composite | `"Two distinct actions: heating (active) and maintaining (passive)"` |

**Example composite step**:
```json
{
  "step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 2,
  "text_raw": "Heat to 95°C and maintain for 30 minutes",
  "extraction_confidence": 0.92,
  "parameters": [
    {
      "parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a",
      "name": "temperature",
      "value": "95°C",
      "normalized_value": 95.0,
      "unit": "celsius",
      "association_method": "explicit",
      "link_confidence": 1.0
    },
    {
      "parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7",
      "name": "duration",
      "value": "30 minutes",
      "normalized_value": 30.0,
      "unit": "minutes",
      "association_method": "explicit",
      "link_confidence": 1.0
    }
  ],
  "transformation_tags": {
    "energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.85, "rationale": "Electrical heating converts to thermal energy"},
    "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude": 0.6, "rationale": "Liquid heated from room temp to 95°C"},
    "information": {"from_form": "temperature_sensor", "to_form": "control_feedback", "magnitude": 0.5, "rationale": "Temperature monitoring for control"}
  },
  "is_composite": true,
  "atomicization_metadata": {
    "method": "manual",
    "confidence": 0.95,
    "rationale": "Two distinct actions: heating (active energy input to reach 95°C) and maintaining (passive control for 30 minutes)"
  },
  "timestamp": "2024-10-28T15:05:00Z"
}
```

---

## Sub-Step Structure

**Sub-steps** (atomic components of composite steps) add these fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| All Stage 2 fields | - | Yes | Inherited (but values may differ from parent) |
| `parent_id` | string (UUID v4) | Yes | `step_id` of the composite parent step |
| `sub_step_order` | integer | Yes | Sequential order within parent (1-based) |
| `order_index` | - | Removed | Not applicable for sub-steps (use `sub_step_order` instead) |

**Critical**: Sub-steps **must re-run Stage 2 encoding** to get transformation tags specific to the atomic action. Don't inherit parent's transformation tags blindly.

### Example: Sub-Step 1 (Heating)

```json
{
  "step_id": "d6ee4b1h-0db0-6e7f-c34c-3d1d37e8g0h0",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "sub_step_order": 1,
  "text_raw": "Heat to 95°C",
  "extraction_confidence": 0.95,
  "parameters": [
    {
      "parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a",
      "name": "temperature",
      "value": "95°C",
      "normalized_value": 95.0,
      "unit": "celsius",
      "association_method": "explicit",
      "link_confidence": 1.0
    }
  ],
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "thermal",
      "magnitude": 0.9,
      "rationale": "Active electrical heating converts to thermal energy to raise temperature from ambient to 95°C"
    },
    "matter": {
      "from_form": "liquid_ambient_temp",
      "to_form": "liquid_elevated_temp",
      "magnitude": 0.7,
      "rationale": "Liquid undergoes thermal elevation from ~25°C to 95°C during active heating phase"
    },
    "information": {
      "from_form": "temperature_sensor",
      "to_form": "setpoint_control",
      "magnitude": 0.6,
      "rationale": "Temperature sensor provides feedback to reach 95°C target"
    }
  },
  "timestamp": "2024-10-28T15:06:00Z"
}
```

### Example: Sub-Step 2 (Maintaining)

```json
{
  "step_id": "e7ff5c2i-1ec1-7f8g-d45d-4e2e48f9h1i1",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "sub_step_order": 2,
  "text_raw": "Maintain temperature for 30 minutes",
  "extraction_confidence": 0.95,
  "parameters": [
    {
      "parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7",
      "name": "duration",
      "value": "30 minutes",
      "normalized_value": 30.0,
      "unit": "minutes",
      "association_method": "explicit",
      "link_confidence": 1.0
    }
  ],
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "thermal",
      "magnitude": 0.4,
      "rationale": "Minimal electrical energy to compensate heat loss and maintain 95°C for 30 minutes (passive control)"
    },
    "matter": {
      "from_form": "liquid_elevated_temp",
      "to_form": "liquid_elevated_temp",
      "magnitude": 0.1,
      "rationale": "Matter state remains constant at 95°C during maintenance phase"
    },
    "information": {
      "from_form": "temperature_sensor",
      "to_form": "control_feedback",
      "magnitude": 0.7,
      "rationale": "Temperature sensor continuously monitors and provides feedback to maintain setpoint over 30-minute duration"
    }
  },
  "timestamp": "2024-10-28T15:07:00Z"
}
```

**Note difference**: Sub-step 1 (heating) has high energy magnitude (0.9), while sub-step 2 (maintaining) has lower energy magnitude (0.4) because it's passive control. This shows why re-encoding is critical.

---

## Complete JSONL File Example

**File**: `.artifacts/550e8400-e29b-41d4-a716-446655440000/stage3/steps_atomicized.jsonl`

```jsonl
{"step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 1, "text_raw": "Add catalyst dropwise over 10 minutes", "extraction_confidence": 0.95, "parameters": [{"parameter_id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d", "name": "duration", "value": "10 minutes", "normalized_value": 10.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0}], "transformation_tags": {"energy": {"from_form": "gravitational", "to_form": "none", "magnitude": 0.1, "rationale": "Minimal gravitational energy"}, "matter": {"from_form": "catalyst_separated", "to_form": "catalyst_mixed", "magnitude": 0.8, "rationale": "Catalyst integrates into mixture"}, "information": {"from_form": "visual_observation", "to_form": "addition_rate_control", "magnitude": 0.4, "rationale": "Visual monitoring"}}, "timestamp": "2024-10-28T15:00:00Z"}
{"step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 2, "text_raw": "Heat to 95°C and maintain for 30 minutes", "extraction_confidence": 0.92, "parameters": [{"parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a", "name": "temperature", "value": "95°C", "normalized_value": 95.0, "unit": "celsius", "association_method": "explicit", "link_confidence": 1.0}, {"parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7", "name": "duration", "value": "30 minutes", "normalized_value": 30.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0}], "transformation_tags": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.85, "rationale": "Electrical heating"}, "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude": 0.6, "rationale": "Thermal elevation"}, "information": {"from_form": "temperature_sensor", "to_form": "control_feedback", "magnitude": 0.5, "rationale": "Temperature control"}}, "is_composite": true, "atomicization_metadata": {"method": "manual", "confidence": 0.95, "rationale": "Two distinct actions: heating and maintaining"}, "timestamp": "2024-10-28T15:05:00Z"}
{"step_id": "d6ee4b1h-0db0-6e7f-c34c-3d1d37e8g0h0", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "sub_step_order": 1, "text_raw": "Heat to 95°C", "extraction_confidence": 0.95, "parameters": [{"parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a", "name": "temperature", "value": "95°C", "normalized_value": 95.0, "unit": "celsius", "association_method": "explicit", "link_confidence": 1.0}], "transformation_tags": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.9, "rationale": "Active heating"}, "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude": 0.7, "rationale": "Thermal elevation during active heating"}, "information": {"from_form": "temperature_sensor", "to_form": "setpoint_control", "magnitude": 0.6, "rationale": "Feedback to reach target"}}, "timestamp": "2024-10-28T15:06:00Z"}
{"step_id": "e7ff5c2i-1ec1-7f8g-d45d-4e2e48f9h1i1", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "sub_step_order": 2, "text_raw": "Maintain temperature for 30 minutes", "extraction_confidence": 0.95, "parameters": [{"parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7", "name": "duration", "value": "30 minutes", "normalized_value": 30.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0}], "transformation_tags": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.4, "rationale": "Passive control with minimal energy"}, "matter": {"from_form": "liquid_elevated_temp", "to_form": "liquid_elevated_temp", "magnitude": 0.1, "rationale": "State remains constant"}, "information": {"from_form": "temperature_sensor", "to_form": "control_feedback", "magnitude": 0.7, "rationale": "Continuous monitoring"}}, "timestamp": "2024-10-28T15:07:00Z"}
{"step_id": "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 3, "text_raw": "Cool to room temperature", "extraction_confidence": 0.85, "parameters": [{"parameter_id": "c2bb067d-6bd7-2a4a-7a0a-9a7a03b4c5d6", "name": "temperature", "value": "room temperature", "normalized_value": 25.0, "unit": "celsius", "association_method": "llm_inferred", "link_confidence": 0.8}], "transformation_tags": {"energy": {"from_form": "thermal", "to_form": "ambient", "magnitude": 0.7, "rationale": "Thermal energy dissipates to environment"}, "matter": {"from_form": "liquid_elevated_temp", "to_form": "liquid_ambient_temp", "magnitude": 0.6, "rationale": "Temperature decreases to ambient"}, "information": {"from_form": "none", "to_form": "none", "magnitude": 0.0, "rationale": "No information transformation"}}, "timestamp": "2024-10-28T15:10:00Z"}
```

---

## Common Atomicization Patterns

### Pattern 1: Explicit Conjunction ("and", "then")

**Original**: "Heat to 95°C and maintain for 30 minutes"

**Atomicized**:
1. "Heat to 95°C"
2. "Maintain temperature for 30 minutes"

**Rationale**: Conjunction "and" indicates two distinct actions.

---

### Pattern 2: Implicit Sequence

**Original**: "Add catalyst dropwise over 10 minutes while stirring"

**Decision**: **Keep atomic** (debatable)

**Rationale**: "While stirring" is a concurrent modifier, not a sequential sub-step. However, if stirring is critical, could split into:
1. "Begin stirring"
2. "Add catalyst dropwise over 10 minutes"

**Best Practice**: Use domain judgment - if both actions are equally important, split. If one is modifier, keep atomic.

---

### Pattern 3: Conditional/Iterative ("until", "repeat")

**Original**: "Stir until solution becomes clear"

**Decision**: **Keep atomic**

**Rationale**: "Until clear" is a completion criterion, not a separate action. Splitting would create artificial distinction.

---

### Pattern 4: Complex Single Action

**Original**: "Heat mixture to 95°C for 30 minutes while monitoring temperature and adjusting power as needed"

**Decision**: **Composite** (but complex)

**Atomicized**:
1. "Heat mixture to 95°C"
2. "Maintain temperature for 30 minutes while monitoring and adjusting"

**Rationale**: Primary split is heat vs. maintain. Monitoring/adjusting are part of maintenance control loop.

---

## Validation Rules

### Composite Step Validation

✅ **Valid composite step**:
- Has `is_composite: true`
- Has `atomicization_metadata` with method, confidence, rationale
- Has 2+ sub-steps with same `parent_id` in file
- Sub-steps have sequential `sub_step_order` (1, 2, 3, ...)

❌ **Invalid composite step**:
- Missing `atomicization_metadata`
- No sub-steps found (orphaned composite marker)
- Sub-steps have gaps in `sub_step_order` (e.g., 1, 3, 4 - missing 2)

### Sub-Step Validation

✅ **Valid sub-step**:
- Has `parent_id` matching a composite step's `step_id`
- Has `sub_step_order` ≥ 1
- Does NOT have `order_index` (atomic steps have `order_index`, sub-steps don't)
- Has `transformation_tags` (re-encoded for atomic action)

❌ **Invalid sub-step**:
- Missing `parent_id`
- `parent_id` doesn't match any composite step
- Missing `sub_step_order`
- Has both `order_index` AND `parent_id` (contradictory)

---

## Usage Workflow

### Creating Stage 3 JSONL

1. **Load Stage 2 JSONL** (`.artifacts/{run_id}/stage2/steps_encoded.jsonl`)

2. **For each step**, use Copilot Chat with atomicization prompt (see `copilot-prompts/stage3-atomicization.md`)

3. **Copilot returns decision**:
   - If atomic: Keep step unchanged
   - If composite: Create parent step + sub-steps

4. **For composite steps**:
   - Write original composite step with `is_composite: true` and `atomicization_metadata`
   - Generate new UUIDs for sub-steps
   - Write sub-steps with `parent_id`, `sub_step_order`
   - **Critical**: Re-run Stage 2 encoding on each sub-step to get transformation tags

5. **Export to Stage 3 JSONL**: `.artifacts/{run_id}/stage3/steps_atomicized.jsonl`

### Reading Stage 3 JSONL

**Python example**:

```python
import json

# Load atomicized steps
all_entries = []
with open('.artifacts/{run_id}/stage3/steps_atomicized.jsonl', 'r') as f:
    for line in f:
        entry = json.loads(line)
        all_entries.append(entry)

# Separate atomic steps, composite parents, and sub-steps
atomic_steps = [e for e in all_entries if 'parent_id' not in e and not e.get('is_composite', False)]
composite_parents = [e for e in all_entries if e.get('is_composite', False)]
sub_steps = [e for e in all_entries if 'parent_id' in e]

# Reconstruct composite step hierarchy
for parent in composite_parents:
    parent_id = parent['step_id']
    children = [s for s in sub_steps if s['parent_id'] == parent_id]
    children.sort(key=lambda x: x['sub_step_order'])
    print(f"Composite: {parent['text_raw']}")
    for child in children:
        print(f"  Sub-step {child['sub_step_order']}: {child['text_raw']}")
```

---

## Troubleshooting

### Issue: Uncertain if step is composite

**Problem**: Step has ambiguous structure (e.g., "Stir vigorously until clear")

**Solution**:
- Set `atomicization_metadata.confidence` < 0.7
- Provide detailed `rationale` explaining ambiguity
- Flag for human review
- Default to **atomic** if uncertain (conservative approach)

### Issue: Sub-step transformation tags same as parent

**Problem**: Forgot to re-encode sub-steps, just copied parent's transformation tags

**Solution**:
- Re-run Stage 2 encoding for each sub-step independently
- Verify transformation magnitudes differ (e.g., heating sub-step should have higher energy magnitude than maintaining sub-step)

### Issue: Too many sub-steps (over-atomicization)

**Problem**: Split "Heat to 95°C for 30 minutes" into 30 separate sub-steps (one per minute)

**Solution**:
- Only split on **distinct actions** (different verbs or phases)
- Don't split based on parameter changes alone
- Rule of thumb: 2-4 sub-steps per composite step (rarely more)

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Transformation-aware atomicization)  
**Related**: [stage4-output-format.md](./stage4-output-format.md), [stage2-output-format.md](./stage2-output-format.md), [copilot-prompts/stage3-atomicization.md](../copilot-prompts/stage3-atomicization.md), [plan.md](../plan.md)
