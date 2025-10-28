# Stage 3: Atomicization Prompt Template

**Purpose**: Use this prompt in GitHub Copilot Chat to identify composite steps and split them into atomic sub-steps while preserving transformation information.

**When to use**: Stage 3 of the pipeline, after transformation encoding is complete.

---

## Copilot Chat Prompt

```
Analyze this process step to determine if it's composite (contains multiple distinct actions). If composite, split it into atomic sub-steps.

**Step text**: "{paste step text here}"

**Decision criteria**:
- **Atomic**: Single, indivisible action (e.g., "Heat to 95°C")
- **Composite**: Multiple actions or sequential operations (e.g., "Mix powders A and B, then heat to 95°C")

**Return JSON**:

If ATOMIC:
{
  "is_composite": false,
  "rationale": "1-sentence explanation why it's atomic"
}

If COMPOSITE:
{
  "is_composite": true,
  "rationale": "1-sentence explanation why it's composite",
  "atomic_steps": [
    {"text": "First atomic action", "order": 1},
    {"text": "Second atomic action", "order": 2},
    ...
  ]
}

**Guidelines**:
- **Atomic = one primary verb**: "Heat", "Mix", "Cool", "Charge"
- **Composite = multiple verbs or sequential phrases**: "Mix and heat", "Charge then discharge", "Heat until dissolved"
- **Preserve original meaning**: Don't add/remove information, just split logically
- **Temporal order**: If step says "A then B", order must reflect that

**Context** (include 2-3 examples):

Example 1 - ATOMIC:
Input: "Heat to 95°C"
Output:
{
  "is_composite": false,
  "rationale": "Single heating action, no multiple operations"
}

Example 2 - COMPOSITE (explicit conjunction):
Input: "Mix powder A and B, then heat to 95°C"
Output:
{
  "is_composite": true,
  "rationale": "Two distinct actions: mixing followed by heating",
  "atomic_steps": [
    {"text": "Mix powder A and B", "order": 1},
    {"text": "Heat mixture to 95°C", "order": 2}
  ]
}

Example 3 - COMPOSITE (implicit sequence):
Input: "Charge at 0.5C to 4.2V, discharge at 1C to 2.5V, repeat 3 cycles"
Output:
{
  "is_composite": true,
  "rationale": "Three distinct actions: charge, discharge, repeat cycle",
  "atomic_steps": [
    {"text": "Charge at 0.5C to 4.2V", "order": 1},
    {"text": "Discharge at 1C to 2.5V", "order": 2},
    {"text": "Repeat cycle 3 times", "order": 3}
  ]
}

Example 4 - ATOMIC (complex but single action):
Input: "Heat lithium carbonate to 800°C in nitrogen atmosphere for 4 hours"
Output:
{
  "is_composite": false,
  "rationale": "Single heating action with multiple parameters (temperature, atmosphere, time)"
}

Now analyze this step: "{paste step text again}"
```

---

## Usage Workflow

1. **Open JSONL file** (`.artifacts/{run_id}/stage2/steps-encoded.jsonl`) in VS Code
2. **Select steps to analyze** (or iterate through all)
3. **Open Copilot Chat** (`Ctrl+Shift+I` or `Cmd+Shift+I`)
4. **Paste the prompt** above (replace `{paste step text here}` with actual text)
5. **Review Copilot's response**:
   - If `is_composite: false`: Mark step as atomic, move to next
   - If `is_composite: true`: Review proposed splits
6. **Adjust if needed** (check logical correctness and preservation of information)
7. **Create new JSONL entries** for atomic sub-steps (see [JSONL Format](#jsonl-format))

---

## JSONL Format for Atomicized Steps

### Original Composite Step

Keep the original step, add `is_composite: true`:

```jsonl
{
  "step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "text_raw": "Mix powder A and B, then heat to 95°C",
  "stage": "atomicized",
  "is_composite": true,
  "transformation_tags": {...}
}
```

### Atomic Sub-Steps

Create new entries with:
- **New UUID** (`step_id`)
- **`parent_id`**: UUID of the composite step
- **`order`**: Sequential position (1, 2, 3...)
- **`stage`**: `"atomicized"`

```jsonl
{
  "step_id": "new-uuid-atom-1",
  "text_raw": "Mix powder A and B",
  "stage": "atomicized",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "order": 1
}
{
  "step_id": "new-uuid-atom-2",
  "text_raw": "Heat mixture to 95°C",
  "stage": "atomicized",
  "parent_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "order": 2
}
```

**Note**: Each atomic sub-step should also be characterized with transformation_tags (re-run Stage 2 encoding on them).

---

## Common Patterns

### Pattern 1: Explicit Conjunctions

**Indicators**: "and", "then", "followed by", "after"

**Examples**:
- "Mix **and** heat" → COMPOSITE (2 steps)
- "Heat **then** cool" → COMPOSITE (2 steps)
- "Charge, **then** discharge" → COMPOSITE (2 steps)

### Pattern 2: Implicit Sequences

**Indicators**: Multiple verbs without conjunctions, comma-separated actions

**Examples**:
- "Charge to 4.2V, discharge to 2.5V" → COMPOSITE (2 steps)
- "Grind, sift, collect powder" → COMPOSITE (3 steps)

### Pattern 3: Conditional/Iterative

**Indicators**: "until", "while", "repeat", "for X cycles"

**Examples**:
- "Heat **until** dissolved" → ATOMIC (single action with completion condition)
- "Heat, cool, **repeat** 3 times" → COMPOSITE (3 steps: heat, cool, repeat)

**Tricky case**: "Heat until X" is usually ATOMIC if "until X" is a stopping condition, not a separate action.

### Pattern 4: Complex Single Actions

**Indicators**: Many parameters but one primary verb

**Examples**:
- "Heat lithium carbonate to 800°C in nitrogen atmosphere for 4 hours" → ATOMIC
- "Charge at 0.5C to 4.2V with CV mode" → ATOMIC

**Rule**: If there's only ONE primary action (heat, charge), it's atomic even with many parameters.

---

## Decision Flowchart

```
Is there more than one primary verb (action)?
├─ No → ATOMIC
└─ Yes → Are they distinct actions?
          ├─ No (e.g., "heat and stir" = simultaneous) → ATOMIC
          └─ Yes (e.g., "heat then cool" = sequential) → COMPOSITE
```

---

## Validation Checklist

After atomicizing 10-20 steps, review:

- ✅ Do atomic steps each have a single clear action?
- ✅ Does the order of atomic steps preserve original meaning?
- ✅ Are all parameters preserved (no information loss)?
- ✅ Does Copilot's split align with domain logic?
- ✅ Can each atomic step be independently characterized with transformations?

**If not**: Manually adjust Copilot's suggestions. You are the final authority.

---

## Edge Cases

### Case 1: Simultaneous Actions

**Example**: "Heat to 95°C while stirring"

**Decision**: **ATOMIC** - "while" indicates simultaneous, not sequential. Single composite action.

### Case 2: Repeating Steps

**Example**: "Charge at 0.5C, discharge at 1C, repeat 3 times"

**Decision**: **COMPOSITE** - 3 steps:
1. "Charge at 0.5C"
2. "Discharge at 1C"
3. "Repeat cycle 3 times"

### Case 3: Conditional Completion

**Example**: "Heat until dissolved"

**Decision**: **ATOMIC** - "until dissolved" is a stopping condition, not a separate action.

---

## Transformation Preservation

**Important**: After splitting composite steps, each atomic sub-step should have its own transformation characterization.

**Workflow**:
1. Split composite step into atomic sub-steps
2. For each atomic sub-step, use Stage 2 prompt to characterize transformations
3. Ensure transformation_tags are added to each atomic step's JSONL entry

**Example**:

Original composite: "Mix powder A and B, then heat to 95°C"

Atomic sub-step 1: "Mix powder A and B"
- Energy: mechanical (motor) → mechanical (rotation)
- Matter: separated_powders → mixed_powder
- Information: mixing_instruction → process_execution

Atomic sub-step 2: "Heat mixture to 95°C"
- Energy: thermal_ambient → thermal_elevated
- Matter: mixed_powder_room_temp → mixed_powder_heated
- Information: temperature_setpoint → heating_control

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Full functionality restored)  
**Related**: [quickstart.md](../quickstart.md), [data-model.md](../data-model.md)
