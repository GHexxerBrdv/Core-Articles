# The Beacon Chain Ethereum 2.0

# What is beacon chain?

before September 15 2022, ethereum was running on the Proof of Work (POW) consensus mechanism. in which the miners used computerns to solve puzzles. whether in morden ethereum the miners are replaced by the validator. 

## Validators

**Who is validator ?**
A validator is someone who:

- Locks up (stakes) 32 ETH
- run ethereum software
- Helps confirm transaction and secure the network
instead of using electricity like miners did, validator uses stacked money as security.

If they behave badly - they lose their staked money
If they behave correctly - they get reward

So think of a validator as security guard who put their money in ethereum to prove they will do their job securly and get reward for it.

## Block Proposer

the Block Proposer is validator who create a block.

Ethereum time is devided in slots and epochs.

1. each slot is 12 seconds long
2. each epoch is 32 slots long

you can visulize it as following

```
|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|
```

as shown in ethereum timeline each slot is 12 seconds long. in each slots one rendom validator is choosed as block proposar. only that validator can create a block. each slot has different validator choosen as proposer. 

as block proposer their job is to:

- collect transactions
- put them into block
- broadcast the block to the network 

after creating a block in each slot. the block will be broadacsted to the network, at this momnet of time most validators are not proposing the block.

**Why each slot has only one block proposer?**

suppose if there are huge numbers of nodes(validators) in the ethereum network, and if all the validator propose the block they the network will be flooded with blocks and becoms slow. So instead of all validator creating block we choose a random validator as block proposer in each slot.

**What if the proposer is not create the block?**

if proposer fails to create the block, then they will miss out the reward. Note that a slot can be empty. if the proposer fails to propose the block so that slot will be called empty

## Attesters

Attesters are those validators who are not proposing any block at any moment of time. thier job is to attest the block created by the block proposer. while attesting any block the attesters put their votes as valid or invalid on the proposed block. Here the vote is called attestation

Think of it like: 
- 1 student do some task
- 30 other student vote if the task done is valid or invalid
- if most aggreed - it becomes official

same happens in this process also. after proposing a block, all the validators attest on that block. if the block has majority valid attestations then it will be included in the blockchain. othervise rejected.

Here the attestions weight is determine by the staked ETH by the validators. If a validator has more staked ETH then thir vote will have more weight then other.

Now the final asnwer of the question `What is beacon chain?`.

The beacon chain is coordinating system of ethereum.
that:
- keeps list of validators
- tracks how much ETH each validator has staked
- records all attestation
- assigns who proposes the block
- rewards honest behavior
- publishes dishonest behavior

it is basically the management system of the ethereum's Proof-of-Stake.

**What if validator do votes maliciously?**

Ethereum designed in such a way that validators can watch other validator or police other validator. 

if a validator:
- votes for two conflics blocks
- propose multiple block in one slot
- tries to cheat the system

in such case other validator can report them.
this lead to slashing(burn all their stakes) - which means the malicious validator will lose their all/partially staked ETH
by doing this all the validator will be financially motivated to behave correclty

# in short
every 12 seconds
- a validator is selected randomly as block proposer
- they build and broadcast the block
- other validators vote on it
- votes are recorded on beacon chain
- honest participants earns reward
- dishonest participants lose money 

## Beacon chain is chain like structure?

Yes, it is a chain like structure as shown below.

```shell
Block 1 -> Block 2 -> Block 3 -> ....
```

in every slot the block proposer create a beacon block and boradcast it.

each block contains:
- who voted
- who proposed
- validator balance
etc

**Note:-** before the ethereum moved to Proof-Of-Stake. everything was handled by the miners. after the merge, the Beacon chain become the consensus backbone

**What is different in beaco chain and transaction chain?**

before answer this question we need to know that the beacon chain does not contain transaction in it's block. for storing transaction there is transaction chain which is different then beacon chain. However each beacon block contain one block of transaction chain. the concepts are same as transaction chain in beacon chain. Like the blocks are connected with previous block with block hash. each beacon block contain it's previous block has which connects and create a full beacon chain.

think of beacon block as a big box which contains all the data about validators, their votes, balances etc, and the transaction block it self. Here each transaction block contain it's previous transaction block hash from previous beacon block which connects and create full transaction chain. in short the beacon chain and the trasaction chain grows and run in parallel, but the catch is that the beacon chain contains the transaction chain inside it. 

it can be confusing as first. but it is way simpler as showen below

```shell
[ Consensus Block1 ]                                                           
    |
    └── Contains:
          - Attestations (votes)
          - Validator updates
          - Rewards / penalties
          - A reference to: 
                ↓                                    
           [ Execution Payload (transaction block) ]      
                - Transactions
                - Gas fees
                - Smart contract changes
                
                |
                |
                |

[ Consensus Block2 ]                                                           
    |
    └── Contains:
          - Attestations (votes)
          - Validator updates
          - Rewards / penalties
          - A reference to: 
                ↓                                    
           [ Execution Payload (transaction block) ]      
                - Transactions
                - Gas fees
                - Smart contract changes
```


# Committees

A committees is a group of validators. for security each slot has commitees of 128 validators. in each slots the validator is selected randomly by RANDAO and shuffles validators to committees. it's possible a validator is proposer and committee member for the same slot, but it's not the norm. the probaility of this happening is 1/32 so we will see it about once per epoch. only the assigned committee is allowed to vote in current slot. 

**why only assign committee can vote?**
if there ishuge numbers of validators then each slot the huge number of votes will broadcasted. and it will explod the network. thats why we choose to only assign comitte can vote in a slot.

# LMD Ghost = Ethereum fork choise rule

this rule says that choose the chain that has the most recent validator votes behind it. mean the chain with the most voting weights. 

let's consider three slot condition.

1. slot 1
  - One validator is chosen as proposer.
  - It creates Block 1.
  - Committee A (a group of validators assigned to Slot 1) votes.
  - 2 validators vote.
  - 1 validator is offline → no vote.

  Block 1 has 2 votes behind it.
  
  Here it is not mandatory for all validators in committee to vote
  
2. slot 2
  - A new proposer creates Block 2.
  - Most validators see Block 2.
  - BUT one validator in Committee B does NOT see it (network delay). there is possible network delay
    
  That validator only knows about Block 1. It votes for the latest block it knows (block 1). That vote is called an LMD GHOST vote.
  
  Why does it vote for block 1?
  - Because the validator are required to vote for what they beleive is the head of the chain
  
  Here Ethereum assums:
  - Network delays happen
  - Honest validators vote based on local view

3. slot 3
  - Now Committee C runs the fork choice rule.
  - they check How much validator weight supports Block 1? and how much in block 2?
  Even though one validator voted for Block 1 in Slot 2,
  Block 2 probably has more total weight.
  So everyone in Committee C independently chooses the same head.
  
  here the rule is deterministic
  
  # Beacon chain checkpoints
  
  A checkpoint is a first block in the first slot of an epoch. if there is not such block, then the checkpoint is the preceding most recent block. there is always one checkpoint block per epoch, and a block can be checkpoint for multiple epoch.
  a checkpoint block is also called as EBB(epoch Boundry Block)
  
  for example if all slots have a block:
  
  Slot 0 → Checkpoint
  
  Slot 32 → Checkpoint
  
  Slot 64 → Checkpoint
  
  Slot 96 → Checkpoint
  
  note here when casting a LMD Ghost rule, a validator also votes for the checkpoint block in current epoch. called target. and this vote is called FFG vote and also includes the prior checkpoint. called the source. 
  
  ## super majority
  
  A vote that made by 2/3 of the total balance of all active validators, is seemed a supermajority. suppose there are three active validators: two have a balance of 8 ETH, and a sole validator with a balance of 32 ETH.  The supermajority vote must contain the vote of the sole validator: although the other two validators may vote differently to the sole validator, they do not have enough balance to form the supermajority.
  
  # Finality
  
  Finality tells how Ethereum finalizes blocks using Casper FFG (Friendly Finality Gadget).
  
  we know about the slots, epoch and checkpoints from above discussion. Finality works at the checkpoint level, not individual blocks.
  
  at the end of every epoch validator votes for:
  1. A source checkpoint
  2. A target checkpoint
  
  If a checkpoint gets ≥ 2/3 of total validator stake voting for it. it becomes justified.
  
  Now this is very important to understand. if checkpoint B is justified and the checkpoint in the immidiate next epoch is also justified then the checkpoint B will be finalized. that means the finality requires two consiqutive justified checkpoints means it will take two epoch to finalize the checkpoint. 