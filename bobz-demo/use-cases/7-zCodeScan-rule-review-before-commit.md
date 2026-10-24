### Use Case 7: ZCodeScan - Custom Rule Enforcement Before Commit

**Description**: Conducts an automated static analysis review of a COBOL program against custom enterprise coding rules defined in `zcodescan/zcodescan-rules.yaml`, classifying violations by severity with minimal fix guidance.

**Prompt**: Review XFRFUN.cbl against the Bank of Z ZCodeScan rule set defined in zcodescan/zcodescan-rules.yaml. Identify every violation, state the rule ID and severity, explain what the violation is and where it occurs (paragraph name and approximate line), and suggest the minimal fix for each. Focus especially on: CICS HANDLE CONDITION usage, SELECT * in embedded SQL, 88-level names not prefixed TEST, missing END-IF or END-EVALUATE, inline PERFORM bodies over 30 lines, and SQLCODE checks after every EXEC SQL.

**Key Points (Why it matters)**:
- **Custom enterprise rules enforcement**: Validates code against project-specific house styles and standards defined in source-controlled YAML rather than generic internet advice.
- **Pre-commit quality gate**: Intercepts unhandled CICS response codes and unstructured jumps before compilation or deployment, preventing production abends.
- **Actionable & prioritized recommendations**: Categorizes findings by severity (HIGH / MEDIUM) and provides surgical, minimal code corrections instead of recommending massive rewrites.
- **Auditable standards**: Demonstrates to compliance and architecture teams that quality and safety rules are automated, consistent, and version-controlled.

**Demo Setup**:
- **Mode**: `Z Code`
- **Active File / Context**: `src/base/cics/cobol/XFRFUN.cbl`
- **Prerequisites / Tooling**: `zcodescan/zcodescan-rules.yaml` in workspace.

**Sample Output**:

### ZCodeScan Violation Report — XFRFUN.cbl (13 Violations Found)

| # | Priority | Rule ID | Severity | Section / Paragraph | Location / Issue |
|---|---|---|---|---|---|
| 1 | 🔴 HIGH | `CheckSqlcodeAfterExecSqlRule` / `RespOptionRule` | **HIGH** | `WRITE-TO-PROCTRAN-DB2` | `ASKTIME` and `FORMATTIME` lack `RESP` checks; CICS LINK calls silently ignore errors. |
| 2 | 🟠 MED | `CicsNoHandleRule` | **MEDIUM** | `PREMIERE` / `A010` (line 272) | `EXEC CICS HANDLE ABEND` creates non-structured flow; replace with explicit `RESP` checks. |
| 3 | 🟠 MED | `GotoRule` / `GotoParagraphRule` | **MEDIUM** | `UPDATE-ACCOUNT-DB2-TO` | Inter-section `GO TO UPDATE-ACCOUNT-DB2` (lines 1282, 1479) for deadlock retries; replace with structured `PERFORM VARYING`. |
| 4 | 🟠 MED | `ProcedureRule` | **MEDIUM** | `UPDATE-ACCOUNT-DB2` | Exceeds 100-line body limit (~606 lines); extract repeated abend-handling logic. |
| 5 | 🟠 MED | `NestedIfLimitRule` | **MEDIUM** | `UPDATE-ACCOUNT-DB2-TO` | Deadlock retry logic nested 8 levels deep (limit is 6). |
| 6 | 🟡 MED | `DisplayUponConsoleRule` | **MEDIUM** | Multiple (20+ sites) | Unsuppressed `DISPLAY` statements across sections; remove or gate behind debug flag. |
| 7 | 🟡 MED | `StopRunRule` | **MEDIUM** | `GET-ME-OUT-OF-HERE` (line 1876) | Unreachable `GOBACK` following `EXEC CICS RETURN`. |
| 8 | 🟡 MED | `EvaluateWhenOtherRule` | **MEDIUM** | `ABEND-HANDLING` (line 1931) | `EVALUATE MY-ABEND-CODE` missing `WHEN OTHER` branch. |

Each violation provides the exact code snippet, violation rationale, and minimal fix to achieve clean static analysis compliance.
