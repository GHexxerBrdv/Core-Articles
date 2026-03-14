# Understand GAS in EVM

**Note:** If you are not familier with EVM, this could be confusing. I recommend you read the [EVM](./EVM.md) first before diving into GAS.

Before diving deep into the GAS mechanics, first we need to understand `why` GAS is needed in EVM.

below is the reason for that:
- Ethereum is turing complete -> means we can run loops. however in normal system we can run infinite loops. But `what if we run an infinite loop in ethereum?` below point is the answer
- Ethereum is single threded machine -> means only one transaction can be proceed at a time. so if one transaction runs an infinite loop, other transactions will be stuck. and ethereum will become useless

To solve this problem, Ethereum introduced GAS. each block in ethereum has a limit of gas that can be used. if a transaction uses more gas than the limit, it will be halted by EVM and rejected.

# What is GAS?

GAS is ethereum unit to calculate the cost of executing a transaction. it is a gas fee that is paid by the sender of the transaction. Ethereum must account for every computation steps performed by transactions and smart contract execution.

Each operation and steps performed by transactions and smart contracrs costs fixed amount of GAS. According to Ethereum Yellow Paper. 

- ADD opcode - 3 gas
- KECCAK256 opcode - 30 gas + 6 gas for each 256 bits of data being hashed
- sending transaction const 21000 gas.

Gas is crucial component of the ethereum ecosystem. it serves two roles.

1. buffer between price of ethere and the rewards to validators
2. defence mechanism against spam and DoS attacks

the initiator of the transaction need to set limit to the amount of calculation they are willing to pay gas for. this system will take out fees from attacker for spam and DoS attacks. as they have to pay for the computations the validators perform.

# Gas accounting

When a transaction is created, the gas limit is set by the sender. the has limit is the actual gas limit that sender is willing to pay. Now when EVM is needed to complate the transaction, it is provided gas supply equal to the gas limit in the transaction. every opcode that executed by the EVM costs gas. Which is predefined. However there is such case in that the gas limit is large but the actual gas used is less. 

In such case the remaining gas will be refunded to the sender. Following is the formula for calculating the refund:

```
refund = gas_limit - gas_used
refund_ether = refund * gas_price
```

Here you can see that the refund is calculated in ether, That means the ether will be paid as gas fees and refund as well. However there is gas price which is set by the sender and is used to convert gas to ether. We will lgo more deep into gas price after. 

Now as we know for executing transaction we have to pay validator a gas fees. Following is the formula for calculating the gas fees:

```
gas_fees = gas_used * gas_price
```

Now what if the gas limit is less then the actual has needed to execute the transaction? In such case the transaction will fail and discarded immediately. But the gas must be paid to the validator even if the transaction fails.

**in conclusion** the gas cost is the number of units of gas required for computation and storage used while executing the transaction. And gas price is the amount of ether per unit of gas a user want to pay for executing the transaction.

## How gas price can effect the transaction?

As we have seen that sender will specify the gas price per uint of gas consumed. However does it mean a user could just pass zero amount as gas price? and get transaction done? is it fair and is it possible?

The answer is yes, it is possible that user can sepcify very less or zero gas price. But there is a catch. The transaction will not execute immediately. The gas price is used to determine the order of transactions in the mempool. Validators always look for the highest gas price first, so a low gas price will be rejected by the network or stays in mempool for infinitely long. According to gas price the validators sort the transactions from highest to lower in the mempool and execute the highest gas price first. this is the main reason why frontrunning attacks are possible. However there are private mempools also, but they are not part od this discussion.

Now think from above. What if validator just take huge numbers of transactions and execute them in same block? The answer is the block gas limit.

# Block gas limit

Every Block in ethereum has a gas limit. If transactions included in a block exeeds the block gas limit then the block will be discarded by the network. suppose there are 3 transactions in a block. And the block gas limit is `X`. If those transactions consumes more than `X` gas then the block will be discarded. To incluade block into the blockchain the gas usage in the block must be less than or equal to the block gas limit.

## Who choose the block gas limit?

Before the EIP1559, miners had a built in mechanism to vote on block gas limit, where miners can increase or decrease the block gas limit in subsequent blocks.

Now the validator vote of gas target, and the block gas limit is defined to be twice the target. 

Now you might have question, that why just not increase the block gas limit and incluade large numbers of transaction? The reason is that increasing the block gas limit would make the network more expensive to run, as validators would need to store and process more gas. Also validating and verifying each blocks for other validators would be more expensive. Only validators who have better computational resources would be able to process more transactions and include them in the block. the validator with less resource will be dead and the consensus will be broken. But the protocol is looking forward to increase gas limit to support more transactions in the future.
