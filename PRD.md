# Bulk Change: Product Requirements

An independent concept for bulk employee changes in an HR and payroll platform with a unified employee record. See the [README](README.md) for the demo flow and [`index.html`](index.html) for the prototype.

> Independent concept. Not affiliated with or endorsed by any company. All names and data are fictional.

## 1. Problem

In an HR platform with one system of record for employee data (an "employee graph"), a change to one employee's record flows automatically to payroll, benefits, device and app access, and third-party tools such as Slack and GitHub.

Admins still change records **one at a time**: search for an employee, edit, save, repeat. That breaks down when dozens or thousands of employees change at once. The need is a **bulk change** capability: changing attributes for many employees in one action.

**Mission:** let admins restructure their entire organization in one action, with every connected system staying in sync automatically.

**Why it matters on this platform:** no standalone tool can show what a change will affect across payroll, benefits, IT and third-party apps before it is committed. Because every system reads from one employee graph, bulk change can preview and propagate those effects in one place. The design is built around that differentiator (screen 4, the impact preview).

## 2. Use cases

| Use case | What changes | Shape of the change |
|---|---|---|
| Re-org | Department, manager, team | Usually one value for everyone (uniform) |
| Performance cycle | Compensation, title, level, bonus multiplier | **A different value per employee** (the hardest case) |
| Office opening, closing or consolidation | Work location, and the tax and benefits rules that follow it | Uniform |
| Cohort onboarding (new joiners, interns) | Device provisioning, app access, SSO | Uniform per cohort |

**Design principle:** design for the performance cycle first. If the system handles a different value for every employee, with paychecks at stake, the uniform cases become easy extensions.

## 3. User segments

| # | Segment | What they do with bulk change | Why one-at-a-time fails them |
|---|---|---|---|
| 1 | HR admins / HRBPs | Carry out re-orgs on behalf of leadership | Dozens of manual updates under deadline pressure, often before a company-wide announcement |
| 2 | **Compensation and performance admins** | Apply comp, title and level changes after reviews, with different values per employee | Hundreds of entries that each differ, where an error means a wrong paycheck |
| 3 | Office / facilities admins | Reassign employees when offices open, close or consolidate | Repetitive identical edits across many records |
| 4 | IT admins | Set up devices, app permissions and SSO for cohorts | Cohorts join together, but access is granted one person at a time. Physical badge access is out of scope because it usually lives outside the platform |
| 5 | Payroll / finance admins | Review and approve bulk comp changes before a payroll run | They need to see timing and accuracy before money moves |
| 6 | Benefits / compliance admins | Handle eligibility effects of location or department changes | A bulk change can quietly affect ACA, COBRA and HSA eligibility |

### Prioritized segment: compensation and performance admins
Segments are prioritized on two factors: **size of the user group** and **how unmet their need is**.
- **Size:** every company runs performance cycles, every year.
- **Unmet need:** the hardest version of the problem, since every employee gets a different value.
- **Stakes:** errors hit paychecks, benefits eligibility and tax withholding.
- **Strategic fit:** solving this makes the other use cases simple extensions.

## 4. Pain points (compensation and performance admins)

The real workflow: finance sends a spreadsheet, the admin finds each employee, edits, saves, repeats about 200 times, and hopes nothing went wrong until payroll runs.

| Stage | What breaks |
|---|---|
| **Input** | Mismatches in the source spreadsheet map the wrong value to the wrong employee before anything reaches the system. Data arrives in different formats (Excel, CSV, PDF, email, Slack) with no standard way in. |
| **Execution** | Manual entry at scale invites serious mistakes: a misplaced decimal turns $200,000 into $20,000, or a 2.0 multiplier into 0.2. Similar titles (Senior PM vs. Senior Manager, Product) get mixed up across 200 records. |
| **Verification** | No review step before commit, so the admin publishes blind. Errors surface only when employees see results or paychecks. |
| **Downstream impact** | The admin can't see which systems a change will affect. A comp change can trigger benefits recalculations, payroll corrections or ACA filing updates. |

## 5. Solution

Bulk change replaces *searching for one employee* with *selecting a cohort*. The admin thinks about the group, not individuals.

### Alternatives considered
| Approach | Verdict |
|---|---|
| Spreadsheet-style editor | Considered |
| Rule-based engine | Considered |
| **Guided wizard with mixed input (uniform values or one per employee)** | **Recommended** |
| Conversational input (natural language) | Roadmap, not v1 |

An earlier comparison of a simple bulk editor and CSV import was rejected because the options weren't distinct: CSV import already sits inside the wizard, and a plain bulk editor fails for performance cycles. The wizard is the only approach that handles all use cases and builds guardrails (validation, impact preview, scheduling) into the flow.

### Flow
| # | Screen | What it does |
|---|---|---|
| 0 | Entry (zero state) | Templates, drafts and recent bulk changes; where the feature lives |
| 1 | Select employees | Filters (department, manager, location) with a live count; CSV import for exact lists; lock badges for records another admin is editing |
| 2 | Select attributes | 300+ attributes via quick-start templates (additive, not replacing) and category accordions |
| 3 | Enter values | One row per employee; rule-based checks give hard errors (red), anomaly flags give soft warnings (amber) |
| 4 | Impact preview | Effects on first-party products and connected third-party apps before anything is committed |
| 5 | Confirm and schedule | Commit now or on a set date; results screen for full success or partial failure |

## 6. Edge cases handled
- Partial failure, with a results screen showing success and failure states
- Downstream integration failures, with a retry for each system
- Two admins editing at the same time, shown with lock badges
- Changes to thousands of employees, with pagination and count badges

## 7. AI (phase 2)
- AI-powered import that maps spreadsheet columns to platform fields and shows a confidence score per match
- AI anomaly detection, kept separate from rule-based validation (for example, a valid value that would mean a 40% pay cut)
- AI compliance insights on the impact preview, flagging ACA and HSA implications

## 8. Key product decisions
- **Guided wizard** chosen over spreadsheet-style, rule-engine and conversational approaches.
- **No AI in the v1 core:** compensation changes can't be taken back once payroll runs, so v1 stays predictable and easy to review.
- **Prevention first:** review steps before commit (preview, flags, scheduling) do the work an undo button would otherwise overpromise.
