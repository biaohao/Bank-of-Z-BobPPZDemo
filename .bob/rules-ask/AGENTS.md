# AGENTS.md

This file provides guidance to agents when working with code in this repository.

# Z Architect / Ask Mode — Non-Obvious Context

## Routing Logic (Critical)

Customer IDs starting with `C` → CICS path. Customer IDs starting with `I` → IMS TM path. This split happens at the REST layer (`openapi.yaml` path prefixes `/cics/...` vs `/ims/...`), not in COBOL. There is no shared COBOL entry point that routes — the two paths are entirely separate program stacks.

## DB2 Table Gotchas

- `CUSTOMER_DATE_OF_BIRTH`, `CUSTOMER_CREATED_DATE`, `CUSTOMER_CS_REVIEW_DATE` are `INTEGER` columns (format YYYYMMDD) — **not** SQL `DATE` type. Queries and joins on these columns cannot use date arithmetic directly.
- The CONTROL table's full qualified name is `STTESTER.CONTROL` (schema-qualified), unlike CUSTOMER/ACCOUNT/PROCTRAN which are unqualified. Sequence numbers for customer and account are stored here, keyed by name+sortcode.

## Audit Trail Architecture

All CICS business transactions write to PROCTRAN (DB2) immediately after the main DML. The PROCTRAN row is **not** written by a shared service — each program has its own `WRITE-PROCTRAN-DB2` section. This means adding a new transaction requires adding a PROCTRAN write in every new program individually.

**Known audit gaps**: `UPDCUST.cbl` (customer update, deliberate) and `IBSCUDAT.cbl` (IMS-path customer update, no PROCTRAN integration at all).

## IMS-Path Architecture

IMS programs use IMS DL/I (`CALL 'CBLTDLI'`) for CUSTOMER data, not DB2. The CUSTOMER table in IMS is an IMS hierarchical database (DBD: `src/base/ims/DBD/`), not the same DB2 CUSTOMER table used by CICS programs. The two customer stores are **separate**.

`IBTRAN.cbl` is the only program that bridges IMS and Java. It runs as a 31-bit COBOL program and calls a 64-bit JVM via JNI (`igzxjni2.x`). The Java side (`nazare-ims-jmp.jar`) queries DB2 for IMS transaction history.

## Frontend

The frontend is vanilla HTML/JS (no framework, no npm). `src/frontend/config.js` detects port to choose API base URL — port 3001 = Docker dev (nginx proxy), anything else = direct z/OS Connect at port 9080. The frontend is packaged into a WAR (`bank-frontend-vanilla.war`) by the `VanillaFrontend` DBB Groovy task.

## Documentation Layout Gotcha

`docs/` is a Jekyll site published to GitHub Pages. The documentation describes the architecture but may lag behind source code. Always trust source code over docs for implementation details.

## Key Config Values (for architecture diagrams / documentation)

| Item | Value |
|---|---|
| DB2 SSID | `DBD1` |
| DB2 plan (CICS) | `BANKZPLN` |
| CICS region | `CICSTS63` / transaction prefix `OMEN` |
| IMS subsystem | `IMSV15` / datastore `IMS2` |
| Sort code (default) | `987654` |
| z/OS Connect port | `9080` (HTTP), `9443` (HTTPS) |
| IMS Connect port | `9977` |
| Application HLQ | `BANKZ` |
