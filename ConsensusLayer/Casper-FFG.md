# Casper FFG - A finality mechanism

What is the Casper FFG? in Casper FFG, the FFG means friendly finality gedget. that means the working of this rule is something related to finalizing something right? the answer is `yes`.

Casper FFG is a finality layer also called `metaconsensus` which not it self consensus, but it is attached of top of the consensus layer. it does not build a chain but it helps to finalize the chain. Now here the finalize means that making chain permenent.

> After finalizing the chain it can not be reverted or changed. That means the re-org is not possible.

In ETHEREUM POS:

- Underlaying protocol is LMD GHOST rule which decides the head to follow further to create new blocks.
- Casper FFG adds a guarantee that which part of chain is permanent.

The reason we needed it is that if only the LMD GHOST rule was used. then the head of the chain could just change in future that makes the chain not permanent, that introduces the re-org problem where chain can be reverted and re organized with new head. if every time this happens then the chain would be broken and the network would be in an inconsistent state. After the Casper FFG we can add finality in the chain and make it permanent.

## How Casper FFG works (Voting logic overview)

Ethereum POS knows all the validators and their stakes already. So the consensus is based on stake-weighted voting. Keep this in mind it will help further to understand how Casper FFG works.

Now there is a safety threshold. Casper FFG follows the BFT (Byzantine Fault Tolerance) protocol. Where the sytsem is safe if `<1/3` validators are malicious. We assume that in the open blockchain there can be malicious actors. so the limit is there. remaining `>=2/3` validators should be legit. and finality depends on that validators vote.

Now there is a catch. if any malicious actor try to act maliciously then there is penalty mechanism in Casper FFG to punish them. They will be slashed (penalized) for their actions. This is called `Accountable safety`.

# Epochs and Checkpoints

Ethereum organizes the time into `Slots`, `Epochs` and `Checkpoints`.

Each `Slot` contains one Block. each slot is 12 seconds long. That means every 12 seconds a new block will be created.

> However slot can be empty also.

Each `Epoch` contains 32 Slots (<= 32 blocks).

And Each `Checkpoint` is first block of each `Epoch`.

Now the reason the checkpoint exist is: Instead of voting on every block, validator vote once per epoch, they vote on checkpoints only. this reduces the voting overhead.

Now each validator votes for source checkpoint and target checkpoint. the goal is to link the chain and build the chain of justified and finalized checkpoints.

See the following structore of one epoch.

```
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 32 |
  ^                                                                                                                                                      ^
  |                                                                                                                                                      |
checkpoint 0                                                                                                                                        checkpoint 32
```

# Justification and Finalization

Now each checkpoint should be justified and finalized. there is two steps to finalize the checkpoint. 

- Each validator broadcast and gether their view on checkpoint. if majority agrees then that checkpoint will be justified.
- after in subsequent round/epoch if validators confirms supports the justified checkpoint, then it will be finalized.

if any checkpoint is finalized that means that block and before it all the blocks will becomes irreversible. that means re-org will not happen after finalizing checkpoint. re-orgs can be only happened if checkpoint is not finalized.

in CasperFFG, votes include source and target checkpoint, which represents the validators commitments to the blockchain state at different point. Every validator vote is not just for single block, instead it has two parts as shown below:

```shell
(source checkpoint) → (target checkpoint)
```

here the source checkpoint is already justified checkpoint, and target checkpoint is what validator whats to justify next. This creates `chain of agreement` that means there could not be random votes. Instead of vote rendomly validators link checkpoint together.

Each vote creates a link `s -> t` where `s`: source checkpoint and `t`: target checkpoint

## Supermajority link - core consensus condition

A link become valid if and only if there exist `>= 2 / 3 ` of total stake votes for the same link. this is called supermajority link

For example if mejority validator agree on link `A -> B` then the network accepts this transition as consensus-backed.

If `A -> B` get `>= 2 / 3` votes then `B` becoms justified. if additionally `A` was already justified then `A` becomes finalize.

Now **What if A is not justified?**

if A is not justified then it wont prevent to make B as justified. But if B justified in this case then link would me look like following

```shell
A(Not Justified) -> B (Justified)
```

In this case A could not be finalized. because it is not justified. This will break the rule and nothing will be finalized in future. 

This case won't be happened but we can consider this answer as of now.

Now **What if validator mis behave?**

If any validator misbehave then there is concept called `Slashing`.

# Slashing

Slashing means economic punishment, if a validator breaks the rule then they will be punished. for punishment their part of staked ETH will be burned according to slashing rules.

slashing can be done in two cases

1. Double Vote -> Validator votes for two different target in the same epoch

```
A -> B
A -> C
```

2. Surrounf Votes -> Validator makes two votes as shown below

```
Vote 1: s1 → t1
Vote 2: s2 → t2

such that:

s1 < s2 < t2 < t1
```

Slashing is neccessary to prevent network to become malicious.

# CasperFFG role in LMD-Ghost Rule

Casper FFG modifies the traditional LMD-GHOST fork choice rule, mandating that nodes prioritize the chain with the highest justified checkpoint; this checkpoint then effectively becomes the starting block for the LMD-GHOST protocol. This adaptation, which is an evolution from the LMD-GHOST protocol's approach, ensures that the network achieves finality by committing to checkpoints that have been agreed upon by a supermajority of validators. It effectively guarantees that once a checkpoint is justified, the network cannot revert beyond it, reinforcing the security and stability of the blockchain. This rule is also designed to maintain network liveness, aligning with Casper's foundational goals.
