# `devices`

The `devices` block catalogs every instrument or tool invoked during the protocol. It drives automation compatibility and ensures calibration data is captured alongside procedure steps.

## Field Reference
| Field           | Type   | Required        | Units | Allowed Types | Constraints / Enum                                                                                                                          | Example                                           |
| --------------- | ------ | --------------- | ----- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `id`            | string | ✅              | -     | string        | Unique                                                                                                                                    | `"d_centrifuge"`                                  |
| `name`          | string | ✅              | -     | string        | -                                                                                                                                        | `"Mini centrifuge"`                               |
| `kind`          | string | ✅              | -     | enum          | `centrifuge`, `pipette`, `thermal_cycler`, `spectrophotometer`, `incubator`, `balance`, `shaker`, `robotic_arm`, `freezer`, `microscope`, `biosafety_cabinet`, `autoclave`, `liquid_handler`, `plate_reader`, `flow_cytometer`, `custom` | `"centrifuge"`                                    |
| `description`   | string | **conditional** | -     | string        | **Required if `kind` = `"custom"`**                                                                                                       | `"Ultrasonic cleaner 40 kHz"`                     |
| `capabilities`  | map    | **conditional** | -     | object        | **Required if `kind: custom` & `validation_mode: strict`**                                                                                | `{ frequency: { unit: "kHz", min: 30, max: 45 } }`|
| `manufacturer`  | string | optional        | -     | string        | -                                                                                                                                        | `"Eppendorf"`                                     |
| `model`         | string | optional        | -     | string        | -                                                                                                                                        | `"5418R"`                                         |
| `calibrated_at` | date   | optional        | -     | ISO 8601      | -                                                                                                                                        | `"2025-01-10"`                                    |

Optional additions such as `serial_number`, `location`, `maintenance_schedule`, `attachments`, or vendor-specific extension namespaces (`automation_ext`, `microscopy_ext`, etc.) can be used for richer integration.

## `kind` Vocabulary
| Kind                | Description                                                                                     | Typical Equipment Example |
| ------------------- | ----------------------------------------------------------------------------------------------- | ------------------------- |
| `centrifuge`        | Spinning device for separating components by density.                                          | [Eppendorf 5420](https://online-shop.eppendorf.com/) |
| `pipette`           | Liquid handling tool for transferring defined volumes.                                         | [Gilson PIPETMAN](https://www.gilson.com/)           |
| `thermal_cycler`    | PCR cycler controlling temperature cycles.                                                      | [Bio-Rad C1000](https://www.bio-rad.com/)            |
| `spectrophotometer` | Measures absorbance/transmittance across wavelengths.                                          | [Thermo Scientific NanoDrop](https://www.thermofisher.com/) |
| `incubator`         | Maintains temperature (and often humidity/CO₂) for cultures.                                    | [Binder CB 160](https://www.binder-world.com/)       |
| `balance`           | Analytical or precision scale.                                                                  | [METTLER TOLEDO XPR](https://www.mt.com/)            |
| `shaker`            | Orbital/linear shaker or rocker platform.                                                       | [Eppendorf Innova 2100](https://www.eppendorf.com/product-media/BR/abo/lab-equipment/shakers/) |
| `robotic_arm`       | Multi-axis arm for automation or sample handling.                                               | [Universal Robots UR5e](https://www.universal-robots.com/) |
| `freezer`           | Cold storage (−20 °C, −80 °C, cryogenic).                                                       | [Thermo TSX Series](https://www.thermofisher.com/)   |
| `microscope`        | Optical or fluorescence microscope.                                                             | [ZEISS LSM 900](https://www.zeiss.com/microscopy/)   |
| `biosafety_cabinet` | Containment enclosure for sterile/BSL operations.                                               | [Labconco Purifier Logic+](https://labconco.com/)    |
| `autoclave`         | Steam sterilizer for decontaminating equipment and media.                                       | [Tuttnauer EZ9](https://tuttnauer.com/)              |
| `liquid_handler`    | Automated pipetting / deck platform.                                                            | [Tecan Fluent](https://www.tecan.com/)               |
| `plate_reader`      | Multi-well plate detection (absorbance, fluorescence, luminescence).                           | [Molecular Devices SpectraMax](https://www.moleculardevices.com/) |
| `flow_cytometer`    | Measures cell populations using lasers and detectors.                                           | [BD FACSCelesta](https://www.bdbiosciences.com/)     |
| `custom`            | Any instrument not covered above; requires descriptive capabilities for validation.             | Custom-built imaging rig                              |

## Defining Capabilities for Custom Devices
- Provide quantitative ranges with units (`frequency`, `volume_capacity`, `temperature_range`, etc.).
- Use arrays of strings for discrete functions (`functions: ["wash", "sterilize"]`).
- Include maintenance metadata when performance drifts over time (e.g., `last_service`, `calibration_certificate` via `attachments`).

## Best Practices
- Ensure `id` matches what appears in `steps[].use` and avoid spaces or uppercase letters (`d_incubator_a`).
- Record `calibrated_at` in ISO 8601 (`2025-01-10`); pair with `attachments` referencing calibration certificates.
- Document device placement (`location: "Room B, Bench 3"`) so technicians know which hardware matches the identifier.
- When multiple devices share a `kind`, use descriptive `name` patterns and include `serial_number` or `asset_tag`.
- Offload complex vendor schemas into `extensions` to keep the core section manageable.

## Validation Notes
- Validators enforce the controlled `kind` vocabulary; unknown terms raise schema errors.
- In `strict` mode, custom devices must declare `capabilities` to describe measurable performance envelopes.
- Cross-references ensure every `use` value in `steps` maps to a declared `devices.id`, preventing orphan references.

## Additional Reading
- [`devices` Field Details](../index.md#63-devices)
- [Behavioral Extensions](../index.md#7-behavioral-extensions)
