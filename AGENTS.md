# AGENTS.md

Guidance for AI coding agents working in this repository. See
<https://agents.md/> for the format. Human contributors should read
[README.md](README.md) first, then [TUTORIAL.md](TUTORIAL.md).

## What this project is

A **tutorial** C++ example that implements the DS/DM base of the BACnet
**B-AS (Authorization Server)** device profile using the CAS BACnet Stack. It
is one of a series - one git repo per BACnet profile. It does **not**
implement AA-AS-B, the profile's one Access-Authorization BIBB - see
[TODO.md](TODO.md) for the stack gap that blocks it. The top priority is that
the code reads like a tutorial a customer can learn from and copy-paste.
Favour clarity over cleverness.

## Layout

This repository is self-contained:

- `main.cpp` - the example device.
- `common/` - the shared helper (vendored).
- `README.md` - what this example is. Keep it short and about THIS example only.
- `TUTORIAL.md` - how to extend and review the example. Long-form material that
  would bloat the README belongs here.
- `docs/PICS.md` - the Protocol Implementation Conformance Statement. Its
  objects-and-properties section is GENERATED from `docs/objects.json`; do not
  hand-edit between the `OBJECTS-PROPERTIES` markers.
- `docs/objects.json` - the input to that generator. Update it in the same change
  as any `main.cpp` change that adds an object or a `GetProperty*` branch.
- `TODO.md` - the AA-AS-B gap, with evidence and the filed stack issue.
- `submodules/cas-bacnet-stack/` - the **CAS BACnet Stack** as a git submodule
  (private; compiled from source). After cloning, run
  `git submodule update --init --recursive`.

The `PROFILE-TABLE` block in README.md is also generated, from the example-series
repository's `docs/profile-table.md`. Edit it there, not here.

## Build

Plain CMake, identical on every platform, in the adapter's default SOURCE mode
(the stack's sources are compiled into the executable - no prebuilt library, no
DLL, no per-platform pre-step):

```bash
git submodule update --init --recursive   # once, if not cloned with --recursive
cmake -B build -S .
cmake --build build --config Release
```

The first build compiles the whole stack (~600 files) and takes a few minutes;
rebuilds after that are incremental and fast. Use `-D CAS_STACK_DIR=...` only if
your stack lives outside the bundled submodule. Do not reintroduce a link-mode
flag or a series-root build script into the documented build: a customer
downloads this repository on its own and must be able to build it with the two
commands above.

## Run

```bash
./build/BACnetExampleBAS [--port 47808] [--deviceID 389023]   # Linux/macOS
.\build\Release\BACnetExampleBAS.exe [--port 47808] [--deviceID 389023]   # Windows
```

Interactive keys while running: `h` help, `q` quit, up/down nudge Analog Input 1.

## Conventions

- Device is named "Rainbow"; objects use the series' colour names; vendor id 389.
- Implement **only** the services and objects the profile's implemented BIBBs
  require - but expose **every required property** of each object for
  Protocol_Revision 24. This repo intentionally does NOT enable WriteProperty
  or ReadPropertyMultiple - they are not part of B-AS's required set.
- DeviceCommunicationControl (DM-DCC-B): the stack runs the enable/disable state
  machine; the `DeviceCommunicationControl` callback just validates `DCC_PASSWORD`
  and logs. The deprecated plain `disable` (1) is rejected by the stack at
  Protocol_Revision >= 20 - only `enable` (0) and `disable-initiation` (2) apply.
  This callback has no fallback error code: it must set `*errorCode` on every
  `false` return.
- Every `GetProperty*` callback ends with `uint32_t* errorCode`. Leave it alone
  on a catch-all decline (the stack's decline-and-fabricate default answers
  required properties this app does not serve); set it only where this device
  knows the read is wrong (`State_Text` out of range is the one case here).
- Match the surrounding code style: `const`-correct parameters, check every stack
  return value, keep `main.cpp` linear and well-commented.
- **AA-AS-B is not implemented, and must not be faked.** Do not add a
  `BACnetStack_AddAuthorizationPolicy` call or any other test-tool-only export
  to this repo - it would require `BACNET_STACK_TESTTOOL`, which this series
  forbids. Re-verify [TODO.md](TODO.md)'s grep against the pinned stack header
  before assuming this is still true; if the export has moved to the customer
  surface, implement AA-AS-B and delete the TODO entry.
- **Never edit `common/` in this repo alone** - it is a vendored copy shared by
  every example in the series, with its own version (`COMMON_VERSION`) and
  changelog (`common/CHANGELOG.md`). To change it: edit, bump the version, add
  a changelog entry, then re-copy `common/` into every example repository.

## How to verify a change

There are no unit tests; verification is behavioural:

1. Build, then run one instance on a clear UDP port.
2. With a BACnet client (e.g. the CAS BACnet Explorer, or `bacpypes3`/`BAC0`),
   send **Who-Is** and confirm **I-Am** from the device instance; send
   **Who-Has** for an object name and confirm **I-Have**.
3. **ReadProperty** every required property of every object and confirm the
   values; confirm `Protocol_Revision` is 24 and `Object_List` lists all objects.
4. Confirm **WriteProperty** and **ReadPropertyMultiple** are rejected with
   `unrecognized-service` - neither is part of this profile's required set.
5. **DeviceCommunicationControl**: confirm `disable-initiation` and `enable`
   SimpleACK, the deprecated `disable` is rejected (service-request-denied), and a
   wrong password (if `DCC_PASSWORD` is set) is rejected (password-failure).
6. If you changed the objects or their properties, regenerate `docs/PICS.md`
   (`python tools/gen-objects-properties.py BACnetProfileExample-B-AS-CPP` from
   the series root) and confirm no row comes out flagged with ⚠.

## Releasing

Bump `APP_VERSION` in `main.cpp` and add an entry to [CHANGELOG.md](CHANGELOG.md),
then tag `vX.Y.Z`. The GitHub Actions workflow builds and publishes the release.

## License

See [LICENSE](LICENSE). The CAS BACnet Stack is a separate, commercially
licensed product and is not covered by it.
