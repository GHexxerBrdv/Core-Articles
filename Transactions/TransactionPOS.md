## Overview

This document explains **end-to-end how a transaction gets executed and finalized in Ethereum Proof-of-Stake (PoS)**.

Ethereum today operates as a PoS blockchain after “The Merge”, where:

* The **Execution Layer** handles transactions and smart contracts.
* The **Consensus Layer** decides which blocks are valid and final.

This separation is fundamental to understanding modern **Ethereum**.

---

# Transaction Creation

## What Happens?

A user creates a transaction that includes:

* `to` address
* `value` (ETH amount)
* `data` (if interacting with a smart contract)
* `gas limit`
* `max fee per gas`
* `max priority fee per gas` (tip)
* `nonce`

The user then **signs the transaction with their private key**.

## How Is It Done?

Usually through libraries like:

* ethers.js
* web3.js
* web3.py

These tools communicate with an Ethereum node using the **JSON-RPC API**.

---

# Gas Fees (EIP-1559 Model)

Ethereum uses the EIP-1559 fee structure.

## Fee Components

### Base Fee

* Automatically adjusted per block
* **Burned** (removed from circulation)

### Priority Fee (Tip)

* Paid directly to the block proposer (validator)
* Incentivizes inclusion in a block

### Total Fee Formula

```
Total Fee = Gas Used × (Base Fee + Priority Fee)
```

Base fee → 🔥 Burned
Tip → 💰 Validator reward

---

# Execution Client Validation

The transaction is submitted to an **execution client**, such as:

* Geth
* Nethermind
* Erigon

## What Does It Check?

* Valid signature?
* Correct nonce?
* Enough ETH balance?
* Sufficient gas?

If valid:

* The transaction enters the **mempool**
* It is broadcast across the execution-layer gossip network

---

# Mempool & MEV (Advanced Concept)

Normally:

* Transactions are broadcast publicly.

However, advanced users sometimes send transactions privately to:

* Flashbots

This allows:

* Protection from front-running
* Custom transaction ordering
* MEV (Maximal Extractable Value) strategies

---

# Slot & Block Proposer Selection

Ethereum operates in **slots**:

* 1 slot = 12 seconds
* 1 epoch = 32 slots (~6.4 minutes)

For each slot:

* One validator is pseudo-randomly selected using RANDAO
* That validator becomes the **block proposer**

---

# Block Construction (Three Clients Working Together)

A validator node consists of:

1. Execution Client
2. Consensus Client
3. Validator Client

---

## Execution Client

Responsibilities:

* Selects transactions from mempool
* Executes them in the EVM
* Produces:

  * Updated state root
  * Receipts
  * Logs
* Creates an **Execution Payload**

---

## Consensus Client

Examples:

* Lighthouse
* Prysm
* Teku

Responsibilities:

* Wraps execution payload into a **Beacon Block**
* Adds:

  * Attestations
  * Slashing info
  * Rewards & penalties
  * RANDAO reveal
  * Sync committee data

This block exists on the Beacon Chain.

---

## Validator Client

Responsibilities:

* Holds validator private keys
* Signs:

  * Block proposals
  * Attestations

---

# Block Propagation

The block proposer:

1. Broadcasts the Beacon Block over the consensus network.
2. Other nodes receive it.
3. They pass the execution payload to their execution client.
4. Transactions are re-executed locally.

If:

* The state root matches
* All rules are followed

The block is considered valid.

---

# Attestations (Voting Process)

Validators then:

* Check the block
* Verify it builds on what they believe is the best chain
* Sign an **attestation**

Ethereum’s fork choice rule is:

### LMD-GHOST

Meaning:

> Follow the chain that has the greatest validator weight behind it.

The more attestations a block gets, the heavier it becomes.

---

# Block Confirmation

Once enough attestations are received:

* The block becomes part of the canonical chain
* Transactions inside it are **confirmed**

However:

Confirmed ≠ Finalized

---

# Finality (Very Important)

Ethereum finality happens at the **epoch level**.

## What is a Checkpoint?

At the start of each epoch:

* A checkpoint block is defined.

Validators attest across the epoch.

---

## Supermajority Link

If:

* ≥ 66% of total staked ETH
* Attest to checkpoint A
* Then later attest to checkpoint B

A → B forms a **supermajority link**

This finalizes the chain between them.

---

## What Finalized Means

If your transaction is in a finalized block:

* It cannot be reverted
* Unless ≥ 33% of total stake is slashed
* Which would cost billions in penalties

Finality typically occurs after **2 epochs (~13 minutes)**.

---

# 🧠 Complete Flow Summary

1. User signs transaction
2. Sent to execution client
3. Validated and added to mempool
4. Proposer selected for slot
5. Execution client executes transactions
6. Consensus client wraps into Beacon Block
7. Block broadcast to network
8. Validators re-execute and attest
9. After supermajority link → finalized

---

# 🏛 Architecture Insight

Post-Merge Ethereum separates:

| Layer           | Responsibility                   |
| --------------- | -------------------------------- |
| Execution Layer | Run transactions, maintain state |
| Consensus Layer | Decide block ordering & finality |

This modular architecture makes Ethereum:

* More secure
* More scalable in future upgrades
* Easier to evolve

---

# Beginner Mental Model

Think of Ethereum PoS like:

* Users → Submit transactions
* Execution Layer → Computes results
* Consensus Layer → Agrees on results
* Validators → Vote on correctness
* Epochs → Lock in finality

---

# Advanced Takeaways

* Fork choice rule: LMD-GHOST
* Finality rule: Casper FFG
* Economic security threshold: 33% stake
* Block time: 12 seconds
* Epoch time: ~6.4 minutes
* Finality time: ~12–13 minutes

---
