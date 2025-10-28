# Lane A Input Format

**Purpose**: Document the expected JSON format for semi-structured patent input files with pre-extracted technical parameters.

**When to use Lane A**: When you have patents with existing parameter extraction (e.g., from NLP pipeline, manual annotation, or structured database export).

---

## JSON Structure

### Minimal Example

```json
{
  "patent_id": "US-10123456-B2",
  "title": "Method for quantum entanglement stabilization",
  "description": "The present invention relates to a method for stabilizing quantum entanglement. The method comprises: Step 1: Prepare a substrate at cryogenic temperature (4K). Step 2: Apply electromagnetic pulse at 2.4 GHz for 500 ns..."
}
```

### Complete Example with Parameters

```json
{
  "patent_id": "US-10123456-B2",
  "title": "Method for quantum entanglement stabilization",
  "abstract": "A novel approach to stabilizing quantum entanglement in solid-state systems using controlled electromagnetic pulses.",
  "description": "The present invention relates to a method for stabilizing quantum entanglement. The method comprises: Step 1: Prepare a substrate at cryogenic temperature (4K). Step 2: Apply electromagnetic pulse at 2.4 GHz for 500 ns. Step 3: Measure coherence time using interferometry...",
  "filing_date": "2023-03-15",
  "grant_date": "2024-01-20",
  "applicant": "Quantum Dynamics Inc.",
  "inventors": ["Dr. Alice Quantum", "Dr. Bob Entangle"],
  
  "technical_parameters": {
    "temperature": [
      {
        "value": "4K",
        "context": "cryogenic substrate preparation",
        "uncertainty": "±0.1K"
      }
    ],
    "frequency": [
      {
        "value": "2.4 GHz",
        "context": "electromagnetic pulse frequency"
      }
    ],
    "time_duration": [
      {
        "value": "500 ns",
        "context": "pulse duration"
      }
    ]
  },
  
  "metadata": {
    "source": "USPTO",
    "extraction_date": "2024-10-28T14:30:00Z",
    "extraction_version": "v1.0.0"
  }
}
```

---

## Field Specifications

### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `patent_id` | string | Unique patent identifier with format `{country}-{number}-{kind}` | `"US-10123456-B2"` |
| `title` | string | Patent title (1-500 chars) | `"Method for quantum entanglement stabilization"` |
| `description` | string | Full patent description text (primary source for step extraction) | `"The present invention relates to..."` |

**Note**: The `description` field should contain the **full technical description** section of the patent, including all process steps, examples, and procedural details.

### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `abstract` | string | Patent abstract (max 5000 chars) | `"A novel approach to..."` |
| `filing_date` | string (ISO 8601) | Filing date `YYYY-MM-DD` | `"2023-03-15"` |
| `grant_date` | string (ISO 8601) | Grant date `YYYY-MM-DD` | `"2024-01-20"` |
| `applicant` | string | Primary applicant/assignee name (max 500 chars) | `"Acme Corporation"` |
| `inventors` | array of strings | List of inventor names | `["John Doe", "Jane Smith"]` |
| `technical_parameters` | object | Pre-extracted parameters (see below) | `{"temperature": [...]}` |
| `metadata` | object | Processing metadata | `{"source": "USPTO"}` |

---

## Technical Parameters Object

**Purpose**: Pre-extracted parameters to be linked with process steps during Stage 1.

**Advantage**: Improves step-parameter association accuracy and reduces manual extraction burden.

### Parameter Categories

Each category is an array of parameter objects with the structure:

```json
{
  "value": "string with unit",
  "context": "where/how parameter is used",
  "uncertainty": "tolerance or uncertainty (optional)"
}
```

### Supported Categories

| Category | Description | Example Values |
|----------|-------------|----------------|
| `temperature` | Temperature parameters | `"95°C"`, `"203°F"`, `"368K"`, `"95-100°C"` |
| `time_duration` | Time/duration parameters | `"30 minutes"`, `"1.5 hours"`, `"3600 seconds"` |
| `pressure` | Pressure parameters | `"2 atm"`, `"202.65 kPa"`, `"1500 mmHg"` |
| `concentration` | Concentration parameters | `"0.5 M"`, `"500 mM"`, `"5% w/v"` |
| `voltage` | Voltage/electrical potential | `"12V"`, `"3.3 kV"`, `"110-220V AC"` |
| `current` | Electrical current | `"2.5 A"`, `"500 mA"`, `"10-20 mA"` |
| `frequency` | Frequency (RF, acoustic) | `"2.4 GHz"`, `"50 Hz"`, `"440 kHz"` |
| `mass` | Mass/weight | `"10 g"`, `"2.5 kg"`, `"100 mg"` |
| `volume` | Volume | `"500 mL"`, `"1 L"`, `"10 μL"` |
| `flow_rate` | Flow rate | `"10 mL/min"`, `"2 L/hr"`, `"500 μL/s"` |
| `rpm` | Rotational speed | `"1000 rpm"`, `"3000-5000 rpm"` |
| `wavelength` | Wavelength (light, radiation) | `"532 nm"`, `"1064 nm"`, `"UV 254 nm"` |
| `ph` | pH parameters | `"7.4"`, `"pH 3.5"`, `"6.8-7.2"` |
| `distance` | Distance/length | `"10 cm"`, `"2.5 mm"`, `"500 μm"` |
| `angle` | Angle | `"45°"`, `"90 degrees"`, `"π/4 radians"` |
| `other` | Unclassified parameters | See below |

### Other Parameters

For parameters not matching standard categories, use `other` with additional `name` field:

```json
"other": [
  {
    "name": "magnetic field strength",
    "value": "1.5 Tesla",
    "context": "applied during cooling step"
  },
  {
    "name": "laser power",
    "value": "10 W",
    "context": "continuous wave excitation"
  }
]
```

---

## Complete Parameter Example

```json
"technical_parameters": {
  "temperature": [
    {
      "value": "95°C",
      "context": "heating step for solvent evaporation",
      "uncertainty": "±2°C"
    },
    {
      "value": "4K",
      "context": "cryogenic substrate preparation",
      "uncertainty": "±0.1K"
    }
  ],
  "time_duration": [
    {
      "value": "30 minutes",
      "context": "heating duration at 95°C"
    },
    {
      "value": "500 ns",
      "context": "electromagnetic pulse duration"
    }
  ],
  "voltage": [
    {
      "value": "12V",
      "context": "applied across electrodes during electroplating"
    }
  ],
  "frequency": [
    {
      "value": "2.4 GHz",
      "context": "electromagnetic pulse frequency for entanglement stabilization"
    }
  ]
}
```

---

## Metadata Object

**Purpose**: Track data source and processing history.

### Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `source` | string | Data source identifier | `"USPTO"`, `"EPO"`, `"manual_extraction"` |
| `extraction_date` | string (ISO 8601) | When data was extracted | `"2024-10-28T14:30:00Z"` |
| `extraction_version` | string | Version of extraction tool/process | `"v1.2.3"` |

```json
"metadata": {
  "source": "USPTO",
  "extraction_date": "2024-10-28T14:30:00Z",
  "extraction_version": "v1.0.0"
}
```

---

## Validation Rules

### Patent ID Format

- **Pattern**: `{COUNTRY_CODE}-{NUMBER}-{KIND_CODE}`
- **Country codes**: 2 uppercase letters (ISO 3166-1 alpha-2)
- **Number**: Digits only
- **Kind code**: Alphanumeric (e.g., `B2`, `A1`, `C`)

**Valid examples**:
- `US-10123456-B2`
- `EP-1234567-A1`
- `CN-100342579-C`

**Invalid examples**:
- `US10123456B2` (missing hyphens)
- `us-10123456-B2` (lowercase country code)
- `US-ABC-B2` (non-numeric number)

### Description Field

- **Minimum length**: 1 character (required)
- **Should contain**: Full technical description section with process steps
- **Avoid**: Only abstract or summary text (too short for meaningful extraction)

---

## Usage Guidelines

### When to Use Lane A

✅ **Use Lane A when**:
- You have structured parameter data from NLP extraction
- Parameters are already normalized and categorized
- You want to improve step-parameter linking accuracy
- Source database provides parameter metadata

❌ **Don't use Lane A when**:
- You only have raw patent text (use Lane B instead)
- Parameter extraction quality is uncertain (may introduce errors)
- You prefer manual parameter extraction during Stage 1

### Best Practices

1. **Include context**: Always provide `context` field for parameters to help with step linking
2. **Preserve units**: Keep original units in `value` field (e.g., `"95°C"` not `"95"`)
3. **Document uncertainty**: Include `uncertainty` field when available (helps assess precision)
4. **Validate patent_id**: Ensure format matches pattern before processing
5. **Full description**: Include complete technical description, not just abstract

---

## Lane A vs. Lane B Comparison

| Aspect | Lane A | Lane B |
|--------|--------|--------|
| **Input format** | JSON with `technical_parameters` | Plain JSON with only `description` |
| **Parameter source** | Pre-extracted and structured | Extracted during Stage 1 from text |
| **Complexity** | Higher (requires pre-processing) | Lower (raw text only) |
| **Accuracy** | Better step-parameter linking | Depends on extraction quality |
| **Use case** | Structured data pipelines | Manual curation, exploratory analysis |
| **MVP scope** | Not prioritized (focus on Lane B) | **Primary focus** |

**For MVP**: Use Lane B (simple text input) to minimize pre-processing complexity. Lane A support can be added later if needed for structured data integration.

---

**Last Updated**: 2025-10-29  
**Version**: 2.0.0 (Simplified for Copilot-driven workflow)  
**Related**: [plan.md](../plan.md), [quickstart.md](../quickstart.md)
