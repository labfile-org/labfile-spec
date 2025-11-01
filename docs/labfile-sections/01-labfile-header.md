# LABFILE Header

The `LABFILE` key is the fixed entry point for every specification-compliant file. It pins the schema version and signals validators that the document uses the Labfile contract.

## Field Reference
| Field      | Type   | Required | Allowed Values | Constraints                                 | Example  |
| ---------- | ------ | -------- | -------------- | ------------------------------------------- | -------- |
| `LABFILE`  | string | ✅       | `"1.0"`        | Must be the first key in the document.      | `"1.0"`  |

## Purpose
- Identify the document as a Labfile artifact.
- Lock the schema version so tooling knows which rules to apply.
- Anchor provenance and attestation metadata to a constant root value for hashing.

## Requirements
- `LABFILE: "1.0"` must appear on the first non-comment line of the YAML file.
- The value is a quoted string; unquoted `1.0` or alternate schema numbers are invalid.
- Any mismatch triggers validation code `S101` and fatal error `E001`.

## Validation Signals
| Code   | Description                                       | Trigger                                             |
| ------ | ------------------------------------------------- | --------------------------------------------------- |
| `S101` | File must begin with `LABFILE: "1.0"`             | Header missing, misplaced, or incorrectly spelled.  |
| `E001` | Invalid LABFILE version                           | Header present but not equal to `"1.0"`.            |

## Example
```yaml
LABFILE: "1.0"
```

## Best Practices
- Avoid blank lines, BOM markers, or comments before the header to preserve canonical hashing.
- When future schema versions are released, the validator will reject older tooling until it advertises support.
- Editors should template the header to prevent accidental typos during authoring.

## Additional Reading
- [Top-Level Structure](../index.md#2-top-level-structure)
- [Field Summary](../index.md#5-field-summary)
- [Syntax Rules](../index.md#4-syntax-rules)
