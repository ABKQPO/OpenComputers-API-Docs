# OpenComputers Lua API Documentation

This local documentation set is derived from the current workspace state of:

- `OpenComputers`
- `OpenSecurity`
- `LogisticsPipes`
- `StorageDrawers`
- `Computronics`
- `OpenPrinter`
- `Draconic-Evolution`
- `SGCraft`
- `OpenModularTurrets`
- `OCGlasses`

## Snapshot

- Last source rescan: `2026-09-05`

- Documentation languages: `English` and `中文`
- Total Markdown documents: `1001`
- English documents: `496`
- Chinese documents: `496`
- English syntax entries scanned from docs: `1226`
- Chinese syntax entries scanned from docs: `530`
- Source-reference repositories scanned:
  - `OpenComputers`
  - `OpenSecurity`
  - `LogisticsPipes`
  - `StorageDrawers`
  - `Computronics`
  - `OpenPrinter`
  - `Draconic-Evolution`
  - `SGCraft`
  - `OpenModularTurrets`
  - `OCGlasses`

## Coverage Counts Per Language

Each language tree currently contains the same section counts:

- Components: `102`
- Commands: `196`
- Integrations: `36`
- Libraries: `130`
- Core Runtime: `6`
- Systems: `13`
- Examples: `5`
- Appendix: `7`

## Language Roots

- [English](./opencomputers-api-en/README.md)
- [中文](./opencomputers-api-zh/README.md)

## Scope Notes

- The documentation set was manually reviewed against source code spread across the repositories listed above.
- The syntax-entry counts above are based on explicit `Syntax:` / `语法:` lines present in the current Markdown pages, so they describe documentation structure density rather than the exact upstream API total by themselves.
- Appendix inventory pages remain in the export to preserve audit context and cross-check history.
- The 2026-08-31 rescan synchronized `OpenComputers` (`c3a147af`), `OpenSecurity` (`d94ead8`), `LogisticsPipes` (`5c0de07`), and `StorageDrawers` (`59b527f`). It added the TecTech BEC component reference, corrected AE/Thaumic Energistics one-based slot and event payload behavior, expanded LogisticsPipes fluid request coverage, and annotated StorageDrawers' existing `drawer` API with its current source-audit status.
- The 2026-09-01 OpenComputers rescan includes PR #219 (`971815a7a` / `ae135bb`) and adds the `me_cellworkbench` AppEng component reference in both languages.
- The 2026-09-05 OpenComputers rescan includes PR #221 (`18c66d3e2` / `093ec4f`) and updates the `bec_diode` TecTech component from a single condensate filter to a multi-slot filter API in both languages.
- The 2026-09-05 scan added `OpenModularTurrets` (`d68cad3b`) and `OCGlasses` (`bb1a1bb`). It documents all five tier-specific turret-base components, the shared turret controls, the `glasses` terminal callbacks, widget object methods, widget types, and terminal signals in both languages.

## Audit Artifacts

- The original working repository also contains auxiliary audit artifacts under `docs/opencomputers-api-work/data/`.
- The working tree currently keeps those audit artifacts alongside the finished documentation pages.
