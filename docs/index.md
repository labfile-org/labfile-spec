# Specification

**Version:** 1.0  
**Status:** Draft for Public Comment  
**Document Type:** Specification  
**Maintainer:** Labfile Core Team  
**URL:** [https://spec.labfile.bio](https://spec.labfile.bio)

---

## 1. Overview

A `.labfile` is a **structured YAML document** that defines a scientific protocol in a way that is both **human-readable** and **machine-verifiable**.  
It serves as a universal, auditable representation of laboratory procedures — bridging human authorship, automated execution, and regulatory traceability.

Each `.labfile` is:

- **Declarative** — describes _what happens_, not _how code runs_
- **Versioned** — the `LABFILE` header fixes the schema version
- **Validated** — syntax, semantics, and units are machine-checked
- **Auditable** — signed with a validator signature for immutability
- **Extensible** — supports external schemas under `extensions:`

---

### 1.1 Goals

| Principle                    | Description                                                                               |
| ---------------------------- | ----------------------------------------------------------------------------------------- |
| **Reproducibility**          | Explicit parameters and units ensure experiments can be reproduced exactly.               |
| **Traceability**             | Each file can link to its data, lineage, and validation record.                           |
| **Interoperability**         | Compatible with automation and FAIR standards (Autoprotocol, SiLA 2, ISA-JSON, RO-Crate). |
| **Auditability**             | Cryptographically signed validation metadata prevents silent edits.                       |
| **Human + Machine Symmetry** | Readable YAML for scientists, strict JSON model for systems.                              |

---

### 1.2 Design Philosophy

| Principle         | Description                                                              |
| ----------------- | ------------------------------------------------------------------------ |
| **Human First**   | Scientists can author and read `.labfile` directly without programming.  |
| **Machine Safe**  | Every field has a defined type and unit, convertible to JSON Schema.     |
| **FAIR-Aligned**  | Built around Findable, Accessible, Interoperable, and Reusable data.     |
| **Deterministic** | Identical input yields identical validated output — hash-stable.         |
| **Extensible**    | Extensions preserve forward-compatibility while isolating custom fields. |

---

## 2. Top-Level Structure

Every `.labfile` follows this ordered layout:

| Field                | Description                                   |
| -------------------- | --------------------------------------------- |
| **LABFILE**          | Specification version (must equal `"1.0"`).   |
| **meta**             | Metadata about authors, license, and project. |
| **materials**        | List of reagents and consumables.             |
| **devices**          | Instruments or tools used in the protocol.    |
| **steps**            | Sequential procedural actions.                |
| **expected_results** | Intended or observed outcomes.                |
| **safety**           | Biosafety and ethics information.             |
| **attachments**      | Related files, data, or analysis results.     |
| **provenance**       | Lineage and origin relationships.             |
| **extensions**       | Optional external namespaces.                 |
| **validation**       | Automatically filled attestation block.       |
| **validation_mode**  | `"strict"` or `"lenient"` validation setting. |

---

## 3. Minimal Example

The example below demonstrates the modern `meta.authors[]` format and optional project website.

```yaml
LABFILE: "1.0"

meta:
  title: "Example DNA Extraction"
  authors:
    - name: "Dr. Alice Smith"
      organization: "Tropic Biology Lab"
      role: "lead"
      website: "https://tropicbio.example.org"
    - name: "Dr. Keiko Yamamoto"
      organization: "Tokyo Bioautomation Center"
      role: "reviewer"
  lab: "Tropic Biology Lab"
  website: "https://labfile.bio/examples/dna-extraction"
  license: "CC-BY-4.0"
  review_status: "draft"
  visibility: "public"
  FAIR_status: true
  derived_from: ["10.5281/zenodo.1234567"]

materials:
  - id: m_buffer
    name: "Lysis buffer"
    purity: 99
    concentration: 10
  - id: m_sample
    name: "Blood sample"

devices:
  - id: d_centrifuge
    name: "Mini centrifuge"
    kind: "centrifuge"
    model: "Eppendorf 5418R"
    calibrated_at: "2025-01-10"

steps:
  - id: s_1
    action: "add"
    with: [m_buffer, m_sample]
    parameters:
      volume: 1 mL

  - id: s_2
    action: "centrifuge"
    use: [d_centrifuge]
    parameters:
      speed: 13000 rpm
      duration: 5 min

expected_results:
  description: "Supernatant contains lysed DNA."
  quantitative_metrics:
    - name: "DNA_yield"
      value: 20
      unit: "µg"
  method: "Spectrophotometer"

safety:
  biosafety_level: "BSL-1"
  ethics_approval_type: "IRB"
  ethics_approval_id: "IRB2025-01"
  ethics_approval_date: "2025-02-01"
  notes: "Wear gloves and protective eyewear."

attachments:
  - type: "raw_data"
    format: "csv"
    path: "./results/dna_yield.csv"
    access_level: "public"
    repository_url: "https://zenodo.org/record/1234567"
    doi: "10.5281/zenodo.1234567"

provenance:
  - relation_type: "derived_from"
    source_type: "labfile"
    doi: "10.5281/zenodo.99999"

extensions:
  my_custom_extension:
    ontology: "https://example.org/schema/biolab/v1"

validation:
  validated_by: "Labfile Validator 0.8.2"
  validated_at: "2025-10-30T14:30:00Z"
  signature: "sha256:acb123def..."

validation_mode: "strict"
```

## 4. Syntax Rules

The `.labfile` syntax follows **deterministic YAML 1.2**, using a constrained subset to guarantee
machine-safe parsing and reproducible hashing.

### 4.1 General Formatting

| Rule                  | Description                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| **Indentation**       | Two spaces per level — tabs are invalid.                                 |
| **Order**             | Top-level keys must appear exactly in the order defined in §2.           |
| **Comments**          | `#` comments allowed anywhere; ignored during validation.                |
| **IDs**               | All `id:` fields must be unique within a file.                           |
| **References**        | Any reference in `with:` or `use:` must point to a declared ID.          |
| **Units**             | All quantitative values must include explicit units (see §6.6).          |
| **Qualitative terms** | Phrases like `"room temperature"` or `"low speed"` are **invalid**.      |
| **Naming**            | Field names use `snake_case`; enumerations are case-insensitive.         |
| **Lists**             | Empty lists (`[]`) are forbidden — omit the key instead.                 |
| **Unknown keys**      | Any undeclared key outside `extensions:` triggers `E120: Unknown field`. |

### 4.2 Validation Modes

| Mode        | Behavior                                                                                |
| ----------- | --------------------------------------------------------------------------------------- |
| **strict**  | All required fields enforced; warnings treated as errors; validator signature required. |
| **lenient** | Allows missing optional metadata and out-of-range warnings; signature optional.         |

```yaml
validation_mode: "strict"
```

Validators, registries, and agents use this flag to decide whether the file must include
a `validation:` block (§6.12) and whether schema deviations block execution.

### 4.3 Canonical Serialization

Before signing, the Labfile Validator:

1. Normalizes indentation and field order.
2. Removes all comments.
3. Serializes deterministically to UTF-8 YAML (canonical form).
4. Computes a SHA-256 hash of the canonical content **excluding** the `validation:` block.

This guarantees that identical Labfiles always yield identical hashes,
making signatures reproducible and long-term verifiable.

### 4.4 Error Codes (excerpt)

| Code     | Meaning                          | Typical Cause                                           |
| -------- | -------------------------------- | ------------------------------------------------------- |
| **E001** | Invalid LABFILE version          | Header not `"1.0"`.                                     |
| **E120** | Unknown field                    | Key not in schema and not inside `extensions:`.         |
| **E205** | Missing unit                     | Numeric value lacks unit.                               |
| **E310** | Invalid reference                | `use:` or `with:` ID not found.                         |
| **E431** | Custom device lacks capabilities | `kind: "custom"` without `capabilities` in strict mode. |
| **E512** | Invalid enumeration              | Value not in allowed enum.                              |
| **E590** | Signature mismatch               | File changed post-validation.                           |

---

## 5. Field Summary

> Master table of all core sections and fields.  
> Fields marked **conditional** become required under the stated conditions.

| Section              | Field                    | Type           | Required        | Units                | Allowed Types | Constraints / Enum                                                                                                                                                                                                                                                       | Example                                                           |
| -------------------- | ------------------------ | -------------- | --------------- | -------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **LABFILE**          | `LABFILE`                | string         | ✅              | —                    | string        | Must equal `"1.0"`                                                                                                                                                                                                                                                       | `"1.0"`                                                           |
| **meta**             | `title`                  | string         | ✅              | —                    | string        | Free text                                                                                                                                                                                                                                                                | `"DNA Extraction"`                                                |
|                      | `authors[]`              | list\<object>  | ✅              | —                    | array         | Each `{ name, organization?, role?, website? }`                                                                                                                                                                                                                          | `[{ name: "Dr. Alice" }]`                                         |
|                      | `lab`                    | string         | ✅              | —                    | string        | Institution or facility                                                                                                                                                                                                                                                  | `"Tropic Biology Lab"`                                            |
|                      | `website`                | string         | optional        | —                    | URL           | Project or protocol URL                                                                                                                                                                                                                                                  | `"https://labfile.bio/p/123"`                                     |
|                      | `date`                   | date           | optional        | —                    | ISO 8601      | `YYYY-MM-DD`                                                                                                                                                                                                                                                             | `"2025-10-30"`                                                    |
|                      | `license`                | string         | ✅              | —                    | string        | SPDX/CC (e.g., `CC-BY-4.0`)                                                                                                                                                                                                                                              | `"CC-BY-4.0"`                                                     |
|                      | `language`               | string         | optional        | —                    | string        | ISO 639-1                                                                                                                                                                                                                                                                | `"en"`                                                            |
|                      | `review_status`          | string         | optional        | —                    | enum          | `draft`, `approved`, `deprecated`, `archived`                                                                                                                                                                                                                            | `"draft"`                                                         |
|                      | `visibility`             | string         | ✅              | —                    | enum          | `public`, `internal`, `private`                                                                                                                                                                                                                                          | `"public"`                                                        |
|                      | `derived_from`           | list           | optional        | —                    | array[string] | DOIs or Labfile IDs                                                                                                                                                                                                                                                      | `["10.5281/zenodo.1234567"]`                                      |
|                      | `FAIR_status`            | boolean/string | optional        | —                    | bool/enum     | `true`, `"compliant"`, `"non_compliant"`                                                                                                                                                                                                                                 | `true`                                                            |
|                      | `compliance`             | list           | optional        | —                    | array[string] | `"GLP"`, `"GMP"`, `"FAIR"`, `"ISO-9001"`                                                                                                                                                                                                                                 | `["GLP"]`                                                         |
| **materials[]**      | `id`                     | string         | ✅              | —                    | string        | Unique per file                                                                                                                                                                                                                                                          | `"m_buffer"`                                                      |
|                      | `name`                   | string         | ✅              | —                    | string        | —                                                                                                                                                                                                                                                                        | `"Lysis buffer"`                                                  |
|                      | `purity`                 | number         | optional        | %                    | float         | 0–100                                                                                                                                                                                                                                                                    | `99`                                                              |
|                      | `concentration`          | number         | optional        | mM, µM, mg/mL, mol/L | float         | ≥0                                                                                                                                                                                                                                                                       | `10`                                                              |
|                      | `storage_temperature`    | number         | optional        | °C                   | float         | −196–200                                                                                                                                                                                                                                                                 | `-20`                                                             |
|                      | `hazards`                | list           | optional        | —                    | array[string] | `"flammable"`, `"toxic"`, …                                                                                                                                                                                                                                              | `["toxic"]`                                                       |
| **devices[]**        | `id`                     | string         | ✅              | —                    | string        | Unique                                                                                                                                                                                                                                                                   | `"d_centrifuge"`                                                  |
|                      | `name`                   | string         | ✅              | —                    | string        | —                                                                                                                                                                                                                                                                        | `"Mini centrifuge"`                                               |
|                      | `kind`                   | string         | ✅              | —                    | enum          | `"centrifuge"`, `"pipette"`, `"thermal_cycler"`, `"spectrophotometer"`, `"incubator"`, `"balance"`, `"shaker"`, `"robotic_arm"`, `"freezer"`, `"microscope"`, `"biosafety_cabinet"`, `"autoclave"`, `"liquid_handler"`, `"plate_reader"`, `"flow_cytometer"`, `"custom"` | `"centrifuge"`                                                    |
|                      | `description`            | string         | **conditional** | —                    | string        | **Required if `kind` = `"custom"`**                                                                                                                                                                                                                                      | `"Ultrasonic cleaner 40 kHz"`                                     |
|                      | `capabilities`           | map            | **conditional** | —                    | object        | **Required if `kind: custom` & `validation_mode: strict`**                                                                                                                                                                                                               | `{ frequency: { unit: "kHz", min: 30, max: 45 } }`                |
|                      | `manufacturer`           | string         | optional        | —                    | string        | —                                                                                                                                                                                                                                                                        | `"Eppendorf"`                                                     |
|                      | `model`                  | string         | optional        | —                    | string        | —                                                                                                                                                                                                                                                                        | `"5418R"`                                                         |
|                      | `calibrated_at`          | date           | optional        | —                    | ISO 8601      | —                                                                                                                                                                                                                                                                        | `"2025-01-10"`                                                    |
| **steps[]**          | `id`                     | string         | ✅              | —                    | string        | Unique                                                                                                                                                                                                                                                                   | `"s_1"`                                                           |
|                      | `action`                 | string         | ✅              | —                    | string        | Controlled action term                                                                                                                                                                                                                                                   | `"centrifuge"`                                                    |
|                      | `with`                   | list           | optional        | —                    | array[string] | Material IDs                                                                                                                                                                                                                                                             | `["m_buffer"]`                                                    |
|                      | `use`                    | list           | optional        | —                    | array[string] | Device IDs                                                                                                                                                                                                                                                               | `["d_centrifuge"]`                                                |
|                      | `parameters`             | map            | optional        | —                    | map           | Quantitative; units required                                                                                                                                                                                                                                             | `{ speed: 13000 rpm }`                                            |
|                      | `execution_mode`         | string         | optional        | —                    | enum          | `manual`, `automated`, `hybrid`                                                                                                                                                                                                                                          | `"manual"`                                                        |
|                      | `runtime.status`         | string         | optional        | —                    | enum          | `pending`, `running`, `completed`, `failed`, `skipped`, `aborted`                                                                                                                                                                                                        | `"completed"`                                                     |
|                      | `documentation_level`    | string         | optional        | —                    | enum          | `standard`, `verbose`, `audit`                                                                                                                                                                                                                                           | `"standard"`                                                      |
|                      | `confirm`                | map            | optional        | —                    | object        | Manual checkpoint                                                                                                                                                                                                                                                        | `{ required: true, message: "Check clarity" }`                    |
|                      | `repeat`                 | map            | optional        | —                    | object        | Loop repetition                                                                                                                                                                                                                                                          | `{ count: 3, interval: 5 min }`                                   |
|                      | `loop`                   | map            | optional        | —                    | object        | While-condition                                                                                                                                                                                                                                                          | `{ condition: { variable: "OD600", operator: "<", value: 0.5 } }` |
|                      | `branch`                 | map            | optional        | —                    | object        | Conditional branch                                                                                                                                                                                                                                                       | `{ condition: {...}, then: "s_5", else: "s_6" }`                  |
| **expected_results** | `description`            | string         | ✅              | —                    | string        | Outcome summary                                                                                                                                                                                                                                                          | `"Color change indicates reaction"`                               |
|                      | `quantitative_metrics[]` | list\<object>  | optional        | various              | array         | `{ name, value, unit }`                                                                                                                                                                                                                                                  | `[{ name: "DNA_yield", value: 20, unit: "µg" }]`                  |
|                      | `method`                 | string         | optional        | —                    | string        | Measurement method                                                                                                                                                                                                                                                       | `"Spectrophotometer"`                                             |
|                      | `confidence_level`       | string         | optional        | —                    | enum          | `high`, `medium`, `low`, `unknown`                                                                                                                                                                                                                                       | `"high"`                                                          |
| **safety**           | `biosafety_level`        | string         | optional        | —                    | enum          | `BSL-1`, `BSL-2`, `BSL-3`, `BSL-4`, `non-applicable`                                                                                                                                                                                                                     | `"BSL-1"`                                                         |
|                      | `ethics_approval_type`   | string         | optional        | —                    | enum          | `IRB`, `IACUC`, `HREC`, `internal`, `none`                                                                                                                                                                                                                               | `"IRB"`                                                           |
|                      | `ethics_approval_id`     | string         | optional        | —                    | string        | —                                                                                                                                                                                                                                                                        | `"IRB2025-01"`                                                    |
|                      | `ethics_approval_date`   | date           | optional        | —                    | ISO 8601      | —                                                                                                                                                                                                                                                                        | `"2025-02-01"`                                                    |
|                      | `notes`                  | string         | optional        | —                    | string        | —                                                                                                                                                                                                                                                                        | `"Handle reagents with gloves"`                                   |
| **attachments[]**    | `type`                   | string         | ✅              | —                    | enum          | `raw_data`, `processed_data`, `report`, `image`, `log`, `archive`, `analysis_script`                                                                                                                                                                                     | `"raw_data"`                                                      |
|                      | `format`                 | string         | ✅              | —                    | enum          | `csv`, `json`, `xlsx`, `yaml`, `xml`, `tiff`, `jpg`, `png`, `zip`                                                                                                                                                                                                        | `"csv"`                                                           |
|                      | `path`                   | string         | ✅              | —                    | string        | File path                                                                                                                                                                                                                                                                | `"./results/output.csv"`                                          |
|                      | `repository_url`         | string         | optional        | —                    | URL           | Dataset or repo URL                                                                                                                                                                                                                                                      | `"https://zenodo.org/record/1234"`                                |
|                      | `doi`                    | string         | optional        | —                    | string        | DOI string                                                                                                                                                                                                                                                               | `"10.5281/zenodo.1234"`                                           |
|                      | `access_level`           | string         | optional        | —                    | enum          | `public`, `restricted`, `private`, `tokenized`, `paid`                                                                                                                                                                                                                   | `"public"`                                                        |
| **provenance[]**     | `relation_type`          | string         | optional        | —                    | enum          | `derived_from`, `variant_of`, `supersedes`                                                                                                                                                                                                                               | `"derived_from"`                                                  |
|                      | `source_type`            | string         | optional        | —                    | enum          | `labfile`, `dataset`, `publication`, `instrument`, `repository`, `external_db`                                                                                                                                                                                           | `"labfile"`                                                       |
|                      | `doi`                    | string         | optional        | —                    | string        | DOI or persistent ID                                                                                                                                                                                                                                                     | `"10.5281/zenodo.99999"`                                          |
| **extensions**       | _namespace_              | map            | optional        | —                    | any           | Custom namespaced fields                                                                                                                                                                                                                                                 | `{ microscopy_ext: {...} }`                                       |
| **validation**       | `validated_by`           | string         | **conditional** | —                    | string        | **Required if `validation_mode: strict`**                                                                                                                                                                                                                                | `"Labfile Validator 0.8.2"`                                       |
|                      | `validated_at`           | date-time      | **conditional** | —                    | ISO 8601      | **Required if `validation_mode: strict`**                                                                                                                                                                                                                                | `"2025-10-30T14:30:00Z"`                                          |
|                      | `signature`              | string         | **conditional** | —                    | string        | `sha256:<hex>` over canonical YAML (excl. `validation:`)                                                                                                                                                                                                                 | `"sha256:acb123def..."`                                           |
| **validation_mode**  | `validation_mode`        | string         | optional        | —                    | enum          | `strict`, `lenient`                                                                                                                                                                                                                                                      | `"strict"`                                                        |

## 6. Field Descriptions

This section describes every key in `.labfile`, their validation logic, and conditional requirements.

---

### 6.1 `meta`

Holds contextual metadata for authorship, licensing, and project information.

```yaml
meta:
  title: "Example DNA Extraction"
  authors:
    - name: "Dr. Alice Smith"
      organization: "Tropic Biology Lab"
      role: "lead"
      website: "https://tropicbio.example.org"
    - name: "Dr. Keiko Yamamoto"
      organization: "Tokyo Bioautomation Center"
      role: "reviewer"
  lab: "Tropic Biology Lab"
  website: "https://labfile.bio/examples/dna-extraction"
  license: "CC-BY-4.0"
  review_status: "approved"
  visibility: "public"
  FAIR_status: true
  derived_from: ["10.5281/zenodo.1234567"]
  compliance: ["GLP", "FAIR"]
```

**Notes**

- `license` is required to ensure legal reusability and must follow SPDX or Creative Commons identifiers.
- `authors` accepts multiple contributors; each can include `role` and `website`.
- `visibility` defines sharing level:
  - `"public"` → visible on registries.
  - `"internal"` → local network distribution only.
  - `"private"` → restricted, signature optional.
- `review_status` controls publication lifecycle (`draft`, `approved`, `deprecated`, `archived`).
- `derived_from` tracks source DOIs or Labfile IDs for lineage provenance.
- `website` at root level refers to the project or publication page.

---

### 6.2 `materials`

Defines every reagent, consumable, or biological material used.

```yaml
materials:
  - id: m_buffer
    name: "Lysis buffer"
    purity: 99
    concentration: 10
    concentration_unit: "mM"
    storage_temperature: -20
    hazards: ["toxic"]
```

**Notes**

- All `id` values must be unique and referenceable under `steps.with`.
- Numeric fields (`purity`, `concentration`, `storage_temperature`) require explicit units.
- Qualitative phrases like `"cold"` or `"ambient"` are invalid.
- Optional fields (`hazards`) use controlled vocabularies.

---

### 6.3 `devices`

Describes instruments or hardware used in the protocol.

```yaml
devices:
  - id: d_centrifuge
    name: "Mini centrifuge"
    kind: "centrifuge"
    manufacturer: "Eppendorf"
    model: "5418R"
    calibrated_at: "2025-01-10"

  - id: d_custom_cleaner
    name: "Sonic Clean 3000"
    kind: "custom"
    description: "Ultrasonic cleaning chamber operating at 40 kHz for glassware sterilization."
    capabilities:
      frequency:
        unit: "kHz"
        min: 30
        max: 45
      volume_capacity:
        unit: "L"
        min: 0.5
        max: 3
      functions: ["wash", "sterilize"]
```

**Notes**

- `kind` is **required** and must match one of the allowed types.  
  (`centrifuge`, `pipette`, `thermal_cycler`, `spectrophotometer`, `incubator`, `balance`, `shaker`, `robotic_arm`, `freezer`, `microscope`, `biosafety_cabinet`, `autoclave`, `liquid_handler`, `plate_reader`, `flow_cytometer`, `custom`)
- `calibrated_at` (optional) improves traceability.
- `capabilities` are **mandatory** for `custom` kinds in `strict` mode.
- Each capability must define a `unit` and either a numeric `min`/`max` range or discrete `functions`.
- Validators check whether local lab equipment meets these capability thresholds.

**Error Example**

```
E431: Custom device 'd_custom_cleaner' missing capabilities (required in strict mode)
```

---

### 6.4 `steps`

Defines sequential procedural actions forming the experiment.

```yaml
steps:
  - id: s_1
    action: "add"
    with: [m_buffer, m_sample]
    parameters:
      volume: 1 mL

  - id: s_2
    action: "centrifuge"
    use: [d_centrifuge]
    parameters:
      speed: 13000 rpm
      duration: 5 min
      temperature: 4 °C
    confirm:
      required: true
      message: "Ensure rotor lid is locked before start."
    repeat:
      count: 3
      interval: 10 min
```

**Rules**

- Each `step` must have a unique `id` and an `action`.
- `with:` refers to `materials.id`; `use:` refers to `devices.id`.
- `parameters:` accept quantitative keys with units.

**Typical parameter keys**

| Key             | Type    | Units          | Range / Notes | Example        |
| --------------- | ------- | -------------- | ------------- | -------------- |
| `volume`        | number  | mL, µL         | 0.1–1000      | `"1 mL"`       |
| `time`          | number  | s, min, h      | ≥0            | `"30 min"`     |
| `temperature`   | number  | °C             | −80–150       | `"37 °C"`      |
| `speed`         | number  | rpm            | 100–30000     | `"13000 rpm"`  |
| `duration`      | number  | s, min, h      | ≥0            | `"5 min"`      |
| `angle`         | number  | °              | 0–360         | `45`           |
| `pressure`      | number  | bar, psi       | ≥0            | `"1 bar"`      |
| `concentration` | number  | mol/L, mg/mL   | ≥0            | `"5 mM"`       |
| `mass`          | number  | g, mg          | ≥0            | `"1.2 g"`      |
| `repetitions`   | integer | —              | 1–1000        | `3`            |
| `wavelength`    | number  | nm             | 180–1100      | `405`          |
| `humidity`      | number  | %              | 0–100         | `60`           |
| `pH`            | number  | —              | 0–14          | `7.4`          |
| `flow_rate`     | number  | µL/min, mL/min | ≥0            | `"500 µL/min"` |
| `mix_speed`     | number  | rpm            | 0–2000        | `"800 rpm"`    |
| `distance`      | number  | mm, cm         | ≥0            | `"5 mm"`       |

**Behavioral controls**

| Field     | Type | Description                                             | Example                                                           |
| --------- | ---- | ------------------------------------------------------- | ----------------------------------------------------------------- |
| `confirm` | map  | Adds a human confirmation checkpoint before proceeding. | `{ required: true, message: "Inspect clarity" }`                  |
| `repeat`  | map  | Repeats the step a given number of times.               | `{ count: 3, interval: 5 min }`                                   |
| `loop`    | map  | Conditional repetition while an expression is true.     | `{ condition: { variable: "OD600", operator: "<", value: 0.5 } }` |
| `branch`  | map  | Conditional jump to another step or sequence.           | `{ condition: {...}, then: "s_5", else: "s_6" }`                  |

**Notes**

- No qualitative conditions allowed (e.g., “until done” is invalid).
- In `strict` mode, all numeric parameters must include explicit units.
- Behavioral blocks (`repeat`, `loop`, `branch`) enable procedural logic while keeping the file declarative.

---

### 6.5 `expected_results`

Defines measurable or qualitative outcomes of the experiment.

```yaml
expected_results:
  description: "Supernatant contains lysed DNA."
  quantitative_metrics:
    - name: "DNA_yield"
      value: 20
      unit: "µg"
    - name: "Purity_ratio"
      value: 1.8
      unit: "A260/A280"
  method: "Spectrophotometer"
  confidence_level: "high"
```

**Notes**

- `description` is required for all experiments.
- `quantitative_metrics` are optional structured records allowing automation and dashboards to display results.
- `method` describes the measurement technique used for validation (e.g., `"Spectrophotometer"`, `"ELISA reader"`).
- `confidence_level` is a qualitative indicator of result certainty (`high`, `medium`, `low`, `unknown`).

---

### 6.6 `safety`

Captures biosafety and ethical compliance requirements.

```yaml
safety:
  biosafety_level: "BSL-1"
  ethics_approval_type: "IRB"
  ethics_approval_id: "IRB2025-01"
  ethics_approval_date: "2025-02-01"
  notes: "Handle reagents with gloves and goggles."
```

**Notes**

- `biosafety_level` aligns with international standards (BSL-1 → BSL-4).
- `ethics_approval_type` may be `IRB`, `IACUC`, `HREC`, `internal`, or `none`.
- `ethics_approval_id` and `ethics_approval_date` provide traceability for institutional compliance.
- Use `notes` for extra warnings or SOP references.

---

### 6.7 `attachments`

Links related datasets, reports, or analysis files.

```yaml
attachments:
  - type: "raw_data"
    format: "csv"
    path: "./results/dna_yield.csv"
    access_level: "public"
    repository_url: "https://zenodo.org/record/1234567"
    doi: "10.5281/zenodo.1234567"
```

**Notes**

- Each attachment must specify both `type` and `format`.
- The `path` is relative to the `.labfile` directory.
- `repository_url` and `doi` enable integration with data repositories (e.g., Zenodo, Figshare).
- `access_level` defines availability:
  - `"public"` — openly accessible
  - `"restricted"` — visible only to collaborators
  - `"private"` — local storage only
  - `"tokenized"` or `"paid"` — controlled programmatically

**Error Example**

```
E312: Attachment missing type or format
```

---

### 6.8 `provenance`

Tracks lineage and origin relationships between `.labfile`s, datasets, or publications.

```yaml
provenance:
  - relation_type: "derived_from"
    source_type: "dataset"
    doi: "10.5281/zenodo.98765"
  - relation_type: "variant_of"
    source_type: "labfile"
    doi: "10.5281/zenodo.87654"
```

**Notes**

- `relation_type` can be:
  - `"derived_from"` — result or transformation of another file.
  - `"variant_of"` — minor modification or alternate protocol version.
  - `"supersedes"` — officially replaces another protocol.
- `source_type` describes the entity:
  - `"labfile"`, `"dataset"`, `"publication"`, `"instrument"`, `"repository"`, or `"external_db"`.
- Each provenance record can include DOI or internal reference.
- Helps establish traceability and versioned history across registries.

---

### 6.9 `extensions`

Namespace for domain-specific fields outside the core schema.

```yaml
extensions:
  microscopy_ext:
    objective: "40x oil"
    fluorescence_filter: "FITC"
    exposure_time: "120 ms"
```

**Notes**

- Each key under `extensions` represents a namespace.  
  For example: `microscopy_ext`, `genomics_ext`, `automation_ext`.
- All keys must be lowercase and use underscores.
- Validators **must ignore unknown namespaces** but still verify YAML structure.
- Namespaces can optionally declare a schema URL:
  ```yaml
  extensions:
    microscopy_ext:
      $schema: "https://schemas.labfile.bio/ext/microscopy/v1"
      objective: "40x oil"
  ```
- Avoid overlapping existing field names from core schema.

---

### 6.10 `validation` and `validation_mode`

Validation metadata is automatically added by the Labfile Validator after successful schema verification.

```yaml
validation:
  validated_by: "Labfile Validator 0.8.2"
  validated_at: "2025-10-30T14:30:00Z"
  signature: "sha256:acb123def..."
validation_mode: "strict"
```

**Rules**

| Mode        | Required Fields                             | Purpose                                  |
| ----------- | ------------------------------------------- | ---------------------------------------- |
| `"strict"`  | `validated_by`, `validated_at`, `signature` | Full attestation for registry submission |
| `"lenient"` | none                                        | Draft validation; fields optional        |

**Notes**

- The `signature` is a SHA-256 checksum over the canonical `.labfile` content (excluding the `validation:` block).
- `validated_at` follows ISO 8601 date-time format with UTC `Z` suffix.
- Validators automatically compute and insert these values.
- Any modification to the file after validation invalidates the signature.

**Error Example**

```
E590: Signature mismatch — file modified post-validation
```

---

### 6.11 `validation_mode` Behavior

- In **strict** mode:
  - Validators enforce all required fields and numeric units.
  - Unrecognized fields trigger hard errors.
  - All validation metadata is mandatory.
- In **lenient** mode:
  - Warnings allowed (e.g., missing optional metadata).
  - Validator may skip signature generation.
  - Suitable for drafts, sandbox testing, or internal sharing.

---

### 6.12 Validation Metadata and Attestation Logic

Describes how validation metadata interacts with registries and agents.

**Workflow**

1. Author creates `.labfile`.
2. Validator performs syntax + semantic validation.
3. If successful:
   - Adds the `validation:` block.
   - Computes SHA-256 signature.
   - Records validator name and version.
4. Registry verifies hash at upload time.
5. Any later change → mismatch → revalidation required.

**Integrity Chain**

| Proof                  | Field          | Function                          |
| ---------------------- | -------------- | --------------------------------- |
| **Validator Identity** | `validated_by` | Identifies tool and version used. |
| **Timestamp**          | `validated_at` | Confirms validation date.         |
| **Cryptographic Hash** | `signature`    | Ensures immutability.             |

This mechanism provides regulatory-grade assurance that `.labfile` instances cannot be silently modified after validation.

---

## 7. Behavioral Extensions

Behavioral extensions allow limited procedural logic inside `.labfile` without converting it into a programming language.  
They describe how steps **repeat**, **branch**, or **wait for conditions**, while maintaining declarative readability.

---

### 7.1 `repeat`

Repeats a step a defined number of times or at fixed intervals.

```yaml
steps:
  - id: s_repeat
    action: "mix"
    parameters:
      speed: 500 rpm
      duration: 30 s
    repeat:
      count: 5
      interval: 10 s
```

**Rules**

- `count` — number of repetitions (≥1).
- `interval` — waiting time between repetitions, with explicit units.
- Repetitions must not introduce side effects (e.g., cumulative additions).
- In `strict` mode, both `count` and `interval` must be defined.

---

### 7.2 `loop`

Executes a step repeatedly while a condition remains true.

```yaml
steps:
  - id: s_loop
    action: "measure"
    use: [d_spectrophotometer]
    parameters:
      wavelength: 600 nm
    loop:
      condition:
        variable: "OD600"
        operator: "<"
        value: 0.5
      check_interval: 5 min
      max_duration: 2 h
```

**Rules**

- `variable` — monitored parameter name (e.g., `"OD600"`).
- `operator` — comparison operator (`<`, `>`, `<=`, `>=`, `==`, `!=`).
- `value` — numeric threshold.
- `check_interval` — polling frequency.
- `max_duration` — safety cap to avoid infinite loops.

**Example Behavior**

> Continue measuring OD600 every 5 minutes until it reaches 0.5, or stop after 2 hours.

---

### 7.3 `branch`

Defines a conditional workflow path based on evaluated criteria.

```yaml
steps:
  - id: s_branch
    action: "analyze"
    parameters:
      wavelength: 450 nm
    branch:
      condition:
        variable: "absorbance"
        operator: ">"
        value: 0.8
      then: "s_cleanup"
      else: "s_repeat_analysis"
```

**Rules**

- `then` and `else` reference valid step IDs.
- Conditions must evaluate deterministically (numeric comparison only).
- Validators verify referenced steps exist and no cycles are created.
- Optional `log_message` field can annotate decisions for audit trails.

**Error Example**

```
E660: Branch 'then' target step not found
```

---

### 7.4 `confirm`

Prompts human confirmation before execution continues.

```yaml
steps:
  - id: s_confirm
    action: "pipette"
    with: [m_buffer]
    parameters:
      volume: 100 µL
    confirm:
      required: true
      message: "Visually confirm that liquid reached 100 µL mark."
      by: "operator"
```

**Rules**

- `required: true` → agent must pause for acknowledgment.
- `message:` gives instruction shown to user/operator.
- `by:` defines responsible role (`operator`, `reviewer`, `supervisor`).
- Skipping confirmation in strict mode triggers:
  ```
  E702: Required confirmation not acknowledged
  ```

---

### 7.5 Execution Modes

Each `step` can specify how it is expected to be carried out.

| Mode        | Description                                  | Agent Behavior                                      |
| ----------- | -------------------------------------------- | --------------------------------------------------- |
| `manual`    | Performed by human operator                  | Validator ignores missing automation metadata       |
| `automated` | Executed by robotic system                   | Requires declared devices and measurable parameters |
| `hybrid`    | Combination — human setup, machine execution | Both must be defined                                |

```yaml
steps:
  - id: s_incubate
    action: "incubate"
    execution_mode: "automated"
    use: [d_incubator]
    parameters:
      temperature: 37 °C
      duration: 30 min
```

---

### 7.6 Runtime Status Tracking

Used by agents and registries to monitor execution progress.

| Status      | Meaning                       |
| ----------- | ----------------------------- |
| `pending`   | Step queued for execution     |
| `running`   | Step in progress              |
| `completed` | Finished successfully         |
| `failed`    | Execution failed              |
| `skipped`   | Intentionally skipped         |
| `aborted`   | Stopped manually or by system |

Agents update these statuses dynamically during workflow runtime but **must not** alter any other step metadata.

---

## 8. Validation Rules and Integrity Model

This section defines the formal rules a `.labfile` must satisfy to be considered valid under version **1.0** of the specification.

---

### 8.1 Structural Rules

| Rule     | Description                                                     |
| -------- | --------------------------------------------------------------- |
| **S101** | File must begin with `LABFILE: "1.0"`.                          |
| **S102** | All top-level keys must follow the prescribed order.            |
| **S103** | Each section must be a valid YAML mapping or sequence.          |
| **S104** | Empty lists or maps are not allowed (omit instead).             |
| **S105** | Field names must use `snake_case`.                              |
| **S106** | Comments may appear anywhere and are ignored during validation. |

---

### 8.2 Identity and Reference Rules

| Rule     | Description                                                              |
| -------- | ------------------------------------------------------------------------ |
| **R201** | All `id:` values must be unique within the file.                         |
| **R202** | `with:` in `steps` must reference valid `materials.id`.                  |
| **R203** | `use:` in `steps` must reference valid `devices.id`.                     |
| **R204** | `branch.then` and `branch.else` must reference existing `steps.id`.      |
| **R205** | `provenance[].doi` must be a valid DOI string (prefix `10.`).            |
| **R206** | `attachments[].path` must point to an existing local or remote resource. |

---

### 8.3 Quantitative Rules

| Rule     | Description                                                                              |
| -------- | ---------------------------------------------------------------------------------------- |
| **Q301** | All numeric values must include a valid unit (e.g., `"5 mL"`).                           |
| **Q302** | Qualitative descriptors like `"room temperature"` or `"high speed"` are invalid.         |
| **Q303** | Units must conform to SI symbols or accepted lab abbreviations (°C, mL, rpm, etc.).      |
| **Q304** | Out-of-range values produce warnings or errors depending on `validation_mode`.           |
| **Q305** | Derived metrics (e.g., `"A260/A280"`) must be expressed as numeric ratios when possible. |

---

### 8.4 Logical Rules

| Rule     | Description                                                                               |
| -------- | ----------------------------------------------------------------------------------------- |
| **L401** | Step execution order must be deterministic.                                               |
| **L402** | No cyclic branching or looping structures allowed.                                        |
| **L403** | Conditional logic must evaluate to a resolvable boolean.                                  |
| **L404** | `repeat` and `loop` blocks cannot coexist in the same step.                               |
| **L405** | Behavioral blocks (`branch`, `loop`, `confirm`) must not modify state variables directly. |

---

### 8.5 Validation Modes

| Mode                | Effect                                                                                 |
| ------------------- | -------------------------------------------------------------------------------------- |
| **Strict**          | All schema, unit, and cross-reference rules enforced. Missing metadata triggers error. |
| **Lenient**         | Structural rules enforced; semantic errors downgraded to warnings.                     |
| **Hybrid (future)** | Context-dependent validation for training or sandbox environments.                     |

Example usage:

```yaml
validation_mode: "strict"
```

**Strict Mode Required For:**

- Registry publication
- Signature generation
- Regulatory submissions

**Lenient Mode Allowed For:**

- Internal drafts
- Human editing stages
- Local sandbox runs

---

### 8.6 Signature Integrity

When validated, the `.labfile` is **sealed** via SHA-256 hash computed over all normalized fields (excluding the `validation:` block).  
This ensures immutability and reproducibility.

**Computed Fields:**

- `validated_by`
- `validated_at`
- `signature`

**Verification Workflow:**

1. The validator parses and normalizes YAML → canonical JSON.
2. Hash is generated:
   ```
   sha256sum canonical.json
   ```
3. Resulting digest is stored under:
   ```yaml
   validation:
     signature: "sha256:abcdef123..."
   ```
4. Registries and agents recompute this hash before execution to verify integrity.

**Any modification to the file after signing requires revalidation.**

---

### 8.7 Attestation Example

```yaml
validation:
  validated_by: "Labfile Validator 0.9.1"
  validated_at: "2025-10-31T11:45:00Z"
  signature: "sha256:e6a1329e6e04b9a2e19f2b9d318dbf76a79c6ed1ce84f5..."
validation_mode: "strict"
```

**Audit Guarantees:**

- Ensures file version and validator version are bound together.
- Prevents retroactive tampering.
- Enables reproducibility and traceable publication through `spec.labfile.bio`.

---

### 8.8 Registry-Level Enforcement

Registries receiving `.labfile` submissions must verify:

| Check           | Enforcement                            |
| --------------- | -------------------------------------- |
| File syntax     | Must parse as valid YAML/JSON          |
| LABFILE version | Must match supported schema            |
| Signature       | Must match recomputed hash             |
| Provenance      | Required for derived or imported files |
| License         | Required for public visibility         |
| Validation mode | Must be `"strict"` for publication     |

If any check fails, registry returns a **400: Validation Error** with structured error JSON.

---

### 8.9 Example Validation Error Report

```json
{
  "labfile_id": "abc-123",
  "spec_version": "1.0",
  "validation_mode": "strict",
  "errors": [
    {
      "code": "Q302",
      "field": "steps[1].parameters.temperature",
      "message": "Qualitative term 'room temperature' is invalid — specify numeric value in °C."
    },
    {
      "code": "R202",
      "field": "steps[2].with",
      "message": "Reference 'm_unknown' not found in materials."
    }
  ],
  "warnings": [
    {
      "code": "Q304",
      "field": "steps[3].parameters.speed",
      "message": "Value 31000 rpm exceeds common centrifuge range (max 30000)."
    }
  ]
}
```

This structured report format will be mirrored by both the CLI and registry APIs for machine-readable validation results.

---

## 9. Example Templates

This section presents canonical `.labfile` templates demonstrating correct structure, validation compliance, and behavioral logic.  
All examples are valid under **Labfile Specification v1.0**.

---

### 9.1 Minimal Protocol

Simplest valid `.labfile` passing strict validation.

```yaml
LABFILE: "1.0"

meta:
  title: "Buffer Preparation"
  authors:
    - name: "Dr. Alice Smith"
      organization: "Tropic Biology Lab"
  lab: "Tropic Biology Lab"
  license: "CC-BY-4.0"
  visibility: "public"

materials:
  - id: m_water
    name: "Distilled water"
  - id: m_naoh
    name: "NaOH pellets"

steps:
  - id: s_1
    action: "add"
    with: [m_water, m_naoh]
    parameters:
      mass: 2 g
      volume: 100 mL
  - id: s_2
    action: "mix"
    parameters:
      duration: 2 min
      mix_speed: 600 rpm

expected_results:
  description: "Clear 0.1 M NaOH buffer prepared."

safety:
  biosafety_level: "non-applicable"
  notes: "Wear gloves and goggles."

validation_mode: "strict"
```

**Key Characteristics**

- Meets minimum required fields.
- No devices or provenance needed.
- All numeric values include explicit units.
- Suitable for human-authored or local `.labfile` testing.

---

### 9.2 Advanced Research Protocol

Includes full metadata, safety, provenance, and attachments.

```yaml
LABFILE: "1.0"

meta:
  title: "Enzyme Activity Assay"
  authors:
    - name: "Dr. Lina Costa"
      organization: "BioAnalytics Institute"
      website: "https://bioanalytics.example.org"
  lab: "BioAnalytics Institute"
  date: "2025-10-30"
  license: "CC-BY-4.0"
  visibility: "public"
  review_status: "approved"
  FAIR_status: "compliant"
  derived_from: ["10.5281/zenodo.98765"]

materials:
  - id: m_substrate
    name: "pNPP substrate"
    concentration: 5
  - id: m_buffer
    name: "Assay buffer"
    pH: 7.4

devices:
  - id: d_plate_reader
    name: "SpectraMax i3x"
    kind: "plate_reader"
    model: "SMX-220"
    calibrated_at: "2025-09-15"

steps:
  - id: s_1
    action: "add"
    with: [m_buffer, m_substrate]
    parameters:
      volume: 2 mL
  - id: s_2
    action: "incubate"
    parameters:
      temperature: 37 °C
      duration: 30 min
  - id: s_3
    action: "measure"
    use: [d_plate_reader]
    parameters:
      wavelength: 405 nm
      repetitions: 3

expected_results:
  description: "Color change to yellow indicates enzyme activity."
  quantitative_metrics:
    - name: "Absorbance"
      value: 0.82
      unit: "A405"
  method: "Spectrophotometer"
  confidence_level: "high"

safety:
  biosafety_level: "BSL-1"
  ethics_approval_type: "IRB"
  ethics_approval_id: "IRB2025-01"
  ethics_approval_date: "2025-02-01"
  notes: "Handle reagents with gloves."

attachments:
  - type: "raw_data"
    format: "csv"
    path: "./results/output.csv"
    repository_url: "https://zenodo.org/record/1234"
    doi: "10.5281/zenodo.1234"
    access_level: "public"

provenance:
  - relation_type: "derived_from"
    source_type: "dataset"
    doi: "10.5281/zenodo.98765"

validation:
  validated_by: "Labfile Validator 0.8.2"
  validated_at: "2025-10-30T14:30:00Z"
  signature: "sha256:acb123def..."
validation_mode: "strict"
```

**Key Characteristics**

- Includes all metadata fields and provenance.
- References validated datasets by DOI.
- Suitable for publication in FAIR registries.
- Contains attestation for audit-ready compliance.

---

### 9.3 Automated / Robotic Workflow

Demonstrates `repeat`, `loop`, and `confirm` behavioral extensions.

```yaml
LABFILE: "1.0"

meta:
  title: "Automated Bacterial Growth Monitoring"
  authors:
    - name: "Dr. Keiko Yamamoto"
      organization: "Tokyo Bioautomation Center"
  license: "CC-BY-SA-4.0"
  visibility: "internal"
  review_status: "draft"

materials:
  - id: m_media
    name: "LB medium"
  - id: m_culture
    name: "E. coli overnight culture"

devices:
  - id: d_incubator
    name: "Thermal incubator"
    kind: "incubator"
    calibrated_at: "2025-08-10"
  - id: d_reader
    name: "Plate reader"
    kind: "plate_reader"
    calibrated_at: "2025-08-11"

steps:
  - id: s_1
    action: "add"
    with: [m_media, m_culture]
    parameters:
      volume: 50 mL

  - id: s_2
    action: "incubate"
    use: [d_incubator]
    parameters:
      temperature: 37 °C
      duration: 8 h
    repeat:
      count: 8
      interval: 1 h
    loop:
      condition:
        variable: "OD600"
        operator: "<"
        value: 0.6
      check_interval: 10 min
      max_duration: 12 h

  - id: s_3
    action: "measure"
    use: [d_reader]
    parameters:
      wavelength: 600 nm
    confirm:
      required: true
      message: "Verify instrument lid is closed"
      by: "operator"

  - id: s_4
    action: "branch"
    branch:
      condition:
        variable: "OD600"
        operator: ">"
        value: 0.9
      then: "s_5"
      else: "s_6"

  - id: s_5
    action: "centrifuge"
    parameters:
      speed: 8000 rpm
      duration: 10 min

  - id: s_6
    action: "wait"
    parameters:
      duration: 30 min

expected_results:
  description: "Growth curve monitored until OD600 reaches 0.9."
  quantitative_metrics:
    - name: "Final_OD600"
      value: 0.92
      unit: "absorbance"
  confidence_level: "medium"

safety:
  biosafety_level: "BSL-1"
  notes: "Disinfect incubator surfaces after run."

validation_mode: "strict"
```

**Key Characteristics**

- Demonstrates all behavioral extensions (`repeat`, `loop`, `branch`, `confirm`).
- Ideal for robotic execution or simulation environments.
- Enables conditional and adaptive automation workflows.

---

### 9.4 Extended Template with Custom Extension

```yaml
LABFILE: "1.0"

meta:
  title: "Fluorescence Microscopy of GFP-tagged Cells"
  authors:
    - name: "Dr. Maria Torres"
      organization: "University of Granada"
  license: "CC-BY-4.0"
  visibility: "public"

materials:
  - id: m_cells
    name: "HeLa-GFP cell line"

devices:
  - id: d_microscope
    name: "Zeiss LSM900"
    kind: "microscope"
    calibrated_at: "2025-09-10"

steps:
  - id: s_1
    action: "observe"
    use: [d_microscope]
    parameters:
      wavelength: 488 nm
      exposure_time: 100 ms

extensions:
  microscopy_ext:
    ontology: "https://example.org/schema/microscopy/v1"
    z_stack_depth: 5 µm
    pixel_size: 0.15 µm
    image_format: "TIFF"

expected_results:
  description: "GFP fluorescence localized to the nucleus."
  confidence_level: "high"

validation_mode: "strict"
```

**Key Characteristics**

- Demonstrates schema-safe `extensions:` usage.
- Adds microscopy metadata for automated systems.
- Fully compatible with strict validation and signature generation.

---
