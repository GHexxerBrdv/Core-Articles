# LMD GHOST rule in consensus mechanism od ethereum pos

LMD GHOST rule is main part of ethereum pos consensus mechanism. it is the fork choise rule algorithem. it is used to select the latest block a validator/node should consider valid in their local view of blockchain. this block is also called the `head` of the chain.


Before diving into it, let's understand something very important. 

Before when there was POW in ethereum, entities was responsible for creating blocks. and adding them to the chain. no need to adher any additional requirements. if they publish a block that satisfy the POW requirements, it is considered valid and accepted by the entier network.

But in POS, entities must stack big amount of ETH as collateral to become a validator. In POW those entities were called miners. right now in POS at least 32 ETH is required just to enter into validator set.

Validators have two main responsibilities.

1. Block proposing
  - Every slot (12 seconds) a validator is pseudorendomly choosen to create and propose the next block of the chain.
2. Creating attestations
  - Every slot (12 seconds), a proportion of the validator is selected to publish their votes on the block they believe is the best head of the chain. Then these votes are shared with every validators in form of an attestation.

When a validator votes for certain block, they are assigned with score. This score is equal to the amount of ETH the validator has staked at the time they have published attestation. Here is catch, This vote is not only vote for the single block. it is also a vote for the entier chain up to that block. That means, the ancestors of that block are also considered valid by the validator.

To make this concept even clear, we assign a score to branch. A `branch` is the link that connects the block with it's parent block.

```
              branch
Block A <--------------- Block B
```

Now what is the score of the branch? the score of the branch is the sum of the score of the block that roots that branch + the score of all it's direct descendant branches.

Lets consider an simple example.

Look following chain:

```
                   (2)                          (1)
Block A (1) <--------------- Block B (1) <--------------- Block C (1)
```

The score of the branch `Block C -> Block B` is the score of `Block C`. Here no descendant branch is present, so the score is just the score of `Block C`.
Now the score of the branch `Block B -> Block A` is the score of `Block B` + the score of the branch `Block C -> Block B`. So the score of `Block B -> Block A` is `1 + 1 = 2`.

Note here each block score is 1 by default.

Suppose we have complex chain example as shown below:

```
            (5)                     (2)                            (1)
Block A' <-------- Block A (1) <--------------- Block B (1) <--------------- Block C (1)
                              \                                  
                                \ (2)                               
                                  \                    (1)          
                                    Block D (1) <--------------- Block E (1)
```

Now in this example the complexity adds at `Block A`, until that the simple rule applies in each individual branch. Now let score of `Block A -> Block A'` is `1` (Block A score) + `2` (score of branch `BlockB -> Block A`) + `2`(score of branch `Block D -> Block A`) = `5`.

Here the score of block is not only influence that single block. but also all its ancestors. the idea is simple, if a validator votes for a block, it also votes for all its ancestors. 

Now we can finally understand the LMD GHOST.

# LMD GHOST

The LMD GHOST is made up of LMD (Latest Message Driven) and GHOST (Greediest Heaviest Observed Subtree).

## LMD

We know that we assign each block and branch a score. to assign the score we have to consider the most recent attestation from the validator. suppose a if you receive two attestation from validator, then you do not have to count them twice. instead you look into them and see which one is most recent and discard the other one.

## GHOST

GHOST is the key aspect of the fork choise rule. the head block is the block which has no further descendants that is part of the fork with the highest vote.