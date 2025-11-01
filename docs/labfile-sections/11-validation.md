# `validation`

The `validation` block records the attestation emitted by authorized validators once a Labfile passes structural and semantic checks. It is generated automatically; authors should not edit it manually.

## Field Reference

| Field          | Type      | Required        | Units | Allowed Types | Constraints / Enum                                           | Example                     |
| -------------- | --------- | --------------- | ----- | ------------- | ------------------------------------------------------------ | --------------------------- |
| `validated_by` | string    | **conditional** | -     | string        | **Required if `validation_mode: strict`**                    | `"Labfile Validator 0.8.2"` |
| `validated_at` | date-time | **conditional** | -     | ISO 8601      | **Required if `validation_mode: strict`**                    | `"2025-10-30T14:30:00Z"`    |
| `signature`    | string    | **conditional** | -     | string        | `sha256:<hex>` over canonical YAML (excluding `validation:`) | `"sha256:acb123def..."`     |

Optional keys frequently emitted by validators:

- `notes` - free-form comments about the validation profile.
- `profile` - identifier for the ruleset used (e.g., `"strict-lims-v1"`).
- `public_key` / `certificate` - materials required for signature verification.
- `report` - relative path to a detailed validation log stored under `attachments`.

## Example

```yaml
validation:
  validated_by: "Labfile Validator 0.8.2"
  validated_at: "2025-10-30T14:30:00Z"
  signature: "sha256:acb123def..."
```

## Integrity Chain

| Proof                  | Field          | Function                          |
| ---------------------- | -------------- | --------------------------------- |
| Validator identity     | `validated_by` | Identifies tool and version used. |
| Timestamp              | `validated_at` | Confirms validation date (UTC).   |
| Cryptographic checksum | `signature`    | Detects any downstream tampering. |

## Best Practices

- Treat the block as append-only: any edit to the Labfile invalidates the signature and requires regeneration.
- Preserve validator public keys in a central registry to allow independent verification.
- When multiple validators run (e.g., internal QA plus registry), store additional attestations under `attachments` or extension namespaces rather than overwriting the core block.
- Include validation reports (PDF/JSON) as attachments for audit trails, referencing them in `validation.notes`.

## Validation Notes

- Validators should refuse to sign if existing attestation data is out of sync with the file hash.
- The signature uses canonical serialization rules defined in the spec; whitespace or key-order differences are normalized before hashing.
- In `lenient` mode, validators may omit the block entirely or leave fields empty, but registries typically require full data before accepting public submissions.

## Additional Reading

- [`validation` Field Details](../index.md#610-validation-and-validation_mode)
- [Attestation Logic](../index.md#612-validation-metadata-and-attestation-logic)
