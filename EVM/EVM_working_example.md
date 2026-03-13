# Understand how EVM works internally with all the components while executing a bytecode.

In this article, i will explain how EVM works with all the components, such as memory, storage and stack while executing a bytecode. i will take here example of small bytecode and explain and undertsand the working of EVM. 

Suppose we have a simple bytecode:

`60425F525F3560AB145F515500`.

Along side the bytecode, we have following calldata as input:

`00000000000000000000000000000000000000000000000000000000000000ab`

Here is a catch, in EVM each opcode is identified by single byte ranging from `0x00` to `0xFF`. for example 0x60 is PUSH1 opcode. 0x5F the PUSH0, and so on. for all opcodes refer to the [EVM opcode list](https://www.evm.codes/).

now according to the bytecode, if we convert it to human readable form we get:

`PUSH1 0x42 PUSH0 MSTORE PUSH0 CALLDATALOAD PUSH1 0xAB EQ PUSH0 MLOAD SSTORE STOP`

This is the human readable form of the bytecode, where each opcode is replaced with its corresponding mnemonic.

let's see how EVM executes this bytecode one by one.

**Note:** assume initial memory/storage and stack are empty.

- PUSH1 0x42 - pushes the value 0x42 onto the stack.


```

[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x42]
stack
```

- PUSH0 - pushes the value 0 onto the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x42, 0]
stack
```

- MSTORE - pops top two values from the stack, interpreting the first one the offset and the second one the value to store in the memory.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                 memory
```

- PUSH0 - pushes the value 0 onto the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x00]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                     memory
```

- CALLDATALOAD - takes top element from the stack, consider it as offset and returns the 32-byte value in the calldata starting at that offset, then pushes it onto the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0xab]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                     memory
```

- PUSH1 0xAB - pushes the value 0xAB onto the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0xab, 0xab]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory
```

- EQ - compares the top two values on the stack and pushes 1 if they are equal, 0 otherwise.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x01]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory
```

- PUSH0 - pushes the value 0 onto the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x01, 0x00]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory
```

- MLOAD - loads the value from memory at the address specified by the top of the stack.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[0x01, 0x42]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory
```

- SSTORE - pops two items from the stack, considering the first item as slot number in storage and the second item as the value to store.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory

[[][],[][],..., [0x0000000000000000000000000000000000000000000000000000000000000042][0x0000000000000000000000000000000000000000000000000000000000000001](slot 0x43)]
  storage
```

- STOP - halts the execution.

```
[00000000000000000000000000000000000000000000000000000000000000ab]
                            calldata
                            
[]            [[0x0000000000000000000000000000000000000000000000000000000000000042](0x00 - 0x1F), [], []]
stack                         memory

[[][],[][],..., [0x0000000000000000000000000000000000000000000000000000000000000042][0x0000000000000000000000000000000000000000000000000000000000000001](slot 0x43)]
  storage
```

So this is the full operation of the EVM for the given example.
