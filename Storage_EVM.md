# Understand the Storage - a component of EVM.

Storage is another core part of the Ethereum. Every contract has its own storage space. Where the data stored. However we know that storage is stored on-chain permanently. unlike memory resets every transaction, the storage only modifies the state of the contract and store modified data on-chain. 

As we know the storage saved on-chain, to access and update on-chain storage data is more expensive then any temporary memory. However if you see while writing view functions it does not costs any gas on-chain even if you access the storage. that is another concept. 

# Example

```solidity
contract Storage {
    mapping(address => uint256) public balance;
    uint256 data;
}
```

above is the example of how contract storage looks like. 

# Layout od storage

In EVM storage is key-value pair same as memory, but the difference here is both key and value is 32 bytes each. contracts only can modify their storage. however the storage is persistent accross the different transactions and blocks. as discussed above it just updates from one state to another. the is a catch in storage. If you try to access such storage where no value has been stored or the value has not been touched yet, accessing it will return a default value of corresponding data type. 

## Example

Suppose every storage location is untouched.

```solidity
contract Storage {
    uint256 data1; // default value 0
    bool data2; // default value false
    bytes32 data3; // default value 0
}
```

As shown in above example accessing the untouched storage variable/locations will return default values. However it is not limited there. as the storage is form of key value each of 32 bytes. we can access any storage location by its key. 

in above example the `data1` is 256 bits(32 bytes) and `data2` is 8 bits(1 byte). So the `data1` will accuire full 32 bytes key space and `data2` will accuire only 1 byte key space. in storage we can call those key space as slots.

so from above example `data1` is stored in slot 0, `data2` is stored in slot 1. Now if you see after `data2` there is `data3` which data type is bytes32. So `data3` will accuire full 32 bytes key space and will be stored in slot 2. Why not in slot 1? Becasue `data2` has accuire 1 bytes and remaining bytes are 31, and 32 bytes can not fit in 31 bytes. That's why we have to accuire next available slot. 

Now what if we want to access slot 3? yes that is possible in SVM. even if there is no variable defined in that slot, we can still access it directly by its slot number. and the default value of an empty slot is 0.

There are total `2^256` slots in the storage of any contract. and those solts initializes to 0 by default after deployment of the contract.

# Transient Storage

Another storage type is transient storage. which is same as storage but it is not persisted across transactions. after a transaction is executed all transient storage values are reset to their default values or we can say it will discarded. This storage has been proposed in `EIP1153`.

This storage can be used as reentrency protection, flashloan protection etc. 
