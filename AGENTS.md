# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Commands (Non-Standard)

There is no local COBOL/PL/I compiler or test runner. All compilation occurs on z/OS via IBM DBB (zBuilder):

- **Single Program Build & Unit Test (TAZ)**:
  Run DBB with `lifecycle: user` and pass the target source file.
- **Incremental Build**:
  `bash .setup/pipeline-local.sh` (Zowe CLI) or trigger VS Code task `Run Pipeline Simulation`.
- **Run Single Integration Test**:
  `BASE_URL="http://<host>:9080/api" bash tests/test_get_customer_cics.sh` (or `test_get_customer_ims.sh`, `test_frontend.sh`).
- **Run All Tests**:
  `BASE_URL="http://<host>:9080/api" FRONTEND_URL="http://<host>:3001" bash tests/run-all.sh` (set `IMS_DISABLED=true` to skip IMS).
- **Static Analysis (ZCodeScan)**:
  Rules configured in `zcodescan/zcodescan-rules.yaml` (Max RC=4; Warnings allowed, Errors fail; source encoding `IBM-1047`).
- **API Provider Asset Regeneration**:
  `cd src/api && ./gradlew build` (regenerates `src/api/src/main/zosAssets/` from `src/api/src/main/api/openapi.yaml`).

## Code Style & Architectural Conventions

- **File Encoding**: COBOL (`.cbl`), Copybooks (`.cpy`), BMS (`.bms`), PL/I (`.pli`), JCL (`.jcl`), and ASM (`.asm`) are strictly **IBM-1047 (EBCDIC)** on z/OS.
- **CICS Program Structure**:
  - Always `COPY ABNDINFO.` in `LOCAL-STORAGE SECTION` for standard abend handling.
  - Obtain sort code via `COPY SORTCODE.` (`77 SORTCODE PIC 9(6) VALUE 987654`); never hardcode sort codes.
  - End CICS tasks via `EXEC CICS RETURN` in `GET-ME-OUT-OF-HERE SECTION`; never use `STOP RUN` or `GOBACK`.
  - Sections must follow pattern `SECTION-NAME SECTION.` $\rightarrow$ `Sxxx010.` $\rightarrow$ `Sxxx999. EXIT.`.
- **DB2 Host Variables & Types**:
  - Group host variables under `01 HOST-xxx-ROW.` with prefix `HV-` (e.g. `HV-CUSTOMER-PHONE`).
  - Dates (`DATE_OF_BIRTH`, `CREATED_DATE`, `CS_REVIEW_DATE`) are stored as **INTEGER** (`YYYYMMDD`), not SQL `DATE`. Compute using: `COMPUTE HV-x = (YYYY * 10000) + (MM * 100) + DD`.
  - Sequence control table is schema-qualified `STTESTER.CONTROL`; key is `'BANKZCUST' || SORTCODE || '  '` (padded to 17 chars).
  - Explicit column lists required; `SELECT *` is forbidden by ZCodeScan.
- **PROCTRAN Audit Requirement**:
  - Every mutating CICS transaction on CUSTOMER/ACCOUNT must write a row to `PROCTRAN` DB2 table (`CRECUST.cbl` and `DELCUS.cbl` are canonical references).
  - Exceptions: `UPDCUST.cbl`, `IBSCUDAT.cbl`, and `BANKDATA.cbl` (data load utility).
- **Number Sequencing (CICS Path)**:
  - Generate sequential numbers using CICS Named Counter (`ENQ/DEQ` on `'BNKCUSTS'` + sortcode). Never insert with hardcoded IDs.
- **Dual Customer Stores**:
  - CICS path (Customer IDs starting with `C`) $\rightarrow$ DB2 `CUSTOMER` table.
  - IMS path (Customer IDs starting with `I`) $\rightarrow$ IMS DL/I hierarchical database (`CUSTOMER.DB`). These stores are completely separate.
- **Build / Link-Edit Stream Gotchas**:
  - IMS batch COBOL programs (`IBACSUM`, `IBGCUDAT`, `IBLOGIN1`, `IBLOGOUT`, `IBSCUDAT`, `LOAD*`) require `ENTRY DLITCBL` in `dbb-app.yaml`.
  - `IBTRAN.cbl` is a 31-to-64-bit JNI bridge to `nazare-ims-jmp.jar`; compile parms `LP(32),JAVAIOP(JAVA64),DLL,RENT,PGMNAME(LONGMIXED)` and entry point `ENTRY IBTRAN(R)` must not be altered.
  - Auto-generated provider files in `src/api/src/main/zosAssets/` must never be edited manually.
