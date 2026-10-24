### Use Case 3: Customer Audit Trail Analysis

**Description**: Leverages Z Understand project-wide indexing to locate all runtime SQL operations touching the DB2 CUSTOMER table across COBOL and PL/I programs, evaluate audit trail compliance against PROCTRAN logging, and identify genuine compliance gaps.

**Prompt**: Using the Z Understand project, find every COBOL program in Bank of Z that reads or writes the `CUSTOMER` DB2 table, and for each one tell me: which SQL operation it performs (SELECT, INSERT, UPDATE, DELETE), which paragraph contains that operation, and whether the program also writes to `PROCTRAN` as part of the same transaction. Then identify any program that reads from `CUSTOMER` but does not write an audit row to `PROCTRAN` — those are the gaps in the audit trail.

**Key Points (Why it matters)**:
- **Compliance audit in seconds, not days**: Replaces tedious manual code grepping and copybook dependency tracing across 30+ programs with an instantaneous, deterministic query.
- **Multi-language awareness**: Discovers SQL access in both COBOL and PL/I batch reporting programs (`BNKSTMT.pli`) through unified indexing.
- **Contextual risk assessment**: Distinguishes genuine audit compliance gaps (`UPDCUST`) from legitimate read-only programs (`INQCUST`, `BNKSTMT`) and test loaders (`BANKDATA`).
- **Clear remediation guidance**: Pinpoints the missing audit logging paragraph and provides exact implementation patterns based on compliant sibling programs (`CRECUST`, `DELCUS`).

**Demo Setup**:
- **Mode**: `Z Architect`
- **Active File / Context**: None
- **Prerequisites / Tooling**: Z Understand project indexed or local analysis database available.

**Sample Output**:

### CUSTOMER Table — Full Usage Inventory
Z Understand returned **11 runtime usages** across **6 programs** (5 COBOL + 1 PL/I):

| Program | SQL Operation | Paragraph | Line | Writes PROCTRAN? |
|---|---|---|---|---|
| `CRECUST.cbl` | INSERT | `WRITE-CUSTOMER-DB2` (line 1139) | 1219 | ✅ Yes — `WRITE-PROCTRAN-DB2` (line 1359) |
| `UPDCUST.cbl` | SELECT + UPDATE | `UPDATE-CUSTOMER-DB2` (line 222) | 265, 363 | ❌ **No — Genuine Audit Gap** |
| `DELCUS.cbl` | SELECT + DELETE | `DEL-CUST-DB2` (line 422) | 441, 642 | ✅ Yes — `WRITE-PROCTRAN-CUST-DB2` (line 774) |
| `INQCUST.cbl` | SELECT (x2) | `READ-CUSTOMER-DB2` (310) / `GET-LAST-CUSTOMER-DB2` (675) | 310, 675 | ❌ No — Read-only / Counter lookup |
| `BANKDATA.cbl` | INSERT + DELETE | `PREMIERE` (676) / `DELETE-DB2-ROWS` (1394) | 676, 1401 | ❌ No — Test Data Loader |
| `BNKSTMT.pli` | SELECT | `GET_CUSTOMER_INFO` | 435 | ❌ No — Batch Statement Reporting |

### Audit Trail Gap Assessment
1. **`UPDCUST.cbl` (⚠️ Genuine Audit Gap)**: The only online mutating transaction that modifies customer demographic records without writing a corresponding `PROCTRAN` audit row. Remediation: Implement a `WRITE-PROCTRAN-CUST-DB2` paragraph after a successful update.
2. **`INQCUST.cbl` & `BNKSTMT.pli` (✅ Compliant Read-Only)**: Inquiries and batch reporting jobs do not alter customer state; omitting audit writes avoids audit log bloat.
3. **`BANKDATA.cbl` (⚠️ Production Risk Flag)**: Test initialization utility that performs bulk inserts/deletes without audit logs; flagged as a risk if executed in production.
