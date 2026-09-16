# BACnet B-AS (Authorization Server) - C++ example

A minimal, copy-paste-friendly example showing how to implement the BACnet
**B-AS (BACnet Authorization Server)** device profile's **DS/DM base** in C++
using the
[CAS BACnet Stack](https://store.chipkin.com/services/stacks/bacnet-stack).
It listens on **BACnet/IP (UDP 47808)**, answers **ReadProperty** requests, is
discoverable via **Who-Is / I-Am** and **Who-Has / I-Have**, and responds to
**DeviceCommunicationControl**.

**[Download a prebuilt binary](https://github.com/chipkin/BACnetProfileExample-B-AS-CPP/releases)**
(Windows and Linux x64) - or build it yourself, see [Build](#build) below.

- **[TUTORIAL.md](TUTORIAL.md)** - how to extend this example and how to review
  it for conformance. Read it when you start turning this into your own device.
- **[docs/PICS.md](docs/PICS.md)** - the Protocol Implementation Conformance
  Statement: every object, every property, and who answers it.

This example claims **only** the B-AS profile's DS/DM base - see
[What this example supports](#what-this-example-supports) for the one BIBB
(AA-AS-B) it does not implement, and why.

> **Versions:** this document describes **example v1.0.0**, built and verified
> against **CAS BACnet Stack 6.0.21** (`6.x` @ `abd4cee1`), at
> **Protocol_Revision 24**, with the vendored `common/` helper at **v2.5.0**.
> Running the example prints all three - if what it prints disagrees with this
> line, trust the program and check `CHANGELOG.md`.

## What is the B-AS (Authorization Server) profile?

**B-AS (BACnet Authorization Server)**, defined in Annex L.14 of ANSI/ASHRAE
135, is the device that other BACnet devices ask to authorize an access
request - the server side of the Access Authorization exchange defined in
Clause 17.

## DeviceCommunicationControl (DM-DCC-B)

`DeviceCommunicationControl` lets a management station tell a device to stop
(or resume) communicating - useful to quiet a device during commissioning. The
CAS BACnet Stack runs the actual enable/disable state machine (and the
optional re-enable timer); this example's `DeviceCommunicationControl`
callback in `main.cpp` validates the optional password and logs what was
asked. At Protocol_Revision 24 the plain "disable" value is deprecated - the
stack itself rejects it and only `enable` and `disable-initiation` take
effect - see the callback's comments in `main.cpp` for the full detail. This
example's password is empty (`DCC_PASSWORD = ""`), so it accepts any request;
set it to require one.

## The device this example creates

```
Device 389023  "Rainbow"   (Vendor 389 - Chipkin Automation Systems)
    │
    ├── Analog Input 1       "Bronze"      Present_Value  21.5    (REAL, degrees Celsius)
    ├── Binary Input 1       "Emerald"     Present_Value  inactive  (0 = inactive / 1 = active)
    ├── Multi-State Input 1  "Hot Pink"    Present_Value  1       (state, 1..3)
    └── Network Port 1       "Vermilion"   the BACnet/IP port     (required on every device)
```

These are the same base-series sensor objects every example in the series
carries; a real Authorization Server would also carry the Access User / Access
Rights / Access Credential objects Clause 17 defines, but those only matter
once AA-AS-B itself can be executed (see below) - adding them without a
working service exchange behind them would not demonstrate anything.

## What this example supports

The example implements the capabilities below - and, deliberately, not
AA-AS-B. This is **not** the full B-AS profile; it is the DS/DM base plus an
honestly-documented gap.

### BIBBs (BACnet Interoperability Building Blocks)

| BIBB | Description | Supported |
|------|-------------|:---------:|
| DS-RP-B | Data Sharing - ReadProperty - B | ✅ |
| DM-DDB-B | Device Management - Dynamic Device Binding - B | ✅ |
| DM-DOB-B | Device Management - Dynamic Object Binding - B | ✅ |
| DM-DCC-B | Device Management - DeviceCommunicationControl - B | ✅ |
| AA-AS-B | Access Authorization - B | ❌ **waiting on stack export** - see [TODO.md](TODO.md) |

**AA-AS-B: waiting on stack export `BACnetStack_AddAuthorizationPolicy`** (and
its siblings `SetAuthorizationPolicyEntry`, `SetAuthorizationPolicyEnabled`,
`AddAuthorizationCache`, `InsertAuthorizationToken`). At the series pin
(`abd4cee1c7f28ca8e1af4720849c4081082bbe82`, 6.0.21) those exports exist only
in `CASBACnetStackTestToolDLL.h`, gated behind `BACNET_STACK_TESTTOOL` - the
stack header itself says they are "a distinct feature that lives on the
test-tool interface," not the customer-facing `CASBACnetStackDLL.h` this
series builds against (see the series' "Customer-facing interface only" rule).
Without a way to seed an Authorization Policy, there is nothing for an
Access-Authorization request to be evaluated against, so AA-AS-B cannot be
executed through the customer DLL surface at this pin. Full evidence,
including the exact header comments, is in [TODO.md](TODO.md). Stack issue:
[chipkin/cas-bacnet-stack#2043](https://github.com/chipkin/cas-bacnet-stack/issues/2043).

### Services (executed / B-side)

| Service | Notes |
|---------|-------|
| ReadProperty | Responds to property reads (DS-RP-B). |
| Who-Is / I-Am | Answers Who-Is with I-Am, and broadcasts an I-Am on start-up (DM-DDB-B). |
| Who-Has / I-Have | Answers Who-Has with I-Have (DM-DOB-B). |
| DeviceCommunicationControl | Stops/resumes communication, optionally timed/passworded (DM-DCC-B). |

### Object types

| Object type | Instance | Name |
|-------------|:--------:|------|
| Device | 389023 | Rainbow |
| Analog Input | 1 | Bronze |
| Binary Input | 1 | Emerald |
| Multi-State Input | 1 | Hot Pink |
| Network Port | 1 | Vermilion |

Every required property of every object, and who answers it, is in
[docs/PICS.md](docs/PICS.md).

## Requires the CAS BACnet Stack (licensed product)

This example **builds against the CAS BACnet Stack, which is a commercial Chipkin
product** - it is not free or open source, and there is no public/trial build.
The stack is referenced here as the **private** git submodule
`submodules/cas-bacnet-stack`; you can only fetch and build it once you have a CAS
BACnet Stack license and access to that repository.

**To get the CAS BACnet Stack (and access to build this example), contact
Chipkin:** <https://store.chipkin.com/services/stacks/bacnet-stack> or
sales@chipkin.com.

You do not need a stack licence to *read* this example, or to run a
[prebuilt release binary](https://github.com/chipkin/BACnetProfileExample-B-AS-CPP/releases).
The licence is what lets you *build* it - that is the part the stack submodule
gates.

## What's in this repository

This is a **self-contained** project. It ships:

- `main.cpp` - the example device.
- `common/` - the shared helper (UDP, callbacks, CLI, keyboard) vendored in.
- `CMakeLists.txt` - the build, the same on Windows, Linux, and macOS.
- `docs/PICS.md` - the conformance statement.
- `TODO.md` - the AA-AS-B gap, with evidence and the filed stack issue.
- `submodules/cas-bacnet-stack/` - the **CAS BACnet Stack as a git submodule**
  (private; requires a license - see above). Its sources are compiled into the
  executable, so there is no library or DLL to build, ship, or install.

## Prerequisites

- A C++17 compiler (MSVC, GCC, or Clang).
- CMake >= 3.15.
- Git (to fetch the stack submodule).

### Windows

- **C++ compiler** - install
  [Visual Studio Community](https://visualstudio.microsoft.com/downloads/)
  (free) and select the **"Desktop development with C++"** workload.
- **CMake** - from <https://cmake.org/download/>, or `winget install Kitware.CMake`.

### Linux / macOS

- Debian/Ubuntu: `sudo apt install build-essential cmake git`
- macOS: `xcode-select --install` and `brew install cmake`

## Build

CMake only, and the same two commands on every platform:

```bash
git clone --recursive https://github.com/chipkin/BACnetProfileExample-B-AS-CPP.git
cd BACnetProfileExample-B-AS-CPP

cmake -B build -S .
cmake --build build --config Release
```

Already cloned without `--recursive`? Run `git submodule update --init --recursive`
first - the build needs the stack submodule.

> **The first build takes a few minutes** - it compiles the entire CAS BACnet
> Stack (~600 source files) into the executable. Rebuilds after that are
> incremental and take seconds.

If your CAS BACnet Stack lives somewhere other than the bundled submodule, point
CMake at it: `cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack`.

## Run

```bash
# Linux / macOS
./build/BACnetExampleBAS

# Windows
.\build\Release\BACnetExampleBAS.exe
```

Expected output:

```
BACnet Authorization Server (B-AS) Example - C++ v1.0.0
CAS BACnet Stack version: 6.0.21.0
Common helper (common/) version: 2.5.0
FYI: Listening for BACnet/IP on UDP port 47808 (Network Port 1).
TX 21 bytes to 192.168.3.255:47808 (broadcast) (Network Port 1)
FYI: Device 389023 ("Rainbow") ready. Vendor ID 389. Press 'h' for help.
```

The `TX` line is the start-up I-Am the device broadcasts to announce itself. It
goes to the **local subnet broadcast** address (here `192.168.3.255`, computed
from the Network Port's interface), not the global `255.255.255.255`. As clients
talk to the device you'll see `RX ... bytes from ...` and `TX ... bytes to ...`
lines showing the traffic.

The device listens on UDP **47808** (BACnet/IP). Allow that port through your
firewall. To use a different port, pass `--port` (see below).

> **A wall of red `Error:` lines at start-up is expected and is not your bug** -
> it is the stack's own debug logging (the device hearing its own broadcast I-Am,
> and a one-time BACnet/SC UUID notice). [TUTORIAL.md](TUTORIAL.md#troubleshooting)
> explains both.

### Command-line options

| Option | Default | Meaning |
|--------|---------|---------|
| `--port <n>` | `47808` | UDP port to listen on (BACnet/IP). |
| `--deviceID <n>` | `389023` | The device's BACnet instance number (BACnet requires this to be configurable). |
| `--help`, `-h` | - | Show usage and exit. |
| `--version` | - | Print the example, stack, and `common/` helper versions, then exit. |

### Interactive commands

While the example runs, these keys are available:

| Key | Action |
|-----|--------|
| `h` | Show the version information and this command list. |
| `q` | Quit. |
| up arrow | Increase Analog Input 1 (`Bronze`) by 1.1. |
| down arrow | Decrease Analog Input 1 (`Bronze`) by 1.1. |

The up/down keys change the live `Present_Value` of the analog input, so a client
re-reading it sees the new value.

## Verify

Use a BACnet client such as the
[**CAS BACnet Explorer**](https://store.chipkin.com/products/tools/cas-bacnet-explorer):

1. **Discover** - send a **Who-Is**. The device replies with **I-Am** from
   instance **389023** (vendor **389**). It also broadcasts an I-Am at start-up.
2. **Who-Has / I-Have** - send a **Who-Has** for object name `Bronze`. The
   device replies with **I-Have**.
3. **Browse the object model** - the device shows five objects: the Device
   (`Rainbow`), the three sensors, and the Network Port (`Vermilion`). Reading
   the Device's `Object_List` returns all five.
4. **Read the Device** - ReadProperty `389023` -> `Object_Name` returns
   `"Rainbow"`; `Protocol_Revision` returns `24`; `Description` returns the
   profile description string; `Protocol_Services_Supported` lists
   `read-property`, `device-communication-control`, `i-am`, `i-have`,
   `who-has`, `who-is` - and nothing else.
5. **Read a sensor** - ReadProperty Analog Input `1` -> `Present_Value` returns
   `21.5`; `Units` returns `degrees-Celsius`. Repeat for Binary Input `1`
   (`"Emerald"`) and Multi-State Input `1` (`"Hot Pink"`).
6. **Confirm the profile boundary** - a **WriteProperty** or
   **ReadPropertyMultiple** to any object is rejected with
   `unrecognized-service` - neither is part of this profile's required set.
7. **DeviceCommunicationControl (DM-DCC-B)** - send `DeviceCommunicationControl`
   with `enableDisable = disable-initiation`. The device answers with a
   SimpleACK and logs the request; send `enable` to restore normal operation.

For a property-by-property review against the conformance statement, see
[TUTORIAL.md](TUTORIAL.md).


## The BACnet profile example series

<!-- PROFILE-TABLE:BEGIN (generated from cas-bacnet-stack-examples/docs/profile-table.md - do not edit here) -->
The CAS BACnet Stack supports every standardized device profile in ASHRAE 135-2024 Annex L, and there is one example repository per profile. Pick the profile your device claims, then the language you build in. "Ask" means the example hasn't been built yet for that language - [contact Chipkin](https://store.chipkin.com/contact-us) if you need one.

### Controllers (Annex L.4)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-SS** Smart Sensor | [B-SS-CPP](https://github.com/chipkin/BACnetProfileExample-B-SS-CPP) | Ask | Ask | Ask | Ask |
| **B-SA** Smart Actuator | [B-SA-CPP](https://github.com/chipkin/BACnetProfileExample-B-SA-CPP) | Ask | Ask | Ask | Ask |
| **B-ASC** Application Specific Controller | [B-ASC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ASC-CPP) | [B-ASC-Node](https://github.com/chipkin/BACnetProfileExample-B-ASC-Node) | Ask | Ask | Ask |
| **B-AAC** Advanced Application Controller | [B-AAC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AAC-CPP) | Ask | Ask | Ask | Ask |
| **B-BC** Building Controller | [B-BC-CPP](https://github.com/chipkin/BACnetProfileExample-B-BC-CPP) | Ask | Ask | Ask | Ask |

### Life safety controllers (Annex L.5)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-LSC** Life Safety Controller | [B-LSC-CPP](https://github.com/chipkin/BACnetProfileExample-B-LSC-CPP) 🚧 | Ask | Ask | Ask | Ask |
| **B-ALSC** Advanced Life Safety Controller | [B-ALSC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ALSC-CPP) | Ask | Ask | Ask | Ask |

### Access control controllers (Annex L.6)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-ACC** Access Control Controller | [B-ACC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACC-CPP) | Ask | Ask | Ask | Ask |
| **B-AACC** Advanced Access Control Controller | [B-AACC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AACC-CPP) | Ask | Ask | Ask | Ask |

### Lighting controllers (Annex L.11)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-LD** Lighting Device | [B-LD-CPP](https://github.com/chipkin/BACnetProfileExample-B-LD-CPP) | Ask | Ask | Ask | Ask |
| **B-LS** Lighting Supervisor | [B-LS-CPP](https://github.com/chipkin/BACnetProfileExample-B-LS-CPP) | Ask | Ask | Ask | Ask |

### Elevator controllers (Annex L.13)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-EM** Elevator Monitor | [B-EM-CPP](https://github.com/chipkin/BACnetProfileExample-B-EM-CPP) | Ask | Ask | Ask | Ask |
| **B-EC** Elevator Controller | [B-EC-CPP](https://github.com/chipkin/BACnetProfileExample-B-EC-CPP) | Ask | Ask | Ask | Ask |
| **B-AEC** Advanced Elevator Controller | [B-AEC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AEC-CPP) | Ask | Ask | Ask | Ask |

### Authentication and authorization (Annex L.14)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-AS** Authorization Server | [B-AS-CPP](https://github.com/chipkin/BACnetProfileExample-B-AS-CPP) | Ask | Ask | Ask | Ask |

### Miscellaneous (Annex L.7, combinable with any one family)

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-BBMD** Broadcast Management Device | [B-BBMD-CPP](https://github.com/chipkin/BACnetProfileExample-B-BBMD-CPP) | Ask | Ask | Ask | Ask |
| **B-ACDC** Access Control Door Controller | [B-ACDC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACDC-CPP) | Ask | Ask | Ask | Ask |
| **B-ACCR** Access Control Credential Reader | [B-ACCR-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACCR-CPP) | Ask | Ask | Ask | Ask |
| **B-RTR** Router | [B-RTR-CPP](https://github.com/chipkin/BACnetProfileExample-B-RTR-CPP) | Ask | Ask | Ask | Ask |
| **B-GW** Gateway | [B-GW-CPP](https://github.com/chipkin/BACnetProfileExample-B-GW-CPP) | Ask | Ask | Ask | Ask |
| **B-DAP** Device Address Proxy | [B-DAP-CPP](https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP) | Ask | Ask | Ask | Ask |
| **B-SCHUB** BACnet/SC Hub | [B-SCHUB-CPP](https://github.com/chipkin/BACnetProfileExample-B-SCHUB-CPP) | Ask | Ask | Ask | Ask |
| **B-GENERAL** General device (Annex L.8) | *(satisfied by every example above)* | — | — | — | — |

### Operator interfaces and workstations (Annex L.1–L.3, L.9–L.10, L.12)

Client-side profiles.

| Profile | C++ | Node.js | C# | Rust | Python |
|---|---|---|---|---|---|
| **B-OD** Operator Display | [B-OD-CPP](https://github.com/chipkin/BACnetProfileExample-B-OD-CPP) | Ask | Ask | Ask | Ask |
| **B-OWS** Operator Workstation | planned | — | — | — | — |
| **B-AWS** Advanced Operator Workstation | planned | — | — | — | — |
| **B-XAWS** Extended Advanced Operator Workstation | planned | — | — | — | — |
| **B-LSAP** Life Safety Annunciator Panel | planned | — | — | — | — |
| **B-LSWS** Life Safety Workstation | planned | — | — | — | — |
| **B-ALSWS** Advanced Life Safety Workstation | planned | — | — | — | — |
| **B-ACSD** Access Control Security Display | planned | — | — | — | — |
| **B-ACWS** Access Control Workstation | planned | — | — | — | — |
| **B-AACWS** Advanced Access Control Workstation | planned | — | — | — | — |
| **B-LOD** Lighting Operator Display | planned | — | — | — | — |
| **B-ALWS** Advanced Lighting Workstation | planned | — | — | — | — |
| **B-LCS** Lighting Control Station | planned | — | — | — | — |
| **B-ALCS** Advanced Lighting Control Station | planned | — | — | — | — |
| **B-ED** Elevator Display | planned | — | — | — | — |
| **B-EWS** Elevator Workstation | planned | — | — | — | — |
| **B-AEWS** Advanced Elevator Workstation | planned | — | — | — | — |

🚧 = in progress. "Ask" = not yet built for that language; contact Chipkin if you need it. Profile definitions: ANSI/ASHRAE 135-2024 Annex L. BIBB definitions: Annex K. Get the stack: <https://store.chipkin.com/services/stacks/bacnet-stack>.
<!-- PROFILE-TABLE:END -->

## References

- **ANSI/ASHRAE Standard 135** (BACnet) - the protocol standard. Object model
  (Clause 12), services (Clause 15), Access control authorization (Clause 17),
  BACnet/IP (Annex J), device profiles (Annex L). Purchase / preview via the
  [ASHRAE store](https://www.ashrae.org/technical-resources/standards-and-guidelines).
- **What is BACnet?** - Chipkin's introduction:
  <https://docs.chipkin.com/protocols/bacnet/>.
- **CAS BACnet Stack** - product page and documentation:
  <https://store.chipkin.com/services/stacks/bacnet-stack>.
- **CAS BACnet Explorer** - client for testing this device:
  <https://store.chipkin.com/products/tools/cas-bacnet-explorer>.
- **Shared helper used by this example** - [`common/README.md`](common/README.md).
- **The AA-AS-B gap** - [`TODO.md`](TODO.md).

See also [TUTORIAL.md](TUTORIAL.md), [docs/PICS.md](docs/PICS.md),
[CHANGELOG.md](CHANGELOG.md), [AGENTS.md](AGENTS.md), and [TODO.md](TODO.md).
