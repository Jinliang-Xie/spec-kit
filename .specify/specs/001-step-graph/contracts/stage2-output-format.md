# Stage 2 Output Format: Transformation Encoding

**Purpose**: Document the JSONL output format for Stage 2 (Transformation Encoding) with energy/matter/information transformation characterizations.

**File location**: `.artifacts/{run_id}/stage2/steps_encoded.jsonl`

**Key Innovation**: This stage adds **transformation tags** showing how energy, matter, and information **convert from one form to another** (not just involvement magnitude).

---

## JSONL Format Overview

Each line inherits Stage 1 fields and adds `transformation_tags` object:

```jsonl
{"step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8", "run_id": "550e8400-e29b-41d4-a716-446655440000", "patent_id": "US-10123456-B2", "order_index": 1, "text_raw": "Heat mixture to 95°C for 30 minutes", "extraction_confidence": 0.98, "parameters": [...], "transformation_tags": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.85, "rationale": "Electrical heating element converts electricity to thermal energy"}, "matter": {...}, "information": {...}}, "timestamp": "2024-10-28T15:00:00Z"}
```

---

## Transformation Tags Object

**Purpose**: Characterize **how** each dimension transforms during this step (from_form → to_form).

### Structure

```json
"transformation_tags": {
  "energy": {
    "from_form": "source energy form",
    "to_form": "resulting energy form",
    "magnitude": 0.0-1.0,
    "rationale": "Explanation of transformation"
  },
  "matter": {
    "from_form": "initial matter state/form",
    "to_form": "resulting matter state/form",
    "magnitude": 0.0-1.0,
    "rationale": "Explanation of transformation"
  },
  "information": {
    "from_form": "source information form",
    "to_form": "resulting information form",
    "magnitude": 0.0-1.0,
    "rationale": "Explanation of transformation"
  }
}
```

### Field Specifications

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `from_form` | string | Yes | Initial state/form of the dimension |
| `to_form` | string | Yes | Resulting state/form after transformation |
| `magnitude` | number (0-1) | Yes | Degree of transformation (0=no change, 1=complete conversion) |
| `rationale` | string | Yes | Human-readable explanation of the transformation |

**Critical Design Principle**: `magnitude` represents **how much transformation occurs**, not just involvement. A step can have high energy involvement but low transformation if it maintains existing energy form.

---

## Energy Transformation

### Common Energy Forms

| Form | Description | Examples |
|------|-------------|----------|
| `electrical` | Electrical energy | AC/DC current, voltage |
| `thermal` | Heat energy | Temperature increase/decrease |
| `mechanical` | Kinetic/potential energy | Motion, force, pressure |
| `chemical_potential` | Chemical bond energy | Battery charging, fuel synthesis |
| `electromagnetic` | EM radiation | Light, microwaves, RF |
| `acoustic` | Sound/vibration energy | Ultrasonic cleaning |
| `gravitational` | Gravitational potential | Lifting, sedimentation |
| `none` | No energy transformation | Use when from_form = to_form |

### Magnitude Guidelines for Energy

| Magnitude | Interpretation | Example |
|-----------|----------------|---------|
| **0.0-0.2** | Minimal transformation (mostly maintains form) | Electrical current flows through wire (electrical→electrical) |
| **0.3-0.5** | Partial transformation (some conversion) | Heating with 30% efficiency (electrical→thermal with losses) |
| **0.6-0.8** | Significant transformation | Standard heating (electrical→thermal ~70% conversion) |
| **0.9-1.0** | Near-complete transformation | Battery charging (electrical→chemical_potential ~95%) |

### Example: Battery Charging

```json
"energy": {
  "from_form": "electrical",
  "to_form": "chemical_potential",
  "magnitude": 0.9,
  "rationale": "Electrical energy from charger converted to chemical potential energy stored in lithium-ion battery with ~90% efficiency"
}
```

### Example: Thermal Heating

```json
"energy": {
  "from_form": "electrical",
  "to_form": "thermal",
  "magnitude": 0.85,
  "rationale": "Electrical heating element converts electricity to thermal energy heating mixture from room temperature to 95°C"
}
```

### Example: Mechanical Mixing

```json
"energy": {
  "from_form": "electrical",
  "to_form": "mechanical",
  "magnitude": 0.7,
  "rationale": "Electric motor converts electrical energy to mechanical rotation of stirring blade at 1000 rpm"
}
```

---

## Matter Transformation

### Common Matter Forms

| Form | Description | Examples |
|------|-------------|----------|
| `solid_crystalline` | Ordered solid structure | Salt crystals, metals |
| `solid_amorphous` | Disordered solid | Glass, polymers |
| `liquid_pure` | Pure liquid | Water, ethanol |
| `liquid_solution` | Dissolved mixture | Salt in water |
| `gas` | Gaseous state | Oxygen, nitrogen |
| `plasma` | Ionized gas | Plasma torch |
| `dispersed_particles` | Suspension/colloid | Powder in liquid |
| `intercalated` | Ions inserted in lattice | Lithium in graphite |
| `separated_phases` | Unmixed components | Oil and water layers |
| `homogeneous_mixture` | Well-mixed single phase | Sugar dissolved in water |

### Magnitude Guidelines for Matter

| Magnitude | Interpretation | Example |
|-----------|----------------|---------|
| **0.0-0.2** | Minimal transformation (state preserved) | Transfer liquid from vessel A to B |
| **0.3-0.5** | Partial transformation | Some dissolution (powder→solution, incomplete) |
| **0.6-0.8** | Significant transformation | Phase change (solid→liquid, complete melting) |
| **0.9-1.0** | Complete transformation | Chemical reaction (reactants→products, full conversion) |

### Example: Lithium Intercalation

```json
"matter": {
  "from_form": "lithium_ions_dispersed",
  "to_form": "lithium_ions_intercalated",
  "magnitude": 0.7,
  "rationale": "Lithium ions migrate from electrolyte and intercalate into graphite anode structure during battery charging"
}
```

### Example: Powder Mixing

```json
"matter": {
  "from_form": "separated_powders",
  "to_form": "homogeneous_mixture",
  "magnitude": 0.8,
  "rationale": "Cobalt oxide and lithium carbonate powders initially separated are mixed into uniform powder blend"
}
```

### Example: Dissolution

```json
"matter": {
  "from_form": "solid_crystalline",
  "to_form": "liquid_solution",
  "magnitude": 0.9,
  "rationale": "Salt crystals completely dissolve in water forming homogeneous solution"
}
```

---

## Information Transformation

### Common Information Forms

| Form | Description | Examples |
|------|-------------|----------|
| `voltage_signal` | Electrical voltage measurement | Sensor voltage output |
| `current_signal` | Electrical current measurement | Ammeter reading |
| `temperature_reading` | Temperature sensor data | Thermocouple output |
| `pressure_reading` | Pressure sensor data | Manometer reading |
| `optical_signal` | Light intensity/wavelength | Spectrophotometer signal |
| `digital_data` | Processed/stored data | Computer file, database |
| `control_command` | Instruction to actuator | PID controller output |
| `user_observation` | Human-interpreted information | Visual inspection |
| `calculated_state` | Derived information | Battery SOC from voltage |

### Magnitude Guidelines for Information

| Magnitude | Interpretation | Example |
|-----------|----------------|---------|
| **0.0-0.2** | Minimal information processing | Relay sensor data without processing |
| **0.3-0.5** | Basic transformation | Unit conversion (Fahrenheit→Celsius) |
| **0.6-0.8** | Significant processing | Calculate average from multiple readings |
| **0.9-1.0** | Complex transformation | ML model inference from sensor data |

### Example: Battery State Calculation

```json
"information": {
  "from_form": "voltage_measurement",
  "to_form": "battery_state_data",
  "magnitude": 0.7,
  "rationale": "Voltage measurements transformed into state-of-charge (SOC) estimate using lookup table or algorithm"
}
```

### Example: Temperature Monitoring

```json
"information": {
  "from_form": "temperature_sensor_signal",
  "to_form": "control_feedback",
  "magnitude": 0.5,
  "rationale": "Temperature sensor data used as feedback for PID controller to maintain setpoint"
}
```

### Example: Spectrophotometry

```json
"information": {
  "from_form": "optical_signal",
  "to_form": "concentration_data",
  "magnitude": 0.8,
  "rationale": "Absorbance at 450nm transformed into concentration value using Beer-Lambert law calibration"
}
```

---

## Complete Example: Battery Charging Step

```json
{
  "step_id": "a3bb189e-8bf9-4c5d-a12a-1b9b15c6e7f8",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10123456-B2",
  "order_index": 1,
  "text_raw": "Charge battery at 0.5C to 4.2V cutoff while monitoring voltage",
  "extraction_confidence": 0.95,
  "parameters": [
    {
      "parameter_id": "f4a8b3c2-1d5e-4f6a-9b8c-3e2d1a0b9c8d",
      "name": "c_rate",
      "value": "0.5C",
      "normalized_value": 0.5,
      "unit": "C",
      "association_method": "explicit",
      "link_confidence": 1.0
    },
    {
      "parameter_id": "e5b9c4d3-2e6f-5a7b-0c9d-4f3e2d1c0b9a",
      "name": "voltage",
      "value": "4.2V",
      "normalized_value": 4.2,
      "unit": "volts",
      "association_method": "explicit",
      "link_confidence": 1.0
    }
  ],
  "transformation_tags": {
    "energy": {
      "from_form": "electrical",
      "to_form": "chemical_potential",
      "magnitude": 0.9,
      "rationale": "Electrical energy from charger converted to chemical potential energy stored in lithium-ion battery with ~90% efficiency during constant-current charging"
    },
    "matter": {
      "from_form": "lithium_ions_dispersed",
      "to_form": "lithium_ions_intercalated",
      "magnitude": 0.7,
      "rationale": "Lithium ions migrate from cathode through electrolyte and intercalate into graphite anode structure, changing spatial distribution and chemical bonding"
    },
    "information": {
      "from_form": "voltage_measurement",
      "to_form": "battery_state_data",
      "magnitude": 0.5,
      "rationale": "Voltage measurements during charging transformed into battery state-of-charge (SOC) and health information via battery management system"
    }
  },
  "timestamp": "2024-10-28T15:00:00Z"
}
```

---

## Complete Example: Thermal Heating Step

```json
{
  "step_id": "b4cc290f-9cga-5d6e-b23b-2c0c26d7f9g9",
  "run_id": "550e8400-e29b-41d4-a716-446655440000",
  "patent_id": "US-10789456-B1",
  "order_index": 3,
  "text_raw": "Heat mixture to 95°C for 30 minutes while stirring",
  "extraction_confidence": 0.98,
  "parameters": [
    {
      "parameter_id": "d3aa078e-7ce8-3b5a-8a1a-0a8a04b5c6d7",
      "name": "temperature",
      "value": "95°C",
      "normalized_value": 95.0,
      "unit": "celsius",
      "association_method": "explicit",
      "link_confidence": 1.0
    },
    {
      "parameter_id": "c2bb067d-6bd7-2a4a-7a0a-9a7a03b4c5d6",
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
      "magnitude": 0.85,
      "rationale": "Electrical heating element converts electricity to thermal energy, raising mixture temperature from room temperature (~25°C) to 95°C"
    },
    "matter": {
      "from_form": "liquid_mixture_ambient_temp",
      "to_form": "liquid_mixture_elevated_temp",
      "magnitude": 0.6,
      "rationale": "Liquid mixture undergoes thermal elevation from 25°C to 95°C, increasing molecular kinetic energy and potentially affecting dissolution or reaction rates"
    },
    "information": {
      "from_form": "temperature_sensor_signal",
      "to_form": "control_feedback",
      "magnitude": 0.4,
      "rationale": "Temperature measurements provide feedback to heating controller to maintain 95°C setpoint during 30-minute duration"
    }
  },
  "timestamp": "2024-10-28T15:05:00Z"
}
```

---

## Common Transformation Patterns

### Pattern 1: Electrochemical Processes

**Charging**:
- Energy: `electrical → chemical_potential` (magnitude ~0.9)
- Matter: `ions_dispersed → ions_intercalated` (magnitude ~0.7)
- Information: `voltage_measurement → battery_state_data` (magnitude ~0.5)

**Discharging**:
- Energy: `chemical_potential → electrical` (magnitude ~0.9)
- Matter: `ions_intercalated → ions_dispersed` (magnitude ~0.7)
- Information: `current_measurement → power_delivery_data` (magnitude ~0.5)

### Pattern 2: Thermal Processes

**Heating**:
- Energy: `electrical → thermal` or `chemical → thermal` (magnitude ~0.7-0.9)
- Matter: `ambient_temp → elevated_temp` (magnitude ~0.5-0.7)
- Information: `temperature_sensor → control_feedback` (magnitude ~0.3-0.5)

**Cooling**:
- Energy: `thermal → ambient` (magnitude ~0.6-0.8)
- Matter: `elevated_temp → ambient_temp` (magnitude ~0.5-0.7)
- Information: `temperature_sensor → control_feedback` (magnitude ~0.3-0.5)

### Pattern 3: Mechanical Processes

**Mixing**:
- Energy: `electrical → mechanical` (magnitude ~0.7)
- Matter: `separated_materials → homogeneous_mixture` (magnitude ~0.8)
- Information: `none → none` or `visual_observation → mixing_quality` (magnitude ~0.2)

**Grinding**:
- Energy: `electrical → mechanical` (magnitude ~0.8)
- Matter: `coarse_particles → fine_particles` (magnitude ~0.9)
- Information: `none → none` or `particle_size_measurement → distribution_data` (magnitude ~0.3)

### Pattern 4: Measurement Processes

**Spectrophotometry**:
- Energy: `electrical → electromagnetic` (magnitude ~0.5)
- Matter: `sample_in_cuvette → sample_in_cuvette` (magnitude ~0.0, no change)
- Information: `optical_signal → concentration_data` (magnitude ~0.8)

**pH Measurement**:
- Energy: `none → none` or `chemical → electrical` (magnitude ~0.2)
- Matter: `solution → solution` (magnitude ~0.0, no change)
- Information: `voltage_signal → ph_value` (magnitude ~0.7)

---

## Validation Rules

### Transformation Consistency

✅ **Valid transformations**:
- `from_form` ≠ `to_form` (actual transformation occurred)
- `magnitude` > 0.3 (meaningful transformation)
- `rationale` explains the conversion process

⚠️ **Questionable transformations** (flag for review):
- `from_form` = `to_form` AND `magnitude` > 0.5 (contradictory)
- `magnitude` < 0.2 (minimal transformation, consider marking as `none`)

❌ **Invalid transformations**:
- Missing required fields (`from_form`, `to_form`, `magnitude`, `rationale`)
- `magnitude` outside 0-1 range

### Common Mistakes to Avoid

1. **Mistake**: `energy: {from_form: "thermal", to_form: "thermal", magnitude: 0.8}`
   - **Problem**: No transformation (from_form = to_form) but high magnitude
   - **Fix**: Identify actual energy source (e.g., `electrical → thermal`)

2. **Mistake**: `matter: {from_form: "solid", to_form: "liquid", magnitude: 0.1}`
   - **Problem**: Phase change is complete transformation (magnitude should be ~0.8-0.9)
   - **Fix**: Increase magnitude or clarify if only partial melting

3. **Mistake**: `information: {from_form: "none", to_form: "data", magnitude: 0.0}`
   - **Problem**: Transformation occurred (none→data) but magnitude is 0
   - **Fix**: Set magnitude to appropriate value (e.g., 0.7 for measurement)

---

## Usage Workflow

### Creating Stage 2 JSONL (Copilot-Assisted)

1. **Load Stage 1 JSONL** (`.artifacts/{run_id}/stage1/steps.jsonl`)

2. **For each step**, use Copilot Chat with transformation encoding prompt (see `copilot-prompts/stage2-transformation-encoding.md`)

3. **Review Copilot's transformation tags**:
   - Verify `from_form → to_form` makes physical sense
   - Check magnitude matches transformation degree
   - Validate rationale explains the conversion

4. **Manually adjust** if needed:
   - Correct form names (use consistent vocabulary)
   - Adjust magnitudes based on domain knowledge
   - Clarify rationales

5. **Append transformation_tags** to each step object

6. **Export to Stage 2 JSONL**: `.artifacts/{run_id}/stage2/steps_encoded.jsonl`

### Reading Stage 2 JSONL

**Python example**:

```python
import json

# Load encoded steps
encoded_steps = []
with open('.artifacts/{run_id}/stage2/steps_encoded.jsonl', 'r') as f:
    for line in f:
        step = json.loads(line)
        encoded_steps.append(step)

# Filter steps with high energy transformation
high_energy_transform = [
    s for s in encoded_steps 
    if s['transformation_tags']['energy']['magnitude'] >= 0.8
]

# Find all electrical→chemical transformations (e.g., battery charging)
charging_steps = [
    s for s in encoded_steps
    if s['transformation_tags']['energy']['from_form'] == 'electrical'
    and s['transformation_tags']['energy']['to_form'] == 'chemical_potential'
]
```

---

## Troubleshooting

### Issue: Unclear what transformation occurred

**Problem**: Step text is vague (e.g., "Process mixture")

**Solution**:
- Set magnitude conservatively (~0.5)
- Use generic forms (e.g., `matter: {from_form: "mixture_initial", to_form: "mixture_processed"}`)
- Flag for domain expert review
- Add note in `rationale` about uncertainty

### Issue: Multiple transformations in one step

**Problem**: Step involves multiple energy/matter/information conversions (e.g., "Heat while stirring and monitoring")

**Solution**:
- Identify **primary** transformation for each dimension
- Use magnitude to reflect relative importance
- Consider splitting step in Stage 3 (Atomicization) if too complex

### Issue: No transformation occurred

**Problem**: Step is purely descriptive or organizational (e.g., "Prepare workspace")

**Solution**:
- Set all magnitudes to 0.0-0.2
- Use `from_form = to_form` with `rationale` explaining no change
- Consider excluding step if it's not a process action

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Transformation-focused schema)  
**Related**: [stage3-output-format.md](./stage3-output-format.md), [stage1-output-format.md](./stage1-output-format.md), [copilot-prompts/stage2-transformation-encoding.md](../copilot-prompts/stage2-transformation-encoding.md), [plan.md](../plan.md)
