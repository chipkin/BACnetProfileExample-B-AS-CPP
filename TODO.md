# TODO

## Blocked Example: AA-AS-B needs stack #2043

The B-AS Profile Function is AA-AS-B (Authorization Server - B, ANSI/ASHRAE
135-2024 K.9.5): "capable of issuing access tokens to authorization clients or
their helpers. See Clause 17." L.14.1 adds "a persistent database of access
policies" and "a User Interface suitable for configuring the authorization
policies". This example cannot do any of it through the stack's customer
interface, so it ships as a **Blocked Example**: the DS/DM base (DS-RP-B,
DM-DDB-B, DM-DOB-B, DM-DCC-B) works and is wire-tested; AA-AS-B is not
claimed and not faked.

**Stack issue:** [chipkin/cas-bacnet-stack#2043](https://github.com/chipkin/cas-bacnet-stack/issues/2043)
(open). On 2026-10-15 the stack owner ruled it **not promoted** for now
(stack PR #3937, commit `aeaa1a37e`): "This blocks the B-AS profile example
(AA-AS-B) until it is done."

**Evidence, re-checked 2026-10-26 at the Stack Pin `6.x` @ `69b6249f`
(6.0.23):**

- `submodules/cas-bacnet-stack/source/CASBACnetStackDLL.h` (the customer
  header) has no `AuthRequest` and no authorization-policy export. It names
  them only in two comments: the Device-Authorization ("AA") functions are "a
  distinct feature that lives on the test-tool interface", and the seeding
  exports "were moved out of this customer header".
- `BACnetStack_AddAuthorizationPolicy`, `SetAuthorizationPolicyEntry` and
  `SetAuthorizationPolicyEnabled` are declared only in
  `CASBACnetStackTestToolDLL.h`.
- The AuthRequest service handler itself is test-tool only:
  `BACnetAuthRequestProcessor.h` is wrapped in
  `#if defined(BACNET_STACK_TESTTOOL) && defined(STACK_OPTION_AUTH_REQUEST)`,
  and so is the `authRequest` dispatch in `BACnetConfirmedRequestProcessor.cpp`.
  A customer build therefore refuses AuthRequest even if the service bit were
  set.
- New at `69b6249f`: the customer header's `BACnetStack_SetServiceEnabled`
  documentation says AuthRequest is a service "which only a test-tool build
  executes" (enabling it returns false), and the AuthRequest host hook the
  stack added (`BACnetStackTestTool_RegisterCallbackProcessAuthRequest`) is a
  test-tool export too.
- On the wire (this example at `69b6249f`): Protocol_Services_Supported leaves
  the auth-request bit clear, and an AuthRequest token-request (confirmed
  service 34) is answered with Reject, unrecognized-service.

The series builds against the customer-facing surface only (no
`BACNET_STACK_TESTTOOL`, no `CASBACnetStackTestToolDLL.h`, no
`BACnetStackTestTool_*` call), so there is no workaround in this repo.

**What #2043's design notes say has to happen first** (the issue's
2026-10-15 comment): re-gate the policy store and the AuthRequest processor
onto `STACK_OPTION_DEVICE_AUTHORIZATION` / `STACK_OPTION_AUTH_REQUEST`, move
the policy exports (and the four policy-term setters) to the customer header,
and settle two open points - the scope parameter's shape, and that issued
tokens are unsigned while cl. 17.4 requires ES256.

**When it closes:** re-run the grep below at the new Stack Pin. If the exports
are in `CASBACnetStackDLL.h` and a customer build answers AuthRequest,
implement the Profile Function (a policy table configured by the example,
AuthRequest answered with a token for a client the policy allows), add a Wire
Test with a Python authorization-client Peer and a Failure Mode (a client
outside the policy gets an Error, not a token), drop the Blocked label, and
delete this entry.

```
grep -n "AuthRequest\|AddAuthorizationPolicy" submodules/cas-bacnet-stack/source/CASBACnetStackDLL.h
grep -n "BACNET_STACK_TESTTOOL" submodules/cas-bacnet-stack/source/BACnetAuthRequestProcessor.h
```

## Series documentation (not this repo)

`docs/profiles.md` in the series root says B-AS "flipped ✅" in an "S178 merge
cash-out". That describes the stack's test-tool surface, not the customer
interface this example uses; it does not hold for this example.

## Coverage gaps the test tool still reports

DeviceCommunicationControl rows 9.24.1.6, .8, .9 and .10 are skipped because the
test tool has no trigger it can drive on a device that only sends I-Am and
I-Have (ref: `chipkin/chipkin-bacnet-testing-tool#355`). The test tool has no
AA-AS-B section, so the Profile Function itself is not measured. No BTL FAIL
remains at this release.
