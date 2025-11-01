# `meta`

The `meta` block captures descriptive information about the protocol: authorship, licensing, visibility, and how the work fits within wider registries.

## Field Reference

| Field           | Type           | Required | Units | Allowed Types | Constraints / Enum                              | Example                       |
| --------------- | -------------- | -------- | ----- | ------------- | ----------------------------------------------- | ----------------------------- |
| `title`         | string         | ✅       | -     | string        | Free text                                       | `"DNA Extraction"`            |
| `authors[]`     | list\<object>  | ✅       | -     | array         | Each `{ name, organization?, role?, website? }` | `[{ name: "Dr. Alice" }]`     |
| `lab`           | string         | ✅       | -     | string        | Institution or facility                         | `"Tropic Biology Lab"`        |
| `website`       | string         | optional | -     | URL           | Project or protocol URL                         | `"https://labfile.bio/p/123"` |
| `date`          | date           | optional | -     | ISO 8601      | `YYYY-MM-DD`                                    | `"2025-10-30"`                |
| `license`       | string         | ✅       | -     | string        | SPDX/CC identifier (e.g., `CC-BY-4.0`)          | `"CC-BY-4.0"`                 |
| `language`      | string         | optional | -     | string        | ISO 639-1 language code                         | `"en"`                        |
| `review_status` | string         | optional | -     | enum          | `draft`, `approved`, `deprecated`, `archived`   | `"draft"`                     |
| `visibility`    | string         | ✅       | -     | enum          | `public`, `internal`, `private`                 | `"public"`                    |
| `derived_from`  | list           | optional | -     | array[string] | DOIs or Labfile IDs                             | `["10.5281/zenodo.1234567"]`  |
| `FAIR_status`   | boolean/string | optional | -     | bool/enum     | `true`, `"compliant"`, `"non_compliant"`        | `true`                        |
| `compliance`    | list           | optional | -     | array[string] | `"GLP"`, `"GMP"`, `"FAIR"`, `"ISO-9001"`        | `["GLP"]`                     |

## How to Use Each Field

#### **`authors[]`** should be ordered by contribution. Include persistent IDs where possible (`orcid`, `ror`, `grid`).

Example:

```yaml
authors:
  - name: "Dr. Alice Smith"
    orcid: "0000-0002-1234-5678"
    organization: "Tropic Biology Lab"
    role: "lead"
```

#### **`review_status`** maps to your governance workflow:

- `draft` - working copy not yet validated.
- `approved` - ready for distribution.
- `deprecated` - superseded, kept for history.
- `archived` - retired, not intended for reuse.

#### **`visibility`** controls publication scope:

- `public` - registry discoverable and requires signatures.
- `internal` - organization-only access; signatures recommended.
- `private` - local sandbox; validation metadata optional.

#### **`FAIR_status`** can be a boolean or controlled string. Use `true` for fully compliant protocols, or provide a status label consumed by your data office.

#### **`compliance`** lists regulatory or quality frameworks satisfied by the protocol (`GLP` for Good Laboratory Practice, etc.).

#### **`derived_from`** should reference persistent identifiers (DOI, accession ID, or a `labfile://` URI) so provenance can be resolved automatically.

## Best Practices

- Keep `license` aligned with data outputs to avoid downstream conflicts. [SPDX](https://spdx.org/licenses/) is the preferred source.
- Provide an institutional landing page in `website` when the protocol has a publication or knowledge base entry.
- Ensure `date` reflects the most recent substantive revision to help registries track versioning.
- Include `language` for multilingual teams so search indexes can filter by locale.
- When multiple compliance regimes apply, sort `compliance` alphabetically for deterministic diffs.

## Validation Notes

- At least one author with `name` is required; validation fails if array is empty.
- Enumerated fields reject unknown values (`E2xx` class errors). Use the wording from the table verbatim.
- Validators inherit ISO formatting checks for `date` and URL validation for `website`.

## Additional Reading

- [`meta` Field Details](../index.md#61-meta)
- [Minimal Example](../index.md#3-minimal-example)
