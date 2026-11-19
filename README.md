# Astelly platform: overview

Astelly is building a marketplace where a small business sells an invoice it is owed, and investors buy it whole or in slices and can sell those slices on. This repository describes how the software is designed and tested. The code itself is in a private repository.

**Status, 17 November 2026.** The design is written and reviewed. The foundation and books layers are built and tested, and the sums layer is next. Nothing is live. Astelly is not yet incorporated or authorised, and nothing here is an offer or an invitation to invest.

## How it is built

One TypeScript program on one PostgreSQL database, divided into 20 modules in eight layers. A module may call the modules below it and never those above; a lower module tells a higher one that something happened only by announcing an event.

| Layer | Modules | Job |
|---|---|---|
| 1 Foundation | Configuration, Event log, Parties | Settings and switches, the audit trail, who exists and what they may do |
| 2 Books | Registry, Ledger, Settlement | Who owns what, every movement of pounds, and the one place either may change |
| 3 Sums | Valuation, Fees | A value for each asset each day, and every fee |
| 4 Asset classes | Invoices | Everything specific to invoices; later asset classes take the same form |
| 5 Markets | Listings, Order book | The first sale of an asset, and resale |
| 6 Schematics | Schematic engine | Standing instructions that buy for the investors who follow them |
| 7 Servicing | Distributions, Payments, Reconciliation, Checks, Document store, Notifications, Reporting | Paying owners, instructing the bank, matching every bank movement, and the checks and records around a sale |
| 8 Interface | API | The only way in, used by Astelly's own website with no shortcuts |

Four rules shape the rest:

- **Ownership is counted in whole units and money in whole pence.** One unit of an invoice is one penny of its face value.
- **Every movement of pounds is recorded twice**, so the books always balance, and the database refuses a posting that does not.
- **Units and pounds change in one step or not at all.** Only one module may change the ownership records or the ledger, so a buyer cannot have paid and own nothing.
- **The software never moves money.** It records what is owed and produces payment instructions; an independent trust company controls the bank accounts, and every bank movement is matched back to the ledger each day.

## How it is tested before any code is written

The design is one note per module: what the module owns, its commands, its queries, its events and its rules. It was tested in two ways before the build began.

- **Scenario traces.** 48 cases were walked step by step through every module, carrying the pounds and units through to the end: an invoice part-sold, part-paid and then written off; a disputed invoice bought back in instalments while its debtor also pays; one bank transfer covering five different things. The two traces found 54 faults, almost all a rule in one module that a general rule in another quietly broke. Each was decided and fixed in the design. In the final 120-day case the books balanced at every checkpoint.
- **Build-readiness reads.** A reader given only the notes for the three lowest layers wrote the database tables and every command's inputs and outputs, and listed each point where they had to guess. The first read found 22 guesses that would change a table or a command; the second found six. All are now settled.

The 48 traced cases become the automated tests the code has to pass.

## How AI is used

The design, the decisions and the testing are the founder's. The code is written with AI coding tools, and every change is checked against the module notes and the tests. Commits in the private repository carry the tool's co-author line.

## Progress

| Date | What |
|---|---|
| 12 November 2026 | Platform designed as 20 modules in eight layers. First scenario trace: 23 cases, 29 faults found and fixed |
| 13 November 2026 | Second scenario trace: 25 cases, 25 faults found and fixed. Two build-readiness reads of layers 1 to 3 |
| 13 November 2026 | Code repository set up: TypeScript in strict mode, tests against a real database, and an automatic check that fails the build if a module breaks the layer rules |
| 15 November 2026 | Database tables and command signatures for layers 1 to 3 written, then read against the design by a fresh reviewer; its 20 findings fixed or decided. The database itself now refuses edits to permanent records, backdated entries and postings that do not balance |
| 15 November 2026 | Foundation layer built: Configuration, Event log and Parties, with the shared sums for pence and a one-off setup command. 83 automated tests |
| 16 November 2026 | Foundation layer re-tested by a second reader who saw only the design: 297 tests written without sight of the code, and the hand-worked figures of every traced case recomputed independently |
| 17 November 2026 | Books layer built: ownership records, the double-entry ledger, and the one module that changes either, all in one step or not at all. 503 automated tests, 399 of them written without sight of the code |
| 17 November 2026 | Independent review of the code of layers 1 and 2, its findings fixed. The books' checks reworked so a settlement costs the same at a million steps of history as at ten thousand, with the full proof run nightly. 513 automated tests |
| 18 November 2026 | Second fresh read of the database rules and the design changes behind them. The faults it found are fixed, each with a test shown to fail without the fix, and looking up a past balance or holding now costs the same however far back. 522 automated tests |
| 19 November 2026 | A further fresh read of those fixes; its faults fixed, and every open design question on the foundation and books layers answered and recorded. 528 automated tests |
| 19 November 2026 | The running program's start-up and its nightly jobs built, then the first parts of the sums layer: the fee rules that stand alone, and the module that gives every sold asset one value each day, with a test showing that a value as at any past day costs the same however long the history. 556 automated tests |
