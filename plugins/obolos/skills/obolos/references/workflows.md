# Workflow and batching patterns

Read this for multi-step protocol work. Below, `<doc>` is an existing data room reference,
`obolos://rooms/<room>/folders/<folder>/doc/<n>`; a local file first goes through `obolos docs upload`.

## Classify every step

1. **Atomic-call set**: all addresses, IDs, calldata and values are known before execution; the calls
   share the intended sender; later calls may rely on earlier state; a failure must revert everything.
   Use one `batch()` and require an atomic executor.
2. **Ordered transactions**: a receipt, emitted ID, deployed address or verified postcondition is
   needed to build the next call. Execute them as separate principal-approved stages.
3. **Asynchronous workflow**: an oracle, threshold network, prover, time window, counterparty or
   governance decision must intervene. Persist state and resume only after the condition is observed.

Never force classes 2 or 3 into account-abstraction batching. AA removes repeated wallet prompts; it
does not manufacture unknown IDs, proofs, elapsed time or external decisions.

## Incorporation

`obolos company incorporate` is one conversation for two founders: one whose legal entity exists, and
one who wants obolos to form it. Hold it in the user's words. Flags carry their answers; never show
them as a menu.

1. **Sign in first.** With a key in the terminal, the flow talks to the obolos API; without a
   session it stops with `cli.needs_login` (see SKILL.md). With no key and no login, `--form` hands
   the company to the app instead (step 6).
2. **A formed request comes first.** When the account has a request obolos formed with no company yet,
   the command asks whether to set the company up from it. Ask the user; pass `--formation <id>` for
   yes, or `--formation none` to set up another company.
3. **Ask whether the legal entity exists**, in plain words: "Is the company already formed, or should
   obolos form it for you?" Pass `--existing` or `--form`.
4. **It exists.** Offer to read their formation documents from files they point at: the certificate of
   incorporation, bylaws or operating agreement, founder stock agreements. Show what you read (legal
   name, jurisdiction and entity type, state, file number, principal address, founders and their
   shares) and pass only what they confirm (`--name`, `--jurisdiction`, `--entity`, `--state`,
   `--file-number`, `--address`, `--founder`). Never invent a legal fact; ask for what no document or
   answer gives. Details they lack go `--details later`; `obolos company details set` adds them later.
5. **obolos forms it.** obolos forms only for an account with a verified email, where the team writes
   about the request. If the command stops with `email_unverified`, ask which email to use, run
   `obolos email verify <address>`, ask for the code it sends, run `obolos email confirm <code>`, then
   re-run. Ask the jurisdiction and entity type first (the CLI names the pairs obolos forms), then the
   formation questions. Before they choose a name, say it needs its entity's ending (Inc., LLC, Ltd
   and so on). A Delaware company takes six answers, which the CLI's questions name: its industry
   (`industry`, exactly one of those the question lists; any other stops the run), what it does in up to 50 characters
   (`description`), and the founder's legal first and last name (`firstName`, `lastName`), home
   address (`street`, `city`, `region`, `postalCode`) and phone with its country code (`phone`).
   Ask the user for each; never fill one in from a guess. The name questions offer the account's
   name split at its first space as the default: read both parts out and ask whether that is their
   legal first and last name. Ask too for up to two more names, with their
   endings, in case the first is taken (`otherNames`, a list); none is fine. Then pass them as one payload:
   `--filing '{"industry": "...", "description": "...", "firstName": "...", ...}'`, or
   `--filing @answers.json`, or `--filing -` for stdin. Say what the rest defaults to and ask if it
   fits: the founder holds every office, the registered agent's address goes on the public record,
   and 10,000,000 shares are authorized at $0.00001. Anything else (co-founders, other shares, the
   company's own address) goes in the same `--filing` as every detail, naming its `people`; leave the
   answer flags (`--industry` and the rest) out then:
   ```json
   {"industry": "Software Publishers", "description": "Software for biology labs", "otherNames": ["Helix Labs Inc."],
    "people": [
      {"firstName": "Ada", "lastName": "Lovelace", "email": "ada@example.com", "phone": "+14155550100",
       "address": {"line1": "1 Main St", "city": "Dover", "region": "DE", "postalCode": "19901", "countryCode": 840},
       "offices": ["PRESIDENT", "TREASURER", "DIRECTOR"], "responsible": true},
      {"firstName": "Alan", "lastName": "Turing", "email": "alan@example.com", "phone": "+14155550101",
       "address": {"line1": "2 Main St", "city": "Dover", "region": "DE", "postalCode": "19901", "countryCode": 840},
       "offices": ["SECRETARY", "DIRECTOR"]}],
    "shares": {"authorized": "10000000", "parValue": "0.00001"}}
   ```
   Exactly one person signs for the EIN (`responsible`). A corporation names one president, secretary
   and treasurer and at least one director, and authorizes at most 2,147,483,647 shares; an LLC gives
   each member's `ownership` instead, adding up to 100. `countryCode` is numeric (840 is the US); a
   `businessAddress` in the same shape puts the company's own address on the record. Where obolos has no
   filer for the entity, the CLI asks instead for each member (`--member "Name|email|ownership"`,
   together 100%) and the principal address, and refuses the answers; where it has one, it refuses the
   members. With the six answers the founder is filed under the account's verified email; with every
   detail, each person under the email given. The account's email is also the contact unless they name another
   (`--contact-email`). Ask which plan the company starts on (`--plan free`, `monthly` or
   `yearly`), reading out the price of each from the CLI's question: Plus by the month takes its
   first month with the formation, Plus by the year includes the formation, and a paid period of
   Plus starts once the company is set up. Last, ask whether to set the company up on obolos now or
   once it is formed (`--deploy now` or `--deploy later`).
   The run stops with `invalid_input`, before anything is sent, on an answer flag that differs from
   the same answer in `--filing`, on `--state`, `--file-number` or `--details` with `--form`, and with
   `--deploy later` on any setup flag (`--ticker`, `--founder`, `--preload`, `--modules`, `--class`,
   `--authorized`, `--traits`, `--par-value`, `--liquidation-preference`, `--voting-weight`,
   `--payment-token`, `--metadata`, `--symbol`): pass those once the company is set up. The shares to
   file go in `--filing`, with every detail.
6. **Review, then send.** Restate the request in the user's words from the flags you passed (name,
   jurisdiction and entity type, industry, what it does, each person with their offices or
   ownership, the shares, contact email, deploy now or later), show the review block, and get a yes
   before `obolos send`, formation requests included. The founder's legal name and home address are
   saved to their obolos account when it has none.
   **Say the price first.** The review states it ("Forming it costs $469, Delaware's $109 filing fee
   included", the Plus price, or included in a yearly Plus plan) with what it buys: the filing, an
   EIN and a registered agent for the first year. Nothing is charged by the request itself.
   A formation request is an obolos API request, not a transaction, but it starts real work; with
   `--deploy later` it is the whole run and the wallet sends nothing. If `obolos send` stops with
   `cli.api_failed`, fix the cause and run `obolos resume <run>`. Never run `obolos company incorporate`
   again for it: a new run sends a second request.
   **With no key in the terminal**, there is no review block and nothing to send: the same command
   exits 6 with `cli.awaiting_payment` and `details.link`. Give the user that link. It opens the app
   with the company, the plan and the details you gathered already filled in; they sign in or create
   their obolos account, check the details and pay. Nothing you gathered reaches a server before
   they do. The link takes the company, the plan and the filing's answers only: the command stops
   with `cli.usage` on any other flag or details too long for a link, and with `invalid_filing` on
   an answer the app would refuse. Once the company is set up, connect this terminal to it with
   `obolos login`.
7. **Payment.** A priced run ends by opening the payment link in the user's browser and waits up to
   15 minutes for the payment; obolos files only a paid request. Off a terminal, add `--no-wait` to
   the command that sends the last stage, `obolos send` or `obolos resume` included: it
   exits 6 with `cli.awaiting_payment` and `details.link`. Give the user the link, wait for them to
   say they paid, then check with `obolos company formation <id>`. `obolos company formation <id>
   --pay` opens the link again on the plan the request has, and `--plan` with it changes the plan
   while it is still unpaid. A request obolos declines is refunded on its own.
   **When the filer turns the details down**, the request stays requested and paid, the list shows
   it as `details refused`, and `obolos company formation <id>` shows why under `filingProblem`;
   nothing is refunded. Tell the user what it says, ask for the corrected answers, restate them and
   get a yes, then send them with `obolos company formation <id> --filing '<json>' --yes`: the same
   answers or every detail as above, checked the same way, and filed at once. **A request on hold**
   (`held` is true, listed as `on hold`) waits on obolos, not the user: the filer stopped the filing,
   or refused it for a reason obolos settles with the filer. A run that meets one ends complete and
   prints `On hold.` with its text; don't `obolos resume` it. Tell them what `filingProblem` says:
   obolos sends the order again once it is settled, or writes to them for what it needs. Send new
   details only when obolos asks for them, with `--filing` as above while the request is still
   requested. A payment shown as disputed holds the filing until the dispute ends.
8. **Once it is incorporated.** obolos writes to the contact email. What is left, in order: sign the IRS form
   for the EIN (a link in the app; the EIN follows in 8 to 11 weeks and nothing waits for it), set
   the company up with `obolos company incorporate --formation <id>`, file and sign the bylaws, and file the
   83(b) election once the founder's shares are issued. It is due within 30 days of the day they are
   issued, not of the Delaware filing and not of their vesting; a late one cannot be made.
9. **After a formation request.** `obolos company formation` lists the requests, their status and
   payment; `obolos company formation <id>` shows one, with the file number and EIN once formed. Once it is formed, `obolos company incorporate
   --formation <id>` sets the company up with the formed name, jurisdiction and details. A company
   deployed with the request is linked by the run, and the formed details reach it on their own.

On the protocol side the run:

1. Resolves `(countryCode, entityType)` and reads the live jurisdiction profile.
2. Previews and predicts the company suite: the modules it starts with and the supported instruments.
   Paid modules picked now (`--modules fundraise,grants,voting`) are activated right after the
   deploy, paid from a credit `--preload` or the company's overdraft.
3. Deploys. Receipt-derived company and module addresses are the stage boundary: a prediction is the
   creator's next deploy and moves on any deploy by that creator, so downstream calls take addresses
   from the receipt, never from a prediction batched ahead of it.
4. Prepares share classes, recorded traits, founder issuance, roles and escrow funding in dependency
   order, batching only calls whose calldata is already known and whose sender authority is valid.
5. Verifies the live cap table, separating chain-enforced facts from record-only charter traits.
6. Registers the company with the obolos API and saves its legal details there (API stages after the
   deploy, filled from its receipt).

## Shares

```bash
obolos shares issue --to <wallet> --shares 1,000 [--class Common] [--document <doc>]
obolos shares cancel --holder <wallet> --shares 1,000 [--class Common] [--document <doc>]
obolos shares freeze --holder <wallet> [--document <doc>]        # Enforcement role; unfreeze releases
obolos shares snapshot take [--at 2026-12-31] [--class Common]  # Snapshooter role; omitted --at means now
obolos shares snapshot list
```

Issuance needs the recipient and the human share quantity; the class is asked only when the company
has several. The workflow checks the issuance module the live jurisdiction profile provides, issuable
capacity, recipient compliance, issuance authority, freeze, simulation and credits, then prepares one
stage. Plain share issuance is never refused because a jurisdiction closed.

Cancelling burns shares the holder actually has (buyback, forfeiture, court order) and is refused past
the balance. A frozen holder can neither send nor receive until released. Snapshots feed
distributions and votes; a past date is refused.

## Founder vesting

```bash
obolos vesting create --to <founder> --shares 480,000 --start 2026-01-01 --vesting 4y/1y/monthly
obolos vesting show <scheduleId> [--reveal]
obolos vesting release <scheduleId> --shares 150,000       # beneficiary, after the cliff
```

Schedules use whole intervals (monthly means 30-day months and 12-month years, daily means 365-day
years), so cliff and duration always divide the cadence. Founder schedules default to non-revocable.
The amount is sealed inside the workflow. When the public escrow floor is below the amount, the run
gains a first `fund-escrow` stage. A release is one command that runs request, verdict relay and
finalize as stages with the beneficiary's key.

When a request gives everything except the amounts, the next question is: "How many shares should vest
for each founder?" Never inspect source or cryptographic internals to avoid asking it.

## Compensation

```bash
obolos pool increase --budget option --shares 500,000
obolos valuation record --fmv 1.00 --document <doc>
obolos grant issue --type option --to <employee> --shares 40,000 --vesting 4y/1y/monthly
obolos grant exercise <grantId> --shares 10,000 [--net]          # holder, with their key
obolos grant exercises [--holder <addr>] [--reveal]              # what's in flight, due and quoted
obolos grant confirm-exercise <grantId> [--paid-on <date>]       # company, once the strike arrives
obolos grant count-exercise <grantId>                            # company, for a net exercise
obolos grant decline-exercise <grantId>                          # company, when it never arrives
obolos grant withdraw-exercise <grantId>                         # holder, before the payment or the count is on record
obolos grant settle-exercise <grantId>                           # finish one whose check is still unpublished
obolos grant record-exercise <grantId> --shares 10,000 --paid-on <date> --document <doc>
```

**Granting.** Read the compensation profile, supported grant kind and tag, class capacity, pool
budget, valuation and recipient transfer compliance. The kind question offers what the live profile
supports. Options need a valuation on record. A budget shortfall asks whether to increase it first,
which adds a stage. Prepare the grant with its escrow and pool calls. Where the protocol selects
encrypted zero instead of reverting, require a private non-zero postcondition before paperwork.

**Exercise for a strike (options and warrants).** No strike is paid on chain.

1. The holder's `grant exercise` only files a request, with their own key (`grants.exercise@1`, one
   stage). The run needs no confidential session, but with a configured key the CLI opens one to check
   the request against what has vested and quote the amount due first.
2. The holder pays the company off-platform, quoting the payment reference the run gives;
   `grant exercises` lists it with the amount due.
3. The company's `grant confirm-exercise` (`grants.exercise.confirm@1`, Share management, warrants
   included: a warrant is equity) publishes the vesting check first, then confirms the payment for the
   request it showed, settling the shares in the same call. `--paid-on` is when the payment arrived,
   the exercise date; `--document` is a receipt in the data room, if any.
4. A request filed before expiry can be confirmed until the company's settlement window after expiry
   closes (14 days unless set, up to a year). After that, confirmation is refused.

When the holder asked for more than has vested, the run skips the confirmation and says to decline.
`grant decline-exercise` (`grants.exercise.decline@1`) clears an unpaid request while the grant is
open; the holder's `grant withdraw-exercise` clears their own. A paid or counted exercise always
settles: nobody withdraws it, cleanup and transfers wait, and either side can finish it with
`grant settle-exercise`. A holder who can't hold the shares then, refused by the class's rules or
frozen, has them parked to claim with `shares claim`.

**RSUs and phantom units.** The holder's `grant exercise` settles them itself.

**Net exercise.** When the company allows it, `grant exercise --net` only files the request. The
company's `grant count-exercise` (`grants.exercise.count@1`) reads the valuation the request was filed
at, which the holder never sees, counts the whole shares it nets, publishes the count and settles
them. A request netting under one whole share, underwater included, counts as zero and settles with
nothing issued, as does one that wasn't vested.

**Recorded exercise.** For a holder who can't request it, the company runs `grant record-exercise`
(`grants.exercise.record@1`) with the reference of their signed notice and the date the notice and
payment arrived. That date must be before expiry (recordable during the settlement window after) and
is the date vesting counts to. The run records, relays the verdict and finalizes, and reports a denial
with the payment to return, or shares parked for a holder who can't hold them yet.

Requests, confirmations and recordings are asynchronous, with a threshold verdict before settlement.
A denied verdict is a valid outcome, not a reason to vary the quantity until one succeeds.

## Fundraising round

```bash
obolos fundraise create --type safe --name Seed --target 1,000,000 --hard-cap 2,000,000 --cap 10,000,000 --discount 20% --agreement ./term-sheet.pdf
obolos fundraise reserve <round> --investor <wallet> --amount 250,000         # a spot, its amount sealed, signed for the company by a document signer
obolos fundraise reservation sign <round> --reservation <id>                  # investor: signs the spot (--accept-terms only once they agree to terms that differ from the round's); an agent asks first, and --in-browser has the user sign in the app
obolos fundraise subscribe <round> --amount 50,000                            # investor, with their key
obolos fundraise reservation countersign <round> --reservation <id>           # the company signs; on an offer, that accepts it. An agent hands it to a signer in the app
obolos fundraise record <round> --reservation <id> --amount 50,000            # the company
obolos fundraise hardcap <round> --amount 2,500,000
obolos fundraise close <round> && obolos fundraise finalize <round>
obolos fundraise publish <round>
```

Confirm the round is one `obolos company show` lists under `jurisdiction.instruments`. One with
`active: false` needs each module in its `activate` list activated first (`obolos company module
activate <module>`); every round runs on `fundraise`. A round opens on an investor policy with rules; a
company with none needs one first (`obolos compliance policy create --preset <id> --class <class>`).
Prepare round creation, document references, whitelist changes and reservations in their natural
stages; counterparty-signed reservations and encrypted investor inputs are not manager batches.

- **Create** makes the round's data room, publishes the round's agreement unsigned in its Term Sheets,
  then opens the round (its document reference is the room), so the run is staged. Give the agreement
  as a document on this machine (.pdf, .md or .txt; save a Word file as a PDF). The run pins its hash
  and stores it unsealed once the user sends the run's first step, the data room, which says so. Don't upload it to a
  data room first; the round refuses an `obolos://` reference.
- **Sealed figures.** Money never moves through the contract, and every amount is sealed. A priced
  round's price per share is sealed too, with a public commitment the round's agreement names. The
  company reads it; adding an address to the whitelist shows it the price for good; `subscribe` first
  unlocks it for an investor the round would accept, then sizes the amount in whole shares.
- **Spots.** The company commits a spot for a named investor at sealed terms. The amount is
  negotiated, so only the hard cap binds it. Countersigning shows the investor the price.
- **Offers.** An investor whose wallet is activated at the company offers at the round's terms with
  `subscribe`. The offer signs the agreement and pays nothing. One offer per person stands open until
  the company countersigns it or the investor withdraws it with `reservation cancel`.
- **Payment.** Once both parties have signed, the investor pays by bank transfer or by sending the
  payment token to the company's vault, and the company runs `record`, naming the shares on a priced
  round.
- **The check behind `record`.** Unless the deal's check already holds a yes (countersign asks for it
  when a fundraise manager signs), `record` first asks the network whether the sealed amount fits the
  hard cap beside what other checked deals hold; for an offer, the round's minimum and the person's
  maximum; on a priced round, that it pays exactly for the shares named. It records with the answer,
  relayed as it lands and only when it is a yes that holds the deal's room. A no stops the run before
  anything is sent, and a record the round refuses publishes nothing. A yes holds the room until the
  record or a cancel, so other records never make it stale.
- **Hard cap.** On an open round, `fundraise hardcap` raises the cap, for example when an investor
  wants to put in more, or lowers it; `--amount 0` lifts it. While payments are recorded or checked
  deals wait for their record, the cap can only be raised or lifted.
- **Close and finalize.** Read back investments before close/finalize and quote the unbounded manager
  cost. Finalize walks the issuance windows until the round is finalized; on a priced round it first
  asks the network whether the sealed total qualifies the round, and the completing window relays the
  answer. Finalization and any later issued-instrument IDs are a state boundary. Verify the issued
  instruments and assemble data room records without exposing confidential terms.
- **Publish** makes the sealed total public, if the company chooses; on a priced round that reveals
  the price too. Reads show a sealed figure as confidential to anyone whose session cannot open it.

## ROFR-controlled transfer

1. Read the class policy, attached transfer-approval rule, ROFR terms and current notice state.
2. Simulate the exact seller, buyer, class and amount.
3. Prepare the next legal state transition: request/notice, holder/company election, decline,
   approval, revocation, or transfer after a valid lapse allowance.
4. Respect election and closing windows. Silence is not immediate permission: never describe an
   unanswered active notice as transferable. Keep a lapsed notice allowance distinct from a standing
   unrestricted transfer.
5. Re-run transfer compliance immediately before the final transfer; jurisdiction and credentials can
   change during the window.

## Compliance sweep

Across each company, read freeze status, jurisdiction profile version, classes, policies, rules,
holders, active financing instruments, round deadlines and compensation capacity. Use public and
aggregate data only. Produce exceptions and deadlines; never decrypt investor economics.

## Conversion

Conversion is an asynchronous state machine:

1. Eligibility and qualifying-financing checks. A series opens on its own when a priced round
   finalizes at the qualified financing threshold. `conversion trigger` only reopens a qualifying
   round's lapsed series, on that round's class, price and pre-money basis. A reopen sweeps what is
   still active of the round's own cohort, by chain order: the instruments recorded between the
   previous qualifying round and this one. Anything recorded later, whatever date it carries, belongs
   to the next qualifying round; a cancelled instrument that is reissued gets a new id and waits with
   them. `getRoundCohort` in the sdk lists a round's cohort.
2. Locate the open conversion (`conversion list`).
3. Pin the basis.
4. Collect proving access and produce ordered proof chunks.
5. Fold the series state.
6. Apply bounded conversion batches.
7. Verify the final cap table, or present rollback/cancel choices within their windows.

Never batch across proof generation or cursor-dependent steps. On a verifier rejection, preserve the
state and evidence; never automatically roll back or regenerate inputs.

## SPV distribution (calculator)

`spv waterfall` allocates exit proceeds through a vehicle's frozen terms and reports the split; it
moves nothing. Give it the vehicle's round (`--round`) and the exits in order, each with its approved
reserves. It reads from the chain every investor's net capital, the carry and the platform's share
fixed when the round opened (`fundraise create --carry 20% --platform-share 20%`), and each spot's
source, fixed when it was committed (`fundraise reserve --source platform`, the fund lead's when
omitted) or named by the investor who subscribed (`fundraise subscribe --source`). Typed `--carry` and
`--contribution` model what the chain does not hold. The result is cumulative: a second exit never
returns capital twice and only charges carry on real profit. Present it as the model the vault will
later pay against, not as a settlement; `distribution waterfall --round` pays an exit from the same
chain inputs.

## Data room and access groups

Folder, room and group creation often emits IDs that later calls need. Prefer SDK receipt-bound
helpers or separate stages. Before preparing membership or key changes, explain the rekey
consequences: who gains access, who loses future access, and which historical ciphertext grants stay
usable.
