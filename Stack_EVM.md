# Understand the Stack a component of EVM.

stack is very basic and simple data structure used in EVM to perform certain operations. Stack works on Last In First Out (LIFO) principal. that means the last element added to the stack is the first one to be removed.

# Stack operations

- push: add an element to the top of the stack.
- pop: remove the top element from the stack.
- peek: view the top element without removing it.
- isEmpty: check if the stack is empty.
- size: get the number of elements in the stack.
- swap: swap the top two elements of the stack.

in EVM stack has fixed size and fixed depth. we can store 32 bytyes element in stack. it can host up to 1024 items at a time. 

# Example:

```
push 1
push 2
swap
pop
```
if we run above operation on stack the stack will be look something like this

```
        [2]     [1]
[1]     [1]     [2]   [2]
push 1  push 2  swap  pop
```

