# AGENTS.md

This file provides guidance to agents when working with code in this repository.

# Z Code Mode — Non-Obvious Coding Rules

## COBOL Code Style Conventions (discovered from source, not enforced by linter)

- Every CICS program declares `COPY ABNDINFO.` in LOCAL-STORAGE — this is mandatory for the shared abend-handler pattern; do not omit it.
- Sort code is always obtained via `COPY SORTCODE.` (defines `77 SORTCODE PIC 9(6) VALUE 987654`). Never hardcode `987654` directly.
- DB2 host variable structs are always declared with the prefix `HV-` (e.g. `HV-CUSTOMER-SORTCODE`). Group them under a single `01 HOST-xxx-ROW.` structure.
- `CUSTOMER_DATE_OF_BIRTH`, `CUSTOMER_CREATED_DATE`, `CUSTOMER_CS_REVIEW_DATE` are `INTEGER` (YYYYMMDD), not SQL `DATE`. Always compute with: `COMPUTE HV-x = (YYYY * 10000) + (MM * 100) + DD`.
- CICS programs use `EXEC CICS RETURN` inside a dedicated `GET-ME-OUT-OF-HERE SECTION` — never code `STOP RUN` or `GOBACK` in CICS programs.
- Sections follow the pattern `SECTION-NAME SECTION. / Sxxx010. / ... / Sxxx999. EXIT.` — every section ends with a `xxx999. EXIT.` exit paragraph.
- ZCodeScan enforces `END-IF`, `END-EVALUATE`, `END-READ`, `END-SEARCH`, `END-CALL` — all scope terminators are **required** (configured in `zcodescan/zcodescan-rules.yaml`).
- Inline PERFORM body limit is **30 lines** (ZCodeScan `InlinePerformLineLimitRule`); PROCEDUREs (paragraphs) are limited to **100 lines**.
- Maximum IF nesting depth is **6** (ZCodeScan `NestedIfLimitRule`).
- `SELECT *` in DB2 is forbidden by ZCodeScan (`SqlAvoidSelectStarRule` severity HIGH) — always name every column.
- `DELETE` and `UPDATE` without a `WHERE` clause will fail ZCodeScan (`SqlWhereRule`). `BANKDATA.cbl` is the only deliberate exception (full-table wipe utility).
- Condition name (level-88) prefix must be `TEST` (configured in `ConditionNamePrefixRule`) — but existing programs use their own conventions; match the file being edited.

## PROCTRAN Write — Mandatory Pattern for New Mutating Operations

After any successful INSERT/UPDATE/DELETE on CUSTOMER or ACCOUNT (CICS path), write a PROCTRAN row. Copy the pattern from `CRECUST.cbl` `WRITE-PROCTRAN-DB2 SECTION` or `DELCUS.cbl` `WRITE-PROCTRAN-CUST-DB2 SECTION`. Valid `PROCTRAN_TYPE` codes are defined as 88-levels in `src/base/cics/copy/PROCTRAN.cpy`.

## Customer / Account Number Generation

- Customer numbers: use CICS ENQ/DEQ Named Counter (`'BNKCUSTS'` + sortcode). See `CRECUST.cbl` `ENQ-NAMED-COUNTER` / `UPD-NCS` sections. Never INSERT a CUSTOMER row with a hardcoded number.
- Account numbers: same pattern in `CREACC.cbl`.

## DBB compileParms Auto-Append Rules

`dbb-app.yaml` appends `CICS` to `compileParms` when `${IS_CICS}` is true, and `SQL` when `${IS_SQL}` is true. Do NOT manually specify these in source `CBL` or `PROCESS` statements — DBB injects them. The only exception is `IBTRAN.cbl`, which has a fully overridden `compileParms` in `dbb-app.yaml`.

## IMS DL/I vs DB2 SQL — Path Distinction

IMS-path programs (`src/base/ims/cobol/`) use `CALL 'CBLTDLI'` with DL/I function codes (`GU`, `GHU`, `GN`, `ISRT`, `REPL`, `DLET`). They do **not** use `EXEC SQL`. The only IMS programs that use DB2 are those with DB2 plans listed in `.setup/config/config.yaml` under `ims.db2_plan_*`.

## IMS Java JMP (src/base/ims/java/)

Java 21, Gradle. JAR output name: `nazare-ims-jmp`. Do NOT change `archiveBaseName` or the entry class structure in `nazare.jmp.controller` — IBTRAN.cbl locates the JNI class by fixed name. Build locally: `cd src/base/ims/java && ./gradlew build` — but the actual z/OS JAR is built via the `ImsJavaBuilder` DBB task.

## z/OS Connect API

The `src/api/src/main/zosAssets/` `.cpy` files are **auto-generated** from `openapi.yaml` by the Gradle `com.ibm.zosconnect.gradle` plugin (v1.5.1). Any hand-edit will be overwritten. To add or change an API operation, edit `openapi.yaml` only, then regenerate.
