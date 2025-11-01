# `materials`

The `materials` array enumerates all consumables, reagents, and samples referenced throughout `steps`. Each entry becomes an addressable resource for procedural actions.

## Field Reference
| Field                 | Type   | Required | Units                              | Allowed Types    | Constraints / Enum                          | Example                       |
| --------------------- | ------ | -------- | ---------------------------------- | ---------------- | ------------------------------------------- | ----------------------------- |
| `id`                  | string | ✅       | -                                  | string           | Unique per file                             | `"m_buffer"`                  |
| `name`                | string | ✅       | -                                  | string           | -                                           | `"Lysis buffer"`              |
| `purity`              | number | optional | %                                  | float            | 0–100                                       | `99`                          |
| `concentration`       | number | optional | mM, µM, mg/mL, mol/L               | float            | ≥0                                          | `10`                          |
| `storage_temperature` | number | optional | °C                                 | float            | −196–200                                    | `-20`                         |
| `hazards`             | list   | optional | -                                  | array[string]    | Controlled vocabulary (`"flammable"`, …)    | `["toxic"]`                   |

Additional metadata such as `lot`, `catalog_number`, `supplier`, `barcode`, `storage_conditions`, or `attachments` can be included for richer traceability even though they are not mandated by the core schema.

## Units and Quantities
- Always pair numeric values with explicit units; omit qualitative words like `"ambient"`.
- When a property requires both magnitude and unit, use either:
  - Separate numeric + unit keys (as shown for `concentration` + `concentration_unit` in the specification example), or
  - Structured objects:  
    ```yaml
    concentration:
      value: 10
      unit: "mM"
    ```
- Temperature limits should stay within the validator range (−196 °C for liquid nitrogen to 200 °C for sterilization settings).

## Hazard Vocabulary
Use standardized terminology to improve cross-lab safety integrations. Suggested values include:

| Hazard Tag     | Meaning / Example Source                                                   |
| -------------- | -------------------------------------------------------------------------- |
| `"flammable"`  | Highly flammable; reference [GHS H225](https://echa.europa.eu/substance-information) |
| `"toxic"`      | Acute toxicity; cross-check supplier Safety Data Sheet (SDS).              |
| `"corrosive"`  | Causes severe skin burns; typically requires acid/base handling PPE.       |
| `"biohazard"`  | Infectious agents or biological waste.                                     |
| `"oxidizer"`   | Strong oxidizing agent; store separately from organics.                    |
| `"cryogenic"`  | Extremely cold materials; mandates face shield and gloves.                 |

## Best Practices
- Keep `id` values short, lowercase, and descriptive (`m_buffer`, `m_cells`). Mirror your LIMS or inventory system to ease integration.
- Capture supplier details (`supplier`, `catalog_number`, `lot`) even if optional; they simplify reordering and audits.
- Create dedicated attachments for certificates of analysis or SDS documents and cross-reference them from `attachments`.
- Use `hazards` and optional `safety_class`/`safety_notes` to align with institutional risk assessments.

## Validation Notes
- Every ID referenced in `steps[].with` must already exist in `materials`; missing references trigger `R`-class integrity errors.
- Validators enforce numeric ranges (e.g., `purity` must fall within 0–100). Out-of-range values raise `Q`-class quantitative errors.
- Units must be SI-derived or part of the validator’s approved unit registry. Spell out prefixes (`µL`, `mg/mL`) exactly.

## Additional Reading
- [`materials` Field Details](../index.md#62-materials)
- [Validation Rules and Integrity Model](../index.md#8-validation-rules-and-integrity-model)
