# BurnWire

Burn-wire release board for the PVDX CubeSat. Generates a constant current of 1.6A through a nichrome wire to sever the retention line holding deployable camera arm in its stowed position.

## Status
- Design phase: manufactured (rev-3.1 sent to fab 2026-08-22)
- Current rev: rev-3.1
- Contributors: Atharv Chowdhary, Nick Cavallo, Jayla Hsiung, Emily Zhang, Kelly Lin, Natalie Cavallo

## System overview

BurnWire was originally forked from another team's Altium-designed board and imported into KiCad, which is the source of most of the quirks documented below. 

**WIP**
Known components on the board (confirmed via schematic/PCB library references, not yet mapped to a full circuit description — fill in as this gets documented properly):

- **ADP2303ARDZ** — synchronous step-down switching regulator. Per standing design notes, this part needs a Schottky diode (e.g. BAT54, here likely the placed **B330B-13-F**) from V_in to the BST pin to prevent bootstrap collapse at 5V input (datasheet Figure 48) — worth confirming this placement is present and correctly oriented.
- **B330B-13-F** — Schottky diode, SMB package (3 footprint variants exist in `libs/footprints.pretty/`: base, `-M`, `-L`; the placed instance uses `-L`).
- **IND_NLV32T_TDK** (TDK NLV32T-3R3J-EF, 3.3µH) and **Bourns SRN6045** — two inductor footprints present; [confirm: both used in the regulator stage, or does one serve the burn-wire drive circuit?]
- **JST B6B-XH-A** 6-pin connector — external interface (power / command / burn-wire line — confirm pinout).
- 0.25Ω 3W power resistor (2512/2010 footprint) — likely current-sense or burn-wire load-adjacent; confirm function.
- Standard passives: 0603/0805 resistors, 0603 100nF decoupling caps, WCAP-ASLI 5x5.5mm bulk electrolytic.
- Test points inherited from the original board's library (`PowerManagmentBoard_V4.0:TP` nickname — currently orphaned, see Open Items).

### Repo layout
```
BurnWire/
├── BurnWire.kicad_pro / .kicad_sch / .kicad_pcb / .step
├── fp-lib-table, sym-lib-table
├── libs/
│   ├── footprints.pretty/
│   └── symbols/
└── manufacturing/
    └── rev-1_2026-08-22/
        └── gerbers/
```
No `schematics/`, `spice/`, or `tools/` — this board has no hierarchical sheets and no LTSpice simulations on file yet.

## Key design notes
- **Inherited from Altium, not yet fully re-linked in KiCad.** Nearly every custom part on the schematic (`Burn Wire Dev Board-altium-import:*`, `Burn Wire Dev Board:root_N_*`, `PowerManagmentBoard_V4.0:*`) has no corresponding entry in `sym-lib-table`. The schematic still opens and renders correctly because KiCad caches each symbol's full geometry inline at placement time — `sym-lib-table` is only consulted when re-editing or re-placing a symbol. Don't take "it opens fine" as evidence the library links are healthy.
- **PCB layer names are non-standard**, carried over from the Altium import (e.g. `Mechanical 1/5/7/13/15`, `User.2/3/4` instead of KiCad's standard `Margin`/`User.Comments`/etc.). Not remapped in this reorg — flagged as a full production-file redo for later, out of scope for repo hygiene.
- **ADP2303 bootstrap diode** — see System overview above; this is a known failure mode for this regulator at 5V input, worth double-checking against the datasheet if this board is ever re-spun.
- `libs/symbols/` now holds four properly filed vendor symbols (B330B-13-F, B6B-XH-A_LF__SN_, NLV32T-3R3J-EF, WCAP-ASLI_5X5.5), but **none are currently referenced by the schematic** — the placed parts use generic `Device:`/`Connector:` symbols instead. These are available for future use, not currently wired in.

## Simulations
None on file for this board yet.

## Manufacturing history
| Rev | Date | Notes | Location |
|-----|------|-------|----------|
| 3.1 | 2026-08-22 | First fab | `manufacturing/rev-1_2026-08-22/gerbers/` |

## Open items / TBD
- PCB layer names still reflect the Altium import (`Mechanical 1/5/7/13/15`, `User.2/3/4`) — needs remapping to standard KiCad layers as part of a full production-file redo.
- `burn_wire:CR_SMB_320-345BE_DIO-L` and `burn_wire:IND_BOURNS_SRN6045` footprint nicknames on the PCB aren't yet re-linked to the `footprints` library in `fp-lib-table` — the backing files exist in `libs/footprints.pretty/` now, they just need the reference updated.
- `Burn Wire Dev Board:0603` and `PowerManagmentBoard_V4.0:TP` footprint nicknames have no backing file anywhere in the repo — genuinely missing, not just unlinked.
- Most schematic symbols are unlinked from `sym-lib-table` (see Key design notes) — re-linking or rebuilding these is a real schematic-level task, not a file move.
- The placed `burn_wire:CR_SMB_320-345BE_DIO-L` instance has a pad offset that doesn't cleanly match either library copy of that footprint (hand-nudged geometry) — worth a visual QC pass against the fabbed rev-1 board.
