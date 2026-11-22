# Operation routing

Map an obolos intent to its SDK domain. Verify exact names and signatures against the installed
version before writing code.

| Intent | SDK areas | Important constraints |
| --- | --- | --- |
| Deploy or inspect a company | `company`, `companyProfiles`, `jurisdictions` | Preview the live jurisdiction profile and predicted suite; Fundraise, Grants and Voting are activated later (`company modules`, `company module activate`) |
| Roles and lifecycle | `company`, `roles` | Check before every write; never self-grant; freeze blocks writes |
| Share classes and cap table | `company`, `shares` | Authorized capacity must include issued supply and dilutive budgets; traits can be record-only |
| Compensation and warrants | `grants` / compatibility `options` | Capability depends on compensation profile, grant kind and scheme tag; some over-budget paths settle to encrypted zero |
| Founder/restricted-share vesting | `vesting` | Releases and some reductions are staged; transfer/revoke/accelerate are high impact |
| Fundraising rounds | `fundraise` | At most one active serialized round; instruments require deployed engines and live profile support |
| SAFEs and notes | `safes`, `notes` | Economics are confidential handles; direct issuance and round issuance must preserve provisions; a payout at an exit is a discharge, not a cancel; a cancel past 7 days needs a document |
| Conversion | `shares`, `conversion`, `proving` | Pin, fold, prove, apply and rollback are stateful stages, not one wallet batch |
| Compliance policies and rules | `compliance`, `hub`, `attestation`, `countries` | Check attached rules, accepted claims and transfer restriction before preparing issuance/transfer |
| ROFR | `compliance` | Record terms, then respect notice, election, closing, assignment and lapse state |
| Distributions | `distributions` | Open, commit, pay, record, cancel and close are treasury actions; claim and claim-pending are holder rights with distinct authority and timing |
| Treasury | `vault` | Withdraw and pause paths require explicit high-impact review |
| Credits | `credits` | Quote manager actions before preparing; holder actions may be unmetered but still need authority |
| Data room and access groups | `dataroom`, `documentGrant`, `accessgroups`, `crypto` | Rekeying changes access; receipt-bound IDs often force staged execution |
| Confidential input and proving | `fhe`, `encryptedInputs`, `proving` | Principal-bound sessions only; proofs and oracle/threshold results create async boundaries |

## Common reads

- Company posture: company info, suite addresses, freeze status, jurisdiction description.
- Cap table: share classes, outstanding and fully diluted totals, holders and balances.
- Financing: round state, reservations/investments, active SAFEs and notes, maturity/status.
- Compensation: grant lists/statuses, pool and budget capacity, valuation metadata.
- Compliance: policies, attached rules, accepted credentials, transfer restriction/message.
- Operations: credit position and quotes, pending exercises/releases/decreases, dataroom structure.

## Common prepared writes

- Company: deploy, metadata, ticker, vault, role changes, view claims.
- Shares: create class, increase authorization, issue shares, fund escrow, record traits.
- Grants: budget changes, valuation, options/RSUs/phantom/warrants, milestone actions, exercise,
  transfer, revoke, release, cleanup, view sharing.
- Fundraise: create/close/finalize/cancel round, whitelist, reserve, invest, record, refund.
- Instruments: issue/cancel/discharge SAFE or note, early-repayment consent, repay, conversion stages.
- Compliance: create policy, deploy/attach/detach rule, set parameters, approve/request/decline
  transfer, record ROFR terms, pause/unpause token.
- Treasury and credits: withdraw/pause, top up/grant credits, price/multiplier administration.
- Dataroom/groups: create and organize rooms, add/share documents, manage group membership and keys.

Every write goes through the safety preflight and returns one `Prepared` operation when possible. Use
`batch()` only when the step classification in [workflows.md](workflows.md) says the calls can safely
share an atomic boundary.

## Error-driven remediation

Choose the remedy from the structured SDK error:

- `module_inactive`: the profile allows the module but the company hasn't activated it. Quote its
  price (`company modules`) and offer the `activateWith` command as its own approved step, since
  activation is paid and permanent.
- `module_unavailable` or another capability error: explain the live profile; never offer a generic
  bypass.
- role error: name the required role and prepare a separate admin-reviewed grant only if requested.
- freeze error: stop.
- compliance error: identify the rule and the legitimate state, credential or approval needed.
- insufficient credits: quote the deficit and prepare a top-up separately. A top-up or preload under
  the ledger's minimum is refused (`below_minimum`, `preload_below_minimum`). As an agent, the user
  loads them: `obolos credits topup` opens their browser.
- `cli.no_agent_here` (exit 6): no agent this terminal is connected as works at the company. Ask the
  user to run the `obolos login --company <company>` it names; the page gives their agent there
  Invest and Read, and the command then goes on as that agent. Never connect it without their word.
- As an agent with Invest, offering into a round, signing its agreement and paying into the vault
  sign and send as the user's wallet at that company; they still need the user's yes before sending.
- user rejection: report the cancellation without retrying.
- network/bundler error: preserve the operation and retry only when the user asks.
- contract invariant/panic: stop and investigate rather than modifying inputs until something passes.
