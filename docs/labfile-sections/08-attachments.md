# `attachments`

The `attachments` array links supplementary assets-raw data, analysis notebooks, calibration sheets-that contextualize the protocol.

## Field Reference

| Field            | Type   | Required | Units | Allowed Types | Constraints / Enum                                                                   | Example                            |
| ---------------- | ------ | -------- | ----- | ------------- | ------------------------------------------------------------------------------------ | ---------------------------------- |
| `type`           | string | ✅       | -     | enum          | `raw_data`, `processed_data`, `report`, `image`, `log`, `archive`, `analysis_script` | `"raw_data"`                       |
| `format`         | string | ✅       | -     | enum          | `csv`, `json`, `xlsx`, `yaml`, `xml`, `tiff`, `jpg`, `png`, `zip`                    | `"csv"`                            |
| `path`           | string | ✅       | -     | string        | Local or relative file path                                                          | `"./results/dna_yield.csv"`        |
| `repository_url` | string | optional | -     | URL           | External dataset or repository link                                                  | `"https://zenodo.org/record/1234"` |
| `doi`            | string | optional | -     | string        | DOI string                                                                           | `"10.5281/zenodo.1234"`            |
| `access_level`   | string | optional | -     | enum          | `public`, `restricted`, `private`, `tokenized`, `paid`                               | `"public"`                         |

Additional optional keys commonly used:

- `hash` / `hash_algorithm` for integrity checks (e.g., `sha256`).
- `description` to provide short human-readable context.
- `created_at` / `updated_at` for version tracking.
- `mime_type` if you need exact media types beyond the `format` enum.

## Enum Guidance

- **Type** communicates the role of the asset. For example, `analysis_script` might point to a Jupyter notebook stored alongside the protocol.
- **Format** reflects the underlying file encoding. If you need to support additional formats (`pdf`, `hdf5`), extend the validator’s configuration or use an `extensions` namespace.
- **Access level** helps registries manage permissions:
  - `public` - no restrictions.
  - `restricted` - requires institutional login or NDA.
  - `private` - local-only, not exposed externally.
  - `tokenized` - presigned URLs or short-lived tokens.
  - `paid` - commercial access via paywall or licensing.

## Linking Strategy

- Use relative `path` values when distributing the Labfile with a repository so consumers can fetch artifacts offline.
- Populate `repository_url` with stable landing pages (Zenodo, Figshare, Dryad). Prefer versioned DOIs for immutable references.
- Mirror important metadata (checksum, size, version) in `attachments` even if already present in external repositories to simplify offline verification.

## Best Practices

- Attach calibration certificates for devices, raw instrument exports, and statistical analysis outputs.
- Pair attachments with `expected_results` metrics by noting filenames or DOIs in `notes`.
- For sensitive data, set `access_level: restricted` and provide retrieval instructions in `description`.
- Consider compressing large raw datasets (`zip`, `tar.gz`) and documenting the compression method.

## Validation Notes

- Validators check RFC 3986 compliance for `repository_url` and `doi` formatting.
- Missing `type` or `format` triggers error `E312` (attachment metadata incomplete).
- When `access_level` is not `public`, ensure usage instructions exist in `description` or `notes` to avoid review delays.

## Additional Reading

- [`attachments` Field Details](../index.md#67-attachments)
- [Provenance Requirements](../index.md#68-provenance)
