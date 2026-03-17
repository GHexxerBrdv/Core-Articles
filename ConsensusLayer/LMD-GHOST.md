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

As name suggests, GHOST (Greediest Heaviest Observed Subtree), this rule will always choose the heaviest vote for the head block and to approach a head block. We will see below what it means.

Before when there was POW, the LMD GHOST rule always start from the initial block of the chain to determine the head block. which means we have to always go to genesis block to find the head block. Now with PoS, the LMD GHOST rule starts from the latest checkpoint which is updated periodically. However the checkpoint is not the discussion of this article. we will see it further. 

We will take a small example to determine the wroking of the rule. Coonsider the following chain:

```
                                                             
Block A'(4) <-------- Block A (4) <--------------- Block B (1) <--------------- Block C (2)
                              \                                  
                                \                                
                                  \                              
                                    Block D (3) <--------------- Block E (1)
```

suppose we are validator who is selected to cast attesation or to create block. Now let's consider we have received other validators votes for the head of the chain, which also runs the LMD GHOST rule. As you can see above we have votes for each block, which is determine by the score when the validators has assigned the votes to each. the block vote is sum of all the votes for that block. also note that as we discussed above we will consider the latest attestation for each validator. so we can prevent double count.

Here now we have to run LMD GHOST rule to cast attestation of to create a block. at this time the rule will run in two steps.

1. first it will assign a score to each branch of the chain from initial block which is Block A' here. the score will be assigned based of the rule we have discussed earlier above.
So chain become something like this

```
                (11)                      (3)                           (2)
Block A'(4) <-------- Block A (4) <--------------- Block B (1) <--------------- Block C (2)
                              \                                  
                                \  (4)                              
                                  \                   (1)           
                                    Block D (3) <--------------- Block E (1)

```

2. Now we start from the initial block and according to GHOST rule, we will greedily choose the branch with the highest score. In this case, the branch with score `11` -> `4` -> `1` will be chosen and the head block will be Block E.

# Incentives

LMD GHOST rule has introduced the incentive for the validators who performs correctly by choosing the branch with the highest score. However it also punish them if they act maliciously. In POS there are two responsibality of the validator. 

1. proposing a bloock.
2. creating attestations

if validator acts good and runs LMD GHOST rule correctly then they will be incentivised. This incentive scheme is designed to reward the validators to secure the network from malicious actors. Now if some validator acts maliciously then their staked ETH will be slashed (burned). As we know to become a validator or to get incluaded into validator list we have to stack minimum of 32 ETH. This stake is deposit to tell the network that the validator will act coorrecly while participating, and this deposits can be burned if the validator acts maliciously. as of now we won't go deeper into slashing mechanisms.

Now we will see those two responsibilities of the validator.

## Proposing a Block

When a validator is selected to propose a block. they must create only one block on top of the head. in rewards they will get priority fees of  all the transaction they include in the block and some additional amount of ETH minted to them. See below diagram for more understanding

```
------------------------
|      tx1             | -----------> fees |
|      tx2             | -----------> fees |
|      tx3             | -----------> fees |
|      ...             | -----------> fees |
|      ...             | -----------> fees |
|      txN             | -----------> fees |
|      (Block)         |                   |
------------------------                   |
                                           |
ETH amount -------------------------------- + ------> validator

```

If they act maliciously and create two blocks instead of one, they will be penalized by the network according to the slashing rules.

> This is the interesting part. In Bitcoin there is no such concept of slashing. However in POW if a miner creates two blocks then they will eventually spending more resources and money. and at the end only one block will be accepted by the network. That means building two blocks in POW is allowed but there is no profit for miners. and it is useless if they create two blocks. That is why there is no such slashing in POW.

## Creating Attestations

When a validator selected to vote on their view of the network in form of attestations, they must cast only one attestation.by doing that they earn some small fee which is much smaller then the block building. If the validator tries to cheat by creating more than a single attestation or contradictory attestations, the protocol explicitly punishes them by slashing a proportion of their stake. 


If the validator keeps behaving maliciously for quite a long time, the protocol has the power of force-ejecting them from the validator set.

