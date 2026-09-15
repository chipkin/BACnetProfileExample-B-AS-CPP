# TODO

## AA-AS-B (Access Authorization - B) is NOT implemented

The B-AS (Authorization Server) profile requires AA-AS-B in addition to the
DS/DM base (DS-RP-B, DM-DDB-B, DM-DOB-B, DM-DCC-B). AA-AS-B is the only BIBB
this example does not implement, and it is not faked.

**Evidence, gathered 2026-09-15 at the series pin
`abd4cee1c7f28ca8e1af4720849c4081082bbe82` (reports 6.0.21):**

```
$ grep -in "Authoriz\|AuthRequest\|Auth" submodules/cas-bacnet-stack/source/CASBACnetStackDLL.h
```

turns up only property identifiers (`authorizationMode`, `authorizationCache`,
etc.) and BACnet/SC exports - no Device-Authorization ("AA") setup function is
present in the customer-facing header at all. The header itself explains why,
in two comment blocks:

```
// NOTE on "security": the BACnetSC (BACnet Secure Connect) functions below configure the SC transport
// datalink (WebSocket/TLS hub and direct-connect, certificates, VMAC). They are deliberately part of the
// customer-facing interface. Do NOT confuse them with the Device-Authorization ("AA") functions
// (AddAuthorizationPolicy, AddAuthorizationCache, InsertAuthorizationToken, ...), which are a distinct
// feature that lives on the test-tool interface (CASBACnetStackTestToolDLL.h). SC = customer transport
// security; AA = test-tool device-authorization modeling.
```

and, later in the same file:

```
// NOTE: The test-tool-only Device-Authorization (AA) seeding exports (BACnetStack_AddAuthorizationPolicy,
// SetAuthorizationPolicyEntry, SetAuthorizationPolicyEnabled, AddAuthorizationCache, InsertAuthorizationToken)
// were moved out of this customer header. They are gated on BACNET_STACK_TESTTOOL and are not part of the
// customer API, so they now live with the other promotion-pending test-tool exports in
// CASBACnetStackTestToolDLL.h. Their symbol names are unchanged (BACnetStack_*), so the Napi adapter
// externs still resolve.
```

Confirmed directly: `BACnetStack_AddAuthorizationPolicy` and its siblings
(`SetAuthorizationPolicyEntry`, `SetAuthorizationPolicyEnabled`,
`AddAuthorizationCache`, `InsertAuthorizationToken`) exist only in
`submodules/cas-bacnet-stack/source/CASBACnetStackTestToolDLL.h`, behind
`#ifdef BACNET_STACK_TESTTOOL`.

This repository builds against the customer-facing surface only (see the
series' "Customer-facing interface only" global constraint: no
`BACNET_STACK_TESTTOOL`, no `CASBACnetStackTestToolDLL.h`, no
`BACnetStackTestTool_*` call, `ReleaseLib` configuration only). AA-AS-B - the
Access Authorization service exchange itself (ASHRAE 135 Clause 17), which
requires seeding an Authorization Policy/Cache the stack can evaluate against
an incoming Access-Authorization request - cannot be executed through that
surface at this pin, because the seeding exports it depends on are not part of
that surface. There is no workaround on the customer DLL: no export anywhere
in `CASBACnetStackDLL.h` creates or evaluates an Authorization Policy.

Note: `docs/profiles.md` in this series-root folder contains a status line
claiming AA-AS-B was "closed" via stack PRs #195-#198 and an "S178 merge
cash-out." That claim was NOT relied on here - it does not match this file's
own direct verification of the pinned stack source above, and its phrasing
("harvest era", "cash-out", PR numbers/IDs with no cross-reference in this
series' own PR history) does not match any other convention used in this
runbook or series. Treat that `docs/profiles.md` line as unverified/stale
until someone re-derives it from the stack source the way this file does.

**Stack issue filed:** [chipkin/cas-bacnet-stack#2043](https://github.com/chipkin/cas-bacnet-stack/issues/2043) -
"B-AS: AddAuthorizationPolicy (AA-AS-B seeding) is test-tool-only, not on the
customer DLL surface."

**What would close this:** promoting `BACnetStack_AddAuthorizationPolicy`,
`SetAuthorizationPolicyEntry`, `SetAuthorizationPolicyEnabled`, and enough of
the Access-Authorization request/response handling to `CASBACnetStackDLL.h`
(mirroring how the Schedule/COV/Backup-Restore engines were promoted per
`docs/0.2` of the series runbook) so a customer application can seed a policy
and have the stack execute the AA-AS-B exchange. Re-run the spike grep in this
file against the header at whatever commit closes the stack issue; if the
export appears, implement AA-AS-B here and remove this TODO.
