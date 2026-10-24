### Use Case 9: PL/I Batch Program Analysis and Targeted Code Change

**Description**: Analyzes the end-to-end business logic of a 912-line batch PL/I statement generation program (`BNKSTMT.pli`) and generates surgical code changes to add a transaction category column while respecting mainframe print layout constraints.

**Prompt**: Explain the business logic of this bank statement generator and add a new column to the output for transaction category. Show me the changes to be made only.

**Key Points (Why it matters)**:
- **First-class Enterprise PL/I support**: Fluently interprets complex PL/I constructs, SQL cursor management, null indicator structures, and `PUT EDIT` line formatting.
- **Single-turn explain + edit**: Combines deep program explanation with precise, ready-to-apply code changes in a single interaction.
- **Proactive line width constraint enforcement**: Identifies the 132-character line printer boundary (`REPORT_LINE CHAR(132)`) and trims transaction description length from 30 to 18 characters to prevent print line truncation.
- **Zero blast-radius certainty**: Confirms that no external copybooks, DB2 tables, cursors, or JCL streams require modification.

**Demo Setup**:
- **Mode**: `Z Architect` (or `Z Code`)
- **Active File / Context**: `src/base/batch/pli/BNKSTMT.pli`
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### 1. Business Logic Summary
Traces the execution of 11 internal procedures (`MAIN` $\rightarrow$ `GET_STATEMENT_PERIOD` $\rightarrow$ `PROCESS_ALL_ACCOUNTS` $\rightarrow$ `GENERATE_STATEMENT` $\rightarrow$ `PRINT_REPORT`), calculating opening balances via `AVAIL_BALANCE + TOTAL_DEBITS - TOTAL_CREDITS` and enforcing a 55-line page overflow mechanism.

#### 2. Targeted Code Edits (`src/base/batch/pli/BNKSTMT.pli`)
```pli
/* Edit 1: Variable declaration (after line 131) */
DCL TRAN_CATEGORY CHAR(12);

/* Edit 2: Header formatting (lines 658-667) */
REPORT_LINE = 'DATE       ID          CATEGORY     DESCRIPTION        AMOUNT       BALANCE';

/* Edit 3: Transaction categorization mapping (lines 763-773) */
SELECT(HV_TRAN_TYPE);
  WHEN('DEB') TRAN_CATEGORY = 'DEBIT';
  WHEN('CRE') TRAN_CATEGORY = 'CREDIT';
  WHEN('TFR') TRAN_CATEGORY = 'TRANSFER';
  OTHERWISE   TRAN_CATEGORY = 'OTHER';
END;
```
