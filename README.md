# Step 4 — Add Email to Customer Info (End-to-End Change Delivery)

> **About this branch:** This branch contains the complete set of code changes required to add an email address field to the customer record. These changes are the direct output of the end-to-end delivery demonstrated in steps 4a, 4b, and 4c — starting with an impact analysis that identified every affected component across the stack (4a), through a detailed implementation plan with exact COBOL edits and byte-position appendices (4b), to the actual code changes applied by Bob across 30+ files spanning COBOL copybooks, BMS maps, COBOL programs, z/OS Connect provider files, and the Web UI (4c). Step 4d shows how to build and deploy those changes into a live z/OS environment using DBB and Wazi Deploy.

> **⚠️ Demoing the scenario live?** If you want to demonstrate steps 4a–4c as a live interactive session — asking Bob to perform the impact analysis, generate the implementation plan, and make the code changes — you should start from the **base branch**, not this one. This branch already contains all the finished code changes, which means there is nothing left for Bob to do. Use this branch only to deploy and demo the running application (step 4d). The files [`bobz-demo/4a-add-email-impact-analysis.md`](bobz-demo/4a-add-email-impact-analysis.md), [`bobz-demo/4b-add-email-implementation-plan.md`](bobz-demo/4b-add-email-implementation-plan.md), and [`bobz-demo/4c-add-email-implementation-summary.md`](bobz-demo/4c-add-email-implementation-summary.md) contain pre-captured outputs from a prior session, provided as reference so you can see what Bob produced — and to set expectations, since the actual output may vary due to the non-deterministic nature of the LLMs Bob uses.


---

## Build, Deploy, and Demo

Pull the changes to the Z environment, run a complete DBB build, package the outputs, deploy via Wazi Deploy, and populate Db2 and IMS with test data.

```bash
.setup/setup-common.sh environment   # if needed
.setup/setup-common.sh install-bank-of-z
```

**DBB** knows which programs include `CUSTOMER.cpy` and recompiles them all. **Wazi Deploy** promotes load modules to CICS.

Open the **Bank-of-Z** Web UI in a browser:
- Navigate to **Create Customer** — the email field is now present in the form
- Create a new customer with an email address, submit
- Navigate to **Customer Details** for that customer — email is displayed and editable

![Customer Details](bobz-demo/4d-add-email-deployment.png)
