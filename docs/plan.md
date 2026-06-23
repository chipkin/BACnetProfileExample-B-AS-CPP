# Plan (STUB): B-AS (Authorization Server) — C++ example

> **STATUS: STUB.** Seed facts below. Expand from
> [`bacnet-profile-plan-template.md`](../../bacnet-profile-plan-template.md) after the
> sample plans ([B-LD](../../BACnetProfileExample-B-LD-CPP/docs/plan.md),
> [B-BC](../../BACnetProfileExample-B-BC-CPP/docs/plan.md)) are reviewed.

**Profile:** B-AS · **Family:** Annex L.14 (Authentication & Authorization) ·
**Role:** B · **Archetype:** Controller · **Difficulty:** 5/5 (**frontier — verify
stack support first**) · **Build wave:** 5 (Phase A, capstone)

**Thesis:** an **Authorization Server** — implements the Clause-17 BACnet
authorization subsystem (AA-AS-B). The only B-role profile whose defining BIBB may
**not yet be available on the standard shipping DLL**.

## Required BIBBs (profiles.md L.14)
`DS-RP-B; DM-DDB-B, DM-DOB-B, DM-DCC-B; AA-AS-B`.

## Services to enable
- ReadProperty (1), DCC (17), baseline discovery, + AA-AS-B (Clause-17).

## Objects (baseline + )
- Authorization-subsystem objects (TBD once AA-AS-B is exposed). Confirm.

## Shared features
- **DEFINE:** F-AUTH (AA-AS-B authorization server).
- **REUSE:** F-DCC (B-ASC).

## Known stack gaps — **possibly blocked (master plan §7 risk 5)**
- profiles.md lists B-AS as **🟡**: AA-AS-B is "an entire unimplemented subsystem
  with no service primitive," shipped to the stack via PRs #195–#198 but **pending
  tool-side wireup PASS evidence**. **Confirm AA-AS-B is exported by the standard
  DLL the example links before committing.** If not, B-AS stays a **documented
  placeholder** (`README` + `TODO.md` explaining the profile and what the stack
  must expose) until the Authorization subsystem ships in the public surface.

## Notes / open questions
1. Is AA-AS-B in the standard `CASBACnetStackDLL.h` yet? (Gate the whole example.)
2. What objects/properties does the Authorization subsystem add?
3. Until answered, this is the one B-role example that may not be buildable —
   keep it last and treat the placeholder outcome as acceptable.
