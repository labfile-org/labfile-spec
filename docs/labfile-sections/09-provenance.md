# `provenance`

The `provenance` section captures lineage-how this protocol relates to prior experiments, datasets, or registry entries. It is essential for reproducibility and regulatory traceability.

## Field Reference

| Field           | Type   | Required | Units | Allowed Types | Constraints / Enum                                                             | Example                  |
| --------------- | ------ | -------- | ----- | ------------- | ------------------------------------------------------------------------------ | ------------------------ |
| `relation_type` | string | optional | -     | enum          | `derived_from`, `variant_of`, `supersedes`                                     | `"derived_from"`         |
| `source_type`   | string | optional | -     | enum          | `labfile`, `dataset`, `publication`, `instrument`, `repository`, `external_db` | `"labfile"`              |
| `doi`           | string | optional | -     | string        | DOI or other persistent identifier                                             | `"10.5281/zenodo.99999"` |

Each provenance record is an object. You can extend entries with additional keys such as `title`, `version`, `access_url`, `commit`, `timestamp`, or `license` to express more context.

## Relationship Types

- `derived_from` - The protocol is a transformation or downstream analysis of the referenced asset (e.g., deriving a derivative dataset).
- `variant_of` - Minor modifications; the core procedure remains similar with slight adjustments.
- `supersedes` - The current Labfile officially replaces the referenced protocol; registries may mark older versions deprecated.

## Source Types

- `labfile` - Another Labfile ID or URL (`labfile://`, HTTPS).
- `dataset` - Data packages deposited in repositories (Zenodo, Dryad).
- `publication` - Journals, preprints, or SOP manuals.
- `instrument` - Device calibration or configuration source.
- `repository` - Git or version-control repositories.
- `external_db` - LIMS, ELN, or regulatory databases.

## Example

```yaml
provenance:
  - relation_type: "derived_from"
    source_type: "dataset"
    doi: "10.5281/zenodo.98765"
    title: "RNA-Seq baseline data"
  - relation_type: "supersedes"
    source_type: "labfile"
    doi: "10.5281/zenodo.87654"
    notes: "Adds automated liquid handling support"
```

## Best Practices

- Use persistent identifiers (DOI, Handle, accession numbers). For Git repositories reference commit SHAs:  
  `repository_url: "https://github.com/labfile-org/example"` + `commit: "a1b2c3d4"`.
- Keep the array ordered chronologically to simplify diff reviews.
- Mirror provenance entries in `attachments` when sharing related reports to close the traceability loop.
- Coordinate with `extensions` if domain-specific lineage metadata is required (e.g., `clinical_ext.study_id`).

## Validation Notes

- Validators enforce enumerated `relation_type` and `source_type` lists; unexpected values raise schema errors.
- DOIs must comply with standard syntax (`10.` prefix). Non-DOI identifiers should include a URI scheme.
- Missing references or unreachable IDs may trigger reference integrity (`R` class) warnings during registry ingestion.

## Additional Reading

- [`provenance` Field Details](../index.md#68-provenance)
- [Identity and Reference Rules](../index.md#82-identity-and-reference-rules)
