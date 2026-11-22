# obolos

The obolos plugin for Claude Code. Incorporate a Delaware company with Claude, and it runs on obolos from the day it is filed: cap table, share issuance, SAFEs, rounds and a data room. Claude also answers from your obolos account: the companies you work in, your holdings and rounds, data rooms and messages.

## How it works

The plugin has two parts:

- **The obolos connector**, an MCP server run by obolos. Claude asks it about your companies, holdings, rounds, data rooms and messages, and through it starts forming a company. It acts for you alone and sees only what you can see in obolos.
- **The obolos skill**, which teaches Claude the obolos CLI and `@obolos/sdk` for what the connector does not do: issuing shares, running a round and other changes to a company. It asks you before anything is sent.

Signing in:

1. Claude opens obolos in your browser.
2. You sign in and approve what Claude may do: read, form companies, messages, and save issuances.
3. obolos gives Claude a key for this connection alone. You can end it at any time in obolos, under Passport › Assistants.

## Installation

```
/plugin marketplace add equitylayer/obolos-equity
/plugin install obolos@obolos-equity
```

Restart Claude Code, then run `/mcp` and sign in to obolos.

The skill installs the obolos CLI the first time it needs it: `npm install -g obolos`, with Node 22.12 or later.

### Try it out

- "Form a Delaware C-Corp for me and my cofounder, with four-year vesting."
- "What's on my To do in obolos?"
- "Which rounds am I in, and what is left for me to do in each?"
- "Summarise the files in Acme's Seed data room."
- "Reply to Acme's last message: I will sign tomorrow."
- "Save a grant of 25,000 options for Grace Hopper at Acme, with employee vesting."

## Skills

| Skill | Description |
| --- | --- |
| `obolos` | Form a company and run it on obolos through the obolos CLI and `@obolos/sdk`: cap tables, share issuance, fundraising and other company workflows, each reviewed before it is sent |

## Connector tools

| Tool | What it does | Permission | Changes anything |
| --- | --- | --- | --- |
| `whoami` | Your obolos account | any | No |
| `get_profile` | The account this connection acts for, for apps that list connected accounts | any | No |
| `help` | How obolos works, from its help articles | any | No |
| `to_do` | Your To do | read | No |
| `list_companies` | Your companies and holdings | read | No |
| `get_company` | One company you work in: cap table, grants, vesting, SAFEs, notes, rounds, distributions, issuances, updates, billing | read | No |
| `get_rounds` | A company's rounds, or one with its spots and offers and who has signed | read | No |
| `get_exercises` | A company's option exercises in progress | read | No |
| `get_votes` | A company's votes, or one with its matters and results | read | No |
| `get_transfers` | A company's transfer rules, and the transfer notices nobody has answered | read | No |
| `get_treasury` | A company's treasury: its balance, what is free and what is set aside | read | No |
| `get_compliance` | The compliance rules in force at a company | read | No |
| `prepare_issuance` | Saves shares, a grant, a warrant, a SAFE or a note for you to issue in obolos | save issuances | Adds a saved issuance |
| `get_portfolio` | Your holdings, convertibles, grants and vesting | read | No |
| `list_deals` | The rounds that are your deals | read | No |
| `get_deal` | One round: its terms, where you stand, your spot or offer | read | No |
| `list_documents` | The files of a data room you can read | read | No |
| `read_document` | One data room file | read | No |
| `get_read_page` | More rows of a list an answer cut short | read | No |
| `list_threads` | Your conversations | messages | No |
| `read_thread` | One conversation | messages | No |
| `send_message` | Sends a message for you | messages | Adds a message |
| `formation_options` | What forming a company takes, with prices | form companies | No |
| `request_formation` | Asks obolos to form a company | form companies | Adds a request |
| `submit_formation_details` | Gives a requested company its details | form companies | Replaces the details |
| `formation_status` | How a formation is going | form companies | No |
| `withdraw_formation` | Withdraws a request the filer does not have yet | form companies | Closes the request |
| `prepare_company_setup` | Prepares a company for obolos with its founders' shares | form companies | Replaces the prepared set-up |
| `company_setup_status` | How a company set-up stands | form companies | No |

The connector never signs, pays or issues shares. What it adds are messages, formation requests, prepared set-ups and saved issuances, each for you to look over: you sign, pay and issue in obolos, at the links it gives, or through the skill.

## Encrypted amounts

Many amounts in obolos, such as investments, prices and grants, are encrypted to the accounts allowed to see them. The connector never receives them, so Claude cannot read them either. Where an answer has one, Claude says so and gives you the link where it opens in obolos.

## Data and privacy

The plugin collects nothing itself: it has no hooks and sends no usage data. The connector keeps what the [obolos privacy policy](https://obolos.io/privacy) describes under "Connected assistants".

## Support

legal@obolos.io
