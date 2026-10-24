### Use Case 5: CICS COBOL Refactor - CRECUST Async Credit Check

**Description**: Demonstrates automated architectural refactoring of a complex COBOL program by extracting an asynchronous multi-agency CICS credit verification routine into a separate service program (`CRCCVERI.cbl`) with a clean COMMAREA interface (`CRCCVERI.cpy`).

**Prompt**: CRECUST is calling 5 credit agency programs with a fixed 3-second wait. I want to refactor the code to isolate the credit check logic into a separate program so the implementation complexity is not in the CRECUST code, and I can easily switch to an API call in the future. Show me the changes to be made only.

**Key Points (Why it matters)**:
- **Separation of concerns**: Removes ~530 lines of asynchronous CICS plumbing (channels, containers, child task tokens, and NOSUSPEND polling loops) from `CRECUST.cbl`, reducing the call site to ~30 lines.
- **Future-proof API modernization**: Encapsulates credit check logic behind a stable COMMAREA contract (`CRCCVERI.cpy`), allowing the backend to be swapped with an external REST API without touching `CRECUST.cbl`.
- **Precise variable scoping**: Retains boundary-crossing state variables (`WS-CREDIT-CHECK-ERROR`) in the caller while migrating 18 exclusively used working-storage fields to the extracted program.
- **CICS runtime correctness**: Accurately applies `GOBACK` instead of `EXEC CICS RETURN` in the LINKed subprogram to ensure proper control transfer back to the caller.

**Demo Setup**:
- **Mode**: `Z Code`
- **Active File / Context**: `src/base/cics/cobol/CRECUST.cbl`
- **Prerequisites / Tooling**: Local workspace repository.

**Sample Output**:

#### 1. New Copybook: `src/base/cics/copy/CRCCVERI.cpy`
```cobol
       01 CCVERI-COMMAREA.
          03 CCVERI-INPUT.
             05 CCVERI-SORTCODE        PIC 9(6) DISPLAY.
             05 CCVERI-FIRST-NAME      PIC X(50).
             05 CCVERI-LAST-NAME       PIC X(50).
             05 CCVERI-DOB             PIC 9(8) DISPLAY.
          03 CCVERI-OUTPUT.
             05 CCVERI-CREDIT-SCORE    PIC 999.
             05 CCVERI-CS-REVIEW-DATE  PIC 9(8) DISPLAY.
             05 CCVERI-SUCCESS         PIC X.
             05 CCVERI-FAIL-CODE       PIC X.
```

#### 2. Modified `CRECUST.cbl` Call Site
```cobol
       CREDIT-CHECK SECTION.
       CC010.
            MOVE SORTCODE          TO CCVERI-SORTCODE OF WS-CRCCVERI-AREA
            MOVE COMM-FIRST-NAME   TO CCVERI-FIRST-NAME OF WS-CRCCVERI-AREA
            MOVE COMM-LAST-NAME    TO CCVERI-LAST-NAME OF WS-CRCCVERI-AREA
            ...
            EXEC CICS LINK
                 PROGRAM('CRCCVERI')
                 COMMAREA(WS-CRCCVERI-AREA)
                 LENGTH(LENGTH OF WS-CRCCVERI-AREA)
                 RESP(WS-CICS-RESP)
                 RESP2(WS-CICS-RESP2)
            END-EXEC
            ...
       CC999.
            EXIT.
```
