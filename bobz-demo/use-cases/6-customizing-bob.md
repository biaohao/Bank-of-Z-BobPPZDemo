### Use Case 6: Customizing Bob (Modes, Skills, Rules, and ZCodeScan Rules)

**Description**: Demonstrates how teams can extend and customize IBM Bob by inspecting and authoring version-controlled custom Modes (`custom_modes.yaml`, e.g. the Z Demo mode), reusable expert Skills (`SKILL.md`), standing Rules (`.bob/rules/*.md`, e.g. write a summary of every interaction to `bobz-demo/logs/`), and static analysis rules (`zcodescan-rules.yaml`).

**Prompt**: What modes are available to me in this project, and what is each one designed for? Also summarize the skills, the Bob rules under `.bob/rules/`, and the custom ZCodeScan rules currently configured.

**Key Points (Why it matters)**:
- **Not a black box**: Shows that Bob is fully extensible through standard YAML and Markdown configuration files checked into the repository alongside application code.
- **Preservation of institutional expertise**: Encodes senior developer guidelines, incident runbooks, and domain review checklists into reusable skills and custom modes so expertise remains across the organization.
- **Standing rules, followed without being asked**: A one-sentence Markdown rule (e.g. "write a summary of every interaction into `bobz-demo/logs/`") gives an automatic audit trail of AI-assisted work, with no mention in the prompt.
- **Enterprise governance & auditability**: Verifies that AI behavior, permissions, allowed file patterns, and coding guardrails are transparent, inspectable, and auditable.
- **Self-documenting and discoverable**: Demonstrates how agents can self-introspect their own capabilities and available workflows on demand.

**Demo Setup**:
- **Mode**: `Ask` (or `Z Demo` / `Agent`)
- **Active File / Context**: `.bob/custom_modes.yaml`, `.bob/rules/my-bobz-rules.md`, or `zcodescan/zcodescan-rules.yaml`
- **Prerequisites / Tooling**: `.bob/` configuration directory present in repository root.

**Sample Output**:

### Available Custom Modes & Extensibility Overview

#### 1. Custom Modes (`.bob/custom_modes.yaml`)
- **`z-code`**: Specialized for mainframe programming (COBOL/PL/I), compiler options, and runtime rule analysis with code editing capabilities.
- **`z-architect`**: Optimized for multi-program dependency mapping, data flow analysis, and cross-layer architecture review.
- **`bobz-demo` (Z Demo)**: Turns demo requests into a Demo Card and a client-facing Agenda with prioritized use cases. Not for coding: can only edit `.md` files under `bobz-demo/demos`, `use-cases`, and `logs`.

#### 2. Reusable Skills (`.bob/` and extensions)
- **`implementation-planning`**: Generates workstream-structured implementation plans with before/after diffs and rollback strategies.
- **`impact-analysis`**: Traces multi-layer blast radius across DB2, copybooks, CICS/IMS, APIs, and UI.
- **`data-dictionary-management`**: Extracts variable definitions and business semantic mappings into `bobz/DD.json`.

#### 3. Rules (`.bob/rules/`)
- **`my-bobz-rules.md`**: Two standing rules: (1) be very concise in wording; (2) write a summary of every interaction into `bobz-demo/logs/`, in a file named with a timestamp followed by a concise description (e.g. `2026-01-15_update-readme.md`).

#### 4. Enterprise Rule Customization (`zcodescan/zcodescan-rules.yaml`)
- Demonstrates version-controlled static analysis enforcing CICS response checking (`RespOptionRule`), SQL formatting (`CheckSqlcodeAfterExecSqlRule`), and structural constraints (`ProcedureRule`, `NestedIfLimitRule`).
