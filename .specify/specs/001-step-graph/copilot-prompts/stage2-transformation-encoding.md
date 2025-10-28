# Stage 2: Transformation Encoding Prompt Template

**Purpose**: Characterize process steps by **form conversions** along three dimensions (energy/matter/information).

**Key Principle**: Focus on **TRANSFORMATIONS** (A→B), not just presence or involvement.

---

## Copilot Chat Prompt Template

```
Analyze this process step and characterize the TRANSFORMATIONS along three dimensions.

**Step text**: "{paste step text here}"

**Return JSON with this structure**:

```json
{
  "energy": {
    "from_form": "initial energy form",
    "to_form": "final energy form",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the energy form conversion"
  },
  "matter": {
    "from_form": "initial matter state/form",
    "to_form": "final matter state/form",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the matter form conversion"
  },
  "information": {
    "from_form": "initial information form",
    "to_form": "final information form",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the information form conversion"
  }
}
```

**CRITICAL**: Focus on FORM CONVERSIONS (transformations), not just presence.

**Magnitude scoring** (degree of transformation):
- **0.0-0.2**: Minimal transformation (minor form change)
- **0.3-0.5**: Moderate transformation (partial form conversion)
- **0.6-0.8**: Significant transformation (major form conversion)
- **0.9-1.0**: Complete transformation (total form conversion)

**Energy Transformation Examples**:
- electrical → chemical (battery charging)
- thermal → mechanical (heat engine)
- mechanical → thermal (friction)
- chemical → electrical (discharging)
- electrical → thermal (resistive heating)

**Matter Transformation Examples**:
- solid → liquid (melting)
- liquid → gas (evaporation)
- dispersed ions → intercalated ions (lithium insertion)
- separate powders → mixed powder (mechanical mixing)
- reactants → products (chemical reaction)

**Information Transformation Examples**:
- raw voltage → state-of-charge data (measurement processing)
- temperature reading → control signal (feedback loop)
- manual observation → recorded data (documentation)
- sensor signal → alarm trigger (threshold detection)
- process state → decision (human judgment)

---

## Few-Shot Examples

### Example 1: Battery Charging

**Step**: "Charge lithium battery at 0.5C to 4.2V"

**Transformation characterization**:
```json
{
  "energy": {
    "from_form": "electrical",
    "to_form": "chemical_potential",
    "magnitude": 0.9,
    "rationale": "Electrical energy is nearly completely converted to chemical potential energy stored in lithium ions"
  },
  "matter": {
    "from_form": "lithium_ions_dispersed_in_electrolyte",
    "to_form": "lithium_ions_intercalated_in_electrode",
    "magnitude": 0.7,
    "rationale": "Majority of lithium ions migrate and intercalate into electrode structure, changing their bonding state"
  },
  "information": {
    "from_form": "voltage_measurement",
    "to_form": "battery_state_data",
    "magnitude": 0.5,
    "rationale": "Voltage readings are transformed into derived battery state-of-charge information"
  }
}
```

---

### Example 2: Thermal Heating

**Step**: "Heat lithium carbonate to 800°C in nitrogen atmosphere"

**Transformation characterization**:
```json
{
  "energy": {
    "from_form": "thermal_ambient",
    "to_form": "thermal_high_temperature",
    "magnitude": 0.8,
    "rationale": "Significant thermal energy is added to raise material from ambient (~25°C) to high temperature (800°C)"
  },
  "matter": {
    "from_form": "lithium_carbonate_room_temp_solid",
    "to_form": "lithium_carbonate_high_temp_solid_or_decomposed",
    "magnitude": 0.6,
    "rationale": "Material undergoes thermal expansion and may partially decompose, changing crystalline structure"
  },
  "information": {
    "from_form": "target_temperature_instruction",
    "to_form": "controlled_heating_process",
    "magnitude": 0.4,
    "rationale": "Temperature setpoint is transformed into controlled heating action via furnace control system"
  }
}
```

---

### Example 3: Mechanical Mixing

**Step**: "Mix cobalt oxide and lithium carbonate powders at 200 RPM for 30 minutes"

**Transformation characterization**:
```json
{
  "energy": {
    "from_form": "electrical_motor",
    "to_form": "mechanical_rotation",
    "magnitude": 0.7,
    "rationale": "Electrical energy is converted to mechanical rotational energy for mixing action"
  },
  "matter": {
    "from_form": "separated_powder_layers",
    "to_form": "homogeneously_mixed_powder",
    "magnitude": 0.8,
    "rationale": "Two distinct powder layers are transformed into a uniformly mixed composite powder"
  },
  "information": {
    "from_form": "mixing_parameters_instruction",
    "to_form": "process_execution_data",
    "magnitude": 0.3,
    "rationale": "RPM and time parameters are transformed into actual mixing process execution record"
  }
}
```

---

### Example 4: Electrochemical Decomposition

**Step**: "Electrolyze water at 2V to produce hydrogen and oxygen"

**Transformation characterization**:
```json
{
  "energy": {
    "from_form": "electrical",
    "to_form": "chemical_bond_energy",
    "magnitude": 0.9,
    "rationale": "Electrical energy breaks H-O bonds and forms H-H and O-O bonds, a complete energy form conversion"
  },
  "matter": {
    "from_form": "water_molecules_H2O",
    "to_form": "hydrogen_gas_H2_and_oxygen_gas_O2",
    "magnitude": 1.0,
    "rationale": "Complete chemical transformation from liquid water molecules to gaseous hydrogen and oxygen"
  },
  "information": {
    "from_form": "voltage_application",
    "to_form": "gas_evolution_observation",
    "magnitude": 0.5,
    "rationale": "Applied voltage information is transformed into observable gas bubble formation data"
  }
}
```

---

## Usage Workflow

1. **Open JSONL file** (`.artifacts/{run_id}/stage1/steps.jsonl`) in VS Code
2. **Select a step's `text_raw` field**
3. **Open Copilot Chat** (`Ctrl+Shift+I` or `Cmd+Shift+I`)
4. **Paste this prompt template** and replace `{paste step text here}` with actual step text
5. **Include 1-2 relevant examples** from above to guide Copilot
6. **Review Copilot's JSON response**:
   - Verify `from_form` and `to_form` are clearly different (not redundant)
   - Check magnitude reflects actual degree of transformation
   - Ensure rationale explains the conversion, not just describes the step
7. **Adjust if needed** based on domain expertise
8. **Copy JSON** and add as `transformation_tags` field to the step in JSONL

---

## Transformation-Focused Validation Checklist

After characterizing 5-10 steps, verify:

- ✅ Do `from_form` and `to_form` describe actual conversions (A→B), not just states?
- ✅ Does magnitude reflect transformation degree (0=no change, 1=complete conversion)?
- ✅ Does rationale explain WHY the transformation occurs, not just WHAT happens?
- ✅ Are transformations specific enough (avoid generic terms like "energy" without form)?

**Common mistakes to avoid**:

❌ **Wrong**: `"from_form": "thermal", "to_form": "thermal"` (no transformation!)
✅ **Correct**: `"from_form": "thermal_ambient", "to_form": "thermal_high_temp"` (temperature change is a transformation)

❌ **Wrong**: `"rationale": "The step involves heating"` (describes action, not transformation)
✅ **Correct**: `"rationale": "Thermal energy is added to transform material from room temperature solid to high-temperature activated state"` (explains conversion)

---

## Advanced: Batch Processing with Copilot

For processing multiple similar steps:

```
Analyze these 5 heating steps and characterize transformations for each.

Steps:
1. "Heat to 95°C"
2. "Heat lithium carbonate to 800°C"
3. "Heat mixture at 200°C for 2 hours"
4. "Cool to room temperature"
5. "Maintain at 150°C"

Return array of transformation_tags objects (same JSON structure as above).

Focus on FORM CONVERSIONS. Note that cooling (step 4) is also a transformation (high temp → low temp).
```

---

**Last Updated**: 2025-10-28  
**Version**: 2.0.0 (Transformation-focused revision)  
**Related**: [data-model.md](../data-model.md), [quickstart.md](../quickstart.md)
