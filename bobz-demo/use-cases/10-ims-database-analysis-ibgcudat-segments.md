### Use Case 10: IMS Database Analysis - IBGCUDAT Segments and CUSTOMER/CUSTACCS Relationship

**Description**: Synthesizes Assembler PSB/DBD descriptors and COBOL DL/I call logic across the IMS subsystem to explain segment access patterns, security permissions, and database cross-reference architectures.

**Prompt**: Show me all the IMS segments that IBGCUDAT accesses, and explain the parent-child relationship between CUSTOMER and CUSTACCS databases.

**Key Points (Why it matters)**:
- **Deep IMS DL/I fluency**: Correctly parses Assembler DBD/PSB files (`CUSTOMER.asm`, `CUSTACCS.asm`, `IBGCUDAT.asm`) and COBOL `CBLTDLI` calls (`GU`, `GHU`, `GN`, `ISRT`).
- **Disproves misleading naming assumptions**: Accurately determines that `CUSTOMER` and `CUSTACCS` have **no hierarchical parent-child relationship** (`PARENT=0`), functioning instead as independent HDAM databases with a multi-valued join key (`SEQ=M`).
- **PSB security boundary interpretation**: Interprets processing options (`PROCOPT=G`), verifying that `IBGCUDAT` is read-only and restricted to a single segment.
- **Subsystem-wide ecosystem visibility**: Automatically produces a full 9-database $\times$ 8-PSB access matrix across all IMS batch and online transactions.

**Demo Setup**:
- **Mode**: `Z Architect`
- **Active File / Context**: None (or `src/base/ims/PSB/IBGCUDAT.asm`)
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### 1. Segment Access for `IBGCUDAT`
- **PSB**: Single PCB referencing `CUSTOMER` database with `PROCOPT=G` (Get/Read only), key length 4 bytes.
- **DBD Alignment**: Accesses the 279-byte `CUSTOMER` root segment keyed on `CUSTID` via qualified SSA `CUSTOMER(CUSTID EQ <value>)`.

#### 2. Relationship Analysis: `CUSTOMER` vs `CUSTACCS`
- **Architecture**: Both databases are independent flat HDAM databases with `PARENT=0`.
- **Cross-Reference Pattern**: `CUSTACCS` acts as an IMS join table linking `CUSTID` (4 bytes) to `ACCID` (8 bytes) with multiple sequence entries (`SEQ=M`) per customer.
- **Design Rationale**: Separate HDAM databases allow account inquiry programs (`IBACSUM`, `IBTRAN`) to access `CUSTACCS` without traversing customer root segments.

#### 3. Subsystem PSB Access Matrix (Excerpt)
| PSB | CUSTOMER | CUSTACCS | ACCOUNT | HISTORY |
|---|---|---|---|---|
| `IBGCUDAT` | **G** (Read) | — | — | — |
| `IBSCUDAT` | **R** (Replace) | — | — | — |
| `IBACSUM` | — | **G** (Read) | **G** (Read) | — |
| `IBTRAN` | — | **G** (Read) | **R** (Replace) | **I** (Insert) |
