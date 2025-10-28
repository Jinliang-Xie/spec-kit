# Patent JSON Data Profile (Canonical)

Date: 2025-10-28
Source folder: `IG_start/data/raw_patent_data`
Scope: Observed schema and value patterns from a representative sample across CN, US, EP, JP, and WO files. English is canonical; a Chinese counterpart exists at `docs/data-profile.zh-CN.md`.

## Sample set reviewed

- WO-9713286-A1.json
- US-7611801-B2.json, US-2023136430-A1.json, US-2021242466-A1.json, US-11604523-B2.json
- EP-1110255-A1.json, EP-4354638-A1.json, EP-4071911-A1.json, EP-4102607-A1.json
- JP-S59127382-A.json, JP-2957756-B2.json, JP-H01144574-A.json
- CN-105162225-B.json, CN-116438709-A.json, CN-209767221-U.json, CN-213124660-U.json

Note: The dataset is larger; this profile will evolve with broader scanning.

## Top-level field inventory (observed)

- `isRelated` (bool-like): often `true`.
- `innovation_object` (string): short label of the object of improvement.
- `innovation_component` (array of strings): may include domain components; sometimes contains literal string `"null"`.
- `technical_effect` (array of strings): descriptive effects; Chinese prose.
- `application_field` (string): comma/顿号-separated list; primarily Chinese.
- `battery_type` (array of strings): high-level use (动力电池/储能/其他...).
- `technical_process` (string): step-like content joined by `->`. Often present and highly informative for step identification.
- `electrochemical_battery_type` (array of strings): chemistry; may include `"null"`.
- `technical_parameters` (object): nested metric entries (see below).
- `cycle_life`, `self_discharge_rate`, `operating_temperature` (object): may contain numeric fields and `unit`.
- `dimensions` (object): `length`, `width`, `height`, `unit`; units vary (mm, cm); zero used as placeholder frequently.
- `weight` (object): `value`, `unit`; often zero or empty unit.
- `cathode_material`, `anode_material`, `electrolyte_type`, `electrolyte_solute` (string): `electrolyte_type` may be `"null"`.
- `separator` (object): `base_material`, `coating`, `thickness{value,unit}`, `porosity{value,unit}`; many zeros.
- `sustainability`, `cost_optimization`, `safety` (object): each contains sub-objects with `value` ("1"/"0") and `actions` string.

## `technical_parameters` structure

Per-parameter objects commonly include:
- Redundant name key mirroring the parameter (e.g., `nominal_capacity.nominal_capacity`)
- `unit` (string): may be empty.
- `value` (string flag): "1" indicates present/claimed; "0" indicates absent/unspecified. Not a numeric magnitude.
- `actions` (string): provenance/explanation text.
- Additional numeric (when present): e.g., `nominal_voltage: 4.5` with `unit: V`.

Observed parameters include: `nominal_capacity`, `nominal_voltage`, `charge_rate`, `discharge_rate`, `power_density`, `energy_density`, `internal_resistance`, `energy_efficiency`.

## Data quirks and anomalies

- Placeholder zeros: numeric fields frequently set to `0` with empty `unit` — these mean "unknown/unspecified", not literal zero.
- String flags: many `value` fields are string flags ("1"/"0"). Treat as booleans `has_value`.
- Literal string "null": appears in arrays (e.g., `electrochemical_battery_type: ["null"]`) and strings (e.g., `electrolyte_type: "null"`). Map to `unknown`.
- Units: mixed use of `mm` and `cm`; temperatures appear as `°C` or `℃`; electrical units include `V` and `C` (C-rate), capacity as `mAh` or `Ah`.
- Language: Field values primarily Chinese, including list items; separators include Chinese comma `，`, enumeration comma `、`, and English comma `,`.

## Parsing and normalization heuristics

1) Steps
- If `technical_process` exists, split by `->` into ordered `Step`s. Preserve order via `order_index`.
- Keep `text_raw`; later refine/atomicize conjunctive content without over-expanding loops/conditions.

2) Parameters
- For each parameter object:
  - Parse numeric magnitude from the redundant key (e.g., `nominal_voltage.nominal_voltage`) when present.
  - Interpret `value: "1"|"0"` as boolean `has_value`.
  - If numeric is 0 and `unit` empty, mark as `unknown` (not 0).
  - Retain `actions` as provenance free-text.
- Dual-track storage:
  - Raw: `raw_text`, `raw_numeric`, `raw_unit`, `source_field`.
  - Normalized: `normalized_value`, `unit_std` (SI or industry-common), `normalization_rules_version`, `confidence`.
- Unit mapping examples:
  - Capacity: mAh <-> Ah (normalize to mAh or Ah consistently; record original).
  - Temperature: `℃` == `°C`.
  - Length: `cm` to `mm` (x10) if standardizing to `mm`.
  - C-rate: unit `C` stays as-is; numeric should be float.

3) Categorical cleanup
- Arrays containing `"null"`: drop or map those entries to `unknown`.
- `electrolyte_type`: map "液态/固态/半固态/其他/未知"; keep raw value.
- Split list-like strings (e.g., `application_field`) by `、`, `，`, and `,`; trim whitespace; attach `lang: zh` when detected.

## Parameter dictionary v0 (observed + suggested)

- nominal_capacity (mAh/Ah)
- nominal_voltage (V)
- charge_rate (C)
- discharge_rate (C)
- power_density (W/kg)
- energy_density (Wh/kg)
- internal_resistance (mΩ)
- cycle_life (cycles)
- self_discharge_rate (%)
- operating_temperature_min/max (°C)
- separator_thickness (mm)
- porosity (%)
- dimensions_length/width/height (+ unit)

## Examples (abridged)

- `technical_process`: "A->B->C" (CN-116438709-A) — split into 3 ordered steps.
- Parameter with numeric + unit: `nominal_voltage: 4.5 V` + `actions` context.
- Parameter flags: `value: "1"` indicates presence; numeric may still be 0 — treat as unknown magnitude but `has_value: true`.
- Categorical with `"null"`: `electrolyte_type: "null"` or `electrochemical_battery_type: ["null"]`.

## Known edge cases to guard

- Entire `technical_parameters` object present but all numeric zeros and empty units.
- Mixed unit systems in `dimensions` and `separator.thickness`.
- Long prose in `actions` with embedded numbers; avoid mis-parsing as magnitudes unless clearly structured.

## Impact on constitution (v0.2.0)

- Reaffirm two-lane input contract and robust parsing rules (0/empty/"null" => unknown; flags => booleans).
- Keep `actions` and `text_raw` for provenance.
- Prioritize `technical_process` for step extraction; annotate when absent.

## Next steps

- Expand sampling coverage and compute simple field presence stats.
- Draft JSON Schemas for `technical_parameters` and `technical_process` presence.
- Wire minimal unit normalization utilities and tests.
