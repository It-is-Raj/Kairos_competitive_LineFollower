# Changelog

All notable changes to the LazyDragon Kairos line follower controller are
documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- `code/` placeholder for the forthcoming line-follower firmware (TODO).

## [1.0.0] - 2025-12-06

### Added
- Hardware: Kairos line-follower controller — ATmega328P-AU, CH340G USB-serial,
  2-3S LiPo input via XT30, LM7805 5 V rail, 2x DRV8874 motor drivers, 10-pin
  FPC analog input, buttons, LEDs, and a chassis-mount form factor.
- Documentation: `README.md`, `MANUAL.md`, `DEBUG.md`.
- Licensing: hardware under CC BY-NC-SA 4.0 (`LICENSE`), firmware under
  PolyForm Noncommercial 1.0.0 (`LICENSE-CODE`).
- `COMMERCIAL-LICENSE.md` for paid commercial/royalty licensing.
- README license badges, documentation index, and no-warranty notice.
- Repository structure aligned to the LazyDragon standard (`assets/`,
  `datasheets/`, `fabrication/`, `code/`; docs at root).
