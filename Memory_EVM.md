# Understand the Memory - a component of EVM.

Memory is core part of the EVM. As we know while executing any transaction on the EVM. it initializes memory with a default value of 0. then perform certain operaions on it. as compare to storage accessing memory is cheaper and faster. because each transaction initializes it's own seprate memory, and after the transaction is completed the memory is deleted. This means the memory is volatile and temporary.

In EVM memory is byte addressable data structure. it is very long array of bytes. every bytes in memory is accessible by 32 byte key. Which means it can contain up to 2^256 bytes of data. as stated above, during initialization all the value in the memory is set to zero. 

# Memory layout

```
Address    Value
----------------
0x00       0x00
0x01       0x00
0x02       0x00
...
0x1F       0x00
0x20       0x00
```
each byte contain one byte as shown above. However there is mentioned that the key is 32 byte long. So we can access memory in following order. 

- if we store 5 at 0x00 then it will not store it as 0x00 -> 0x05. instead it stores it as:

```
Address   Value
----------------
0x00      00
0x01      00
0x02      00
...
0x1E      00
0x1F      05
```

Here the key will become 0x00 to 0x1F. 

While accessing memory we can read or write to any address. if we want to read some specific byte then we can not read that directly. instead we need to load full 32 bytes from memory and then extract the specific byte we want.

This is how the memory works in EVM.