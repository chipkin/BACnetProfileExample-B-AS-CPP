# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_No changes yet._

## [1.1.0] - 2026-10-27

This is a **Blocked Example**. The CAS BACnet Stack cannot yet issue access tokens
(AA-AS-B, ANSI/ASHRAE 135-2024 K.9.5 / L.14.1) through its customer interface. See
the note at the top of the README.

- **Does today:** a discoverable BACnet/IP device that serves its Device, Network
  Port and three sensor objects. It answers ReadProperty, Who-Is/I-Am, Who-Has/I-Have
  and DeviceCommunicationControl (the profile's data-sharing and device-management
  base).
- **Does not do today:** it does not answer AuthRequest (a client gets Reject,
  unrecognized-service), issues no tokens, and AA-AS-B is not claimed in the PICS.

Built on CAS BACnet Stack 6.x (`69b6249f`, 6.0.23).

### Added

- **`--deviceName <text>`** sets the Device's `Object_Name` at start-up. The default
  is unchanged. `--help` lists the option, no longer lists a `--dcc-password`
  option the example never read, and `--xml` (print each frame as XML) now works.
- **Automated discovery self-test** under `tests/` reads every property of every
  object.
- **TUTORIAL.md** (extending the example, troubleshooting) and **docs/PICS.md**
  (Annex A style).

### Changed

- **Device renamed** from "Rainbow" to "Chipkin Example B-AS" so examples on one
  network can be told apart. Object names such as "Bronze" are unchanged.
- **Build now compiles the stack source directly into the executable**:
  `cmake -B build -S .` then `cmake --build build --config Release` on every
  platform; the separate static-library step is gone.
- **Protocol_Revision is 30** and segmentation is `segmented-both`.
- **PICS Database_Revision is 5**, as the stack counts the objects added at start-up
  (cl. 12.11).
- The AA-AS-B BIBB is named as Annex K.9.5 names it, "Authorization Server - B".

### Fixed

- **Application_Software_Version and Firmware_Revision** report the real build
  version and stack version instead of a fixed "1.0.0".
- **Requests no longer go unanswered on Windows** in rare timing cases (shared
  `common/` 3.5.0).

## [1.0.0] - 2026-09-15

### Added

- First **B-AS (Authorization Server)** profile example in C++, covering the
  profile's data-sharing and device-management base: **DS-RP-B** (ReadProperty),
  **DM-DDB-B** (Who-Is/I-Am), **DM-DOB-B** (Who-Has/I-Have) and **DM-DCC-B**
  (DeviceCommunicationControl, with an optional password).
- Objects: Device 389023, Analog Input 1 "Bronze", Binary Input 1 "Emerald",
  Multi-State Input 1 "Hot Pink" and Network Port 1 "Vermilion". All are read-only,
  since this profile does not require DS-WP-B.
- Built on CAS BACnet Stack 6.x (`abd4cee1`, 6.0.21).

### Known gap

- **AA-AS-B (Access Authorization) is not implemented.** The stack functions it
  needs are not available to a customer build. See [TODO.md](TODO.md).
