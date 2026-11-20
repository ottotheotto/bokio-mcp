# Roadmap: Obsonator fork

This fork of [diblaze/bokio-mcp](https://github.com/diblaze/bokio-mcp) is the **data layer** for an
in-house "CFO" agent. It reads Bokio safely and does the arithmetic. The CFO itself (financial
calendar, handbook, scheduled briefings, nudges) lives outside this repo, as a Claude skill plus
scheduled tasks.

## Principles

- **Stay thin.** Keep the diff against upstream small so upstream fixes can be merged in regularly.
- **Upstream first.** Anything generic (marked _upstream candidate_) goes to diblaze as a PR, not
  just here.
- **Read-only.** This fork never changes the books.
- **Public repo.** No real accounting data, company ids, names or amounts in code, fixtures or
  issues. Follow upstream's rules in `AGENTS.md`.

## Phase 1: Hardening (do first)

### 1. Read-only by construction
Remove every `WRITE.` tool and the upload path (`fileForm`) from the build, instead of relying on
`BOKIO_ALLOW_WRITES`. This also removes the risk of an upload tool reading arbitrary local files.
- Done when: `tools/list` has no mutating tools whatever the environment says; smoke test asserts it;
  `fileForm` and `readFile` are gone from `src/`.

### 2. Token from the macOS Keychain
Read the Bokio token from the Keychain (e.g. `security find-generic-password`), falling back to
`BOKIO_TOKEN`. No plain-text token in MCP client config.
- Done when: README documents the Keychain setup; tests cover both paths. _Upstream candidate._

### 3. Lock the API host
Drop the `BOKIO_BASE_URL` override (or allow only `api.bokio.se`).
- Done when: requests can only go to `https://api.bokio.se/v1`.

### 4. Local call log
Append one line per Bokio call (time, method, path, status; never the token or response body) to
a local log file outside the repo.
- Done when: the log path is configurable and documented. _Upstream candidate._

## Phase 2: CFO tools (the value)

All computed from the SIE export like the existing computed tools: integer öre inside, kronor at
the edge.

### 5. Result per business side
P&L split by Bokio tag group or cost centre (e.g. design practice vs. CRM practice).
- Depends on: consistent tagging in Bokio (bookkeeping habit, not code).
- _Upstream candidate_ (generic "P&L by dimension").

### 6. Salary vs. brytpunkt tracker
Year-to-date salary per employee from the salary accounts, compared against a configurable
threshold (the brytpunkt for the year). Payroll isn't in the Bokio API, so this reads the books.

### 7. VAT helpers
- Moms summary for a period, laid out like the VAT return boxes.
- Periodisk sammanställning: EU service sales per customer VAT number per quarter.
- _Upstream candidate._

### 8. Receivables ageing
Unpaid customer invoices bucketed by age (0–30, 31–60, 61–90, 90+ days), plus average days to pay
per customer. _Upstream candidate._

### 9. Cash outlook
Bank balance today minus known outflows (salaries, AGI, moms, supplier invoices) plus expected
inflows (open customer invoices), over 30/60/90 days. Recurring amounts come from config, not code.

## Phase 3: Later

### 10. Always-on host
Run on a home Mac mini over HTTP with authentication, so scheduled CFO runs work when the laptop
is closed. Only after Phase 1 is done.

## Outside this repo (the CFO layer)

- Notion "CFO handbook": company facts, decisions, open questions, financial calendar database.
- A Claude skill that reads the handbook and these tools and writes the monthly and quarterly CFO memos.
- Scheduled tasks for the monthly letter, the quarterly review and season playbooks.
- Hard deadlines go to Todoist as single dated tasks; everything else stays in the memo.
