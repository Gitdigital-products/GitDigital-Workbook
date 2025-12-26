# GitDigital-Workbook
GitDigital Products Workbook repo
Below is a “living blueprint” for a single GitHub Organization workbook (a mono-repo called `GitDigital-Workbook`) that turns GitHub itself into your operating-system.  
Every file, label, issue template, project view and automation is explained twice:  
1. WHAT it is (the mechanic)  
2. WHY it exists (the management outcome you care about)

You can copy-paste the skeleton today and evolve it as the company grows.

--------------------------------------------------
1. REPO STRUCTURE (one repo to rule them all)
--------------------------------------------------
```
GitDigital-Workbook/
├─ README.md                 ← 30-second “how to use this workbook”
├─ docs/
│  ├─ 00-index.md            ← table of contents for humans
│  ├─ 01-ops-model.md        ← the operating model you are reading now
│  ├─ 02-glossary.md         ← define “Initiative”, “OKR”, “Bet”, etc.
│  └─ 03-change-log.md       ← log of every policy change (date, who, why)
├─ .github/
│  ├─ ISSUE_TEMPLATE/
│  │  ├─ initiative.yml      ← big multi-week body of work
│  │  ├─ task.yml            ← ≤3-day piece of work
│  │  ├─ bug.yml             ← defect report
│  │  ├─ request.yml         ← customer/internal request
│  │  └─ retro.yml           ← sprint/phase retrospective
│  ├─ workflows/
│  │  ├─ issue-opened.yml    ← auto-label, assign, link parent
│  │  ├─ pr-merged.yml       ← auto-close or move task
│  │  ├─ stale.yml           ← nudge inactive issues
│  │  └─ metrics.yml         ← push issue data to Google Sheet nightly
│  └─ labels.yml             ← single source of truth for all labels
├─ data/
│  ├─ okr/
│  │  ├─ 2025-Q1.json        ← machine-readable OKR sheet
│  │  └─ 2025-Q1.md          ← human-readable OKR sheet
│  ├─ kpi/
│  │  └─ weekly.csv          ← exported KPI csv (updated by workflow)
│  └─ risks.md               ← risk register (link from initiatives)
├─ templates/
│  ├─ initiative.md          ← blank markdown for new initiative
│  ├─ task.md                ← blank markdown for new task
│  └─ retro.md               ← blank retro canvas
└─ scripts/
   ├─ new-initiative.sh      ← CLI to scaffold an initiative
   ├─ okr-check.py            ← validate OKR syntax & progress
   └─ gsheet-sync.py          ← push GitHub data to Google Sheet
```

--------------------------------------------------
2. ISSUE TYPES & LIFE-CYCLE
--------------------------------------------------
We use ONLY FOUR native GitHub issue types. Everything else is a label or project field.

| Issue type  | Opened by | Closed by | Purpose |
|-------------|-----------|-----------|---------|
| initiative  | PM/Lead   | PM        | A bet that moves an Objective ≥1 quarter |
| task        | Anyone    | Assignee  | ≤3 days, moves 1 Key-Result |
| bug         | Anyone    | QA/Lead   | Production defect |
| request     | Customer/CS | PM    | Feature ask or support ticket |

Life-cycle rules (enforced by workflows):
- When an `initiative` is opened, automation creates a GitHub Project named after it and pre-populates 5 default tasks (“Setup”, “Design”, “Build”, “QA”, “Launch”).
- When a linked `task` PR is merged, the task auto-closes and the initiative progress bar (project field “% Done”) recalculates.
- If an issue is idle >14 days, the stale-bot adds label “stale” and mentions the assignee; after 7 more days it escalates to the team lead.
- Every night a workflow pushes issue title, state, assignee, labels, cycle-time into the Google Sheet “GitDigital Ops Metrics”.

--------------------------------------------------
3. LABEL TAXONOMY (single source in `.github/labels.yml`)
--------------------------------------------------
Labels answer three questions at a glance:
1. What DOMAIN? (product area)
2. What TYPE? (bug, chore, feature)
3. What PRIORITY? (P0-P3)

Example subset:
```
- name: "domain:mobile"
  color: "0052CC"
  description: "iOS/Android codebase"

- name: "type:feature"
  color: "0E8A16"

- name: "priority:P0"
  color: "B60205"
  description: "All hands on deck; page people"

- name: "okr:KR1.2"
  color: "FEF2C0"
  description: "Links to Q1 KR1.2"

- name: "risk:high"
  color: "D93F0B"
```

The workflow `issue-opened.yml` applies labels automatically from issue-template dropdowns, keeping the taxonomy clean.

--------------------------------------------------
4. PROJECTS (V2) – THE REAL DASHBOARD
--------------------------------------------------
We create two tiers of GitHub Projects:

A. **North-Star Project** (1 per quarter)  
   - View mode: Table + Progress field  
   - Rows = Objectives (custom field “Objective”)  
   - Columns = Key-Results (custom field “KR”)  
   - Each KR cell contains linked initiatives; %Done rolls up from child tasks.  
   - Automation: when any child task closes, a workflow updates the KR’s “progress %” field and writes a comment “KR1.2 now 67 %”.

B. **Initiative Projects** (1 per initiative)  
   - View mode: Board (To Do → In Progress → Review → Done)  
   - Swim-lanes by assignee  
   - Built-in “Burn-up” chart (GitHub native) shows scope creep.

--------------------------------------------------
5. OKR & KPI STORAGE (human + machine)
--------------------------------------------------
- Human file: `data/okr/2025-Q1.md` – nice markdown for all-hands deck.  
- Machine file: `data/okr/2025-Q1.json` – consumed by:
  - `okr-check.py` (validate that every KR has ≥1 initiative)  
  - nightly workflow that posts a Slack summary “3/7 KRs on track, 2 at risk”.

KPI csv (`data/kpi/weekly.csv`) is appended every Sunday by workflow that queries GitHub’s GraphQL API for:
- cycle time (created→closed)  
- throughput (issues closed / week)  
- open bug count by priority  
- % initiatives on schedule (progress ≥ expected week-of-quarter)

--------------------------------------------------
6. AUTOMATION PLAYBOOK
--------------------------------------------------
| Trigger | Workflow file | What it does | Why it matters |
|---------|---------------|--------------|----------------|
| issue opened | `issue-opened.yml` | Auto-label, assign owner, link parent initiative, comment checklist | Removes human triage time |
| PR merged | `pr-merged.yml` | Closes linked task, updates initiative %Done, posts “ shipped” in #releases | Everyone sees momentum without extra meetings |
| schedule (nightly) | `metrics.yml` | Push issue data to Google Sheet | Historical trends, board-level KPIs |
| schedule (Sun) | `okr-summary.yml` | Read JSON, compute KR health, post Slack | Early warning system |
| label “risk:high” added | `risk-alert.yml` | Auto-create “risk” issue in `GitDigital-Workbook` and tag CTO | Single risk register |

--------------------------------------------------
7. HUMAN RITUALS (calendar hooks)
--------------------------------------------------
- **Monday 09:30** – “North-Star Review” (30 min)  
  Facilitator opens the North-Star Project, walks through red KRs, drags initiatives to “At Risk” if behind.  
  Output: comments in KR cards with next step & owner.

- **Wednesday 16:00** – “Initiative Demo” (30 min)  
  Any initiative at ≥80 % Demo column must show working software in staging.  
  If demo fails, initiative automatically moved back to In-Progress and label “demo-failed” added.

- **Friday 15:00** – “Retro Slot”  
  Any initiative that closed ≥1 task that week must open a `retro.yml` issue and schedule 15-min retro.  
  Retro issues are collected in `docs/03-change-log.md` so lessons compound.

--------------------------------------------------
8. SETUP CHECKLIST (copy into README.md)
--------------------------------------------------
1. Create empty repo `GitDigital-Workbook` under `github.com/GitDigital-Products`.  
2. Clone locally, run `scripts/bootstrap.sh` (creates labels, secrets, initial OKR json).  
3. Add Google Sheet API key to repo secret `GSHEET_API_KEY`.  
4. Invite team; point them to `docs/01-ops-model.md`.  
5. Schedule the three calendar rituals above.  
6. Every quarter, duplicate `data/okr/YYYY-Qx.json`, reset progress, archive old North-Star Project.

--------------------------------------------------
9. EXTENSIONS (when you outgrow it)
--------------------------------------------------
- Replace Google Sheet with Metabase + Postgres by changing `metrics.yml` to POST into your warehouse.  
- Add “Estimates” custom field and use story-points; burn-down instead of burn-up.  
- Swap GitHub Projects for Jira/GitLab if you hit 500+ engineers—export is JSON, so migration is painless.  
- Turn `GitDigital-Workbook` into a GitHub Template Repository so new portfolio companies get the same OS in one click.

--------------------------------------------------
10. TL;DR FOR YOUR FUTURE SELF
--------------------------------------------------
This workbook is not a wiki; it is an **executable operating model**.  
Issues = work. Labels = context. Projects = dashboard. Workflows = muscle memory.  
If it isn’t tracked in an issue, it doesn’t exist.  
If the North-Star Project is green, the company is healthy.
