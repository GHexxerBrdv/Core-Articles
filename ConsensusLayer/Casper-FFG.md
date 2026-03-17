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

