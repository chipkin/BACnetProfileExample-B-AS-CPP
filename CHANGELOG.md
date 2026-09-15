# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
