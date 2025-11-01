# `safety`

The `safety` block centralizes risk assessments, biosafety classifications, and mitigation measures required to run the protocol responsibly.

## Field Reference

| Field                  | Type   | Required | Units | Allowed Types | Constraints / Enum                                   | Example                         |
| ---------------------- | ------ | -------- | ----- | ------------- | ---------------------------------------------------- | ------------------------------- |
| `biosafety_level`      | string | optional | -     | enum          | `BSL-1`, `BSL-2`, `BSL-3`, `BSL-4`, `non-applicable` | `"BSL-1"`                       |
| `ethics_approval_type` | string | optional | -     | enum          | `IRB`, `IACUC`, `HREC`, `internal`, `none`           | `"IRB"`                         |
| `ethics_approval_id`   | string | optional | -     | string        | Institutional approval identifier                    | `"IRB2025-01"`                  |
| `ethics_approval_date` | date   | optional | -     | ISO 8601      | Approval date with `YYYY-MM-DD` format               | `"2025-02-01"`                  |
| `notes`                | string | optional | -     | string        | Additional handling instructions                     | `"Handle reagents with gloves"` |

You may extend the section with arrays such as `controls[]`, `hazards[]`, `ppe[]`, or `references[]` if you want to link into a local safety management system or Standard Operating Procedures (SOPs).

## Enum Notes

- **Biosafety levels (BSL)** indicate containment requirements:
  - `BSL-1` - routine teaching labs; minimal risk.
  - `BSL-2` - moderate risk organisms; requires biological safety cabinets.
  - `BSL-3` - high-risk airborne pathogens; controlled facility entry.
  - `BSL-4` - maximum containment for dangerous agents.
  - `non-applicable` - no biological agents involved.
- **Ethics approval types** map to governance bodies:
  - `IRB` (Institutional Review Board) - human subjects research.
  - `IACUC` - animal care and use committee.
  - `HREC` - human research ethics committee (common outside the US).
  - `internal` - company or lab-internal committee.
  - `none` - ethics approval not required (explain rationale in `notes`).

## Documentation Guidance

- Record PPE expectations (`notes`, `ppe[]`) such as gloves, goggles, or respirators.
- Link to SOPs or emergency procedures using `references` objects (`{ id, title, url }`).
- Capture waste disposal guidance, spill response, or decontamination steps to satisfy institutional audits.
- When work spans multiple BSL zones, document location-specific requirements (`notes: "Phase 1 in BSL-1, transfer to BSL-2"`).

## Alignment with Materials and Devices

- Cross-reference high-risk materials via `materials[].hazards` to keep messaging consistent.
- Tie in equipment dependencies (e.g., `biosafety_cabinet` IDs) describing where certain steps must be performed.

## Validation Notes

- Enumerations are case-sensitive; deviations raise schema errors.
- Dates must be ISO 8601; missing or malformed entries trigger `S`-class syntax warnings.
- When used with `attachments`, ensure referenced SOP documents exist and match the declared access level.

## Additional Reading

- [`safety` Field Details](../index.md#66-safety)
- [Structural Rules](../index.md#81-structural-rules)
