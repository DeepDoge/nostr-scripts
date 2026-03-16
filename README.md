# Nostr-Scripts

> Gonna impl this later some time, rn just collecting my ideas.
> Probably gonna make it after making bitcoinatlas

Nostr-Scripts is an isolation-first scripting layer, without global state or ledger, anchored to Bitcoin with a single 32-byte hash per batch, and using open networks like Nostr, uses external payment systems that can be verified offline like lightning.

It is not a global virtual machine and not a shared account ledger.

Each user only downloads and replays the script states they actually care about — their own scripts and any explicit dependencies — and proves those states back to Bitcoin through compact Merkle proofs.

---

## Core Idea

Nostr-Scripts is built on the idea of isolated script states.

A script is a small, self-contained state machine.
It defines its own ID, its own state or state root, a sequence of state **transactions**, optional dependencies on other scripts, and optional references to external payment proofs.

There is no global world state to sync.
Your client only interacts with the minimal dependency graph required for your activity.

---

## Data Flow

Nostr-Scripts separates responsibilities across three layers:

1. **State and logic** — stored in scripts and their transactions.
2. **Transport and availability** — Nostr or any broadcast layer for discovering transactions.
3. **Settlement and finality** — Bitcoin anchoring each batch with a 32-byte commitment.

Payments happen externally and are not tracked inside Nostr-Scripts.

---

## Scripts and Transactions

A script transaction typically includes the script ID, the previous state hash or version, the new state hash or diff, references to dependent script states, and optional external payment proofs.

Aggregators collect transactions from Nostr, order them, commit them into a Merkle structure, and publish the Merkle root to Bitcoin.
The transactions stay off-chain; Bitcoin only commits the hash.

---

## What a Client Actually Does

A Nostr-Scripts client does not download or replay a global history.

It only:

1. Selects a script to interact with.
2. Fetches the script’s transactions, dependency transactions, Merkle proofs, and the Bitcoin anchors containing the batch roots.
3. Replays only that small local subgraph.
4. Verifies script logic, dependency state, Merkle inclusion, Bitcoin anchoring, and external payment proofs.

Your entire node consists of a few scripts, their transactions, their proofs, and the Bitcoin anchors. Nothing global.

---

## External Payments

Nostr-Scripts does not become a balance ledger.
Value moves in systems that supply client-verifiable receipts, such as Lightning, Ark v2, Fedimint, or on-chain Bitcoin.

Scripts may require those proofs:

* “This update is valid only if a Lightning invoice preimage is provided.”
* “This action requires a valid Ark spend proof.”
* “This transaction depends on an on-chain payment receipt.”

Nostr-Scripts never tracks full payment histories.
It only validates the provided proofs according to the script rules.

---

## Isolation and Dependencies

Each script is an isolated island of state.

It can operate independently or declare explicit dependencies on other scripts.
When a script depends on another, the client fetches and verifies only those transactions, forming a small verifiable state graph.

Users never download unrelated scripts or unrelated activity.

---

## Anchoring and Proofs

Aggregators batch script transactions into a Merkle tree and publish the root to Bitcoin.

To verify a script state, the client only needs the transactions for that script, transactions for its dependencies, the Merkle proofs, and the Bitcoin transaction containing the root.

From these, the client reconstructs the script state and its anchoring to Bitcoin.

---

## Mental Model

Nostr-Scripts is not a blockchain.

It is a structure for verifiable, local script states that anchor collectively to Bitcoin without global replication.

Many scripts, each with its own state machine.
Optional dependencies forming small graphs.
External systems handle value.
Bitcoin provides the anchor.

Users verify only the part of the world they touch — nothing more.

## New ideas

We don't need scripts to be bind to aggrigators. We can have a aggrigator market, aggregating and anchoring calls with a proof of work.
