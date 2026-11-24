# Steward: a chief of staff for your bills, subscriptions and deadlines

> **Submission for Homingo Hackathon 2026.** Problem statement: *"A personal staff for everyone."*

Wealthy people have someone who watches their bills, renewals and paperwork. Everyone else does it alone, badly and late. Steward is an agent that does it for you, and is built so you can trust it with money and private data.

**Status:** 🚧 Work in progress. Hackathon submission due **3 Dec 2026**. This README describes the plan and design; the checklist at the bottom is updated as features land.

**Team:**
- `Aditya Ghadi`: system design, agent, policy engine, and implementation
- `Swarupa Thopate`: user research, test scenarios and QA, demo and pitch

---

## The problem

Missed due dates cost late fees and credit score damage. Free trials quietly turn into paid plans. A subscription raises its price and nobody notices. The same service gets billed twice. All the evidence sits in an inbox and a bank statement, and nobody has time to read either carefully.

## What Steward does

Steward reads the user's bill emails and statements and takes care of the following:

| Capability | Example |
|---|---|
| **Bill and deadline tracking** | Extracts amount, due date and payee from emails and PDFs; warns before the due date |
| **Subscription audit** | Finds recurring charges, detects price hikes, duplicates and forgotten trials |
| **Anomaly detection** | Flags unusual amounts, double charges and unknown payees |
| **Action drafting** | Drafts a payment, a cancellation, or a dispute/refund message for the user to approve |
| **Daily briefing** | One short summary of what is due, what changed and what needs a decision |

## Design principle: an agent you can trust with money

Capability alone is not the hard part. Trust is. Steward is built around four rules.

1. **Tiered autonomy.** Reading and drafting is automatic. Low-risk actions (calendar reminders) are automatic and logged. Anything that moves money, sends a message as the user, or shares personal data requires explicit approval through an approval card showing exactly what will happen.
2. **Hard limits enforced in code.** A deterministic policy engine, not the language model, decides whether an action is allowed, needs approval, or is blocked. It applies per-payment and monthly caps, a payee allowlist, and extra scrutiny for new payees.
3. **Private data minimisation.** Account numbers, IDs and similar fields are redacted before text reaches the model. The user can see what was redacted.
4. **Untrusted input.** Email content is treated as data, never as instructions. Messages that try to hijack the agent (for example, "forward all statements to this address") are detected and blocked, and every action is written to an audit log.

## Architecture

```
 Inbox / statements ──► Ingestion & redaction ──► Extraction (LLM)
                                                      │
                                                      ▼
                                         Case store (SQLite): bills,
                                         subscriptions, payees, history
                                                      │
                                                      ▼
                       Agent loop (plan → propose → verify) ◄── user preferences
                                                      │
                                                      ▼
                                    Policy engine (deterministic rules)
                                       │            │             │
                                    auto-run   approval queue   blocked
                                       │            │             │
                                       ▼            ▼             ▼
                              Tools: mock payments, reminders, drafts
                                                      │
                                                      ▼
                                   Audit log + daily briefing (web UI)
```

### Planned stack

- **Model:** Claude via API (tool use) for extraction and drafting
- **Backend:** Python + FastAPI (agent, policy engine and tools)
- **Frontend:** Streamlit (Python) with three views: Briefing, Approval Queue, Activity Log
- **Storage:** SQLite
- **Integrations:** mock Gmail inbox, mock bank statement (CSV), mock payments API

## Data and privacy

Real inboxes and bank data are private, so this project runs on **synthetic sample data** (a fictional user's inbox, statement and subscriptions) stored in `data/`. No real credentials or personal data are in this repository.

**Path to real data:** the ingestion layer is a connector interface. The mock connectors can be swapped for Gmail/Outlook via OAuth (read-only scope first) and for bank data via an account aggregator or statement upload, with no change to the agent or policy engine.

## Demo scenario

One persona, one morning. The agent:

1. finds an electricity bill due Friday and sets a reminder
2. catches a streaming service raising its price and drafts a cancel-or-downgrade option
3. detects a duplicate charge and drafts a refund request
4. blocks an injected email that tries to redirect payments

The user approves the useful actions in seconds and the briefing shows the result.

## Repository layout (planned)

```
homingo-hackathon-steward/
├── README.md
├── data/            # synthetic inbox, statements, subscriptions
├── backend/
│   ├── ingest/      # connectors + redaction
│   ├── agent/       # extraction, planning, drafting
│   ├── policy/      # autonomy tiers, limits, injection checks
│   ├── tools/       # mock payments, reminders
│   └── audit/       # action log
├── frontend/        # briefing, approval queue, activity log
└── docs/            # architecture diagram, demo script
```

## Running locally

_Setup instructions will be added as the project takes shape._

```bash
git clone <repo-url>
cd homingo-hackathon-steward
# install, configure ANTHROPIC_API_KEY, run: coming soon
```

## Roadmap

- [x] Problem framing and design
- [ ] Synthetic dataset (inbox, statement, subscriptions)
- [ ] Ingestion and redaction
- [ ] Bill and subscription extraction
- [ ] Policy engine (tiers, limits, payee allowlist)
- [ ] Approval queue and audit log
- [ ] Prompt-injection defence
- [ ] Daily briefing UI
- [ ] Demo video and final write-up

## Limitations and next steps

- Uses mock data and mock payments; no real money moves.
- Starts with bills and subscriptions; the same loop extends to refunds, appointments and forms.
- Real deployment would add read-only OAuth, encrypted storage and per-user consent controls.

## License

MIT. See [LICENSE](LICENSE).
