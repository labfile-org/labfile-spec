# `extensions`

The `extensions` object is a namespaced escape hatch for community- or vendor-specific fields that do not belong in the core schema.

## Field Reference

| Field (namespace) | Type | Required | Units | Allowed Types | Constraints / Enum                 | Example                     |
| ----------------- | ---- | -------- | ----- | ------------- | ---------------------------------- | --------------------------- |
| `_namespace_`     | map  | optional | -     | any           | Custom namespaced fields per owner | `{ microscopy_ext: {...} }` |

Each top-level key represents a namespace that you control. Names must be lowercase with underscores to avoid collisions (e.g., `microscopy_ext`, `automation_ext`).

## Namespace Structure

```yaml
extensions:
  microscopy_ext:
    $schema: "https://schemas.labfile.bio/ext/microscopy/v1"
    objective: "40x oil"
    fluorescence_filter: "FITC"
    exposure_time:
      value: 120
      unit: "ms"
```

Recommended metadata for every namespace:

- `$schema` or `ontology` - URL to a JSON Schema / OpenAPI / LinkML document describing the shape.
- `version` - semantic version of the extension payload (`"1.2.0"`).
- `provider` - organization maintaining the namespace.
- Domain-specific payload - any fields that your tooling understands.

## Best Practices

- **Isolation:** Keep namespace keys unique to your organization or consortium (`labname_feature_ext`). Never overload core field names.
- **Documentation:** Publish schemas and changelogs where validators can retrieve them. Versioned URLs help with deprecation management.
- **Validation:** If extensions impact robotic execution, attach signature metadata inside the namespace or in `validation.notes`.
- **Discoverability:** Reference extensions in `attachments` or knowledge bases so collaborators know where to find schema docs.
- **Interoperability:** Include fallback summaries for humans (`description`, `notes`) when the consumer lacks the custom validator.

## Validation Notes

- Core validators ensure namespaces do not collide with reserved keys but treat payloads as opaque by default.
- Registries may choose to fetch `$schema` and perform additional checks; plan for network failures by keeping critical fields simple.
- Keep payload sizes reasonable; large binary data should remain in `attachments`.

## Additional Reading

- [`extensions` Field Details](../index.md#69-extensions)
- [Extended Template Example](../index.md#94-extended-template-with-custom-extension)
