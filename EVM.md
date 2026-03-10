# What is the EVM?

EVM is the execution engine of the Ethereum. Thisnk of it as CPU of Ethereum. Whenever a transaction is made on ethereum, the EVM comes into action.

There can be two types of transactions on ethereum.

1. Transfer funds to another account from EOA to EOA.
2. Execute a smart contract.

While in case one no need to run any code in ethereum. EOA funds transfer is handled by the EOA itself. However in case 2 where we have to execute a smart contract, the EVM comes in the picture. 

during executing a smart contract, the EVM loads the bytecode of the smart contract, executes the functions, update the storage and returns the result.

EVM conseist of millions of executable objects, here we can refer smart contracts as an object, A smart contract is an object with code and storage. this makes the ethereum as word comuter as it contains millions of executable objects, which can be account address, and smart contracts. All the smart contracts bytecode runs in the EVM. That's why we can consider EVM as the cpu of ethereum. Where you submit instructions on the ethereum and EVM executes and updates the state.

The ethereum is actually the state machine, where the state is updated from one state to another. And the state update is invoked by the transactions.

> old_state + transaction = new_state

This makes ethereum a deterministic ssystem. Where every time the same transaction updates the state in the same way.

# EVM is called turing complate

As per the latest book `mastering ethereum`, the EVM is called `quasi-Turing complete`. 

A system is turing complete if it can run any computation for example running infinite loopss that can never stop. Bitcoin is not turing complate, as we can only develop scripts on that, but the ethereum made a change by introducing smart contracts.

Here is the catch, ethereum is turing complate, so you can run loops in ethereum smart contracts, however even if it is turing complate you can not run the infinite loops. 
consider that if any infinite loop runs in ethereum then it will never stops, that means the halting problem is unsolvable in ethereum. that results in high resource consumption and ethereum being stuck.

That is why ethereum introduces the gas system. Gas is a measure of computational complexity and is used to limit the execution of smart contracts. Even if a user submits the transaction that consist of infinite loop, after a certain gas limit is reached, the transaction will be reverted. that means every action and opertion in ethereum is using gas. This is solving the halting problem in turing complate system. 

In EVM each insstruction cost gas as shown below

ADD -> 3 gas
SLOAD -> 100
etc.

if gas becoms zero, the execution stops and the transaction is reverted. the state is not updated. this is why the EVM called `quasi-Turing complete`.

# Architecture

The EVM using stack based architecture. all the instructions are executed on a stack. even solidity compiles to stack operations.

EVM uses a 256-bit numbers as word size. mainly to facilitate the hashing and elliptic curve operations.

EVM has several data components.

1. Code(ROM)
  - This is the contract bytecode. After deployment of the contract, the bytecode is stored. Which is immutable and stored permanently.
2. Memory
  - This is the temporary memory used for execution while running transactions.
  - after completing a transaction, the memory is cleared.
3. Transient storage
  - transient storage that lasts only for a single transaction
  - This is similar to temporary storage across calls in a transaction. this was introduced in `EIP-1153`
  - this is used in temporary locks, reentrancy guards.
4. Permanent storage
  - This is the contract's permanent storage. consider it as state variables in contract.

All the data components costs gas differently. for example memory is cheaper than storage. and storage is cheaper than code in a way.
This means accessing each data component cost different amount of gas.

