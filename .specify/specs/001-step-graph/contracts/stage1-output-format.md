# Stage 1 Output Format: Identified Steps

**Purpose**: Document the JSONL output format for Stage 1 (Step Identification) with extracted process steps and associated parameters.

**File location**: `.artifacts/{run_id}/stage1/steps.jsonl`

---

## JSONL Format Overview

Each line is a valid JSON object representing one process step. Example structure:

```jsonl
{"step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 1, "text_raw": "Heat the reaction mixture to 95°C for 30 minutes under nitrogen atmosphere", "extraction_confidence": 0.98, "parameters": [...], "timestamp": "2024-10-28T14:30:00Z"}
{"step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 2, "text_raw": "Add catalyst dropwise over 10 minutes while maintaining temperature", "extraction_confidence": 0.92, "parameters": [...], "timestamp": "2024-10-28T14:31:00Z"}
```

**Note**: JSONL (JSON Lines) format = one JSON object per line (no commas between lines).

---

## Step Object Structure

### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `step_id` | string (UUID v4) | Unique identifier for this step | `"a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8"` |
| `run_id` | string (UUID v4) | Identifier for this processing run | `"550e8400-e29b-41d4-a716-446655440000"` |
| `patent_id` | string | Source patent identifier | `"US-10123456-B2"` |
| `order_index` | integer | Sequential order of step (1-based) | `1`, `2`, `3`, ... |
| `text_raw` | string | Original step text from patent (unmodified) | `"Heat the reaction mixture to 95°C"` |
| `extraction_confidence` | number (0-1) | Confidence that this is a valid process step | `0.98` |
| `parameters` | array | Associated technical parameters (see below) | `[{...}, {...}]` |
| `timestamp` | string (ISO 8601) | When step was extracted | `"2024-10-28T14:30:00Z"` |

### Example: Minimal Step

```json
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 1,
  "text_raw": "Heat the reaction mixture to 95°C",
  "extraction_confidence": 0.98,
  "parameters": [],
  "timestamp": "2024-10-28T14:30:00Z"
}
```

---

## Parameters Array

**Purpose**: Technical parameters associated with this step (temperature, time, voltage, etc.).

### Parameter Object Structure

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `parameter_id` | string (UUID v4) | Yes | Unique parameter identifier | `"f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d"` |
| `name` | string | Yes | Parameter type/name | `"temperature"`, `"duration"`, `"voltage"` |
| `value` | string | Yes | Value with unit | `"95°C"`, `"30 minutes"`, `"12V"` |
| `normalized_value` | number | No | Numeric value (if normalizable) | `95.0`, `30.0`, `12.0` |
| `unit` | string | No | Normalized unit | `"celsius"`, `"minutes"`, `"volts"` |
| `association_method` | string | Yes | How parameter was linked to step | `"explicit"`, `"llm_inferred"`, `"lane_a_linked"` |
| `link_confidence` | number (0-1) | Yes | Confidence in step-parameter association | `1.0`, `0.85`, `0.7` |
| `link_evidence` | string | No | Justification for association | `"Direct mention in step text"` |

### Association Methods

| Method | Description | When to use |
|--------|-------------|-------------|
| `explicit` | Parameter directly mentioned in step text | "Heat to **95°C**" |
| `llm_inferred` | Parameter inferred by human curator from context | "Heat until boiling" → inferred ~100°C for water |
| `lane_a_linked` | Parameter from Lane A pre-extracted data linked to step | Parameter extracted separately, linked via context matching |

### Example: Step with Parameters

```json
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 1,
  "text_raw": "Heat the reaction mixture to 95°C for 30 minutes under nitrogen atmosphere",
  "extraction_confidence": 0.98,
  "parameters": [
    {
      "parameter_id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d",
      "name": "temperature",
      "value": "95°C",
      "normalized_value": 95.0,
      "unit": "celsius",
      "association_method": "explicit",
      "link_confidence": 1.0,
      "link_evidence": "Direct mention: 'to 95°C'"
    },
    {
      "parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a",
      "name": "duration",
      "value": "30 minutes",
      "normalized_value": 30.0,
      "unit": "minutes",
      "association_method": "explicit",
      "link_confidence": 1.0,
      "link_evidence": "Direct mention: 'for 30 minutes'"
    },
    {
      "parameter_id": "d6cc5e4f-3f7a-6c8d-0d0e-5g4f3e2d1c0b",
      "name": "atmosphere",
      "value": "nitrogen",
      "association_method": "explicit",
      "link_confidence": 1.0,
      "link_evidence": "Direct mention: 'under nitrogen atmosphere'"
    }
  ],
  "timestamp": "2024-10-28T14:30:00Z"
}
```

---

## Complete File Example

**File**: `.artifacts/550e8400-e29b-41d4-a716-446655440000/stage1/steps.jsonl`

```jsonl
{"step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 1, "text_raw": "Heat the reaction mixture to 95°C for 30 minutes under nitrogen atmosphere", "extraction_confidence": 0.98, "parameters": [{"parameter_id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d", "name": "temperature", "value": "95°C", "normalized_value": 95.0, "unit": "celsius", "association_method": "explicit", "link_confidence": 1.0, "link_evidence": "Direct mention: 'to 95°C'"}, {"parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a", "name": "duration", "value": "30 minutes", "normalized_value": 30.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0, "link_evidence": "Direct mention: 'for 30 minutes'"}], "timestamp": "2024-10-28T14:30:00Z"}
{"step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 2, "text_raw": "Add catalyst dropwise over 10 minutes while maintaining temperature", "extraction_confidence": 0.92, "parameters": [{"parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7", "name": "duration", "value": "10 minutes", "normalized_value": 10.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0, "link_evidence": "Direct mention: 'over 10 minutes'"}], "timestamp": "2024-10-28T14:31:00Z"}
{"step_id": "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 3, "text_raw": "Cool to room temperature", "extraction_confidence": 0.85, "parameters": [{"parameter_id": "c2bb067d-6bd7-2a4a-7a0a-9a7a03b4c5d6", "name": "temperature", "value": "room temperature", "normalized_value": 25.0, "unit": "celsius", "association_method": "llm_inferred", "link_confidence": 0.8, "link_evidence": "Inferred: 'room temperature' typically ~25°C in lab context"}], "timestamp": "2024-10-28T14:32:00Z"}
```

---

## Validation Rules

### Extraction Confidence Thresholds

| Confidence | Interpretation | Action |
|------------|----------------|--------|
| **0.9-1.0** | High confidence - clear, well-formed process step | Auto-include |
| **0.7-0.89** | Moderate confidence - valid step but some ambiguity | Include, flag for review |
| **0.5-0.69** | Low confidence - uncertain if this is a process step | Flag for human review |
| **0.0-0.49** | Very low confidence - likely not a process step | Exclude (or flag for review) |

**MVP Guideline**: Include only steps with confidence ≥ 0.7 to maintain quality.

### Parameter Link Confidence

| Confidence | Interpretation | Example |
|------------|----------------|---------|
| **1.0** | Explicit mention in step text | "Heat to **95°C**" |
| **0.8-0.99** | Strong inference from context | "Heat until dissolved" → 95°C (solvent boiling point known) |
| **0.5-0.79** | Moderate inference (uncertain) | "Heat gently" → ~60-80°C (inferred from "gently") |
| **0.0-0.49** | Weak/speculative inference | Avoid unless flagged for review |

**Best Practice**: Only include parameters with `link_confidence ≥ 0.7` for automated processing.

---

## Usage Workflow

### Creating Stage 1 JSONL

**Option 1: Manual Curation (Recommended for MVP)**

1. Read patent description
2. Identify process steps manually
3. Extract parameters from step text
4. Create JSONL file with one step per line
5. Generate UUIDs for `step_id` and `parameter_id`
6. Set `extraction_confidence` and `link_confidence` based on certainty

**Option 2: Copilot-Assisted Extraction**

1. Use Copilot Chat with Stage 1 prompt template
2. Paste patent description
3. Review Copilot's extracted steps
4. Manually adjust `text_raw`, parameters, and confidence scores
5. Export to JSONL format

**Note**: No automated API calls - human curator validates all extractions.

### Reading Stage 1 JSONL

**Python example**:

```python
import json

steps = []
with open('.artifacts/{run_id}/stage1/steps.jsonl', 'r') as f:
    for line in f:
        step = json.loads(line)
        steps.append(step)

# Filter by confidence
high_confidence_steps = [s for s in steps if s['extraction_confidence'] >= 0.9]

# Get all parameters
all_params = []
for step in steps:
    all_params.extend(step['parameters'])
```

**Command-line example** (using `jq`):

```bash
# Count total steps
cat steps.jsonl | wc -l

# Extract all step texts
cat steps.jsonl | jq -r '.text_raw'

# Filter high-confidence steps
cat steps.jsonl | jq 'select(.extraction_confidence >= 0.9)'

# Get all temperature parameters
cat steps.jsonl | jq '.parameters[] | select(.name == "temperature")'
```

---

## Common Patterns

### Pattern 1: Heating Step

```json
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 1,
  "text_raw": "Heat mixture to 95°C for 30 minutes",
  "extraction_confidence": 0.98,
  "parameters": [
    {"parameter_id": "...", "name": "temperature", "value": "95°C", "normalized_value": 95.0, "unit": "celsius", "association_method": "explicit", "link_confidence": 1.0},
    {"parameter_id": "...", "name": "duration", "value": "30 minutes", "normalized_value": 30.0, "unit": "minutes", "association_method": "explicit", "link_confidence": 1.0}
  ],
  "timestamp": "2024-10-28T14:30:00Z"
}
```

### Pattern 2: Electrochemical Step

```json
{
  "step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10654321-A1",
  "order_index": 5,
  "text_raw": "Charge battery at 0.5C to 4.2V cutoff",
  "extraction_confidence": 0.95,
  "parameters": [
    {"parameter_id": "...", "name": "c_rate", "value": "0.5C", "normalized_value": 0.5, "unit": "C", "association_method": "explicit", "link_confidence": 1.0},
    {"parameter_id": "...", "name": "voltage", "value": "4.2V", "normalized_value": 4.2, "unit": "volts", "association_method": "explicit", "link_confidence": 1.0}
  ],
  "timestamp": "2024-10-28T14:35:00Z"
}
```

### Pattern 3: Measurement Step

```json
{
  "step_id": "c5dd301g-0dhb-6e7f-c34c-3d1d37e8g0h0",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10789456-B1",
  "order_index": 3,
  "text_raw": "Record absorbance at 450 nm using spectrophotometer",
  "extraction_confidence": 0.92,
  "parameters": [
    {"parameter_id": "...", "name": "wavelength", "value": "450 nm", "normalized_value": 450.0, "unit": "nanometers", "association_method": "explicit", "link_confidence": 1.0}
  ],
  "timestamp": "2024-10-28T14:33:00Z"
}
```

---

## Troubleshooting

### Issue: Step text is too long

**Problem**: Step contains multiple actions (composite step)

**Solution**: Split into separate steps with sequential `order_index` values, or wait until Stage 3 (Atomicization) to handle splitting

**Example**:
```
Original: "Heat to 95°C and maintain for 30 minutes"
Split into:
  Step 1: "Heat to 95°C" (order_index=1)
  Step 2: "Maintain temperature for 30 minutes" (order_index=2)
```

### Issue: Parameter value has no unit

**Problem**: Parameter like "Heat for 30" (missing unit)

**Solution**: 
- If unit is clear from context, add it: `"value": "30 minutes"`
- If unit is ambiguous, flag for human review or exclude parameter

### Issue: Uncertain parameter-step association

**Problem**: Parameter appears in patent but unclear which step it belongs to

**Solution**:
- Set `association_method="llm_inferred"`
- Set `link_confidence` < 1.0 (e.g., 0.7-0.8)
- Provide `link_evidence` explaining the inference
- Flag for human review if confidence < 0.7

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Simplified for Copilot-driven workflow)  
**Related**: [stage2-output-format.md](./stage2-output-format.md), [lane-a-format.md](./lane-a-format.md), [plan.md](../plan.md)
