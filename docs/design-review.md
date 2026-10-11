# Ardu-CAN design review

## Scope and basis

This review describes the checked-in KiCad source and exports. Checks were run
with **KiCad CLI 10.0.6** on 2026-10-11. The circuit and PCB geometry were not
changed as part of this portfolio-preparation work.

## Verified design facts

- MCU: U8 `STM32F412CGUx`.
- CAN transceiver: U5 `TJA1051TK-3`; J1/J2 carry the `CAN` value.
- Sensor: U4 `ICM-42688-P`.
- Regulator: U6 `NCV8705MT33TCG`.
- Input protection: U1 `LM66100DCK`, shown on the reverse-polarity-protection
  sheet.
- Additional protection: D1 `NUP2105L` and ESD9C3.3ST5G parts.
- X1 is `LFSPXO072801REEL`.
- Outline: 37 × 37 mm from the board Edge.Cuts coordinates; configured thickness
  is 1.6 mm.
- The board setup specifies six copper layers.
- The root schematic links seven named circuit sheets. Firmware source was not
  found in the repository.
- The [ST STM32F412 family datasheet](https://www.st.com/resource/en/datasheet/stm32f412cg.pdf)
  lists an on-chip bxCAN controller; the board separately uses a TJA1051 CAN
  transceiver. Firmware and tests proving CAN operation are not present.

These are design-file observations. They do not establish actual powered
behavior, ratings, or compliance.

## Automated checks

### PCB DRC

The CLI completed the DRC but reported **458 violations**:

| Finding | Count | Notes |
| --- | ---: | --- |
| Via diameter | 172 | The reported 0.4 mm vias are below the configured 0.5 mm minimum |
| Annular width | 172 | The current via geometry conflicts with the configured annular-width rule |
| Library footprint mismatch | 44 | Reconcile the board footprints with the configured library sources |
| Silkscreen overlap | 22 | Review text/graphics against pads and clearances |
| Copper clearance | 19 | Resolve against the intended fabrication rules |
| Copper-to-edge clearance | 17 | Recheck after finalizing board outline and fabricator rules |
| Dangling track | 6 | Investigate and reconnect or intentionally remove |
| Malformed courtyard | 4 | Review the affected footprints/courtyards |
| Silkscreen over copper | 1 | Review the affected marking |
| Footprint-library issue | 1 | Resolve the reported missing/unavailable library association |
| Unconnected item | 1 | Resolve before release |

Counts are per reported finding and can overlap when one physical issue triggers
multiple checks; they are not a severity waiver. The board should not be labeled
fabrication-ready until the issues are corrected and a new DRC report is
reviewed.

### Schematic ERC

The CLI reported **61 violations: 11 errors and 50 warnings**. The error set
includes sheet-pin/hierarchical-label mismatches, unconnected sheet pins,
undriven power-input pins, and power-output pin conflicts. Warnings include
off-grid endpoints, unconnected wire endpoints, pin-type connections, and a
footprint-library configuration issue. Resolve findings against intended
connectivity; do not suppress them solely to obtain a clean count.

Complete CLI output is retained in
[`pcb-drc-report.txt`](pcb-drc-report.txt) and
[`schematic-erc-report.txt`](schematic-erc-report.txt).

### Exports and supporting files

- A full eight-page schematic PDF was exported from the source schematic.
- A KiCad schematic BOM CSV was exported. It has references, values, footprint
  names, quantities, and DNP fields, but does not establish sourcing,
  lifecycle, alternates, or procurement status.
- A STEP assembly file is present and has a KiCad assembly-export header. The
  KiCad 3D board renderer also loaded the board and its available component
  models. This does not constitute independent STEP/CAD validation.
- No Gerber, drill, pick-and-place, or fabrication job output files were
  present at review time.
- No firmware or hardware-test evidence was present.

## Before commercial fabrication

1. Resolve all ERC errors and review the warnings against intended circuit
   behavior.
2. Resolve all DRC errors/findings against the chosen board house's design
   rules, particularly via geometry, annular ring, clearance, copper-edge
   distance, unconnected items, silkscreen, and footprint-library mismatch.
3. Confirm symbols, footprints, 3D model paths, component part numbers, and
   connector pinouts against current manufacturer documentation.
4. Verify mechanical hole/outline intent, connector mating access, enclosure
   fit, and assembly clearances.
5. Confirm final layer stack-up, material, copper weight, finish, and
   fabrication constraints with the selected supplier.
6. Generate Gerber/drill and assembly outputs from the reviewed release
   revision; inspect those outputs independently and provide a complete,
   sourced BOM if purchasing support is required.
7. Record the agreed validation scope. Do not claim prototype testing,
   certification, or production readiness without evidence.

## Licensing note

No project-wide license was added. Review each included library/model license,
including the ICM-42688-P library's `License.txt`, before redistribution or
commercial reuse.
