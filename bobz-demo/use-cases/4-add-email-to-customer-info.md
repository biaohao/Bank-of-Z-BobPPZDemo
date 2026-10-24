### Use Case 4: Add Email to Customer Info (End-to-End Change Delivery)

**Description**: Demonstrates the full lifecycle of an end-to-end application change across the entire mainframe and hybrid stack: performing impact analysis, generating an exact implementation plan, applying code edits across 30+ files, and building, deploying, and verifying the live change on z/OS.

#### Sub-Use Cases Navigation:
- [4a — Impact Analysis](#4a--impact-analysis)
- [4b — Implementation Plan](#4b--implementation-plan)
- [4c — Code Changes](#4c--code-changes)
- [4d — Build, Deploy, and Demo](#4d--build-deploy-and-demo)

---

### 4a — Impact Analysis

**Description**: Performs a comprehensive multi-layer blast-radius analysis for adding an optional `CUSTOMER_EMAIL CHAR(50)` field right after the phone number, identifying every affected file from DB2 DDL to the web UI.

**Prompt**: I want to add an email address field to the customer data model. The field should be optional, max 50 characters, stored in the CUSTOMER DB2 table as CUSTOMER_EMAIL CHAR(50). Also, place the email field right after phone number. Please do a full impact analysis: identify every file that needs to change, explain why, and flag any risks. Complete the end-to-end analysis including the frontend web UI. Ignore the IMS path for now.

**Key Points (Why it matters)**:
- **Cross-stack blast radius in one query**: Identifies all affected components across DB2 DDL, COBOL copybooks, COBOL business programs, BMS 3270 screen maps, z/OS Connect descriptor/mapping files, OpenAPI specs, and frontend HTML/JS.
- **Catches hidden traps upfront**: Detects inline manual COMMAREA structures in programs like `BNK1DCS.cbl` and `BNK1CCS.cbl` (which have no `COPY` statements) that would otherwise be missed and cause production outages.
- **First-class deployment sequencing**: Determines the strict build and deployment order (DDL first $\rightarrow$ copybooks $\rightarrow$ COBOL recompile $\rightarrow$ DBRM bind $\rightarrow$ z/OS Connect $\rightarrow$ Web UI).

**Demo Setup**:
- **Mode**: `Z Architect`
- **Active File / Context**: None
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### Scope of Identified Components (CICS Processing Path)

| Layer | Artifact / File | Change Required |
|---|---|---|
| **DB2 Schema** | `CUSTOMER` Table / `.setup/jcl/cics/Db2-create.j2` | Add `CUSTOMER_EMAIL CHAR(50)` column |
| **COBOL Copybooks** | `CUSTDB2.cpy`, `CUSTOMER.cpy`, `CRECUST.cpy`, `INQCUSTZ.cpy`, `UPDCUST.cpy` | Add `CUSTOMER_EMAIL` / `COMM-EMAIL` definitions |
| **COBOL Programs** | `CRECUST.cbl`, `INQCUST.cbl`, `UPDCUST.cbl`, `BANKDATA.cbl` | Add host variables, update SQL INSERT/SELECT/UPDATE clauses |
| **BMS Screen Maps** | `BNK1CCM.bms` (Create, row 21), `BNK1DCM.bms` (Display, row 20) | Add `EMAIL` / `CUSTEML` `DFHMDF` field definitions |
| **CICS Presentation** | `BNK1CCS.cbl`, `BNK1DCS.cbl` | Sync inline structs without COPY to pass email across LINKs |
| **z/OS Connect API** | `openapi.yaml`, Request/Response YAML mappings, Provider `.dai`/`.cpy` | Add `email` property and JSONata `$exists` null guards |
| **Frontend Web UI** | `customer-create.html`, `customer-details.html`, `api.js` | Add email form inputs, validation, and API bindings |

#### Critical Risk Matrix
- **COMMAREA Length & Alignment**: 50-byte expansion shifts subsequent field offsets; requires synchronized recompilation of all callers.
- **Inline Struct Desynchronization**: `BNK1DCS` and `BNK1CCS` use inline structures without `COPY` and require manual layout updates.

---

### 4b — Implementation Plan

**Description**: Produces a spec-driven, workstream-organized implementation plan specifying the exact before/after code edits for every file, byte offset calculations, and a comprehensive rollback matrix.

**Prompt**: Based on the impact analysis, create a detailed implementation plan for adding the CUSTOMER_EMAIL field. Organize it into workstreams. For each file change, show the exact edit — the before and after code. Also cover rollback. Do not make any changes to any files — produce the plan document only.

**Key Points (Why it matters)**:
- **Exact before/after code edits**: Provides ready-to-execute code diffs across COBOL host variables, SQL clauses, BMS field definitions, JSONata expressions, and HTML inputs.
- **Precise byte-position arithmetic**: Computes exact byte offsets and start positions for COMMAREA buffers and z/OS Connect `.dai` descriptors.
- **Structured rollback procedures**: Details rollback strategies for partial deployment failures (e.g., DDL deployed vs. programs bound vs. API deployed).

**Demo Setup**:
- **Mode**: `Z Architect`
- **Active File / Context**: `bobz-demo/sample-outputs/4a-add-email-impact-analysis.md` (or completed impact analysis report)
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### Workstream Structure & Excerpted Edits

- **Workstream A: DB2 DDL Migration**
  ```sql
  ALTER TABLE BANKZ.CUSTOMER ADD COLUMN CUSTOMER_EMAIL CHAR(50);
  ```
- **Workstream B: COBOL Copybooks (`CUSTDB2.cpy`, `CUSTOMER.cpy`, `CRECUST.cpy`, etc.)**
  ```cobol
  * Insert after CUSTOMER_PHONE / COMM-PHONE:
  05 CUSTOMER-EMAIL PIC X(50).
  ```
- **Workstream C: BMS 3270 Maps (`BNK1CCM.bms`, `BNK1DCM.bms`)**
  ```assembly
  EMAIL   DFHMDF POS=(21,18),LENGTH=50,ATTRB=(UNPROT,IC,NORM),COLOR=GREEN
  ```
- **Workstream D: COBOL Programs (`CRECUST.cbl`, `UPDCUST.cbl`, `INQCUST.cbl`, `BNK1DCS.cbl`)**
  - Add `03 HV-CUSTOMER-EMAIL PIC X(50).` to `HOST-CUSTOMER-ROW`.
  - Update `EXEC SQL INSERT/SELECT/UPDATE` statements and synchronize inline structs in `BNK1DCS.cbl`.
- **Workstream E: z/OS Connect Mapping YAMLs & Descriptors**
  - Map `COMM-EMAIL` from `$exists($body.email) ? $body.email : ""`.
- **Workstream F: Frontend Web UI Forms & Validation**
- **Workstream G: Deployment Sequence & Rollback Plan**

---

### 4c — Code Changes

**Description**: Executes the implementation plan end-to-end, making code modifications across 30+ files in all layers of the stack and resolving runtime and compile-time issues.

**Prompt**: Using the implementation plan, make all the code changes required to add the CUSTOMER_EMAIL field. Work through each workstream in order. For each file, show me what you changed and why. Flag any risks you encounter as you go — especially anything related to inline structs, COMMAREA byte alignment, or z/OS Connect descriptor files.

**Key Points (Why it matters)**:
- **AI that does the work, not just describes it**: Modifies DB2 DDL, COBOL logic, BMS screen definitions, API mappings, and Web UI forms without requiring manual developer calculations.
- **Honest defect recovery**: Identifies and fixes real-world mainframe issues (such as COBOL Area A/B alignment, unsynced inline structs causing CICS `LENGERR`, and COMMAREA byte drift).
- **Cross-stack consistency**: Guarantees naming and typing alignment across all 5 tiers of the application.

**Demo Setup**:
- **Mode**: `Z Code`
- **Active File / Context**: `bobz-demo/sample-outputs/4b-add-email-implementation-plan.md`
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### Summary of Executed Changes (15 Commits across 30+ Files)
- **Database & Copybooks**: Updated `CUSTDB2.cpy`, `CUSTOMER.cpy`, `CRECUST.cpy`, `INQCUSTZ.cpy`, `UPDCUST.cpy`, `DELCUS.cpy`.
- **BMS 3270 Maps**: Added `EMAIL` input field to `BNK1CCM.bms` and `CUSTEML` display field to `BNK1DCM.bms`.
- **COBOL Programs**: Updated host variables, SQL clauses, and inline linkage structures in `CRECUST.cbl`, `INQCUST.cbl`, `UPDCUST.cbl`, `BNK1CCS.cbl`, `BNK1DCS.cbl`, and `BANKDATA.cbl`.
- **z/OS Connect & OpenAPI**: Updated `openapi.yaml`, 4 operation mapping YAMLs, and synchronized all `.dai` startPos offsets across 4 provider asset sets.
- **Frontend UI**: Integrated email field into `customer-create.html`, `customer-details.html`, and `api.js`.

#### Self-Detected & Resolved Defects:
1. **Area A Indentation Error**: Corrected `END-EXEC.` misplaced in Area A in `CUSTDB2.cpy` which caused compile RC=8 across 4 programs.
2. **Inline Struct `LENGERR`**: Updated `WS-CHILD-DATA` in `CRECUST.cbl` (inline struct without COPY) to add `WS-CHILD-EMAIL`, resolving runtime CICS container length errors.
3. **Byte Offset Drift**: Realigned `COMM-EMAIL` position to follow phone number across all `.dai` files to prevent data corruption.

---

### 4d — Build, Deploy, and Demo

**Description**: Demonstrates pulling the changes to the live z/OS environment, executing DBB compilation, deploying via Wazi Deploy to CICS/Liberty, and verifying the new email field in the Bank of Z Web UI.

**Prompt**: Build and deploy the changes to the z/OS environment, and verify the customer creation and inquiry flows in the web application.

**Key Points (Why it matters)**:
- **True end-to-end delivery**: Delivers a live, working feature from initial prompt to z/OS production runtime in a single session.
- **z/OS-native enterprise tooling**: Uses IBM DBB for dependency-based compilation, Wazi Deploy for artifact deployment, and Liberty on z/OS for API/web serving.
- **Tangible business outcome**: Demonstrates real-time verification in the browser UI (creating a customer with email and viewing persistent details).

**Demo Setup**:
- **Mode**: `Z Code` / Terminal / Browser
- **Active File / Context**: None
- **Prerequisites / Tooling**: Access to z/OS environment or live Bank of Z Web UI (port 3001/9080).

**Sample Output**:

#### Live Verification Steps & Results
1. **DBB Rebuild on z/OS**:
   ```bash
   .setup/setup-common.sh install-bank-of-z
   ```
   *Outcome:* DBB detects copybook dependency updates and automatically recompiles all dependent CICS COBOL programs; Wazi Deploy updates CICS load libraries and Liberty server assets.
2. **Web UI Verification**:
   - **Customer Creation**: Navigated to `customer-create.html`, filled customer details with email `john.smith@bankofz.com`, and submitted $\rightarrow$ Received HTTP 201 with newly allocated customer ID.
   - **Customer Inquiry**: Opened `customer-details.html` for the new customer ID $\rightarrow$ Confirmed email address is retrieved from DB2, rendered in the form, and editable via update requests.
