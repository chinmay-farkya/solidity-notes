# Solidity Notes
# Solidity Notes

This is a collection of the interesting details of Solidity which need to be remembered. This corpus has been accumulated by me during my study of Solidity over the past few months. I have named it Solidity 451.
Some syntax highlighting ⇒
When you see this anywhere in the text:

1. OMG = Validity has to be checked
2. WTF = To reason why
3. IDK = learn more about this

Feel free to add or correct me on anything. Please spare the numbering and formatting, I moved it from my Notion page.

1. Sending ether to a contract B from a contract(for eg. using .send or .transfer), executes the code of B’s fallback function if it exists(and receive doesn’t exist). 

![Untitled](https://user-images.githubusercontent.com/93861625/234314831-00fe6e9c-8330-4ed1-944d-c729dd074a70.png)

1. Division result is auto rounded towards zero
2. Division by zero causes panic error, escapes unchecked
3. Always check the return value of send. Send fails if call stack depth is at 1024(can be forced by the caller)
4. Using low level functions to call a contract = handing over control to it
5. Before using delegatecall, ensure that storage layout is in same order in both contracts
6. enum types are not part of the ABI, they are just a solidity abstraction
7. Delete keyword is simply a reassignment of elements to their default values (ie.zero)
8. .call bypasses function existence check, type checking and argument packing
9. The evm considers a call to non-existing contract to always succeed, so there is a check of extcodesize > 0 when making an external call. But call, staticcall, delegatecall, send, transfer do not include this check
10. On receiving funds from a selfdestruct / coinbase miner reward, the contract can not react to it, and it doesn’t require a contract to have receive or fallback functions.
11. extcodesize > 0 check is skipped by the compiler if the function call expects return data. the ABI decoder will catch the case of a non-existing contract Because such calls are followed up by abi decoding the return data, which has a check for `returndatasize` is being at least a non-zero number. So for empty contracts, they would always revert in the end.
12. Always send 1 wei to precompiled contracts to activate them when testing in private blockchains, otherwise it may lead to OOG
13. Functions called from within an `unchecked` block do not inherit the property. Bitwise operators do not perform overflow or underflow checks
14. After a failed call, Do not assume that the error message is coming directly from the called contract: The error might have happened deeper down in the call chain and the called contract just forwarded it (bubbling up of errors)
15. Calling a function on a different contract (instance) will perform an EVM function call and thus switch the context such that state variables in the calling contract are inaccessible during that call (except if you use delegatecall)
16. After contract creation, The deployed code does not include the constructor code or internal functions only called from the constructor
17. Dont use this.f inside constructor
18. Internal is the default visibility level for state variables.
19. Internal function calls do not create an EVM message call. They are called using simple jump statements. Same for functions of inherited contracts. 
20. If you have a `public` state variable of array type, then you can only retrieve single elements of the array via the generated getter function
21. Without a payable keyword in function declaration, it will auto reject all ether sent to it. It will revert. 
22. Modifiers can also be defined in libraries but their use is limited to functions of the same library
23. Multiple modifiers are applied to a function by specifying them in a whitespace-separated list and are evaluated in the order presented. **Modifier Order Matters**
24. The `_` symbol can appear in the modifier multiple times. Each occurrence is replaced with the function body. Symbols introduced in the modifier are not visible in the function
25. For values of immutable variables, 32 bytes are reserved in the code, even if they would fit in fewer bytes
26. The compiler does not reserve a storage slot for constant and immutable variables, and every occurrence is replaced by the respective value directly in the code. 
27. The code of free functions is included in all contracts that call them, similar to internal library functions (free functions are those that exist at file level, outside of a contract. 
28. Functions defined outside a contract are still always executed in the context of a contract. They still can call other contracts, send them Ether and destroy the contract that called them
29. the opcode `STATICCALL` is used when `view` functions are called, For library `view`
 functions `DELEGATECALL` is used(if interacting with already deployed library). This means library `view` functions do not have run-time checks that prevent state modifications ?? Omg
30. For pure functions, the opcode `STATICCALL` is used, which does not guarantee that the state is not read, but at least that it is not modified. It is not possible to prevent functions from reading the state at the level of the EVM.
31. The data returned from fallback function will not be ABI-encoded. Instead it will be returned without modifications (not even padding) WTF
32. Return parameters are not taken into account for overload resolution for function calls to overloaded functions, only function arguments are matched. (Overload resolution is nothing in practice because function dispatcher will match the selectors) 
33. Events are inheritable. The Log and its event data is not accessible from within contracts (not even from the contract that created them.
34. it is possible to “fake” the signature (topic0) of another event using an anonymous event
35. Errors are inheritable. the revert data of inner calls is propagated back through the chain of external calls by default. Low-level calls do not throw an exception. 
36. When a contract inherits from other contracts, only a single contract is created on the blockchain, and the code from all the base contracts is compiled into the created contract. This means that all internal calls to functions of base contracts also just use internal function calls (`super.f(..)` will use JUMP and not a message call
37. The overriding function may only change the visibility of the overridden function from `external` to `public` .The mutability may be changed to a more strict one following the order: `nonpayable`can be overridden by `view`and `pure`. `view`can be overridden by `pure`. `payable`is an exception and cannot be changed to any other mutability
38. Functions without implementation(that need to be overriden somewhere) have to be marked `virtual` outside of interfaces. In interfaces, all functions are automatically considered `virtual`
39. Public state variables can override external functions if the parameter and return types of the function matches the getter function of the variable. They themselves cannot be overriden
40. Before the constructor code is executed, state variables are initialized to their specified value if you initialize them inline, or their [default value](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#default-value) if you do not.
41. The order in which the base classes are given in the `is` directive is important: You have to list the direct base contracts in the order from “most base-like” to “most derived”
42. Interfaces cannot have any functions implemented, can only inherit from other interfaces, all declared functions should be external, can be inherited by contracts
43.  Libraries are deployed only once at a specific address and their code is reused using the `DELEGATECALL` feature of the EVM. This means that if library functions are called, their code is executed in the context of the calling contract, i.e. `this` points to the calling contract, and especially the storage from the calling contract can be accessed
44. Library functions can only be called directly (i.e. without the use of `DELEGATECALL`
) if they do not modify the state (view or pure) Omg
45. Libraries are stateless and undestroyable. Cannot be inherited and cant receive ether
46. In the EVM, the code of internal library functions that are called from a contract and all functions called from therein will at compile time be included in the calling contract, and a regular `JUMP` call will be used instead of a `DELEGATECALL`. This is case of using library in the same file instead of interacting with already deployed library) Omg
47. Calling a public library function with `L.f()`results in an external call (`DELEGATECALL`
 to be precise). `msg.sender`, `msg.value`and `this` will retain their values in this call
48. It is possible to obtain the address of a library by converting the library type to the `address`
 type, i.e. using `address(LibraryName)`
49. As the compiler does not know the address where the library will be deployed, the compiled hex code will contain placeholders of the form `__$30bbc0abd4d6364515865950d3e0d10953$__`
. The placeholder is a 34 character prefix of the hex encoding of the keccak256 hash of the fully qualified library name, which would be for example `libraries/bigint.sol:BigInt`
 if the library was stored in a file called `bigint.sol` in a `libraries/` directory. Such bytecode is incomplete and should not be deployed. Placeholders need to be replaced with actual addresses.
50. For computing function selector of libraries, the storage pointers in argument encoding is encoded as a `uint256` value referring to the storage slot to which they point
51. The actual code stored on chain for a library is different from the code reported by the compiler as `deployedBytecode`
52. In using A for B, all public + internal functions of library A are attached to the type of B
53.  Inline assembly bypasses several important safety features and checks of Solidity
54. If you access variables of a type that spans less than 256 bits (for example `uint64`, `address` or `bytes16` ), you cannot make any assumptions about bits not part of the encoding of this type. Especially, do not assume them to be zero (dirty higher order bits )
55. Most arithmetic operations ignore the fact that types can be shorter than 256 bits, and the higher-order bits are cleaned when necessary
56. During memory allocation, when you point at a memory location, There is no guarantee that the memory has not been used before and thus you cannot assume that its contents are zero bytes. There is no built-in mechanism to release or free allocated memory
57. If you annotate an assembly block as memory-safe, but violate one of the memory assumptions, this **will** lead to incorrect and undefined behavior that cannot easily be discovered by testing IDK
58. Packed encoding of abi can be ambiguous
59. ecrecover returns zero on error IDK what error ? (In this case it doesn’t actually return zero but the precompile never gets back with a new value so you are left with the default zero address value when you declared the address variable. ie. when we write address a = ecrecover(bla bla), it never gets assigned on erorr and thus remains zero.
60. Structs and array data always start a new slot, and items following these also start from a new storage slot
61. Due to their unpredictable size, mappings and dynamically-sized array types cannot be stored “in between” the state variables preceding and following them. Instead, they are considered to occupy only 32 bytes .The elements they contain are stored starting at a different storage slot that is computed using a Keccak-256 hash. For dynamic arrays, this slot stores the number of elements in the array, For mappings, the slot stays empty. 
62. If the storage location of the dynamic array ends up being a slot p after applying the storage layout rules, this slot stores the number of elements in the array (byte arrays and strings are an exception). Array data is located starting at keccak256(p) and it is laid out in the same way as statically-sized array data would: One element after the other, potentially sharing storage slots if the elements are not longer than 16 bytes
63. For bytes and string, it is stored together with the length in the same slot. characters are stored in leftmost bytes of the slot and 2 * length is stored at rightmost byte. If the string is greater than 31 bytes,The string is split into 32-byte long chunks and put starting in the slot index calculated by `keccak256(stringDeclarationSlotIndex)` (similar to arrays), and only the length of the string is saved in the string declaration slot index. Then just add 1 to the hash to find remaining data of next 32-byte chunks. 
64. For short type dynamic arrays, is length byte reserved like bytes and string ? IDK
65. This means that you can distinguish a short array from a long array by checking if the lowest bit is set: short (not set) and long (set). because IDK the bit math here
66. Elements in memory arrays in Solidity always occupy multiples of 32 bytes (this is even true for `bytes1[]` , but not for `bytes` and `string`
67. One should not expect the free memory to point to zeroed out memory
68. The input data for a function call is assumed to be in the format defined by the [ABI specification](https://docs.soliditylang.org/en/v0.8.17/abi-spec.html#abi). Among others, the ABI specification requires arguments to be padded to multiples of 32 bytes (ABI is just a standardized format for uniform interaction)
69. Arguments for the constructor of a contract are directly appended at the end of the contract’s creation code, also in ABI encoding. The constructor will access them through a hard-coded offset, and not by using the `codesize` opcode, since this of course changes when appending data to the code
70. before writing a value to memory, the remaining bits of this value need to be cleared because the memory contents can be used for computing hashes or sent as the data of a message call. Also before writing to storage..   (when a value is shorter than 256 bit, the remaining should be cleaned). The solidity compiler does this for certain operations
71. If you use inline assembly to access Solidity variables shorter than 256 bits, the compiler does not guarantee that the value is properly cleaned up. Moreover, we do not clean the bits if the immediately following operation is not affected
72. the Solidity compiler cleans input data when it is loaded onto the stack WTF
73. The number of runs (`--optimize-runs` ) specifies roughly how often each opcode of the deployed code will be executed across the life-time of the contract. a larger “runs” parameter will produce longer but more gas efficient code. (max 2**32 - 1) and vice versa
74. The compiler appends by default the IPFS/swarm hash of the metadata file to the end of the runtime bytecode
75. Since the bytecode of the resulting contract contains the metadata hash by default, any change to the metadata might result in a change of the bytecode. This includes changes to a filename or path, and since the metadata includes a hash of all the sources used, a single whitespace change results in different metadata, and different bytecode
76. do not rely on this metadata hash sequence to start with `0xa2 0x64 'i' 'p' 'f' 's`
77. For the ABI spec, We assume that the interface functions of a contract are strongly typed, known at compilation time and static. We assume that all contracts will have the interface definitions of any contracts they call available at compile-time
78. In function signature, parameter types are listed with a comma, without whitespace
79. Address payable, contract, enum and struct are not supported by ABI, so they are represented by their respective equivalents (struct as tuple)
80. `len(a)` is the number of bytes in a binary string `a`. The type of `len(a)`i s assumed to be `uint256` IDK bytes.concat method
81. In Function argument encoding, uints are padded with zero bytes upto 32 bytes (on the left). bool is encoded as a uint8. Similarly, return types
82. Events, error returns all are encoded as functions in ABI encoding
83. Never trust error data. The error data by default bubbles up through the chain of external calls, which means that a contract may receive an error not defined in any of the contracts it calls directly. Furthermore, any contract can fake any error by returning data that matches an error signature, even if the error is not defined anywhere
84. Through `abi.encodePacked()`, Solidity supports a non-standard packed mode where:
- types shorter than 32 bytes are concatenated directly, without padding or sign extension
- dynamic types are encoded in-place and without the length.
- array elements are padded, but still encoded in-place

Furthermore, structs as well as nested arrays are not supported

1. 
- During the encoding, everything is encoded in-place. This means that there is no distinction between head and tail, as in the ABI encoding, and the length of an array is not encoded.
- The direct arguments of `abi.encodePacked` are encoded without padding, as long as they are not arrays (or `string` or `bytes`).
- The encoding of an array is the concatenation of the encoding of its elements **with** padding.
- Dynamically-sized types like `string`, `bytes` or `uint[]` are encoded without their length field.
- The encoding of `string` or `bytes` does not apply padding at the end, unless it is part of an array or struct (then it is padded to a multiple of 32 bytes).
1. Since packed encoding is not used when calling functions, there is no special support for prepending a function selector. Since the encoding is ambiguous, there is no decoding function. In general, the encoding is ambiguous as soon as there are two dynamically-sized elements, because of the missing length field.
2. If you use `abi.encodePacked` for signatures, authentication or data integrity, make sure to always use the same types and check that at most one of them is dynamic. Unless there is a compelling reason, `abi.encode` should be preferred
3. All through encoding, whenever padded, `bytesNN`
 types are padded on the right while `uintNN`/ `intNN` are padded on the left
4. `using A for B`
 only affects the contract it is mentioned in. It is not inherited
5. If a byte array in storage is accessed whose length is encoded incorrectly, a panic is caused. A contract cannot get into this situation unless inline assembly is used to modify the raw representation of storage byte arrays
6. Explicit conversions between literals and `address` type (e.g. `address(literal)`) have the type `address` instead of `address payable`. The global variables `tx.origin`and `msg.sender` have the type `address` instead of `address payable`
7. `payable(0)` is valid and is an exception to the explicit conversion rules
8. `enum` definitions cannot contain more than 256 members. This will make it safe to assume that the underlying type in the ABI is always `uint8`
9. Negate unsigned integers by subtracting them from the maximum value of the type and adding 1 (e.g. `type(uint256).max - x + 1`, while ensuring that x is not zero
10. The Solidity compiler only interprets Natspec tags if they are on external or public functions
11. As a special case, if no tags are used then the Solidity compiler will interpret a `///`
 or `/**` comment in the same way as if it were tagged with `@notice`
 (describes to the end user what this does)
12. 

![](https://user-images.githubusercontent.com/93861625/234341434-9ddc10dc-df06-45de-b444-5d8346c9b537.png)

1. The following are considered reading from state: 1) Reading from state variables 2) Accessing address(this).balance or <address>.balance 3) Accessing any of the members of block, tx, msg (with the exception of msg.sig and msg.data) 4) Calling any function not marked pure 5) Using inline assembly that contains certain opcodes
2. For ERC-20 tokens that support it, `[permit` is an alternative to the standard `approve` call](https://forum.openzeppelin.com/t/why-is-erc20permit-better-than-approve-transferfrom/7478)
: it allows an off-chain secure signature to be used to register an allowance. The permitter is approving the beneficiary to spend their money, by signing the permit request. The `permit`
 approach has several advantages: there is no need for a separate transaction (spending gas) to approve a spender, allowances have a deadline, transfers can be batched, and more
1. EVM halting exceptions are errors in the stack machine.(subset of runtime errors, but not directly part of contract interaction) These include out of gas, wrong opcode, stack underflow/overflow, invalid jump destinations etc.(when jumpdest is not present at jump location), or state modification by static call.
2. three type of errors - compile time(during compilation to bytecode), runtime(reverts) and EVM halting exceptions (related directly to how the evm processes instructions)
3. Arithmetic operations can be performed using two different time units, because EVM only works with seconds under the hood.
4. Variables(and function parameters) cannot be appended directly with time units. Ex variable days is not applicable. but, variable * 1 days is an alternative syntax that can be used
5. Read operator precedence cheatsheet : [https://docs.soliditylang.org/en/v0.8.18/cheatsheet.html#order-of-precedence-of-operators](https://docs.soliditylang.org/en/v0.8.18/cheatsheet.html#order-of-precedence-of-operators)
6. The base unit for ether units is wei. so when you write 1 ether it gets converted to 10^18 under the hood. This value after converting is used in any expressions
7. In EVM denominations (hex encoding), 4 bits = 1 character, so 1 byte contains 2 characters 
8.  writing a variable with ether suffix is not applicable. “total ether” is not valid.  total * 1 ether can be used.
9. When using send, transfer or call for native token transfers, the value parameter’s default unit is wei, unless otherwise specified.
10. The value of a constant can be read in constructor, while immutable can’t be read even if it has been assigned in the very same constructor. Also immutable cannot be overriden if it has been assigned already at declaration.
11. If a `constant`i s simply defined in the contract but not used anywhere, its value will not be stored anywhere in the contract code (= bytecode). `constant` are simply inline wherever they are read from and used in the contract logic and its functions. Constant has to be initialized at the point of declaration
12. `constant` is never padded into the contract bytecode. On the contrary, for `immutable`, a full 32-byte word is reserved in the contract bytecode
13. *Prior to Solidity 0.8.9, padding worked a bit differently in code; in code, all types were zero-padded, even if they would ordinarily be sign-padded. This did not affect which side they are padded on. IDK padding properly*
14. constant cannot be defined with external visibility.
15. types allowed to be declared : 

![types allowed to be](https://user-images.githubusercontent.com/93861625/234342188-841a5e19-fe4b-475d-b072-eb0e45ad4c53.png)

1. The `constant` variable can be defined standalone inside a `.sol`file outside of a `contract { … }` block (file-level)
2. `storage` consists of a key-value store that maps 256bits words to 256 bits words
3. The EVM `memory` is used to hold temporary values and is erased between external function calls. However, it is cheaper to use. It is specific to the context of a specific contract (environment). Meaning the whiteboard/scratchpad is cleared when the execution context change from one contract to another (when a new contract gets on top of call stack)
4. The calldata behaves mostly like memory and is a byte-addressable space. You have to specify an exact byte offset for the number of bytes you want to read.
5. The stack is where most of the local variables created inside a function reside
6. The bytecode contains a lot of information and logic about the contract, including the dispatcher, and the contract metadata
7. For arrays (fixed or dynamic sized arrays, like `uint256[]`), `bytes`, `string`, struct and mappings, you have to explicitly provide the data area where the value is stored. This can be `storage`, `memory`or `calldata`
8. 

![For arrays (fixed](https://user-images.githubusercontent.com/93861625/234342239-4441ba55-7ecd-43fe-aa75-7aa7a272789d.png)

1. When `storage` is used as a reference for a function parameter, it is a pointer to the contract storage (that is why it is only allowed with internal, private calls because then these references can be passed as arguments to call such functions)
2. For memory = we can always copy any data in memory (whether it comes from the contract’s storage or the calldata). For storage and calldata = we can only assign values coming from the data location specified (either directly or via a reference of the same type.
3. a collection of storage and memory pointers explanations : [https://github.com/CJ42/All-About-Solidity/blob/master/articles/data-locations/DataLocationsReferences.sol](https://github.com/CJ42/All-About-Solidity/blob/master/articles/data-locations/DataLocationsReferences.sol) : If you want to create a storage/calldata/memory pointer to a storage/calldata pointer, then it has to point to an initialized storage slot first. (Uninitialized references work from memory to memory) storage can only point to storage and calldata can only point to calldata
4. When a `storage` variable is created inside a function, this basically acts as a storage
pointer. The storage pointer simply references data already allocated in storage. You can reassign the storage pointer to point it somewhere else in the storage. = references some existing value in storage = no new storage is created. However, we can override the contract storage by assigning a new value directly to this lookup/pointer variable
5. When we assign a `storage` referenced data to a `memory` referenced variable, we are copying data from storage → memory. since we have copied from storage to memory, we are operating on a copy of the data, not on the actual data that reside in storage. Therefore, 
any modification made to this memory variable will not propagate back to the contract storage and will not modify the contract state.
6. No new storage or calldata can be created from inside a function. Any simple data type variables declared are actually stored on the stack. 
7. mappings inside function bodies can only be assigned the data location `storage`to act as a reference to a mapping that already exists inside the contract storage.
8. Gas usage of storage slots :
- initializing a storage slot (for the first time, or if the slot does not contain any value) from a zero to a non-zero value costs 20,000 gas
- editing the value at a storage slot costs 5,000 gas
- deleting the value at a storage slot give a refund of 15,000 gas
1. The storage of a smart contract is free to read externally (from an EOA or eth node). An EOA just uses a rpc url to access an eth node that exists physically somewhere and stores all storage etc. data. In such a case, no gas has to be paid. However, gas has to be paid if the read operation is part of a transaction that modifies the state on the contract, another contract, or on the blockchain.
2. the storage of a smart contract is a word-addressable space. This is opposite to memory or call data which are linear data locations (growing bytes arrays), where you access data through offsets (indexes in the bytes array).
3. A smart contract’s storage consists of 2²⁵⁶ slots (staring at position 0), where each slot can contain values of size up to 32 bytes. Under the hood, the contract storage is a key-value store, where 256 bits keys map to 256 bits values
4. the layout of the contract’s storage is set in stone at the time of contract creation. The layout is “shaped” based on your contract-level variable declarations, and such layout cannot be changed by future method calls.
5. In storage slot packing, the first item in a storage slot is stored starting filling from lower-order (in lower bits) and according to the order of declaration
6. The layout of contract storage is also based on inheritance. If a contract inherits from other contracts, its storage layout follows the inheritance order.
- state variables defined in the most base contract start at slot 0.
- state variables defined in the following derived contract are placed in sub-sequential slots (slot 1, 2, 3, etc…).
1. If possible through the inheritance, state variables from different parent and child contracts do share the same storage slot.
2. The `SLOAD` opcode is available in inline assembly. It can be used to easily retrieve the whole word value stored at a specific storage slot
3. When`storage` is specified in a function parameter, this mean that the argument passed to the function must be a state variable( or storage reference/pointer)
4. When a local variable is assigned the value of the state variable(in a function body), Since this local variable is of elementary type (a `uint`), the value is copied/cloned from the contract storage to the local variable (on the stack). Any changes made to the local variable do not propagate to the contract storage. however if it is a complex type (struct/array/mapping), we would need to define storage/memory keyword with this local variable declaration.
5. In inline assembly, the slot and byte offset of a variable can be read using a.slot and a.offset, but you cannot assign to these values. The offset value retruned is bytes from right to left (ie. counting from lower bytes to higher bytes) because it starts filling in from the right when we declare.
6. Struct/array/mapping always occupy a full storage slot and thus .offset will always return zero IDK bytes and string too ?
7. When the EVM encounters data larger than 32 bytes (complex types like `string`, `bytes`, `struct` or arrays), it cannot process them on the stack because these items are too large. Therefore, the EVM needs to take this data and process it elsewhere. It has a dedicated place for this: the memory. By putting such variables in memory, the EVM can then deliver them to the stack in smaller chunks, one after the other.
8. memory is also used for complex operations built-in Solidity, like abi-encoding, abi-decoding or hashing functions via keccak256. For these specific cases, imagine that the memory acts as a scratchpad or a whiteboard for the EVM.(0x00 to 0x40)
9. All the bytes in memory are initially 0. The memory is not erased and cleared per say. Each new instance of the EVM memory is specific to an execution context, the current contract execution. any external call switches the execution environment to the target contract and provides a fresh instance of evm memory.
10. Memory is a linear byte array. When you interact with the EVM memory, you read from or write to (what I call) *“memory blocks”* that are 32 bytes long.
11. 

![Memory is a lin](https://user-images.githubusercontent.com/93861625/234342329-6efe800a-51c6-4290-a07e-a212b84b6ecc.png)

1. The max amount of data(no. of bytes) you can put in memory is the maximum value of an `uint64` number (2^64). If the offset specified is more than that, it will revert.
2. You can specify `memory` only inside function, not outside functions at the contract level.
3. The following data and values are always in memory by default:
- Function arguments of complex types.
- Local variables (inside function bodies) of complex types(struct, array, bytes, string etc.)
- Values returned from functions, regardless of their type (this is done via the **return** opcode).
- Any complex value type returned by a function must specify the keyword `memory`
1. reads are limited to a width of 256 bits, while writes can be either 8 bits or 256 bits wide.
2. The `MSIZE` opcode returns the highest byte offset accessed in memory in the current execution environment. The size will always be a multiple of word (32 bytes). what msize tracks is the highest offset ever accessed in the current execution. A first write or read to a bigger offset will trigger a memory expansion.
3. If you write to memory in assembly block via `mstore`or through similar opcodes that write to memory like `calldatacopy`, the free memory pointer is not updated automatically. You are responsible to do it manually yourself. In normal solidity writing like bytes memory a = hex’abcd’ updates the free memory pointer.
4. the free-memory pointer as *“a reference to the first unused word in memory”*. It enables to know where in memory (at which offset) there is free space to write data to. This is to avoid overriding data already present in memory.
5. You must ensure to always fetch the free memory first in assembly, and write to the location in memory pointed by the free memory pointer, if you don’t want to end up overwriting something in memory that already had some content. Once writing in memory, you must ensure that you update the free memory pointer with a new free memory offset
6. when calldata data is used to call function that has defined memory reference parameter, this data will first get copied to memory(using calldatacopy) and then supplied to function body(work of function wrapper)

![when calldata data](https://user-images.githubusercontent.com/93861625/234342657-4043885e-26a8-4ec7-9d73-a372d9d9ca34.png)

1. So when you see a variable inside a Solidity function with the keyword `memory`, you are dealing with a reference to a location in memory. Local variables that refer to memory evaluate to the address of the variable in memory not the value itself. Such variables can also be assigned to, but note that an assignment will only change the pointer and not the data :: not clear about this statement but when you edit a memory reference it gets propagated back to the data stored just like storage pointers do 
2. 

![data stored just like storage pointers](https://user-images.githubusercontent.com/93861625/234342707-f462cd6a-b362-4f77-9abf-56174e552bd1.png)

1. simply writing uint[] memory data; doesn’t allocate memory. it has to be initialized with a value. The value is then stored in the memory and variable points to its allocated location. or use new keyword(only fixed size arrays) to allocate memory without initializing immediately.
2. when we write bytes data; as a state variable, it is just a storage reference to the storage slot allocated.
3. Memory is expanded by a word (256-bit), when accessing (either reading or writing) a previously untouched memory word (i.e. any offset within a word).
4. At the time of expansion, the cost in gas must be paid. Memory is more costly the larger it grows (it scales quadratically). Because the larger memory grows, the more gas it will consume each time you interact with it. But the gas cost for writing to memory does not only depend on how much data you are writing to memory. It also depends on the actual memory size
5. 

![At the time of expansion](https://user-images.githubusercontent.com/93861625/234342740-0c15c01e-b1b0-4d58-8b5c-082a7271006d.png)

1. Calldata It refers to the raw hex bytes sent along any message call transaction between two addresses. the calldata is a byte-addressable space, laid out continuously just as memory. For reading, you can load 32 bytes at a time
2. 
- the first 4 bytes correspond to the selector of the function signature.
- the remaining bytes correspond to the input parameters of the function.
Each input argument is always 32 bytes long. If its type is smaller than 32 bytes, the argument is padded. IDK what about complex type parameters ? read this : [https://ethereum.stackexchange.com/questions/131455/understanding-dynamic-types-in-calldata](https://ethereum.stackexchange.com/questions/131455/understanding-dynamic-types-in-calldata)
1. input arguments are padded either on the right or the left depending on their type according to abi specs (for example,intN `uintN` or `address` or others are padded on the left, while string, bytes and `bytesN` are padded on the right).
2. “One good way to think about the difference (between calldata and memory) and how they should be used is that calldata is allocated by the caller, while memory is allocated by the callee.”
3. Calldata is Almost unlimited in size = virtually unlimited in size, without a fixed boundary.
4. when reading values from the calldata, the values are copied on the stack.
5. the calldata, like the memory, will also be bounded to the block gas limit. However, the cost of allocating more bytes in the calldata is always linear in comparison to the memory that goes quadratically as the memory size grows
6. Each byte of calldata has a cost:
- 4 gas for zero bytes `0x00`
- 16 gas for non-zero bytes. so the bigger the calldata more gas it uses
1. `msg.sig`= function identifier (the first 4 bytes of the keccak256 hash of the function signature). This is the equivalent of accessing the first 4 bytes of the calldata
2. The function signature function(uint,address) never includes spaces or the data location specified for a parameter while declaring the function
3. You can also extract as follow, using calldata slices.

```
bytes4 selector = msg.data[4:]
```

1. In constructors when the creation code runs, there is no 4-byte offset. This is because in constructors, the calldata is empty (the special variable *`msg.sig`* is padded to contain 4 zero bytes). and any parameters are appended at the end of creation code. IDK how are these parameters encoded ? with names ?
2. The global `msg.data` gives you access to the entire calldata, including the function selector. calldata = selector + any arguments passed to the function(input data)
3. The `delegatecall` operates with data in memory. Therefore, for the proxy’s fallback function to work, all the calldata must first be copied in memory
4. A bytes data literal string is written as “abc”. a bytes data hex literal string is written as hex”abc”. hex just concatenates 0x to the literal, so ⇒ “0xabc” . Each character in a string is 1 byte
5. 

![a bytes data literal](https://user-images.githubusercontent.com/93861625/234342802-7f6b6188-6051-4bac-adc4-6c6a019f996c.png)

1. encodePacked should be used when creating a payload for calling a contract because it prevents appending raw bytes to the selector we are passing to it, otherwise abi.encode will add 28 0x00 bytes first(default behavior) and then append other parameters, which is not correct (If you are not using dynamic types inputs like string and bytes). That is why use `abi.encodePacked`to encode the selector in place, taking only exactly 4 bytes. The difference between encode and encodepacked is just that it doesn’t pad extra bytes for shorter types just for the sake of working with 32 bytes, and selector encoding doesnt need that padding.

![encodePacked should be used when creating 1](https://user-images.githubusercontent.com/93861625/234342880-cd20bec9-146a-4e74-8f94-f5bfe28d1557.png)

1. in encodewithsignature we are passing signature and in encodewithselector we are passing hex selector

![encodePacked should be used 2](https://user-images.githubusercontent.com/93861625/234342917-882e4720-1ec5-429a-a606-a6922ff775de.png)

1. 

![encodePacked should be used when 3](https://user-images.githubusercontent.com/93861625/234342963-3673c118-e1f2-4942-9a07-6c69971d40ed.png)

1. 

![encodePacked should be used when 4](https://user-images.githubusercontent.com/93861625/234343008-b8e2882a-f47a-4f47-a930-484be4707d3f.png)

1. The syntax *`data[start:end]`*is only available for *`bytes`* variables pointing to *`calldata`*, not *`memory`. This is calldata slicing. used with [msg.data](http://msg.data) or a bytes calldata variable via a function parameter.* The slice returned is of type `bytes`
2. When a `calldata` reference is passed to an internal function that accepts `memory` as a function argument, the EVM can move this value from calldata to memory in two ways:
- using the opcodes `calldataload` (to load the value from calldata into the stack), and then `mstore` (to write the loaded value in memory).
- using the opcode `calldatacopy` to move the value from the calldata directly in memory in a single step.
1. in reality, values are not *“moved”* from the calldata in the real sense of the term. They are rather copied. Both opcodes *`calldataload`*and *`calldatacopy`* create copies. Data in calldata is immutable and only copies of it are made..
2. This makes us see that calldata is a data location that goes only in one direction: values from calldata can only be loaded. Therefore, values from calldata are always:
- loaded (= copied) into the stack using `calldataload` or `calldatacopy`.
- the loaded/copied values are then moved either in `storage` , `memory` or `code` (for `immutable` in constructors). IDK can internal function defined with storage parameters be called with calldata parameters internally ? how is value moved from calldata to storage ?
1. In the context of a constructor, the data location `calldata` is not available as a function parameter. The reason is that when a contract is deployed, and the constructor runs, the calldata is always empty. During a deployment transaction, the contract creation code does not go into the `data`field (= calldata). Instead, the contract creation code goes into the `init` field of the transaction. The **`data`** field is used for message calls, which can be made only to addresses that have some code stored in them
2. The EVM uses a 256-bit word machine, which facilitates the Keccak256 hash scheme and elliptic-curve computations.
3. There are two main rules regarding the stack and Solidity.
- Cheapest to use data location (among all others)
- The stack is only available in the function scope.
1. two Opcodes of stack : 
- `DUP`: instruct to duplicate the `n`th stack item and to put it on top of the stack, where `n` can be `1` up to `16`.
- `SWAP`: instruct to swap the item on top of the stack with the `n`th item, where `n` can be `1` up to `16`.
1. In general, opcodes take(consume) one, two, or more elements from the top of the stack ⇒ as parameters, depending on the instruction. The result of the opcode is pushed back on top of the stack.
2. Layout of stack : 
- words on the stack are 256 bits long.
- limited in size = the stack can hold up to 1,024 values.
- Only the 16th topmost items on the stack can be accessed.
1. The stack is a padded data location, large end of a word is located on the left on the stack. It is a word-addressed byte array. the stack is a padded data location. This means that each variable of direct types (e.g., `address`, `uintN`, `bytesN`, etc.) are padded on the stack to occupy a full word.
2. It is also possible to move stack elements to storage or memory to get deeper access to the stack. But it is not possible to access arbitrary elements deeper in the stack without first removing elements from the top of the stack.
3. All variables defined + assigned inside a function are stored on the stack. They are referred to as local variables. (two types : direct types, the pointers types: which are references to variables stored in other data locations (storage, memory, or calldata)) IDK how are pointer types stored on stack ?
4. [https://ethdebug.github.io/solidity-data-representation/#user-content-types-overview-overview-of-the-types-direct-types-table-of-direct-types](https://ethdebug.github.io/solidity-data-representation/#user-content-types-overview-overview-of-the-types-direct-types-table-of-direct-types)

![ethdebug github](https://user-images.githubusercontent.com/93861625/234343135-12ff6843-89d5-4eb6-aac3-9a1184d490c5.png)

1. At the end of each function execution, the stack will pop as many elements from the stack as needed to end up having no more items left on the stack (or only one, being the function selector related to the current stack frame). I call this “stack balancing.” This is why while debugging you may see selector being preserved deep in the stack.
2. block scopes using `{}` within a function help to reduce “stack too deep” errors. this is because the value of any variables defined inside a block scope is removed from the stack on the block scope end and is not accessible anymore.
3. The “stack” (as a single word) refers to the EVM stack used to manipulate data and consume opcodes arguments. The “call stack” refers to the current execution environment (the contract) where the EVM is running. It is made of an array of addresses of contracts being called. These addresses are pushed and popped from the call stack as `CALL`opcodes are being run. (Note : call opcode is used anyways for message calls but when you use low-level calls directly in solidity it removes some of the safety checks that the compiler adds)
4. The last `address`on the call stack (the `address` on the top of the stack) is the contract `address`where the EVM is currently running opcodes. The contract `address` the most buried at the bottom of the call stack is the one that was initially called by the EOA.
5. Stack too deep error : *There are several situations in which this or similar 
errors occur. Most are related to too many local variables, parameters 
or return variables in functions. It might also happen if you have 
expressions that are too deeply nested.*
6. 

![expressions that are too deeply nested](https://user-images.githubusercontent.com/93861625/234343195-9ea907a5-b0dc-4285-bb1d-179f5400ad63.png)

1. 

![expressions that are too deeply nested 2](https://user-images.githubusercontent.com/93861625/234343228-99317630-d66e-41f9-9af6-c0782205fbf4.png)

1. Each byte(2 digits) in the contract bytecode code is the hexadecimal representation of an opcode. This instruction data, stored in the code (the opcodes that make the smart contract logic), is persistent and, as described above, part of the account state field.
2. The opcodes `CODESIZE` and `CODECOPY` enable you to read and copy the bytecode of the contract we are currently executing. Finally, `EXTCODESIZE` and `EXTCODECOPY` enable you to read and copy the bytecode of another external contract from a contract, providing its address.
3. Code is always a multiple of 32 bytes words. Variables stored in the contract bytecode like `constant`or `immutable` are placed wherever the compiler places them among the code. IDK where ?
4. 

![Code is always a multiple of](https://user-images.githubusercontent.com/93861625/234343261-ab939cb1-a8d6-47b5-8f24-08b2861d2654.png)

1. Aside from these three main sections, a smart contract’s bytecode also includes three smaller sections:
- the [free memory pointer](https://medium.com/better-programming/solidity-tutorial-all-about-memory-1e1696d71ee4)
- the calldata check: ensures we send at least four bytes. If not, it uses the `receive` / `fallback` function as the default function handler. OMG this is inside the function dispatcher i guess
- [the contract metadata.](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-vi-the-swarm-hash-70f069e22aef/)
1. A dispatcher is that part of a runtime bytecode that checks if the function requested by the user to execute exists in the smart contract. The existence is checked using the function selector. You can imagine the dispatcher as a `switch` case statement. The dispatcher will start comparing (using the `[EQ](https://www.evm.codes/#14?fork=arrowGlacier)` opcode) our calldata with all the function signatures inside it. IDK how is the order of these selector matching decided ?
- If the existence check passes (meaning the function exists in the contract), it jumps to its function body to execute its logic.
- If the existence of the function is not found, it either executes the `fallback` function of the smart contract or reverts if the contract does not contain a `fallback` function. P.S. non-reverting fallback functions are a problem
1. The bytecode of a smart contract is not directly stored under the account state. Instead, it is the `codeHash` that is stored. the `codeHash` is simply the keccak256 hash of the contract’s bytecode.
2. So a smart contract’s bytecode is stored in the low-level database of the Ethereum client (leveldb for geth client) under the key corresponding to the keccak256 hash of the contract bytecode
3. ** We would need to hash the code again before hashing the 4 elements together.

![We would need to hash the code again](https://user-images.githubusercontent.com/93861625/234343331-98fcf3ab-8bca-4ab1-ac14-13b735c18d08.png)

1. 

There are two types of code for smart contracts, as shown below:

- the creation code: this is the contract’s bytecode that includes the instruction for deploying the contract and running the `constructor` logic.
- the runtime code: this is the final bytecode of the contract once it has been deployed on the blockchain.
1. The main difference is that the creation code runs only once when the contract is deployed. In contrast, the contract runtime code is the bytecode of the contract sitting on the network that is executed once the contract is called
2. The creation code contains the logic to return the runtime code and thus (indirectly the runtime code)

*→ “init code” = creation code, or maybe at some places referred to as “creation part of the code”*

*→ “deployed bytecode” = runtime code*

1. The creation bytecode is equivalent to the input data of the transaction that creates a contract (init field of txn), provided the sole purpose of the transaction is to create the contract. **The creation code is the code that generates the runtime bytecode. The creation code not only contains the logic to run the constructor but also contains the logic to return the contract runtime code and save this bytecode on the blockchain under the address of the deployed smart contract.**
2. IDK read in detail what does the optimizer actually do ?

![what does the optimizer ](https://user-images.githubusercontent.com/93861625/234343371-bb706021-6c2c-40f8-b7be-bc49ab149fa6.png)

1. Usually, creation code is larger than the runtime code due to constructor logic. When we deploy a contract that does not contain a `constructor`, the creation code is still bigger than the runtime code. (around + 32 bytes extra, because of other components like free memory parameter, payable constructor value check, and copying of runtime code associated with the address and returning it from memory) creation code =<init code><runtime code><constructor parameters>
2. Solidity has multiple ways to access a smart contract's bytecode. 3 of these return a bytes memory value.
- `<address>.codehash`
- `<address>.code`
- `type(ContractName).creationCode`
- `type(ContractName).runtimeCode`
1. What differentiates these properties is that `.code`and `.codehash`are properties that read from the blockchain(reads from nodes’ storage ), while `.creationCode`and `.runtimeCode`actually return some `bytes memory` that are inlined inside the bytecode of the smart contract where it is used. Also, address.code returns the runtime code, since the creation code is not stored on the blockchain.
2. Implicit conversion between two data types is allowed in Solidity if no information is lost in the process. *If the compiler does not allow implicit conversion but you know what you are doing, an explicit type conversion is sometimes possible. This is risky because it allows you to bypass some security features of the compiler*
3. When converting an unsigned integer to a larger type, **left-padding occurs** ,meaning zeroes (= 0 bits) are added to the left. When converting an unsigned integer to a smaller type, the **high order bits (the bits “the most on the left”) end up being discarded. This will lead to data loss.** 
4. When explicitly converting to a smaller-bytes range, the right-most bytes are discarded (= the *“higher order bytes”). This basically mean that Solidity truncates from the right hand side, until the length in bytes is equal to new length of the bytes specified in the type casting. Remember that 1 bytes = 2 hex characters. When converting to larger bytes, zero padding is added to the right.*
5. `uintM` and `bytesN`cannot be implicitly converted between each other ❌. Explicit conversion is allowed in both directions as long as both the `uintM` and `bytesN`have the same size(no. of equivalent bits)
6. uintM to bytesN table of equivalence = [https://gist.github.com/CJ42/01ccb48bcbee928bd9ee8d522859f7fe#file-bytes-uint-equivalence-md](https://gist.github.com/CJ42/01ccb48bcbee928bd9ee8d522859f7fe#file-bytes-uint-equivalence-md)
7. Any hexadecimal literal can be implicitly assigned to a`bytesN` as long as the literal has the same number of bytes mentioned in the type. Eg. bytes4 a = 0xbeefbeef or hex’beefbeef’ 
8. When a uint number is converted to the bytesN type, the number is not stored in the bytes type but its hexadecimal representation is. eg. when we convert 200 to bytes1 type it gets value 0xc8. This means bytes datatype is meant to deal with only raw hexadecimal representation under the hood. For the reverse process of converting hexadecimal representation into uint, it gets converted to the decimal number.
9. a hexa number literal doesnt mean it can only have “numbers” it can have alphabets also. Replace number literal with just literal when reading this. 

![a hexa number literal](https://user-images.githubusercontent.com/93861625/234343420-16530de0-d780-4170-8e0e-902918fa1ffd.png)

eg. uint8 = 12; uint16 = 0x1234; uint32 = 0x123456 (when we assign uint16 = 0x1234 its value is not actually 1234 but 4660.

1. Any explicit or implicit conversion that seems confusing first gets converted to binary type and then decimal/hex type. Binary representation is the intermediate form. 
2. Here, we can still undercast smaller type variables though. 

![we can still undercast smaller](https://user-images.githubusercontent.com/93861625/234343465-a16338ce-40a0-42d4-a22a-cd2c7aaec8f1.png)

Since 0.8.0 this gives type conversion error

1. **Conversion from bytes to bytesN**
- Implicit conversion is not allowed ❌
- Explicit conversion is allowed since Solidity 0.8.5 🙌 🙂

Below is an example: gets truncated similar to uint, but starting from lower bits. 

```
bytes memory data = new bytes(5);
bytes2 firstTwoBytes = bytes2(data);
```

1. **Conversion between bytes and strings**
- Implicit conversion is not allowed ❌ on either side (`bytes` to `string`, or `string` to `bytes`)
- Explicit conversion is allowed ✅
- When converting string to bytes, each character gets converted to its utf-8 representation in hex.  character takes 1 byte. When doing the reverse, all such utf-8 hex repres are converted back to string characters.

```
string memory a = "All About Solidity";
bytes memory b = bytes(a);bytes memory c = new bytes(5);
string memory d = string(c);
```

1. 

Any hex literal can be implicitly converted to an `address` type if it passes the following two requirements:

rule 1: must have the correct size: 20 bytes long.

rule 2: must have a valid checksum

1. 

![must have a valid checksum](https://user-images.githubusercontent.com/93861625/234343508-8fd7d24b-09ac-41f7-b3b8-5c4dd2eb48d2.png)

1. **Conversion from uint160 to address(what address is returned ?**
- Implicit conversion is not allowed from `uint160` to `address` ❌
- Explicit conversion is allowed from `uint160` to `address` ✅. It converts the decimal repres to hexadecimal repres through the binary form and just starts to fill the address type from lower bits and rest zero padded upto 20 bytes.

```
uint160 someNumber = 5_689_454_112;address convertedAddress = address(someNumber);
```

> NB: prior to Solidity 0.8.0 (up to Solidity 0.7.6), it was possible to convert explicitly any integer type uintN to an address (via casting). Since Solidity 0.8.0, explicit conversion is only allowed with uint160.(even though the value needn’t be so big. even 500 works.
> 
1. **Conversion from address to uint160** 
- Implicit conversion is not allowed from `address` to `uint160` ❌
- Explicit conversion is allowed from `address` to `uint160` ✅ : returns decimal form of this hexa address.
1. An `address from`can be explicitly converted to `address payable` via `payable(from)`
2. any explicit conversion into `address` type (using `address(…)`) always returns a non-payable `address` type. Therefore, any Address Literal, `bytes20` or `uint160` value can be explicitly converted to an `address payable` as follow:

```
// conversion from Address Literal to address payableaddress to = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;
address payable payableTo = payable(to);// conversion from bytes20 to address payablebytes20 from = bytes20(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4);
address payable payableFrom = payable(address(from));
// conversion from uint160 to address payableuint160 u = 12345;
address payable converted = payable(address(u));
```

1. Converting from address to bytes20 removes the checksum. All explicit conversions are done using address a = address(bla bla); **

Explicit conversion can be done from an `address` to a `Contract` type.

```
address creator = TokenCreator(msg.sender);
```

1. **continue in a “for” loop : so the basic idea is it needs to evaluate condition again but in case of a for loop the increment step comes in between because it is outside of function body**

In a `for` loop, it **1) updates the expression** (*e.g: incrementing a counter*), **2)** **re-evaluates the condition** (after the expression has been updated), and finally, **3) jump inside the body of the loop.**

**continue in a “while” loop**

In a `while` loop, it 1) **jumps back ⬆️** to ***re-evaluate the initial condition*.**

- if the condition is true **❓**✅ ****→ **2.a)** it will **jump inside the body of the loop.**
- if the condition is false **❓**❌ → **2.b)** the loop will terminate.

**continue in a “do … while” loop**

In a `do ... while` loop, it **1)** **jumps straight** ️️ ⬇️ to ***evaluate the final condition.***

- if the condition is true **❓**✅ ****→ **2.a)** it will run the loop body again.
- if the condition is false **❓**❌ → **2.b)** the loop will terminate.
1. A statement like if(1) is not valid in solidity because non-booleans are not automatically converted to booleans
2. Types of compiler errors :

![Types of compiler errors](https://user-images.githubusercontent.com/93861625/234343581-28327a37-8d29-4b5e-981a-940e302d584e.png)

1. There are multiple situations where a `warning` occurs. Including:
- shadowing (variable or built-in symbol)
- statement having no effect
- unreachable code
- when a function defines a type being returned, but there is no explicit `return` statement inside the function body. (either name it, or return a variable explicitly)
- when a function state mutability can be restricted (e.g: from nothing to `view` or `pure`).
- when the return value of a low level `.call` , `.staticcall` or `.delegatecall` is not used.
1. A `ParserError` occurs when the Solidity source code does not conform to the Solidity language rules. For instance:
- when a line does not end with a semi colon
- when a variable is declared but no value is assigned to it (no `=` symbol) and the variable does not end with a semi colon `;` )
1. `DocstringParsingError`occurs when some content within the Natspec comments is invalid, preventing the compiler from parsing a Natspec comment block.  Such error type related to Natspec will generally occur in the following scenarios:
- when a @param tag does not contain the name of the function parameter or an invalid name.
- when trying to use a Natspec tag that does not exist (e.g: `@test` ).
- when trying to use a Natspec tag that is unavailable for the specific definition, it is assigned to (e.g: using the `@title` tag above a `function` , the `@title` tag being available only to document above `contract` definitions).
1. `SyntaxError`: occurs when the Solidity syntax is invalid. Below are some examples of `SyntaxError`:
- declaring variables outside of loop blocks. ? WTF
- using `break` or `continue` outside of a `for` , `while` or `do while` loop.
- defining empty `struct`
- when underscores `_` are used incorrectly as separators for number literals.
- when the body of the `modifier` does not include the function body placeholder `_;`.
1. `DeclarationError` includes:
- incorrect contract or variable name.
- impossible inheritance (`Linearization of inheritance graph impossible`). (based on C3)
- when two `constructor` are declared in the same contract.
1. List of solidity errors - [https://solidity-debugger.io/](https://solidity-debugger.io/)
2. Examples of `TypeError` include:
- type conversions.
- incorrect visibilities (e.g: in libraries).
- when a contract that does not implement all the functions (some functions are just like “interface” functions) is not marked as abstract.
- when not specifying the data location like `storage` , `memory` or `calldata` for some variables of complex types (struct, mapping, arrays, etc…)
- when explicitly converting a non-payable address to a contract that has a payable `fallback` or `receive` function (so that can receive ethers). WTF
1. One important thing to note about `UnimplementedFeatureError`
 is that the compiler will not point you to the line where compiling fails and where the error occurs (unlike the other errors). Whether you do it via Remix or via the `solc` CLI. (ex. moving a whole array of struct from memory to storage)
2. **Crash Errors**

**In some cases, the following errors should be reported in the Github repo of the Solidity compiler if they are encountered.**

- `InternalCompilerError`: Internal bug triggered in the compiler – this should be reported as an issue.
- `Exception`: Unknown failure during compilation – this should be reported as an issue.
- `CompilerError` relates to an invalid use of the compiler stack, settings or configs. This include the infamous `“stack too deep”` error that causes a lot of developer a lot of issues.
- `FatalError`: Fatal error not processed correctly – this should be reported as an issue.
- `YulException`: Error during Yul Code generation – this should be reported as an issue.
- The difference between the errors defined as Fatal and the others is that the parser will stop parsing any line that follows where it encounters a fatal error. It will then throw and abort.
1. **The ABI is the standard way to interact with contracts in the Ethereum ecosystem. Both from outside the blockchain and for contract-to-contract interaction”**
2. The ABI of a smart contract will describe its **public (which means public and external functions)** interface and how to interact with it. It defines which functions you can invoke and guarantees that the function will return data in the format you are expecting. the ABI **defines clear specifications of how to encode and decode data and contract calls**
3. The same goes backwards. The ABI specifies how to read and decode data out of transactions since all the data specified in transactions is encoded as raw hexadecimal. Therefore in Ethereum and any EVM based chain, the ABI is basically how contracts calls are encoded for the EVM (so that the EVM understands which instructions to run).
4. For a Dapp interface, the ABI of a Solidity smart contract is represented as an array of objects.(JSON) Each object can correspond to:
- A method (function) in the contract that is publicly callable (= can be called by anyone, except if restrictive modifiers are attached to it). Internal/private functions are excluded.
- an event definition
- A custom error
- a fallback() or receive() function.
- getter functions of public state variables
1. **JSON ABI specification of a function**
- **type:** `function`, `constructor`, `event` , `receive` (for the [receive ether function](https://docs.soliditylang.org/en/develop/contracts.html#receive-ether-function)), or `fallback` (for the [default fallback function](https://docs.soliditylang.org/en/develop/contracts.html#fallback-function)) or error
- **name**: the name of the function.
- **inputs**: the function parameters as an array with the name and type of each parameter.
- **outputs**: an array of return value(s) by the function
- **stateMutability**: `view`, `pure` , `payable` or `nonpayable` (= default mutability of a function).

> NB1: for the constructor and fallback and receive functions, :They have only two fields : type (receive/fallback/constructor) and stateMutability
> 
> 
> ***NB2:** for the `fallback` function, the input is always empty, since the fallback function cannot accept arguments.*
> 
1. Each object in inputs field is made of the following properties : name, type and internaltype of parameter. This is incorrect : outputs field is a separate field, not inside inputs. Both have same structure : name, type and internalType of each parameter object

![Each object in inputs](https://user-images.githubusercontent.com/93861625/234343669-34086fa3-1b19-488c-981e-ec95b8f701cc.png)

1. **JSON ABI specification of events (outputs and stateMutability don’t make sense here)**
- **type:** always `event`.
- **name:** the name of the event.
- **inputs:** array of parameters that can be passed to the event.
- **anonymous:** `true` if the event was declared as `[anonymous](https://docs.soliditylang.org/en/v0.8.13/contracts.html?highlight=anonymous#events)` in the Solidity code. otherwise `false`.
1. Only difference with function abi inputs field is : indexed field in place of internalType 

![Only difference with function abi](https://user-images.githubusercontent.com/93861625/234343705-7bf55666-1b34-4cf1-9101-900989f39101.png)

1. In ABI encoding, Most of the static types in Solidity like `address`, uint256 or bytes32 are encoded as 32 bytes words. Padding applicable based on type (address on left, bytes on right)
2. The solidity built-in function `abi.encode` enables to encode any Solidity types into raw bytes(with proper padding), that can be interpreted directly by the EVM. Note that multiple arguments can be given to this function.
3. OMG 

![multiple arguments can be given](https://user-images.githubusercontent.com/93861625/234343754-c4e2d553-0cf9-4906-98fa-64631710f468.png)

1. ASCII UTF8 encoding table reference to breakdown hex notation for strings : [https://www.alpharithms.com/ascii-table-512119/](https://www.alpharithms.com/ascii-table-512119/)
2. 

![UTF8 encoding table reference](https://user-images.githubusercontent.com/93861625/234343794-0805527c-0a0c-4f7a-9feb-9108a28e46fd.png)

abi.encodepacked would just return the hex-encoded raw bytes for the input data (directly concatenated)

1. The functions `abi.encodeWithSignature(...)` and `abi.encodeWithSelector(...)` can be used in Solidity to prepare payloads in raw bytes for external contract calls. Such payloads can then be passed as parameters to the low level Solidity  `.call(...)` ,  `.delegatecall(...)` and  `.staticcall(...)`  functions. These two functions will encode the passed arguments, starting from the second parameter.
2. 

![These two functions will encode the passed](https://user-images.githubusercontent.com/93861625/234343849-a5a7def3-7e40-4f25-a08c-f67247985a68.png)

1. 

> Enums used to be defined in the ABI by the smallest uint type large enough to hold all values defined in the enum.
> 
> 
> *For example, an `enum` of 255 values or less would have been mapped to a `uint8` by the ABI, while an `enum` of 256 values or more would have been mapped to a `uint16`.*
> 
> *This has changed since Solidity version `0.8.0` , as `enum` cannot have more than 256 members anymore.*
> 
1. The calldata for a message call/function call is generated according to the abi specs, into raw bytes by the evm when we call the function
2. import “./MySolidityFile.sol”;

When the Solidity compiler encounters this statement, two things happen.

- All global symbols (what I call “*Solidity Objects”*) defined in `“./MySolidityFile*.sol”`* get imported into the current global scope.
- All global symbols imported inside `“./MySolidityFile*.sol”`* get also imported into the current global scope.
1. In Solidity, everything imported inside a file A gets subsequently available (imported) in other files that import A. This global import syntax is not recommended, as it makes it hard to understand **where modules come from** or **where modules are defined (and breaks contracts on updation of import statements)**
2. The import syntax with curly braces (specific objects) helps locate where the building blocks that make your smart contracts (`abstract contract`, `interface`, `library`, etc.) come from and which files they are **actually defined**
3. Import Aliases :
- **Aliases for** **global imports:** can then be used to refer to specific objects defined in the file, using the dot “.” syntax (like object properties).
- **Aliases for specific imports:** enable to rename objects or symbols imported from a Solidity file, for instance, to avoid naming collision (or to give better name for better code clarity if needed).
1. 

![Import Aliases](https://user-images.githubusercontent.com/93861625/234343904-2e8cff46-2d7d-49b4-910a-1344f8b0d7a7.png)

All the symbols and objects defined in `“./Endpoints.sol"`(as well as imported in `“./Endpoints.sol”`) are now available in the format `Endpoints.TOWER_ADDRESS_A.`They are now members of this global symbol and can be accessed like you access properties of a JavaScript object using the dot notation.

1. import { Point as Coordinate, GPS } from “./Endpoints.sol”;  for aliases of spicific import
2. Such aliases are useful in two cases:
- **Avoid naming collisions:** in cases where what you want to import is already imported by other files. If you encounter naming collisions while importing, use aliases to solve this.
- **Prevent unexpected behavior** (if the compiler did not flag it, and a variable you are using references the wrong variable imported from elsewhere).
- **Rename what you are importing:** if you want to use more meaningful names in your context and for your code clarity.
1. Importing from github URL or npm ( for eg. “@openzepplin” ) works directly only in remix. In VS code, you have to install the npm package using npm install or add it to to your dependencies list in hardhat/brownie project config or import it from a local file. : [https://medium.com/@raman.shinde15/imports-in-solidity-7aec77c50423](https://medium.com/@raman.shinde15/imports-in-solidity-7aec77c50423)
2. These are called solidity objects : IDK if enum etc. declared inside of contracts can be imported at other places

![These are called solidity objects](https://user-images.githubusercontent.com/93861625/234344053-3e1aae2d-04bb-45df-bb65-c6c61f87ce68.png)

1. Smart contracts interfaces are their skeleton or backbone. It defines the contract’s functionalities and how to trigger them. This way, Dapps or other smart contracts can know how to interact with any smart contract implementing a specific interface since the interface is known. The main idea behind using interfaces is to provide a customizable and re-usable approach for your contracts. **The implementation becomes a custom way to perform a specific set of functionalities.**
2. Encoding dynamic types in calldata : [https://ethereum.stackexchange.com/questions/131455/understanding-dynamic-types-in-calldata](https://ethereum.stackexchange.com/questions/131455/understanding-dynamic-types-in-calldata)
3. An interface contract in Solidity will lead to two main pieces of information:
- the **bytes4 selector** of each of functions, based on their *function signature* ****(function name + inputs if any -)
- the **interface id (**ERC165) : which can be used to check if a contract implements ERC165 and what interface ID it supports based on what interfaces it has implemented.
1. The contracts inheriting from the interface will have to implement all the functions defined in the interface. Otherwise, the Solidity compiler will throw an error. If the contract does not implement all functions (in other words, **if some functions implementations are missing**), it can only be used as an **abstract** contract and cannot be deployed directly. The other contract that will derive from it will have to implement the functions missing the implementations*. Abstract contracts and interfaces cannot be deployed.* A contract can’t be deployed even if the contract has all its functions implemented, but defined with the `abstract`keyword.
2. **Starting from Solidity 0.8.8, the override keyword is not required when overriding an 
interface function except for the case where the function is defined in multiple bases. All interface functions are virtual by default, no need of the virtual keyword. The overriding function is not virtual by default and cannot be overriden automatically in future inheritances.**
3. Functions part of an interface can also be `view` functions. state modifiers like `view`, `pure`  and `payable`must match inside the implementation contract. The functions defined inside an `interface`contract can only be declared as `external`. the contract implementing an interface can define the visibility of the implemented functions as either `external`
 or `public` : similar to how abi only includes public/external methods. This is different from overriding of contract-inherited functions, there state mutability and visibility can be changed to a stricter one but for interface-inherited functions it has to match. 
4. for the ABI’s second purpose :

![can be changed to a stricter one but](https://user-images.githubusercontent.com/93861625/234344112-4308b1a8-9453-460f-975b-a7711e927d1b.png)

1. The JSON artifact of the interface contract has an empty bytecode field (0x) because interfaces do not have any function implementation, any logic inside, and therefore are not deployable.
2. Any types defined inside an interface can be accessed in your solidity (file level) via the syntax Interface.yourtype IDK
3. Interfaces cannot inherit from other contracts. However, they can inherit from other interfaces
4. Function overloading is also allowed inside interfaces. The reason is functions defined inside the interfaces resolve to their bytes4 function selector.
5. Events like functions also have their own signature. The signature is constructed similarly by hashing the event name with the event parameter types, separated by commas without spaces). Defining events in interface helps modularity and composability. IDK Error signatures ?
6. ****What cannot be declared inside an interface in Solidity?**** 
- State variables (enum and struct allowed)
- Constant variables
- modifiers
- constructors (since they cannot be instantiated)
1. If there is no constructor specified in the contract, the contract will assume the default empty constructor, equivalent to `constructor() {}`
2. If a base contract has arguments in constructor, derived contracts need to specify all of them. This can be done in 2 ways: if base constructor has no arguments it needn’t be invoked at derived constructor.

```solidity
contract Lion is Animal(4, true) {}
```

```solidity
contract Lion is Animal {
    
    constructor(string memory _name)
        Animal(_name, 4, true)
    {
        // ...
    }
}
```

1. ***NB: specifying the arguments in both places** (inheritance list + modifier in derived constructor) **will generate an error**.*

***NB2:** If a derived contract does not specify the arguments to all of its base contracts’ constructors (and this should be done in order left to right most base like to most derived like), it will be abstract.*

1. If the modifier does not have argument, you can omit the parentheses `()` : modifier A { ..}
2. The symbol `_;`is called a merge wildcard. It merges the function code with the modifier code where the `_;` is placed. it *“returns the flow of execution to the original function code”. It is mandatory*
3. The modifiers will be executed in the order they are defined, so *from left to right. This is an important quirk to consider.*
4. **Modifiers are inheritable properties of contracts and may be overridden by derived contracts. All function visible in the contract are visible to the modifier. But symbols introduced in a modifier are not visible in the function** (*because the modifier might change if it is overridden in the derived contract).*
5. 

![Modifiers are inheritable](https://user-images.githubusercontent.com/93861625/234344169-9178aa68-63e4-4a44-bad5-81e9190d6446.png)

1. In Solidity, mappings values are represented by the hash of their keys. The 32 byte hash is an hexadecimal value that can be converted to a decimal number. This number represents the slot number where the value for a specific key is hold in storage.
2. In fact when mapping is declared, space is reserved for it sequentially like any other type(but nothing stored at this slot), but the actual values are stored in a different slot. There’s no length to be stored, and individual values need to be located elsewhere.
3. . represents concatenation here

![represents concatenation](https://user-images.githubusercontent.com/93861625/234344237-87c35e1c-3c0f-49fb-9628-b2c2641a3b06.png)

1. ***NB: Note the value of slot and keys are represented in hexadecimals., and left padded with empty bytes and pasted into memory from 0x00 to 0x40 WTF if bytes32 is key, is it left padded ?***
2. The data for a key is not stored in a mapping, but rather its **keccak256** hash is used for storing the value the key data references to. Because of this, **mappings do not have a length**. If the mapping value is a non-value type, the computed slot marks the start of the data. If the value is of struct type, for example, you have to add an offset corresponding to the struct member to reach the member.
3. Key and value types allowed in a mapping : 

![Key and value types allowed](https://user-images.githubusercontent.com/93861625/234344290-002aeb06-1b85-4213-8ef5-5b851484ca14.png)

1. **Finding the storage slot for a specific key (abi.encode is used because we need padding) convert into uint256 to view it human-readable number**

```
function findMapLocation(uint256 slot, uint256 key) public pure returns (uint256) {
    return uint256(keccak256(abi.encode(key, slot)));
}
```

1. Mappings can be passed as parameters inside functions, but they must satisfy the following two requirements.
- Mapping can be used as parameter only for private and internal functions.
- The data location for the function parameter (our mapping) can only be storage.
1. **mappings are virtually initialized. In other words, all possible variables are initialised by default.** So by default, when a mapping is declared, every possible key is mapped to a value *whose bytes representation are all zeros. This is because a mapping cannot be iterated over.*
2. The solution of the problem(not knowing how many keys actually have values) is to have a counter that tells you the length of the mapping where the last value is stored.(IterableMapping is a proper pattern by sol-by-example)

```
contract IterableMapping {   
 mapping(uint => address) someList;
 uint public totalEntries = 0;   
 function addToList() public returns(uint) {
        someList[totalEntries] = address(0xABaBaBaBABabABabAbAbABAbABabababaBaBABaB);
        return ++totalEntries;
    }}
```

1. When a struct is used as a value type inside a mapping, If the struct contains an array, it will not be returned via the getter function created by the *“public”* keyword. You have to create your own function for that, that will return an array.
2. • Mappings can’t be used as return value for function. And particular variables can’t be used as the value type. 
3. The EVM in Ethereum know that a smart contract is created when an Externally Owned Account (EOA):
- sends a transaction
- specifies a **zero-address** (**`0x0000000000000000000000000000000000000000`** ) as a recipient.
1. The `this` keyword in Solidity refers to the current contract. (follows delegatecall context conventions) It is convertible to the `address` type and enables you to :
- get the contract’s address: `address(this)``
- get the number of ether hold by the contract: `address(this).balance`
- get one of the contract’s function selector (=function signature): `this.yourFunction.selector`
- call a function within the contract defined as `external`, using `this.externalFunction()` syntax.
1. A contract needs to be marked as `abstract` in two cases:
- at least one of its functions is not implemented.
- It inherits from an `abstract` contract and does not implement all its non-implemented functions.
1. **When a contract inherits from other contracts, only a single contract is created on the blockchain. The code from all the base contracts is compiled into the created contract. This means contract C is A, B { .. } doesn’t require A/B to be already deployed. This means that all internal calls to functions of base contracts will use internal function calls (jump statements)**
2. The order of inheritance for a contract is based on [C3 Linearization](https://en.wikipedia.org/wiki/C3_linearization) **You have to list the direct base contracts in the order from “most base-like” to “most derived”(while inheriting as well as in constructor)., otherwise the error : “Linearization of inheritance graph impossible”. *The order in which parent contracts are declared on the child contracts determines the layout of the storage.***
3. Use `super.functionName()`if you want to call the function one level higher up in the flattened inheritance hierarchy. One of the extra benefit of using `super`is that it uses JUMP internally and not a message call.
4. For a function overriding multiple functions from multiple contracts in the inheritance hierarchy, the override(..) order should be similar as declared inheritance order WTF

you must specify all the overridden contracts between parentheses, as follow:

```
contract Teacher is Person, Employee {

    function greetings() public pure override(Employee, Person) returns (string memory) {
        return "Hello students ! Welcome to 'All About Solidity' series";
    }

}
```

**In other words, you have to specify all base contracts that define the same function**

1. This is best example of inheritance : [https://solidity-by-example.org/inheritance/](https://solidity-by-example.org/inheritance/) . 

*When a function is called that is defined multiple times in different contracts, parent contracts are searched from right to left (contract D is B, C or override(B,C) ⇒ C is rightmost so super refers to this even if B and C are completely different contracts at same level of inheritance.*

1. 

![This is best example of inheritance](https://user-images.githubusercontent.com/93861625/234344434-f8946368-fcc1-4a5d-83e2-b0f31949ce5f.png)

1. After calling selfdestruct, **contract’s code and storage are removed from the state. But it is still part of the history of the blockchain (eg. if we read storage and state trie at specific block numbers, we will find that information)**
2. Someone (= any other contract or address) can still send Ether to a removed contract. In this case, the Ether are forever lost. IDK can a contract be recreated at that address ? see Ethereum Engineering Group’s video
3. **View** functions do not require any gas in order to be executed if : because the information is stored on all nodes and we send the request to the node (via RPC URLs)
- It is called *externally*
- It is called *internally* by another `view` function.

If a **view** function is called *internally* (within the same contract) from another function **which is not a view function,** it will still cost gas. This is because this other function creates a transaction on the Ethereum blockchain, and will need to be verified by every node on the network.

1. Function overloading : when two functions with different parameters exist in the same contract. Overloading must not be confounded with **function overriding**, which happen only with contract inheritance.
- **overriding :** function has the same name, inputs and outputs, but logic is different
- **overloading** : function has the same name, but inputs and outputs and logic are different.
1. Overloading works for inherited functions too. *since only one constructor is allowed in a smart contract, overloading for constructors is not supported.*
2. Overload resolution : **Solidity chooses overload functions by matching the function declarations in the current scope to the argument provided in the function call.** *Return parameters are not taken into account for overload resolution. (basically matching of incoming calldata’s first 4 bytes with function selectors possible in the contract , by the function dispatcher)*
3. Bear in mind that if you use `uint` as a function parameter, like `withdraw(uint amount)` , the formula for function signature will be: (uint is just an alias)

```
keccak256(withdraw(uint256))
```

1. You can get the function selector of the `f()` (any function of a contract) by using `this.f.selector` (or contractType.f.selector in external contracts) in another function. it returns a value of `bytes4`
2. [External functions are more efficient](https://medium.com/@gus_tavo_guim/public-vs-external-functions-in-solidity-b46bcf0ba3ac) than public ones when they receive large arrays of data, because the data is not copied from calldata to memory. Instead, data passed to an `external` function is simply read directly from the calldata. Only `external`functions support exception handling. These exceptions can be caught via `try`and `catch`statements.
3. One Dimensional and Multi-Dimensional arrays can be both of Fixed-Size or Dynamic-Size
- **`T[k]` : One Dimensional, Fixed-size**
- **`T[]` : One Dimensional, Dynamic-size**

Multi-dimensional arrays are essentially nested arrays (An array that contain other arrays). These however, come in three forms in Solidity. We use two-dimensional array in this first example…

- **`T[k][k]` : Two-Dimensional, Fixed-size**
- **`T[][]` : Two-Dimensional, Dynamic-size**
- **`T[][k]` or `T[k][]` : Two-Dimensional, Mixed-size**

… but you will see that **multi-dimensional arrays can have any level of nesting** ! 3,4,5,6 dimensional too

1. Array Slices: Array slices are a view on a contiguous portion of an array. They are written as x[start:end], where start and end are expressions resulting in a uint256 type (or implicitly convertible to it). The first element of the slice is x[start] and the last element is x[end - 1]. 1. Both start and end are optional: start defaults to 0 and end defaults to the length of the array. Array slices are only implemented for calldata : used for [msg.data](http://msg.data) and msg.sig byte arrays. Array slices are useful to ABI-decode secondary data passed in function parameters.
2. Dynamic-size and fixed-size can be mixed in a nested array. ***The maximum level of nesting for nested arrays is 15. If you try to create a variable with 16 nested arrays, you will get a ‘stack too deep’ error.** This is because the stack pointer in the EVM cannot access a slot in the stack that is deeper than is 16th element (counting from the top downwards)*
3. You cannot use the `.push`method in an array that has a fixed length. **You must specify the index. ie. array[i] = value. .push has to be used only with dynamic-size storage arrays. memory arrays have a fixed size and you need to allocate the space previous assigning it.**
4. when accessing elements of multi-dimensional arrays, As opposed to the array definition, the notation is reversed, array access is in the **right to left order. In accessing values, first it goes into the array and then into its specific elements of base element type array instead while defining it goes to first define an individual element and then the actual array**
5. An array literal is ***a comma-separated list of one or more expressions, enclosed in square brackets***( [...] ). For example [1, a, f(3)]  IDK this casting thing for first element? Also dynamic vs fixed-size array declarations ? When you write uint[10] or uint[] = new uint[](5) {memory array} It is fixed size meaning you have to write the length. 

![An array literal is a](https://user-images.githubusercontent.com/93861625/234344494-749adf91-6bb0-436b-a3d3-edf4042df265.png)

1. 

A **memory array** is an array created within a function. To create a memory array, use the **`new`** keyword.

```
function memoryArray(uint len) {

    // memory array of length 7, containing uint
    // x.length == 7
    uint[] memory x = new uint[](7);

    // memory array of length `len`, containing  bytes
    // y.length == len
    bytes[] memory y = new bytes(len);
}
```

1. Note that **you can’t assign a fixed-size memory array to a dynamic size memory array WTF dynamic memory arrays do not exist.** 
2. *You can only change the `.length` of a dynamic-size array if it is referred as `storage` IDK arrays in solidity*
3. When you delete an array, It will automatically create an array of length 0 for dynamic arrays or set each item of the array to 0 for static arrays. If you want to delete just a single item like delete myArray[index] , you should be aware that this will create a gap in the array.
4. *`.push(new_element)` ,* when executed:
- append a new element at the end of the array
- returns nothing

This member function is only available for **dynamic sized storage arrays** and **bytes. IDK bytes expansion using push**

> Note: you can’t use push to append a new element to a string
> 
1. Solidity enables you to create functions that return arrays (both fixed and dynamic size) (all 1,2 and multi dimensional)
2. If an array is returned by a function, the data location specified **always have to be `memory` .**

> NB: you cannot return an array of mappings in Solidity.
> 
1. • It is not possible to use arrays of arrays within external functions.
2. There are 3 ways to assign values to a struct : struct a { uint b; uint c }  a name;

name.b = 1;  name.c = 2;

name = a({b: 1,c: 2});  this is called JSON object notation.

name = a(1,2);

1. The members of a struct can be of any type (except the struct itself, which will give a **recursive struct.** Struct can also contain `array`and `mapping` (including a fixed or dynamic size array of its own struct type)
2. If you assign Struct type to a **local variable** you always have to mention the data location: storage, memory or calldata. (same condition for arrays : storage arrays can only point to already existing arrays, or memory arrays can be created and allocated space; mappings(only storage pointers); )
3. You can use the Solidity built-in function `delete` on a variable of **Struct** type to reset all its members *(This will act like if the variable would be declared as a Struct without assigning any value to its members). **`delete` has no effects on mappings.** So if you delete a struct, it will reset all members that are not mappings and also recurse into the members unless they are mappings.*
4. Ethereum and the EVM is a Virtual Machine that uses the [Big Endian format.](https://github.com/ethereum/solidity-examples/issues/54)
 In the EVM, all data (regardless of its Solidity type) is stored big-endian at the low level inside the virtual machine. • With **Big-Endian,** the ****first byte of binary representation of the multibyte data-type is stored first (whether it is stored in the left or right of allocated space is different for different data types and it is because of padding/alignment rules) Strings are left-aligned whereas numbers are right-aligned. so bytes representing them follow the underlying string/number rule.
5. However, depending on its type ( `bytesN` , `uintN` , `address`, etc…), the data is laid out differently. The sentence *“how data is layout”* refers to **where the high order bits are placed (strings do not have concept of high order bits, they are just stored starting from first character on the leftmost byte of resulting type)**
- **right-padded:** `string` , `bytes` and `bytesN`.
- **left-padded**: `intN` / `uintN` (signed/unsigned integers), `address` and other types.
1. *`byte` is an alias for `bytes1` and therefore stores a single byte. use bytes for arbitrary-length raw byte data*
2. The term `bytes`in Solidity represents a dynamic array of bytes. It’s a shorthand for `byte[]` • You can push(appending a byte at the end), pop and length
3. *use string for arbitrary-length string (UTF-8) data ie. “string” This string literal is interpreted in its raw byte form when assigned to a bytes32 type.*
4. Bytes with a fixed-size variable can be passed between contracts. However, strings can’t be passed between contracts because they are not fixed size variables. Solidity does not have string manipulation functions, but there are third-party string libraries (https://github.com/Arachnid/solidity-stringutils)
5. **Comparison operators**

The following comparison operators applied to bytes evaluate to a `bool` value `true` or `false` .

`<=`, `<`, `==`, `!=`, `>=`, `>` IDK how do these actually compare under the hood ?

1. The following Bit operators are available in Solidity : `&`(**AND),** `|`**(OR),** `^`**(XOR)** and  **(NEGATION).**
2. These bitwise operations, when used with variables result in a final value obtained by the bitwise operation on each bit of the input variables. eg. a & b (write bits of a and b in two rows and then perform the & operation vertically, final bits group together to form a value)
3. & (both 1s is true), | (either of them is 1 then true), xor(both values should be different for true), ~ (This is also called an *inversion operation.* With this operation, 0 becomes 1 and 1 becomes 0.)
4. *The shifting operator works with any integer type as right operand (but returns the type of the left operand), which denotes the number of bits to shift by. Shifting by a negative amount causes a runtime exception.. eg. << x (left shift a no. by x bits);* ``>> **x` (right shift by `x` bits) : All bits are simply shifted to given direction by given no. of bits (eg. 10101010 << 2 = 10101000 and 10101010 >> 2 = 00101010)**
5. Index access(read only) on bytes : *if `x` is of type `bytesI` (where `I` represents an integer), then `x[k]` for 0 <= k < I returns the `k`th byte  ;; (looks like an array access).* Accessing individual bytes by index is possible for all `bytesN` types. The highest order byte(leftmost byte) is found at index 0. `.length` (read-only). : yields the fixed length of the byte array
6. When number is converted to bytesN it gets filled in starting from rightmost bit(lower order).
7. When bytesN type is converted to a larger bytes type, it just adds bytes to the right ie. padding zero bytes.
8. Bytes are actually stored in the order from the highest order byte to lowest byte aligned for the string characters. so 0x616263 = “abc”  where 61 = a, 62 = b, 63 = c ; so each string character(even whitespace) occupies one byte
9. All data types have different layout and way of storing bits like uint has a continuous stream of bits determining the value, string stored in bytes has one character stored in each byte, address is just a hex representation of a number(after converting to binary bits) (requires research on how low level stuff is manipulated during type conversion)
10. There is one difference between byte[] and bytes ie. *The type `byte[]` is an array of bytes, but due to padding rules, it wastes 31 bytes of space for each element (except in storage). It is better to use the `bytes` type instead.*
11. The fixed length `bytes32` can be used in function arguments to pass data in or return data out of a contract. The variable length `bytes` can be used in function arguments also, but only for internal use (inside the same contract), because the interface (ABI) does not support variable length type.
12. To get first N bits of a binary representation(bytes), create a mask of N bits(all 1s) and run the & operation with the binary rep. Take the first N bits of the final value and discard other bits
13. To get last n bits of a binary rep(bytes a), uint8(a) % 2 ** n gives the uint value of last n bits 
14. **Libraries can be seen as implicit base contracts of the contracts that use them. Once deployed on the blockchain (only once), it is assigned a specific address and its properties / methods can be reused many times by other contracts in the Ethereum network.**
15. ****Deploying common code as library will save gas as gas depends on the size of the contract too. Using a base contract instead doesn’t save gas because in solidity inheritance works by copying code. Solidity libraries can be used to add member functions to data types
16. 

![Deploying common code as library](https://user-images.githubusercontent.com/93861625/234344589-2d17021e-dbab-4473-a0e2-2c57fc4f5bb3.png)

1. **Ideally, libraries are not meant to change state of contract, it should only be used to perform simple operations based on input and returns result** However, libraries can still implement some data type : OMG but libraries are used with delegatecall ?
- `struct` and `enum` : these are user-defined variables.
- any other variable defined as `constant` (immutable), since constant
1. Here are two scenarios when deploying libraries:
- **Embedded Library:** If a smart contract is consuming a library which have only **internal functions,** then the EVM simply embeds library into the contract (pulled and hardcoded into code at compile time). Instead of using delegate call to call a function, it simply uses JUMP statement(normal method call). There is no need to separately deploy library in this scenario.
- **Linked Library :** On the flip side, if a library contain **public or external functions** then library needs to be deployed. The deployment of library will generate a unique address in the blockchain. This address needs to be linked with calling contract.
1. To use a library in your contract, first import it. **`using** LibraryName **for** Type` (or * to attach it to all types) . This directive can be use to attach library functions (from the library `LibraryName`) to any type (`Type`).
2. When you call a library function, these functions will receive the object they are called on as their first parameter, like syntax a.transfer(b) so a and b both are parameters
3. Another syntax for this is :
using MathLib for MathLib.MathConstant ; 
4. **If library functions are called, their code is executed in the context of the calling contract. (delegatecall)  WTF many of this info on libraries may be incorrect or outdated.** 
- They will not be explicitly visible in the inheritance hierarchy…
- …but calls to library functions look just like calls to functions of explicit base contracts (`L.f()` if `L` is the name of the library).
1. *Library functions can only be called directly (without the use of `DELEGATECALL`) if they do not modify the state (`view` or `pure` functions)*
2. Despite the fact libraries do not have storage, they can modify their linked contract’s storage, by passing a `storage` argument to their function parameters.(can’t access if not supplied)
3. A library can have functions which are not implemented. a use case is callback functions
4. In the same way that libraries don’t have storage, they don’t have an event log. However, libraries can dispatch events. Any event created by the library will be saved in the event log of the contract that calls the event emitting function in the library. But *the contract ABI does not reflect the events that the libraries it uses may emit.*
5. In a library, • You can use a **mapping** as a parameter for any function visibility : **public, private, external and internal.**
6. It is possible to include libraries inside other libraries. it is possible to use modifiers in libraries. However, we can’t export them in our contract, because modifiers are compile-time features. Therefore, modifiers only apply to the functions of the library itself, and cannot be used by an external contract of the library.
7. 4 main types of runtime errors can be generated in Solidity: `Error(string)`, Panic(uint256), custom `error` and `invalid`
8. 
- `Error(string)` → triggered via the built-in functions `require` and `revert` .
- `Panic(uint256)` → triggered via the built-in function `assert` , or created by the compiler in certain situations.
- custom `error` → triggered via `revert CustomError()` .
- `invalid` opcode → triggered only via assembly.
1. 

![4 main types of runtime](https://user-images.githubusercontent.com/93861625/234344640-f666e59b-0743-4118-8f49-c6f24a7663fd.png)

1. The built-in error `Error(string)` is used for *“revert with error message”*.

An `Error(string)` exception (with an error message provided in the `string` parameter) occurs in the following situations:

1. Calling `require(x, "error message")` where `x` evaluates to `false` , and you have provided an `error message` .
2. If you use `revert()` or `revert("description")`.
3. If you perform an external function call targeting a contract that contains no code.
4. If your contract receives Ether via a public function without a `payable` modifier (including the constructor and the fallback function).
5. If your contract receives Ether via a public getter function.

The provided `string` is abi-encoded as if it were a call to the function `Error(string)` .

1. `Panic(uint256)`**errors should not be present in bug-free code. A Panic error that occurs at runtime in a smart contract already deployed on a network is likely a sign of either a bug in the smart contract code or the smart contract design requirements not being completely or correctly fulfilled.**
2. A `Panic(uint256)` type of error throws an exception with a single argument: a `uint256` number representing a hex error code. The table below list and describes each of these error codes :

![describes each of these eroor codes](https://user-images.githubusercontent.com/93861625/234344690-3d77193c-9dc2-4598-bc1d-d380d86760ed.png)

1. ***Note:** `Panic(uint256)` exceptions used to use the invalid opcode before Solidity 0.8.0, which would consume all the gas available. This is not the case anymore. Since the major release 0.8.x of Solidity, Panic(uint256) errors use the `REVERT` opcode.* 
2. Custom errors can be defined either at the contract or file levels. Passing the named parameters to the error is possible in the same way as with function and events arguments. The named arguments can be written as an object, and the error params can be defined in any order, or do it like function parameters:in order white spaced direct values

```solidity
function send(address receiver, uint amount) public {
    if (amount &gt; balances[msg.sender]) {
        revert InsufficientBalance({
            requested: amount,
            available: balances[msg.sender]
        });
    }
```

Currently, up to solc 0.8.19, it is not possible to use custom errors in combination with `require()`
 . Therefore, to use custom errors, you should use a syntax with an `if`
 statement that evaluates to false: if(bla bla) revert CustomError(paramteres);

1. custom errors are part of the contract JSON ABI generated by the `solc`
 compiler. You can access the bytes4 selector of a custom error using the `.selector`
 syntax the same way as you would access the bytes4 selector of a function.
2. However, it is not possible to access the selector of a custom `error` in inline assembly. Also, you can retrieve the list of custom errors selectors using the `solc` compiler option `--hashes` since Solidity release v0.8.12
3. The `invalid` opcode (instruction `0xfe`) can be used only in inline assembly. It is not available as a built-in function in Solidity.

```solidity
function runInvalid() public {
    assembly {
        invalid()
    }
}
```

It is used in a not so known contract from the OpenZeppelin contracts package: the `MinimalForwarder`

1. The word **Enum** in Solidity stands for **Enumerable**. They are user defined type(like struct) that contain human readable names for a set of constants, called **member. Every index no. of an enum is one-to-one correspondence with the entity declared at that index position.** *Enums are especially useful when we need to specify the current state / flow of a smart contract*
2. ***Note :** you can’t use **numbers** (positive or negative) or **boolean** (true or false in lowercase) as members for an **`enum`**. However, True and False (Capitalised) are accepted. An enum needs at least one member to be defined.*
3. *Enum are explicitly convertible to and from all integer type. The options are represented by subsequent unsigned integer values starting from 0, in the order they are defined. Under the hood, enums are integers, not strings. Solidity will automatically handle converting name to ints. OMG I think it should be uints and not ints*
4. we could create our function using integers instead of enums as function parameters. The syntax would change to myCard.suit = Suit(_value); when value is uint instead of myCard.suit = _value; when value is enum(named Suit) datatype function parameter. (This means if we want uints to be converted and used as enum member, it requires wrapping the uint in an enum type’s Name so it gets assigned the corresponding entity value. 
5. Natspec comments : apart from documentation, they are also used to annotate conditions for formal verification

:@custom - custom tag, semantics is application defined 

@inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)

![they are also used to annotate conditions](https://user-images.githubusercontent.com/93861625/234344913-dd021343-edc2-488e-9b56-806fb252cfe5.png)

1. *NatSpec currently only interprets tags on functions if they are **external** or **public**. You can still use similar comments for your internal and private functions, but those will not be parsed.*
2. If no tags are used (only `///`), then the comment applies to `@notice` [single line = ///, multi line = /** …*/]  in natspec, whereas in normal solidity - // for single line and /* … */ for multi line
3. 
- `@param` **must** be followed by the name of the variable(s) passed as the function argument(s).
- `@return` can be followed by either the **variable name** or the **variable type** returned by the function.

multiple param and return tags for multiple parameters / returned values (one for each)

1. The Solidity compiler automatically generates a JSON file, the contract metadata, that contains information about the current contract. This JSON file includes the compiler version, ABI, but **also the NatSpec documentation**
2. The Solidity compiler `solc` can produces two type of documentation file (in JSON format) after compiling your smart contract.
- **Developer documentation :** to be used by developers > `solc --devdoc my_contract.sol`
- **User documentation :** for the end user as a notice when a function is executed >`solc --userdoc my_contract.sol`
1. A wallet can use the NatSpec user documentation **to display a confirmation message to the user whenever they interact with the contract**
, together with requesting authorization for the transaction signature.
2. In function documentation, you may use dynamic expressions for all natspec tags (based on actual parameter values of the current message call). Use any Javascript/Paperscript expression encapsulated in backticks. This script will be run on a EVM Javascript environment that has access to `message` and all parameters.
3. Use atom plugin to generate natspec comments automatically - boilerplate documentation
4. In Ethereum, an address relates to a public key. It is linked to its hash identifier. An Ethereum address is the last 20 bytes of the Hash of a given public key, then represented in hexadecimal. The hashing algorithm used is **Keccak-256**.
5. Each contract’s address is derived from the contract creation transaction, as a function of the originating account and nonce. ***NB:** the public key is derived from the private key using ECDSA (Elliptic Curve Digital Signature Algorithm). Ethereum uses the same type of curve that bitcoin does : **secp256k1**.f. Each public key is 64 bytes* [https://ethereum.stackexchange.com/questions/760/how-is-the-address-of-an-ethereum-contract-computed](https://ethereum.stackexchange.com/questions/760/how-is-the-address-of-an-ethereum-contract-computed)
6. Addresses are case-insensitive. All wallets are supposed to accept Ethereum addresses expressed in capital or lowercase characters, without any difference in interpretation. ([Since EIP-55, UpperCase addresses are used to validate a checksum](https://github.com/Ethereum/EIPs/blob/master/EIPS/eip-55.md))
7. In Solidity, from the sender perspective:
- You can send Ether to a variable defined as `address payable`
- You cannot send Ether to a variable defined as `address`
- In a smart contract, You can’t define a contract as address payable type if its code doesnt have a payable receive /fallback function
1. *A statement like `<address>.balance` is considered as reading from the state, **because it accesses the blockchain data**. Therefore, any function in Solidity that return an `<address>.balance` can be defined as `view`*
2. `_address.**transfer()**`
- Reverts on failure and throw an exception on any error.

Under the hood, the `.transfer()` function queries the balance of an address, by applying the property balance, before to send the ether.

1. address.send() returns false on failure, Fails if call stack depth is at 1024 or out of gas
2. call staticcall and delegatecall : this high-level syntax is only available when the target contract’s interface is available at compile time. All the low-level functions define accept one argument: the **raw message**
3. call :

![this high-level syntax  ](https://user-images.githubusercontent.com/93861625/234344975-69724870-c5e6-4754-8695-e589df274e2d.png)

1. using `delegatecall` from A to B not only enable B to overwrite contract A storage. If contract B call another contract C, contract C will see that `msg.sender` is contract A.
2. tx.origin returns the originator of the full call chain(an EOA). block.coinbase() returns the miner of the current block
3. The new ABI coder (v2) is able to encode and decode arbitrarily nested arrays and structs. The set of types supported by the new encoder is a strict superset of the ones supported by the old one. Contracts that use it can interact with ones that do not without limitations. The reverse is possible only as long as the non-abicoder v2 contract does not try to make calls that would require decoding types only supported by the new encoder. 
4. .This abi coder pragma applies to all the code defined in the file where it is activated, regardless of where that code ends up eventually. This means that a contract whose source file is selected to compile with ABI coder v1 can still contain code that uses the new encoder by inheriting it from another contract. This is allowed if the new types are only used internally and not in external function signatures.
5. For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. Immutable variables are evaluated once at construction time. 
6. When a function has multiple return types, the statement return (v0, v1, ..., vn) can be used to return multiple values. The number of components must be the same as the number of return variables and their types have to match, potentially after an implicit conversion. If you use an early return to leave a function that has return variables, you must provide return values together with the return statement 
7. You can either explicitly assign value to named return variables and then leave the function or you can provide return values (either a single or multiple ones) directly with the return statement (even if you have named them)
8. The modifier can choose not to execute the function body at all and in that case the return variables are set to their default values just as if the function had an empty body.
9. Free Functions: Functions that are defined outside of contracts are called “free functions” and always have implicit internal visibility. Their code is included in all contracts that call them, similar to internal library functions.
10. Indexed Event Parameters: Adding the attribute *indexed*
 for up to three parameters adds them to a special data structure known as “topics” instead of the data part of the log. If you use arrays (including string and bytes) as indexed arguments, its Keccak-256 hash is stored as a topic instead, this is because a topic can only hold a single word (32 bytes). All parameters without the *indexed* attribute are ABI-encoded into the data part of the log(with proper padding etc.)
11. The deployed code does not include internal functions only called from the constructor
12. Value Types: Types that are passed by value, i.e. they are always copied when they are used as function arguments or in assignments — Booleans, Integers, Fixed Point Numbers, Address, Contract, Fixed-size Byte Arrays (bytes1, bytes2, …, bytes32), Literals (Address, Rational, Integer, String, Unicode, Hexadecimal), Enums, Functions.
13. Reference Types: Types that can be modified through multiple different names. Arrays (including Dynamically-sized bytes array *bytes* and *string),* Structs, Mappings. Only the reference types have data location definitions 
14. For dynamically-sized arrays, bytes and string, the default value is an empty array or string. For the enum type, the default value is its first member.
15. in the expression f(x) || g(y), if f(x) evaluates to true, g(y) will not be evaluated even if it may have side-effects. This is called the common short-circuiting rule
16. address payable conversion via payable(address) : For contract-type, this conversion is only allowed if the contract can receive Ether, i.e., the contract either has a receive or a payable fallback function.
17. Contract Type: Every contract defines its own type. Contracts can be explicitly converted to and from the *address* type. Contract types do not support any operators. The members of contract types are the *external* functions of the contract(meaning we can access them as Contract.func(…) ) including any state variables marked as public.
18. String Literals: String literals are written with either double or single-quotes ("foo" or ‘bar’, similar for hex “.. “ or hex’ .. ‘). They can only contain printable ASCII characters and a set of escape characters
19. Unicode Literals: Unicode literals prefixed with the keyword *unicode* can contain any valid UTF-8 sequence. They also support the very same escape sequences as regular string literals.
20. 
- *push()*: appends a zero-initialised element at the end of the array and returns a reference to the element
- *push(x)*: appends a given element at the end of the array and returns nothing
1. delete
- delete a assigns the initial value for the type to a
- For integers it is equivalent to a = 0
- For arrays, it assigns a dynamic array of length zero or a static array of the same length with all elements set to their initial value
- *delete a[x]* deletes the item at index x(set to “zero” ) of the array and leaves all other elements and the length of the array untouched
- For structs, it assigns a struct with all members reset
1. 
- *blockhash(uint blockNumber)* returns *(bytes32)*: hash of the given block - only works for 256 most recent, excluding current, blocks, values for all other blocks will be zero.
- *block.chainid (uint)*: current chain id
- *block.coinbase (address payable)*: current block miner’s address
- *block.difficulty (uint)*: current block difficulty
- *block.gaslimit (uint)*: current block gaslimit
- *tx.gasprice (uint)*: gas price of the transaction
- *gasleft() returns (uint256)*: remaining gas
1. For an interface type I, the following type information is available: *type(I).interfaceId*: A bytes4 value containing the EIP-165 interface identifier of the given interface I. This identifier is defined as the XOR of all function selectors defined within the interface itself - excluding all inherited functions. Also type(contract).name : The name of the contract
2. for type(C).creationcode and .runtimecode : This property cannot be accessed in the contract itself or any derived contract. It causes the bytecode to be included in the bytecode of the call site and thus circular references like that are not possible.
3. The *try* keyword has to be followed by an expression representing an external function call or a contract creation (*new ContractName()* ). Errors inside the expression are not caught (for example if it is a complex expression that also involves internal function calls), only a revert happening inside the external call itself.
4. Solidity supports different kinds of catch blocks depending on the type of error:
- *catch Error(string memory reason) { ... }*: This catch clause is executed(note that this is only when these catch statements are defined in else clauses ) if the error was caused by revert("reasonString") or require(false, "reasonString") (or an internal error that causes such an exception).
- *catch Panic(uint errorCode) { ... }*: If the error was caused by a panic, i.e. by a failing assert, division by zero, invalid array access, arithmetic overflow and others, this catch clause will be run.
- *catch (bytes memory lowLevelData) { ... }*: This clause is executed if the error signature does not match any other clause, if there was an error while decoding the error message, or if no error data was provided with the exception. The declared variable provides access to the low-level error data in that case.
- *catch { ... }*: If you are not interested in the error data, you can just use *catch { ... }* (even as the only catch clause) instead of the previous clause.
1. Do not assume that the error message is coming directly from the called contract: The error might have happened deeper down in the call chain and the called contract just forwarded it.
2. State Variable Shadowing: This is considered as an error. A derived contract can only declare a state variable x, if there is no visible state variable with the same name in any of its bases.
3. Function overriding can only change visibilities from public to external and state mutabilities to a stricter one. payable is an exception and cannot be changed to any other mutability.
4. Functions without implementation have to be marked virtual outside of interfaces. Public state variables can override external functions if the parameter and return types of the function matches the getter function of the variable. Function modifiers can override each other.
5. Name Collision Error: It is an error when any of the following pairs in a contract have the same name due to inheritance: 1) a function and a modifier 2) a function and an event 3) an event and a modifier. Because selectors would mix up
6. In storage layout packing, the variables are stored in order right to left (lower-order aligned)
7. The elements of structs and arrays are stored after each other, just as if they were given as individual variable values.

Topics confusing :

1. Exact storage layout of complex data types and their packing 
2. Internal bit-layout of types (which byte of data is stored left or right aligned)
3. What happens to bit-layout/alignment on conversion between types
