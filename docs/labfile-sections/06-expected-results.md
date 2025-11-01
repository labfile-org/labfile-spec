# `expected_results`

The `expected_results` section documents the outcomes that should follow a successful execution-qualitative descriptions and quantitative metrics that support experiment verification.

## Field Reference

| Field                    | Type          | Required | Units   | Allowed Types | Constraints / Enum                 | Example                                          |
| ------------------------ | ------------- | -------- | ------- | ------------- | ---------------------------------- | ------------------------------------------------ |
| `description`            | string        | ✅       | -       | string        | Outcome summary                    | `"Supernatant contains lysed DNA."`              |
| `quantitative_metrics[]` | list\<object> | optional | various | array         | Each `{ name, value, unit }`       | `[{ name: "DNA_yield", value: 20, unit: "µg" }]` |
| `method`                 | string        | optional | -       | string        | Instrument or assay used           | `"Spectrophotometer"`                            |
| `confidence_level`       | string        | optional | -       | enum          | `high`, `medium`, `low`, `unknown` | `"high"`                                         |

## Metric Structure

Quantitative metrics should be explicit and machine-readable:

```yaml
quantitative_metrics:
  - name: "Purity_ratio"
    value: 1.8
    unit: "A260/A280"
    target: ">= 1.8"
    tolerance: "±0.1"
```

- Supply canonical SI units whenever possible; for ratios such as `A260/A280`, treat the unit as dimensionless but still provide a descriptive string.
- Include optional `target`, `tolerance`, or `notes` keys to communicate acceptable variance bands.

## Confidence Levels

- `high` - empirical result reproduced consistently with strong evidence.
- `medium` - outcome observed but with moderate variance or smaller sample size.
- `low` - exploratory or preliminary data; treat deviations cautiously.
- `unknown` - no reliability statement can be made (e.g., new protocol).

## Best Practices

- Connect `method` to the device or assay actually used (e.g., `"LC-MS"` or `"Flow cytometer"`). When possible, match it to an entry in `devices` by ID via `notes`.
- Provide supplemental plots, raw data, or analysis notebooks through the `attachments` section and reference them in `expected_results.notes`.
- Use `confidence_level` to inform downstream dashboards which metrics can gate quality checks.
- Document statistical treatments (mean, median, confidence intervals) either in `notes` or attached analysis files for reproducibility.

## Validation Notes

- Every metric entry requires a `name`; validators reject unnamed objects.
- Numeric `value` fields should be accompanied by `unit` unless dimensionless. Missing units raise `Q`-class warnings in strict mode.
- Enumerated `confidence_level` values are validated verbatim-other strings are rejected.

## Additional Reading

- [`expected_results` Field Details](../index.md#65-expected_results)
- [Validation Error Report Example](../index.md#89-example-validation-error-report)
