### Use Case 8: COBOL Funds Transfer Program - Logic and Failure Path Analysis

**Description**: Conducts an in-depth code and architecture analysis of the 2,060-line CICS funds transfer program `XFRFUN.cbl`, uncovering deadlock prevention locking mechanisms and tracing partial failure rollback paths.

**Prompt**: Explain what this program does, including how it handles the case where the source account update succeeds but the target account update fails.

**Key Points (Why it matters)**:
- **Exhaustive failure path tracing**: Pinpoints exact `COMM-FAIL-CODE` values, explicit `SYNCPOINT ROLLBACK` invocations, and CICS abend codes (`TO  `, `WPCD`, `WPCT`) across all partial-failure scenarios.
- **Architectural pattern discovery**: Identifies the implicit deadlock prevention strategy (sorting account numbers and locking the lower account number first) without requiring explicit documentation.
- **Transactional integrity & audit trail analysis**: Traces how post-update `PROCTRAN` logging failures safely trigger full unit-of-work rollbacks across both debit and credit operations.
- **Operational risk awareness**: Highlights design assumptions such as the absence of overdraft checking and the storm-drain failure suppression logic for CPSM workload routing.

**Demo Setup**:
- **Mode**: `Z Architect`
- **Active File / Context**: `src/base/cics/cobol/XFRFUN.cbl`
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

### Partial Failure & Rollback Handling in `XFRFUN.cbl`

| Failure Scenario | `COMM-FAIL-CODE` | Rollback Mechanism | `PROCTRAN` Written? |
|---|---|---|---|
| Target (TO) account not found | `'2'` | Explicit `SYNCPOINT ROLLBACK` (lines 407, 1111) | ❌ No |
| Target (TO) account DB2 SQL error | `'3'` | CICS ABEND `'TO  '` $\rightarrow$ CICS UOW backout | ❌ No |
| Source (FROM) account not found | `'1'` | Explicit `SYNCPOINT ROLLBACK` (line 407) | ❌ No |
| Audit (`PROCTRAN`) insert fails | — | CICS ABEND `'WPCD'`/`'WPCT'` $\rightarrow$ Full UOW backout | ❌ Rolled back |
| Both account updates succeed | — | Transaction commits successfully | ✅ 2 rows written |

#### Key Architectural Findings:
1. **Deadlock Prevention**: Compares `COMM-FACCNO < COMM-TACCNO` to always acquire DB2 row locks in numeric order regardless of transfer direction.
2. **Deadlock Retry Loop**: Catches `SQLCODE -911` (`SQLERRD(3) = 13172872`) and retries up to 5 times with a 1-second delay before issuing `ABEND 'RUF2'`.
