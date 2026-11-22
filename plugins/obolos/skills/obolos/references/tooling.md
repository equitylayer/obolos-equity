# Tooling: how an agent calls obolos

Read this when deciding how a shell-capable agent operates an obolos company. First commands, human
units, exit codes and sending are in SKILL.md.

## Three layers

| Layer | Owns | Package |
| --- | --- | --- |
| Skill | Intent routing, the questions to put to the user, the safety posture | `obolos` |
| CLI | Company selection, human units in flags, hash-verified run files, simulation, the signer boundary, RPC transport | `obolos` |
| SDK | Typed reads, `Prepared` builders, and the workflow engine (`@obolos/sdk/workflows`): input normalization, preflight, staging, confidential sealing, postconditions | `@obolos/sdk` |

The CLI holds no business logic: every command calls `client.workflows.prepare(id, input)` and the
workflow chooses the builders and stages. Websites and backends can enter at the workflow, module or
ABI layer; the agent enters through the CLI.

## Command map

| Domain | Commands |
| --- | --- |
| Setup | `doctor`, `skills install|list` |
| Account | `login`, `logout [agent]`, `agents`, `agents use <agent>`, `agents check <address>`, `email verify`, `email confirm` |
| Company | `company show`, `company roles`, `company incorporate`, `company details [set]`, `company formation [id] [--pay] [--filing <json>]`, `company modules`, `company module activate <name>`, `company upgrades`, `company upgrade schedule|cancel|execute`, `company set metadata|ticker`, `company role grant|revoke|renounce`, `companies`, `use`, `whoami` |
| Identity | `identity status` |
| Shares | `shares issue`, `shares authorize`, `shares captable`, `shares claim`, `shares cancel`, `shares freeze|unfreeze`, `shares parked list|cancel`, `shares snapshot take|list`, `shares class create`, `shares escrow fund` |
| Vesting | `vesting create|release|revoke|accelerate|transfer|list|show`, `vesting milestone request|approve|reject|cancel`, `vesting excess withdraw` |
| Compensation | `grant issue|exercise|exercises|confirm-exercise|count-exercise|decline-exercise|withdraw-exercise|settle-exercise|record-exercise|accelerate|transfer|attach-document|revoke|release-unvested|cleanup|list|show`, `grant milestone …`, `pool increase|decrease|show`, `pool decrease request|finalize|cancel`, `valuation record|list` |
| Fundraising | `fundraise create|whitelist|reserve|subscribe|record|publish|hardcap|close|finalize|cancel|refund|list|show|investments|reservations`, `fundraise reservation attach|sign|file|countersign|cancel|sweep` |
| Messages | `messages list|read|post|open|wait`: an agent's round threads |
| Instruments | `safe issue|cancel|discharge|list|show`, `note issue|cancel|discharge|early-repayment|repay|list|show`, `instruments access`, `conversion trigger|settle|rollback|cancel|list|show` |
| Compliance | `compliance presets`, `compliance policy create [--preset 506b|usOnlyPrivate|3c7Private|regS|noUs|506c|usOnly|3c7] [--class]`, `compliance rule attach|detach`, `compliance set …`, `compliance pause|unpause|show|check` |
| Transfers | `transfer request|approve|decline|revoke|send|show`, `rofr set|show` |
| Voting | `vote room|create|extend|cancel|list|show|access|cast|withdraw|proxy|certify|abort` |
| Distributions | `distribution dividend|waterfall|record|pay|top-up|claim|claim-pending|close|cancel|show`, `spv waterfall` |
| Documents | `docs room create`, `docs folder create`, `docs upload|replace|share|grant|revoke|read|sign|list|show` |
| Treasury | `treasury show|deposit|withdraw|pause|unpause`, `credits show|topup|quote` |
| Passport | `passport attest|activate` |
| Runs | `send <run>`, `status <run>`, `resume <run>`, `simulate <plan>` |
| Low level | `sdk actions|read|prepare|batch`, `decode <revert-data>`, `workflow list|describe|run` (hidden) |

## Run files and reads

The review block's preflight lists `capability`, `authority`, `freeze`, `simulation` and `quote`, each
`passed`, `not-configured` or `skipped`. A failed check refuses before anything is written. A run file edited by
hand is refused.

Run files hold the private cross-stage state needed to resume. They are written owner-only (`0600`;
new run directories `0700`). Treat them as confidential local artifacts: never print or attach them.
For automation use `--json`. Run output is then a redacted `obolos-run-view/v1` projection with
workflow, stage and check status plus hashes, never normalized business inputs, private state,
calldata or decrypted stage outputs. Its `file` field points to the private resumable artifact for
`obolos send|status|resume`. Beside it, `company` names the company the run acts at: `address`, `id` or
a `note`, and `from` when `./.obolos/config.json` alone chose it.

Reads (`show`, `list`, `captable`, `check`) return results directly. Add `--reveal` to decrypt the
confidential values the acting wallet may see; without it they show as confidential. Reveal when the
user you work for asks for a sealed figure, with the narrowest read that shows it (`--reveal` on a
`list` or `captable` decrypts every row), and tell them what it reads.

## Signer boundary

The CLI accepts no `--private-key` value; SKILL.md says how the user supplies a key. The acting wallet
is the key's address, or after `obolos login` the user's wallet the agent wallet works for. Preparing
needs only that wallet, and sealing confidential inputs binds them to it. With a key, decrypting,
publishing verdicts and sending need the key. As an agent wallet, decrypting needs Read on that wallet
at the company, sending needs the power that makes each call, and a command that signs as the wallet
itself, such as publishing a verdict, is the user's to do in the app. Never ask the user for a key.

A terminal can hold several agent wallets; each `obolos login` adds one and puts it in use. At a
company the CLI acts as the agent in use when it holds powers there, otherwise as a connected agent
that does. `obolos agents` lists them and `obolos agents use <name>` changes the one in use; pick
one only on the user's word. A name two agents share exits 2 listing them; name the agent by its
address instead.

## Low-level access

`obolos sdk actions|read|prepare|batch` expose single SDK builders for integration work and are not an
operational path. Builders that take sealed inputs are refused there and point to the command that
fronts them. `obolos decode <revert-data>` decodes protocol errors.

## SDK use during development

When the user explicitly asks to build, review or debug an integration, a reviewable TypeScript
program may import `@obolos/sdk` and `@obolos/sdk/workflows` and use its `Prepared`, `batch`,
`sendCall`, `toObolosError`, address maps, jurisdiction helpers and workflow engine. A missing
workflow is a product gap to call out, never a reason for hand-written calldata.
