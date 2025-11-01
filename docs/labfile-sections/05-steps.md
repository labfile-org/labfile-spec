# `steps`

The `steps` array defines the ordered procedural actions of the protocol. Each item captures intent (`action`), resources (`with` or `use`), parameters, and optional control flow directives.

## Field Reference

| Field                 | Type   | Required | Units | Allowed Types | Constraints / Enum                                                | Example                                                           |
| --------------------- | ------ | -------- | ----- | ------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------- |
| `id`                  | string | ✅       | -     | string        | Unique                                                            | `"s_1"`                                                           |
| `action`              | string | ✅       | -     | string        | Controlled action term (e.g., `add`, `incubate`, `centrifuge`)    | `"centrifuge"`                                                    |
| `with`                | list   | optional | -     | array[string] | Material IDs defined in `materials[]`                             | `["m_buffer"]`                                                    |
| `use`                 | list   | optional | -     | array[string] | Device IDs defined in `devices[]`                                 | `["d_centrifuge"]`                                                |
| `parameters`          | map    | optional | -     | map           | Quantitative; units required                                      | `{ speed: 13000 rpm }`                                            |
| `execution_mode`      | string | optional | -     | enum          | `manual`, `automated`, `hybrid`                                   | `"manual"`                                                        |
| `runtime.status`      | string | optional | -     | enum          | `pending`, `running`, `completed`, `failed`, `skipped`, `aborted` | `"completed"`                                                     |
| `documentation_level` | string | optional | -     | enum          | `standard`, `verbose`, `audit`                                    | `"standard"`                                                      |
| `confirm`             | map    | optional | -     | object        | Manual checkpoint                                                 | `{ required: true, message: "Check clarity" }`                    |
| `repeat`              | map    | optional | -     | object        | Loop repetition                                                   | `{ count: 3, interval: 5 min }`                                   |
| `loop`                | map    | optional | -     | object        | While-condition                                                   | `{ condition: { variable: "OD600", operator: "<", value: 0.5 } }` |
| `branch`              | map    | optional | -     | object        | Conditional branch                                                | `{ condition: {...}, then: "s_5", else: "s_6" }`                  |

## Parameter Guidelines

Quantitative parameters must include units (especially in `strict` mode). Common keys drawn from the specification:

| Key             | Type    | Units (examples) | Valid Range / Guidance       | Example        |
| --------------- | ------- | ---------------- | ---------------------------- | -------------- |
| `volume`        | number  | mL, µL           | 0.1–1000 (context dependent) | `"1 mL"`       |
| `time`          | number  | s, min, h        | ≥0                           | `"30 min"`     |
| `duration`      | number  | s, min, h        | ≥0                           | `"5 min"`      |
| `temperature`   | number  | °C               | −80–150                      | `"37 °C"`      |
| `speed`         | number  | rpm              | 100–30000                    | `"13000 rpm"`  |
| `angle`         | number  | °                | 0–360                        | `45`           |
| `pressure`      | number  | bar, psi         | ≥0                           | `"1 bar"`      |
| `concentration` | number  | mol/L, mg/mL     | ≥0                           | `"5 mM"`       |
| `mass`          | number  | g, mg            | ≥0                           | `"1.2 g"`      |
| `repetitions`   | integer | -                | 1–1000                       | `3`            |
| `wavelength`    | number  | nm               | 180–1100                     | `405`          |
| `humidity`      | number  | %                | 0–100                        | `60`           |
| `pH`            | number  | -                | 0–14                         | `7.4`          |
| `flow_rate`     | number  | µL/min, mL/min   | ≥0                           | `"500 µL/min"` |
| `mix_speed`     | number  | rpm              | 0–2000                       | `"800 rpm"`    |
| `distance`      | number  | mm, cm           | ≥0                           | `"5 mm"`       |

## Control Blocks

| Field     | Required Keys                                 | Behavior                                                                             |
| --------- | --------------------------------------------- | ------------------------------------------------------------------------------------ |
| `confirm` | `required`, `message`, optional `by`          | Inserts a manual checkpoint; execution pauses until the operator acknowledges.       |
| `repeat`  | `count`, optional `interval`, extras          | Executes the same step a fixed number of times with optional wait intervals.         |
| `loop`    | `condition`, `check_interval`, `max_duration` | Continues while a logical condition remains true. Requires explicit exit safeguards. |
| `branch`  | `condition`, `then`, `else`                   | Directs control flow to another step ID depending on a condition outcome.            |

Define conditions using measurable variables (`variable`, `operator`, `value`). Qualitative statements (“until done”) are rejected.

## Execution Metadata

- **`execution_mode`** communicates whether automation or manual operation is expected.  
  Use `hybrid` for human-in-the-loop robotics.
- **`runtime.status`** captures live execution states for digital twins, dashboards, or audit logs.
- **`documentation_level`** can increase verbosity of machine-generated reports (`audit` for regulated studies).

## Best Practices

- Keep IDs stable and descriptive; prefix steps by phase (`prep_1`, `run_2`).
- Group related operations with dedicated notes or attachments for SOP references.
- Avoid hidden dependencies-reference every material/device explicitly via `with`/`use`.
- Parameter units should be canonical (SI). If devices operate in alternative units, provide conversions in `notes`.
- Use `branch` sparingly; when complex logic is required consider splitting into multiple Labfiles with explicit transitions.

## Validation Notes

- Controlled vocabularies for `action` are enforced by the validator’s ontology; invalid verbs raise semantic errors (`Axxx` codes).
- Strict mode requires all numeric parameters to carry units; missing units raise `Q`-class errors.
- Control blocks must define exit conditions; otherwise deterministic validation fails.
- Cross-references ensure every `with`/`use` item exists, preventing orphaned resources.

## Additional Reading

- [`steps` Field Details](../index.md#64-steps)
- [Behavioral Extensions](../index.md#7-behavioral-extensions)
