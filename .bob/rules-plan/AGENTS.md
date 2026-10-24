# AGENTS.md

This file provides guidance to agents when working with code in this repository.

# Z Architect / Plan Mode — Non-Obvious Architectural Constraints

## Two Completely Separate Customer Stores

CICS programs use the **DB2 CUSTOMER table** (declared in `src/base/cics/copy/CUSTDB2.cpy`). IMS programs use an **IMS hierarchical database** with a CUSTOMER segment (defined in `src/base/ims/DBD/`). These are not synchronised. A change to the DB2 CUSTOMER schema does not affect the IMS CUSTOMER segment and vice versa.

## PROCTRAN is Not a Shared Service — It's Copy-Paste

There is no shared `WRITE-PROCTRAN` utility program. Each CICS program that needs to audit has its own private `WRITE-PROCTRAN-DB2 SECTION`. Any new operation that must be audited requires its own copy of this section. This is an architectural debt — plan for it when adding new transaction types.

## Customer Number Serialisation is CICS-Coupled

Customer number generation relies on CICS Named Counter (ENQ/DEQ on resource `'BNKCUSTS'` + sortcode) backed by the DB2 CONTROL table. This mechanism is **not available in batch or IMS**. Any batch process that needs to create customers must use a different approach (direct CONTROL table update without ENQ/DEQ).

## DBB Build Auto-Detection of CICS/SQL

DBB auto-appends `CICS` and `SQL` to compile parms by detecting `EXEC CICS` and `EXEC SQL` in source. This means:
- A program that conditionally uses EXEC CICS will always get CICS compile parms if the text appears anywhere in expanded source.
- Copybooks containing `EXEC SQL DECLARE` (like `CUSTDB2.cpy`) will trigger SQL compile parms for any program that COPYs them, even if the program has no runtime SQL.

## IMS DL/I Programs Require ENTRY DLITCBL in Link-Edit

Without `ENTRY DLITCBL`, IMS cannot schedule the program. This is not automatically added — it must be listed explicitly in `dbb-app.yaml` `linkEditStream` for each IMS batch COBOL program. The list is explicit (not a glob): `IBACSUM`, `IBGCUDAT`, `IBLOGIN1`, `IBLOGOUT`, `IBSCUDAT`, all `LOAD*` programs.

## IBTRAN is a 31→64-bit JNI Bridge — Cannot Be Refactored Casually

`IBTRAN.cbl` must remain 31-bit (`LP(32)`) and produce a DLL-style load module. Changing compile parms, the link-edit stream, or the Java package/class structure will break the IMS-Java integration. The bridge uses `igzxjni2.x` (path in `config.yaml` under `global.igzxjni2`) which is fixed to COBOL for z/OS v6.5.

## Deployment Types Determine Target Library

`dbb-app.yaml` `deployType` controls which PDS the load module goes into. Mistakes here cause runtime `PGMNOTFOUND` errors:
- `CICSLOAD` → CICS program library (programs with `EXEC CICS`)
- `IMSLOAD` → IMS program library (`src/base/ims/cobol/`)
- `PSBLOAD` / `DBDLOAD` → IMS ACBGEN libraries (assembled, not linked)
- `LOAD` → standard batch load library (default)

## z/OS Connect Provider CPY Files are Generated — Plan Gradle Regeneration

`src/api/src/main/zosAssets/*.cpy` are generated from `openapi.yaml` by the `com.ibm.zosconnect.gradle` plugin. When planning API changes, include a Gradle regeneration step before any COBOL changes that reference the updated provider copybooks. The COBOL programs reference the provider CPYs via `COPY` statements.

## Frontend Has No Build Step

The frontend (`src/frontend/`) is static vanilla HTML/JS. The `VanillaFrontend` DBB Groovy task just packages it into a WAR. There is no transpilation, bundling, or npm step. Direct file edits are immediately deployable.
