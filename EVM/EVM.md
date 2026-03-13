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

# The EVM Bytecode Operations

Now there is a question, if EVM is ececuting the bytecode of smart contracts, then what actually it is doing while executing? The answer is it is executing the bytecode instructions on the stack and put result onto the stack. if you see the bytecode, it is a sequence of opcodes and stack operations that the EVM executes.

here the EVM instruction set comes in the picture.

The EVM instruction set provides most of the operations such as:
- Arithmetic and bitwise logic operations
- Execution context inquiries
- Stack, memory, and storage access
- Control flow operations
- Logging, calling, and other operators

Alongside the bytecode operations, The EVM can access the account informations such as balance, code, and storage and block information.

To executes the opcodes, the EVM takes operands from the stack and performs the operation, then puts the result back onto the stack.

The available opcodes can be divided into the following categories:

## Arithmetic operations

- ADD -> Add the top two stack items
- MUL -> Multiply the top two stack items
- SUB -> Subtract the top two stack items
- DIV -> Integer division
- SDIV -> Signed integer division
- MOD -> Modulo operation
- SMOD -> Signed modulo operation
- ADDMOD -> Addition modulo any number
- MULMOD -> Multiplication modulo any number
- EXP -> Exponential operation
- SIGNEXTEND -> Extend the length of a two's complement signed integer
- SHA3 -> Compute the Keccak-256 hash of a block of memory

## Stack operations

- POP -> Remove the top item from the stack
- MLOAD -> Load a word from memory
- MSTORE -> Save a word to memory
- MSTORE8 -> Save a byte to memory
- SLOAD -> Load a word from storage
- SSTORE -> Save a word to storage
- TLOAD -> Load a word from transient storage
- TSTORE -> Save a word to transient storage
- MSIZE -> Get the size of the active memory in bytes
- PUSH0 -> Place value 0 on the stack
- PUSHx -> Place x byte item on the stack, where x can be any integer from 1 to 32 (full word) inclusive
- DUPx -> Duplicate the x-th stack item, where x can be any integer from 1 to 16 inclusive
- SWAPx -> Exchange 1st and (x+1)-th stack items, where x can be any integer from 1 to 16 inclusive

## Process-flow operations

- STOP -> Halt execution
- JUMP -> Set the program counter to any value
- JUMPI -> Conditionally alter the program counter
- PC -> Get the value of the program counter (prior to the increment corresponding to this instruction)
- JUMPDEST -> Mark a valid destination for jumps

## System operations

- LOGx -> Append a log record with x topics, where x is any integer from 0 to 4 inclusive
- CREATE -> Create a new account with associated code
- CALL -> Message-call into another account, i.e., run another account's code
- CALLCODE -> Message-call into this account with another account's code
- RETURN -> Halt execution and return output data
- DELEGATECALL -> Message-call into this account with an alternative account's code, but persisting the current values for sender and value
- STATICCALL -> Static message-call into an account, i.e., it cannot change the state of any account
- REVERT -> Halt execution, reverting state changes but returning data and remaining gas
- INVALID -> The designated invalid instruction
- SELFDESTRUCT -> Halt execution and, if executed in the same transaction a contract was created, register account for deletion. Note that its usage is highly discouraged  and the opcode is considered deprecated

## Logic operations

- LT -> Less-than comparison
- GT -> Greater-than comparison
- SLT -> Signed less-than comparison
- SGT -> Signed greater-than comparison
- EQ -> Equality comparison
- ISZERO -> Simple NOT operator
- AND -> Bitwise AND operation
- OR -> Bitwise OR operation
- XOR -> Bitwise XOR operation
- NOT -> Bitwise NOT operation
- BYTE -> Retrieve a single byte from a full-width 256-bit word

## Environmental operations

- GAS -> Get the amount of available gas (after the reduction for this instruction)
- ADDRESS -> Get the address of the currently executing account
- BALANCE -> Get the account balance of any given account
- ORIGIN -> Get the address of the EOA that initiated this EVM execution
- CALLER -> Get the address of the caller immediately responsible for this execution
- CALLVALUE -> Get the ether amount deposited by the caller responsible for this execution
- CALLDATALOAD -> Get the input data sent by the caller responsible for this execution
- CALLDATASIZE -> Get the size of the input data
- CALLDATACOPY -> Copy the input data to memory
- CODESIZE -> Get the size of code running in the current environment
- CODECOPY -> Copy the code running in the current environment to memory
- GASPRICE -> Get the gas price specified by the originating transaction
- EXTCODESIZE -> Get the size of an account's code
- EXTCODECOPY -> Copy an account's code to memory
- RETURNDATASIZE -> Get the size of the output data from the previous call
- RETURNDATACOPY -> Copy data output from the previous call to memory

## Block operations

- BLOCKHASH -> Get the hash of one of the 256 most recently completed blocks
- COINBASE -> Get the block's beneficiary address for the block reward
- TIMESTAMP -> Get the block's timestamp
- NUMBER -> Get the block's number
- PREVRANDAO -> Get the previous block's RANDAO mix. This opcode replaces the DIFFICULTY one since The Merge hard fork.
- GASLIMIT -> Get the block's gas limit


# Ethereum as state machine

Ethereum is transaction based state machine. 

it works like following:

> current_state + transaction → new_state

Every transaction changes the global state. as EVM computes the all the transacations. the job of the EVM is as following:

> take the current state → execute transaction → produce the next state

Ethereum maintains a word state. Think of it as a database that stores the account informations.
The word state is mapping of address to the account state.

> account address -> account state

in ethereum each account has address of 160 bits (20 bytes). this account address can be EOA or smart contract address. Each account address has a corresponding account state.

Here is an minimal info about what each account state contains:

```
AccountState [
    nonce -> this depends on account type (EOA or contract)
          -> for EOA, it means the number of transactions sent
          -> for contract account it means the number of contract created
    balance -> the amount of ETH owned in wei
    code -> Only contract account normally has code
    storage -> only the smart contract account has storage
            -> this is permement
            -> stored on-chain
            -> expensive
]
```

> EOA does not have code and storage, that is why those fields will be empty for EOA accounts. However after the pectra fork this has been changed. by introducing EIP7702. in this EIP the EOA can now behave as smart contract account ny deligate execution to a contract.

## How this works?

now in account state, instead of code being epmty.

> code -> delegation marker

delegation marker has specific formate such as `0xef0100 || contract_address`, The EOA will delegate execution to the `contract_address`.

> Here the `0xef` is used because it is not allowed in contract bytecode according to EIP3541. so this marker can not be confused with real code.

### What happens when an EOA delegates execution?

When someone calls the EOA:
- EVM sees delegation marker
- It loads code from target contract
- It executes it in the EOA context

meaning: 

```
msg.sender = EOA
storage = EOA storage
balance = EOA balance
```
but

```
code = contract code
```
This is similar to delegatecall semantics.

Here two opcodes comes in picture. 

1. EXTCODESIZE(eoa) -> This will return 23 bytes (the delegation marker)
2. CODESIZE -> This will return the size of delegated contract

Now there is a risk related to storage.

delegated EOA can use the storage. if we change the delegated contract the storage layout could break. So the recommended standard uses namespace storage which prevent collision. which is `EIP7201`

### How EVM starts the execution?

when transaction call a contract. the EVM instance is created. After that, things happens in order as following. 

- Load contract bytecode -> `ROM <- bytecode`
- initialize the program counter -> `PC = 0`
- load storage -> storage = `contract storage`
- initialize memory -> `memory = 0`
- loads environment variables -> `msg.sender, msg.value, block.number, block.timestamp`.
- set gas supply -> usually gas comes from transaction gas limit.

Now what happens when gas runs out?

if has runs out, the execution stops immediately. the state changes do not apply.

But even if state has not been changed, following things will be changed.

- sender nonce
- sender pays gas fees

Now this all happens in the sendbox model.

### EVM executing in sendbox model

every time EVM executes transaction, it executes in sendbox copy of state.

Think of it as:

```
real world state -> sandbox state copy -> execute contract -> success ? commit : discard
```

so if execution fails sendbox will be destoyed. during execution all the state changes has been done in sendbox copy of state. After finalizing the tx and state update the sendbox will be copied in the actual state. This is how EVM works internally.

Now there is a catch. What if there are recursive calls in one tx? means some smart contract calls itself or another smart contract. 

```
User -> Uniswap Router -> Uniswap Pool -> ERC20 Token
```

in this case, each call creates new EVM instance. each instance has it's own stack, memory and gas allocation. However we have to forward gas between contracts. Suppose contract A calls contract B. in that can we have to check `gas_b <= gas_remainint_a`. if the inner call fails then called contract state changes will be discarded and caller contract has to handle the error or revert.

