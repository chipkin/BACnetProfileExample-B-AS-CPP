# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - unreleased

### Changed

- **`common/` 3.3.0 -> 3.4.0** and **`wiretest/` 0.5.0 -> 0.6.0**, both copied
  verbatim (`common/` from B-SS-CPP `5d78299`, `wiretest/` from B-LS-CPP). With
  `common/` 3.4.0 a broadcast the stack addresses to this socket's own address
  goes to the subnet broadcast instead; `wiretest/` 0.6.0 only adds helpers.
  `tests/discovery_test.py` now sends its Who-Is a second time, with a visible
  `[RETRY]` line, if no I-Am comes back in 3 s: Who-Is is unconfirmed and the
  example's UDP receive can drop a datagram on Windows. Discovery results are
  unchanged: 11/11 on Windows and Linux, no `[RETRY]`.
- **Labelled a Blocked Example.** The B-AS Profile Function (AA-AS-B: issuing
  access tokens with AuthRequest, ANSI/ASHRAE 135-2024 K.9.5 / L.14.1) cannot
  be performed through the stack's customer interface: the AuthRequest handler
  and the policy exports are test-tool only at the new pin, and the stack
  owner ruled on 2026-10-15 that they are not promoted for now
  ([stack #2043](https://github.com/chipkin/cas-bacnet-stack/issues/2043)).
  README, PICS, TODO.md and `main.cpp` say so and what the example does today.
- **Stack Pin moved to `6.x` @ `64b2cfac5` (6.0.23)** from `abd4cee1` (6.0.21),
  per the series rule that a pin moves to the latest `6.x` on any update. No
  source change was needed.
- **Protocol_Revision 24 -> 30** in the README, PICS, AGENTS.md and comments
  (30 is the stack's default at the pin). Segmentation is now
  `segmented-both`, by the stack (stack #2992); PICS section 5 updated.
- **`common/` 3.0.0 -> 3.3.0**, copied verbatim from B-SS-CPP `b640a81`
  (additive keys, and broadcasts with no connection string are sent).
- The AA-AS-B BIBB is named as K.9.5 names it, "Authorization Server - B"
  (was "Access Authorization - B"). README/PICS version lines were stale
  (said v1.0.0, common/ 2.5.0); fixed. The sample start-up output now shows
  the I-Am decode the program prints.

### Added

- **Discovery Wire Test** (`tests/discovery_test.py`, `tests/run_wire_tests.py`,
  copied from B-SS-CPP `da3e96f`): Who-Is, Object_List, every object's
  Property_List, and a ReadProperty of every listed property; any Error,
  Reject, Abort or silence fails it. 11/11 on Windows and Linux at the new
  pin. CI runs it on Linux after the smoke test.
- `wiretest/` 0.5.0, the series' Python harness, vendored verbatim from
  B-BBMD-CPP `75aedd8`.

## [1.0.2] - unreleased

### Changed

- **Device renamed from the series' colour placeholder "Rainbow" to "Chipkin
  Example B-AS"** so devices from different examples in the series are
  distinguishable from each other on the same BACnet network - every example
  previously announced the identical Object_Name "Rainbow", which made two
  examples on one subnet indistinguishable by name. Sub-object names (Analog
  Input 1 "Bronze", etc.) are unchanged - only the Device object's name
  changed. `docs/colour-table.md` (series root) updated to match. APP_VERSION
  bumped 1.0.1 -> 1.0.2.

## [1.0.1] - unreleased

### Fixed

- **`Application_Software_Version` (12) and `Firmware_Revision` (44) were
  hardcoded and stale** - both were literal `"1.0.0"` string constants
  (`APPLICATION_SOFTWARE_VERSION` and `FIRMWARE_REVISION` in `main.cpp`)
  that never tracked the example's real version as it advanced, and
  `Firmware_Revision` was never meant to be this example's own version at
  all - it names the underlying platform. Fixed the same way as the
  identical bug in the B-SCHUB-CPP example:
  `Application_Software_Version` now reads `APP_VERSION` directly (one
  source of truth, can't drift from `--version`'s own banner again), and
  the separate stale `APPLICATION_SOFTWARE_VERSION` constant was removed.
  `Firmware_Revision` is now built at runtime from the CAS BACnet Stack's
  own `BACnetStack_GetAPIMajorVersion()`/`GetAPIMinorVersion()`/
  `GetAPIPatchVersion()`/`GetAPIBuildVersion()` (the same 4 calls
  `common/CASExampleHelper.cpp`'s `PrintVersion()` already uses for the
  startup banner), populated once into `g_firmwareRevision` right after
  `LoadBACnetFunctions()` succeeds; the old hardcoded `FIRMWARE_REVISION`
  constant was removed.

### Changed

- Restructured documentation to match the series' README + TUTORIAL + PICS
  shape: `README.md` cut down to this example only (series-framing, the
  generic profile explanation, "What the profile requires" prose, "Before you
  ship", "Get the code", "Link mode", "Troubleshooting", "Extending the
  example", "Objects and properties", and the CC0 paragraph all removed or
  moved out); added `TUTORIAL.md` (extending the example, per-object-type
  serving checklist, "who serves what", reviewing your device, and
  troubleshooting - including the DeviceCommunicationControl and AA-AS-B
  entries); added `docs/PICS.md` (ANSI/ASHRAE 135 Annex A shape, with a Device
  object added to `docs/objects.json` as the generator's first entry).
- Switched the documented build from a prebuilt **STATIC** library
  (`tools/build-stack-static.sh`) to the adapter's default **SOURCE** mode:
  `cmake -B build -S .` / `cmake --build build --config Release`, identical on
  every platform and matching every other repository in the series.
  `CMakeLists.txt`'s header comment, `AGENTS.md`, and
  `.github/workflows/release.yml` (link-mode assertion, metrics JSON, matrix
  `lib:` entries, and packaged artifact list) updated to match.
- The `## Footprint` numbers still reflect the v1.0.0 STATIC-library build;
  the table now notes that the next release refreshes them under the
  documented SOURCE build.

## [1.0.0] - 2026-09-15

### Added

- First implementation of the BACnet **B-AS (Authorization Server)** profile
  example's DS/DM base: **DS-RP-B** (ReadProperty), **DM-DDB-B** (Who-Is/I-Am),
  **DM-DOB-B** (Who-Has/I-Have), and **DM-DCC-B**
  (DeviceCommunicationControl, with an optional password and the stack-managed
  enable/disable/disable-initiation state machine).
- Base-series objects: Device 389023 "Rainbow", Analog Input 1 "Bronze",
  Binary Input 1 "Emerald", Multi-State Input 1 "Hot Pink", Network Port 1
  "Vermilion" - the same read-only sensor set as B-SS, with no writable
  outputs (this profile does not require DS-WP-B).
- Seeded from `BACnetProfileExample-B-ASC-CPP`; `common/` vendored verbatim
  from `BACnetProfileExample-B-SS-CPP` at 2.5.0; CAS BACnet Stack pinned to
  `6.x` @ `abd4cee1` (reports 6.0.21), linked as a **STATIC** library.
- `docs/objects.json` and the generated **Objects and properties** README
  block (`tools/gen-objects-properties.py`); the series profile table block
  (`tools/sync-profile-table.sh`); a `## Footprint` placeholder row, filled
  at release.

### Known gap

- **AA-AS-B (Access Authorization) is not implemented.** At the pinned stack
  commit, the seeding exports it needs
  (`BACnetStack_AddAuthorizationPolicy` and siblings) exist only on the
  test-tool interface (`CASBACnetStackTestToolDLL.h`, behind
  `BACNET_STACK_TESTTOOL`), which this series' examples do not build against.
  See [TODO.md](TODO.md) for the full evidence and the filed stack issue
  ([chipkin/cas-bacnet-stack#2043](https://github.com/chipkin/cas-bacnet-stack/issues/2043)).
