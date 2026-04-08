# Changelog

## v1.4.0 — 2026‑04‑08

### Added
- New relay configuration model:
  - SET RELAY_MODE <NORMAL|LATCHED>
  - SET HAS_RX_RELAY <ON|OFF>
- Persistent config fields:
  - relayMode
  - hasRXRelay
- Status output now includes:
  - RX Relay Present
  - human‑readable Relay Mode
- Boot banner now reports:
  - relay hardware type (TX‑only / TX+RX)
  - current relay mode

### Changed
- Simplified relay mode handling:
  - internal TX‑only / TX+RX modes are derived from HAS_RX_RELAY
  - invalid TX+RX modes on TX‑only hardware are automatically mapped to TX‑only equivalents

### Notes
- Existing configurations default to HAS_RX_RELAY=ON unless overridden.
