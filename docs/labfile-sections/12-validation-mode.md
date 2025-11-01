# `validation_mode`

`validation_mode` reflects the intent behind the validator run-whether strict schema enforcement is required or leniency is allowed for exploratory work.

## Field Reference

| Field             | Type   | Required | Units | Allowed Types | Constraints / Enum  | Example    |
| ----------------- | ------ | -------- | ----- | ------------- | ------------------- | ---------- |
| `validation_mode` | string | optional | -     | enum          | `strict`, `lenient` | `"strict"` |

## Mode Semantics

| Mode      | Characteristics                                                                              | Impact on `validation` block                                  |
| --------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `strict`  | All schema, semantic, and ontology rules enforced; numeric parameters must include units.    | `validated_by`, `validated_at`, and `signature` **required**. |
| `lenient` | Noncritical warnings permitted; validators may skip signature generation; useful for drafts. | Attestation fields optional; warnings logged for reviewers.   |

## When to Use Each Mode

- **Production release / registry submission:** use `strict` to guarantee reproducibility and integrity checks.
- **Internal experimentation, collaborative drafting, or schema prototyping:** temporarily use `lenient`. Convert back to `strict` before public dissemination.
- **Automation gating:** robotics pipelines can read `validation_mode` to decide whether to block execution on warnings.

## Governance Tips

- Log the reason for switching modes in `validation.notes` or commit messages to preserve an audit trail.
- Pair mode transitions with version tags (e.g., `v1.0.0-draft` vs `v1.0.0`) so consumers know whether to trust the signature.
- If a registry overrides the declared mode (e.g., enforcing `strict`), record the override decision in `validation.notes` or an `extensions` namespace.

## Validation Notes

- Mode must always be declared, even if the validator default is stricter; absent values are treated as `strict` by most tooling.
- Changing any part of the Labfile after validation requires rerunning the validator, regardless of mode.
- Validators may emit warning codes (`Wxxx`) in lenient mode; treat them as actionable TODOs before promoting to `strict`.

## Additional Reading

- [Validation Modes](../index.md#42-validation-modes)
- [`validation` Field Details](../index.md#610-validation-and-validation_mode)
