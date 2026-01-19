# Solidity & Blockchain Interview Questions

## Table of Contents
1. [Ethereum & Smart Contract Basics](#ethereum--smart-contract-basics)
2. [Solidity Language Fundamentals](#solidity-language-fundamentals)
3. [Data Types & Collections](#data-types--collections)
4. [Structs & Enums](#structs--enums)
5. [Function Visibility & Modifiers](#function-visibility--modifiers)
6. [Memory Locations & Gas](#memory-locations--gas)
7. [Development Environment](#development-environment)
8. [Wallets & Networks](#wallets--networks)
9. [Inheritance & Code Reuse](#inheritance--code-reuse)
10. [Advanced Types & Solidity Changes](#advanced-types--solidity-changes)
11. [Gas Optimization](#gas-optimization)
12. [Contract Interactions](#contract-interactions)
13. [Events & Logging](#events--logging)
14. [Access Control Patterns](#access-control-patterns)
15. [Advanced Security](#advanced-security)
16. [Libraries](#libraries)
17. [Hashing & Randomness](#hashing--randomness)
18. [Assembly in Solidity](#assembly-in-solidity)

---

## Ethereum & Smart Contract Basics

### Q1: What is an Ethereum smart contract?
**Answer:**
A smart contract is a small program that runs on the Ethereum blockchain. It's self-executing code with the terms of agreement directly written into lines of code.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleStorage {
    uint256 private data;
    
    function set(uint256 _data) public {
        data = _data;
    }
    
    function get() public view returns (uint256) {
        return data;
    }
}
```

### Q2: What makes an Ethereum smart contract so special compared to other programs?
**Answer:**
Several unique properties make smart contracts special:

| Property | Description |
|----------|-------------|
| **Unstoppable** | Once deployed, cannot be stopped - even by the creator, government, or Ethereum developers |
| **Immutable Code** | The code cannot be modified after deployment |
| **Hack Resistant** | Cannot be hacked if the code is correct (bugs are the vulnerability) |
| **Transparent** | Code is visible and verifiable on the blockchain |
| **Trustless** | No need to trust a third party; the code enforces the rules |

**Important Distinction**: While the code is immutable, the **data** (state) of the smart contract CAN be modified through function calls.

### Q3: Can a smart contract interact with other smart contracts?
**Answer:**
Yes, and this is what makes smart contracts so powerful. Any smart contract can call functions on other smart contracts, triggering a chain of executions through the blockchain. This enables composability - building complex decentralized applications from simpler building blocks.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IExternalContract {
    function getValue() external view returns (uint256);
}

contract Caller {
    function callExternal(address _contractAddress) public view returns (uint256) {
        IExternalContract externalContract = IExternalContract(_contractAddress);
        return externalContract.getValue();
    }
}
```

### Q4: Can an Ethereum smart contract call an API on the web?
**Answer:**
**No.** A smart contract can only execute its own code and interact with other smart contracts on the Ethereum blockchain. Smart contracts cannot make HTTP requests or access external data directly.

**Solution - The Oracle Pattern:**
If you need external data in your smart contract, use the Oracle pattern where an external API calls a function on your smart contract and fills it with data.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract PriceOracle {
    uint256 public ethPrice;
    address public oracle;
    
    constructor() {
        oracle = msg.sender;
    }
    
    // Only the oracle can update the price (called from off-chain)
    function updatePrice(uint256 _price) external {
        require(msg.sender == oracle, "Only oracle can update");
        ethPrice = _price;
    }
}
```

**Key Point**: Data flow is always from the outside world TO the blockchain, never the other way around.

### Q5: Can a smart contract store a lot of data?
**Answer:**
**No, not practically.** In a smart contract, storing data costs gas, and gas consumption is capped in each Ethereum block. This creates two limitations:

1. **Technical Limit**: Storage is indirectly limited by block gas limits
2. **Economic Limit**: Since storing data costs money (gas), you probably don't want to store large amounts anyway

**Best Practice**: Store only essential data on-chain. Use off-chain solutions (IPFS, Arweave) for large data and store only hashes on-chain for verification.

### Q6: Can a smart contract be written in another language than Solidity?
**Answer:**
Yes, there are other smart contract languages:

| Language | Description |
|----------|-------------|
| **Solidity** | Most popular, JavaScript-like syntax |
| **Vyper** | Python-like syntax, focuses on security and simplicity |
| **LLL** | Low-Level Lisp-like Language, very low-level |
| **Yul** | Intermediate language, can be compiled to bytecode |
| **Fe** | Rust-inspired language, still in development |

However, **Solidity is by far the most popular** with the largest ecosystem, most documentation, and best tooling support.

---

## Solidity Language Fundamentals

### Q7: Is Solidity a dynamically or statically typed language?
**Answer:**
Solidity is a **statically typed language**, which means variable types must be defined at compile time.

| Statically Typed (Solidity) | Dynamically Typed (JavaScript) |
|-----------------------------|-------------------------------|
| Types defined at compile time | Types determined at runtime |
| More verbose but safer | Less verbose but more error-prone |
| Errors caught at compilation | Errors found at runtime |

**Why Static Typing for Blockchain?**
Since smart contracts are immutable after deployment, bugs are extremely costly. Static typing helps catch errors before deployment, reducing the risk of costly mistakes.

```solidity
// Solidity - must declare types
uint256 amount = 100;
address recipient = 0x123...;
string memory name = "Token";

// JavaScript - no type declaration needed
// let amount = 100;
// let recipient = "0x123...";
```

### Q8: Is Solidity compiled or interpreted?
**Answer:**
Solidity is **compiled**. Before running a smart contract, you must first compile it to EVM bytecode.

| Compiled (Solidity) | Interpreted (JavaScript) |
|--------------------|-----------------------|
| Code compiled before execution | Code compiled on-the-fly (JIT) |
| Results in bytecode | Direct execution |
| Compilation step required | Run immediately after writing |

The Solidity compiler (`solc`) transforms human-readable Solidity code into EVM bytecode that the Ethereum Virtual Machine can execute.

### Q9: What is the file extension of Solidity files?
**Answer:**
The file extension is **`.sol`**

```
MyContract.sol
Token.sol
Voting.sol
```

### Q10: Can a single Solidity file have several smart contracts?
**Answer:**
**Yes.** You can define multiple contracts in a single file using the `contract` keyword multiple times.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// First contract
contract TokenA {
    string public name = "Token A";
}

// Second contract in same file
contract TokenB {
    string public name = "Token B";
}

// Third contract
contract TokenFactory {
    function createTokenA() public returns (TokenA) {
        return new TokenA();
    }
}
```

**Best Practice**: For organization, it's generally better to have one contract per file, but when deploying, it's often more convenient to combine related contracts.

### Q11: What is the typical layout of a Solidity smart contract?
**Answer:**
A Solidity smart contract follows this structure:

```solidity
// SPDX-License-Identifier: MIT
// 1. License identifier (required since Solidity 0.6.8)

pragma solidity ^0.8.0;
// 2. Pragma statement - specifies compiler version

import "./OtherContract.sol";
// 3. Import statements (optional)

// 4. Contract declaration
contract MyContract {
    // 5. State variables
    uint256 public data;
    address public owner;
    
    // 6. Events
    event DataChanged(uint256 newValue);
    
    // 7. Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    // 8. Constructor
    constructor() {
        owner = msg.sender;
    }
    
    // 9. Functions
    function setData(uint256 _data) public onlyOwner {
        data = _data;
        emit DataChanged(_data);
    }
    
    function getData() public view returns (uint256) {
        return data;
    }
}
```

### Q12: What is the difference between state and local variables?
**Answer:**

| State Variables | Local Variables |
|-----------------|-----------------|
| Declared at contract level | Declared inside functions |
| Persisted on the blockchain | Exist only during function execution |
| Cost gas to store and modify | Cheaper, stored in memory/stack |
| Accessible by all functions | Only accessible within their scope |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract VariableExample {
    // State variable - stored on blockchain
    uint256 public stateVar = 100;
    
    function example() public view returns (uint256) {
        // Local variable - exists only during this function call
        uint256 localVar = 50;
        return stateVar + localVar;
    }
}
```

### Q13: What is the problem with the following Solidity code?
```solidity
contract Problem {
    address data;
    
    function set() public {
        address data = msg.sender;
    }
}
```

**Answer:**
The `set` function redefines the `data` variable inside its body using `address data = ...`. This creates a **local variable that shadows the state variable** defined above.

The state variable `data` will never be modified because the function is assigning to a new local variable with the same name.

**Fix:** Remove the `address` keyword when referencing `data` inside the function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fixed {
    address data;
    
    function set() public {
        data = msg.sender; // Now modifies state variable
    }
}
```

### Q14: What is the problem with this code where argument shadows state variable?
```solidity
contract Problem {
    uint256 data;
    
    function set(uint256 data) public {
        data = data;
    }
}
```

**Answer:**
The function argument `data` **shadows the state variable** `data`. Inside the function, `data` refers to the argument, not the state variable, making it impossible to access the state variable.

**Fix:** Use a different name for the argument, conventionally prefixed with underscore:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fixed {
    uint256 data;
    
    function set(uint256 _data) public {
        data = _data; // Now correctly assigns to state variable
    }
}
```

### Q15: What are the two variable visibilities for state variables in Solidity?
**Answer:**
The two main visibilities are **private** and **public**:

| Visibility | Description |
|------------|-------------|
| **private** | Only accessible by functions inside the same contract |
| **public** | Accessible by anyone; automatically creates a getter function |
| **internal** | Like private, but also accessible by derived contracts |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Visibility {
    uint256 private privateVar = 1;   // Only this contract
    uint256 public publicVar = 2;     // Anyone can read
    uint256 internal internalVar = 3; // This contract + children
}
```

### Q16: What is the default visibility of state variables?
**Answer:**
The default visibility is **internal** (not private as commonly mistaken). However, best practice is to always explicitly declare visibility.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DefaultVisibility {
    uint256 data; // internal by default - bad practice
    uint256 internal data2; // explicit - good practice
}
```

### Q17: Are private variables really private?
**Answer:**
**No!** Private variables are only private to the EVM (Ethereum Virtual Machine) - meaning they can't be accessed by other smart contracts.

However, **all data on the blockchain is public**. Anyone can:
- Use tools to analyze blockchain data
- Read any storage slot of any contract
- Access "private" variables with special tools

```javascript
// Using ethers.js to read private variable
const value = await provider.getStorageAt(contractAddress, 0);
```

**How to Handle Truly Private Data:**
1. **Don't put it on-chain** - store sensitive data off-chain
2. **Use hashes** - store hash of data instead of raw data
3. **Encryption** - encrypt data before storing (but key management is complex)

### Q18: What data types do you use often in Solidity and why?
**Answer:**

| Type | Use Case | Example |
|------|----------|---------|
| **uint256** | Ether/token amounts, counters | `uint256 balance = 1 ether;` |
| **address** | Identifying users and contracts | `address owner = msg.sender;` |
| **string** | Names, metadata | `string name = "MyToken";` |
| **bool** | Flags, conditions | `bool isActive = true;` |
| **bytes32** | Hashes, fixed-size data | `bytes32 hash = keccak256(...);` |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CommonTypes {
    uint256 public totalSupply;           // Token amounts
    address public owner;                  // User/contract addresses
    string public name;                    // Human-readable names
    bool public paused;                    // Feature flags
    bytes32 public merkleRoot;            // Cryptographic hashes
    mapping(address => uint256) balances; // Complex associations
}
```

---

## Data Types & Collections

### Q19: What are the two container types in Solidity?
**Answer:**
The two container types are **mappings** and **arrays**:

| Mappings | Arrays |
|----------|--------|
| Key-value store | Ordered list |
| O(1) lookup | O(n) for search, O(1) for index access |
| Cannot iterate | Can iterate |
| No length property | Has length property |
| All keys "exist" (default values) | Only defined indices exist |

### Q20: How to declare an array of integers in Solidity?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ArrayExample {
    // Dynamic array - can grow/shrink
    uint256[] public dynamicArray;
    
    // Fixed-size array
    uint256[10] public fixedArray;
    
    // Array with initial values
    uint256[] public initializedArray = [1, 2, 3];
}
```

### Q21: How to declare a mapping of address to boolean in Solidity?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MappingExample {
    // address is the key, bool is the value
    mapping(address => bool) public whitelist;
    
    function addToWhitelist(address _user) public {
        whitelist[_user] = true;
    }
    
    function isWhitelisted(address _user) public view returns (bool) {
        return whitelist[_user];
    }
}
```

### Q22: How to declare a nested mapping (mapping of address to mapping of address to boolean)?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract NestedMapping {
    // First key: owner address
    // Second key: spender address  
    // Value: approval status
    mapping(address => mapping(address => bool)) public approvals;
    
    function approve(address _spender) public {
        approvals[msg.sender][_spender] = true;
    }
    
    function isApproved(address _owner, address _spender) public view returns (bool) {
        return approvals[_owner][_spender];
    }
}
```

### Q23: How to add data to an array declared as a state variable?
**Answer:**
Use the `push` method:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ArrayPush {
    uint256[] public numbers;
    
    function addNumber(uint256 _num) public {
        numbers.push(_num); // Adds to end of array
    }
    
    function removeLastNumber() public {
        numbers.pop(); // Removes from end of array
    }
}
```

### Q24: How to add data to a mapping declared as a state variable?
**Answer:**
Use bracket notation:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MappingAdd {
    mapping(address => uint256) public balances;
    
    function setBalance(address _user, uint256 _amount) public {
        balances[_user] = _amount; // Bracket notation
    }
}
```

### Q25: How to loop through an array?
**Answer:**
Use a `for` loop:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ArrayLoop {
    uint256[] public numbers;
    
    function sum() public view returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < numbers.length; i++) {
            total += numbers[i];
        }
        
        return total;
    }
}
```

**Warning**: Be careful with unbounded loops - they can run out of gas!

### Q26: What is the difference between uint8 and uint16?
**Answer:**

| Type | Bits | Max Value | Use Case |
|------|------|-----------|----------|
| uint8 | 8 bits | 255 (2^8 - 1) | Small numbers, flags |
| uint16 | 16 bits | 65,535 (2^16 - 1) | Medium numbers |
| uint256 | 256 bits | ~1.15 × 10^77 | Default, large numbers |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract UintTypes {
    uint8 public small = 255;      // Max: 255
    uint16 public medium = 65535;  // Max: 65,535
    uint256 public large = type(uint256).max; // Huge number
}
```

**Note**: `uint` is an alias for `uint256`.

---

## Structs & Enums

### Q27: What are the two ways of defining custom data structures in Solidity?
**Answer:**
**Structs** and **Enums**:

| Structs | Enums |
|---------|-------|
| Complex data with multiple fields | Variants of the same data type |
| Like objects in JavaScript | Like a fixed set of options |
| Can contain any types | Integer-based under the hood |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CustomTypes {
    // Struct - complex data structure
    struct User {
        uint256 id;
        string name;
        bool active;
    }
    
    // Enum - variants
    enum Status {
        Pending,   // 0
        Active,    // 1
        Completed, // 2
        Cancelled  // 3
    }
    
    User public user;
    Status public status;
}
```

### Q28: When would you use a struct versus an enum?
**Answer:**

**Use Struct when:**
- Representing complex data with different fields
- Grouping related data together
- Creating object-like structures

**Use Enum when:**
- Creating variants of the same concept
- Defining a fixed set of states
- Improving code readability for status/type values

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StructVsEnum {
    // Enum for order status (variants of same concept)
    enum OrderStatus { Pending, Shipped, Delivered, Cancelled }
    
    // Struct for order data (multiple different fields)
    struct Order {
        uint256 id;
        address buyer;
        uint256 amount;
        OrderStatus status;  // Enum can be a field in struct!
    }
}
```

### Q29: What are the two ways to instantiate a struct?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StructInstantiation {
    struct User {
        uint256 id;
        string name;
        bool active;
    }
    
    User public user1;
    User public user2;
    
    function createUsers() public {
        // Method 1: Positional arguments (order matters!)
        user1 = User(1, "Alice", true);
        
        // Method 2: Named arguments (order doesn't matter)
        user2 = User({
            name: "Bob",
            active: false,
            id: 2
        });
    }
}
```

### Q30: How to instantiate a struct that has an inner mapping?
**Answer:**
This is tricky because **mappings cannot be instantiated in memory**. You must reference the storage location directly:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StructWithMapping {
    struct User {
        uint256 id;
        string name;
        mapping(address => bool) friends;
    }
    
    mapping(uint256 => User) public users;
    
    function createUser(uint256 _id, string memory _name) public {
        // Can't do: users[_id] = User(_id, _name, ???)
        // Instead, reference storage and assign fields:
        
        User storage newUser = users[_id];  // Reference non-existent entry
        newUser.id = _id;
        newUser.name = _name;
        // Mapping is automatically initialized
        
        // Add a friend
        newUser.friends[msg.sender] = true;
    }
}
```

### Q31: When would you use an array versus a mapping?
**Answer:**

| Use Array When | Use Mapping When |
|----------------|------------------|
| Need to iterate through all values | Need quick lookup by key |
| Order matters | Order doesn't matter |
| Need to know total count | Don't need to count entries |
| Data set is small | Data set is large |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ArrayVsMapping {
    // Array - when you need to iterate
    address[] public participants;
    
    // Mapping - when you need quick lookup
    mapping(address => uint256) public balances;
    
    // Common pattern: use BOTH for different needs
    address[] public userList;
    mapping(address => bool) public isUser;
    
    function addUser(address _user) public {
        require(!isUser[_user], "Already exists");
        userList.push(_user);
        isUser[_user] = true;
    }
}
```

### Q32: How to combine array and mapping to allow both iteration and rapid lookup?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CombinedPattern {
    struct User {
        uint256 id;
        string name;
    }
    
    // Array for iteration - stores all user IDs
    uint256[] public userIds;
    
    // Mapping for rapid lookup - ID to User struct
    mapping(uint256 => User) public users;
    
    function createUser(uint256 _id, string memory _name) public {
        users[_id] = User(_id, _name);
        userIds.push(_id);
    }
    
    // Quick lookup by ID - O(1)
    function getUser(uint256 _id) public view returns (User memory) {
        return users[_id];
    }
    
    // Iterate through all users
    function getAllUsers() public view returns (User[] memory) {
        User[] memory allUsers = new User[](userIds.length);
        
        for (uint256 i = 0; i < userIds.length; i++) {
            allUsers[i] = users[userIds[i]];
        }
        
        return allUsers;
    }
}
```

### Q33: How to define an in-memory array of fixed size?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemoryArray {
    function createArray() public pure returns (uint256[] memory) {
        // Declare and instantiate with fixed length
        uint256[] memory numbers = new uint256[](3);
        
        // Assign values by index (no push for memory arrays!)
        numbers[0] = 10;
        numbers[1] = 20;
        numbers[2] = 30;
        
        return numbers;
    }
}
```

**Key Point**: Memory arrays must have a fixed length at creation time and cannot use `push()`.

### Q34: How to add values to an in-memory array?
**Answer:**
You cannot use `push` with memory arrays. Instead, assign to specific indices:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemoryArrayValues {
    function fillArray(uint256 _size) public pure returns (uint256[] memory) {
        uint256[] memory arr = new uint256[](_size);
        
        for (uint256 i = 0; i < _size; i++) {
            arr[i] = i * 10; // Assign by index, not push!
        }
        
        return arr;
    }
}
```

### Q35: How to get the list of all keys in a mapping?
**Answer:**
**It's not possible!** Mappings don't track their keys. Unlike JavaScript's `Object.keys()`, Solidity mappings don't provide a way to enumerate keys.

**Solutions:**
1. Maintain a separate array of keys
2. Know the keys beforehand
3. Use events to log key additions

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TrackableMapping {
    mapping(address => uint256) public balances;
    address[] public keys; // Maintain keys separately
    mapping(address => bool) public keyExists;
    
    function setBalance(address _user, uint256 _amount) public {
        if (!keyExists[_user]) {
            keys.push(_user);
            keyExists[_user] = true;
        }
        balances[_user] = _amount;
    }
    
    function getAllKeys() public view returns (address[] memory) {
        return keys;
    }
}
```

### Q36: How to create an in-memory mapping?
**Answer:**
**It's not possible!** Mappings can only exist in storage, not memory.

```solidity
// This will NOT compile:
// mapping(address => uint256) memory tempMap;

// Workaround: use arrays for temporary key-value needs
```

### Q37: What happens if you access a key of a mapping that does not exist?
**Answer:**
Solidity returns the **default value** for that type. No error is thrown!

| Type | Default Value |
|------|---------------|
| uint | 0 |
| bool | false |
| address | 0x0000...0000 |
| string | "" (empty) |
| bytes | 0x (empty) |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DefaultValues {
    mapping(uint256 => bool) public flags;
    mapping(address => uint256) public balances;
    
    function checkDefaults() public view returns (bool, uint256) {
        // These keys don't exist, but no error!
        bool flag = flags[999];        // Returns false
        uint256 bal = balances[address(0x1)]; // Returns 0
        
        return (flag, bal);
    }
}
```

**Contrast with Arrays:** Accessing a non-existent array index WILL throw an error!

---

## Function Visibility & Modifiers

### Q38: What are the four function visibilities in Solidity?
**Answer:**
From most restrictive to most permissive:

| Visibility | Callable From | Use Case |
|------------|--------------|----------|
| **private** | Same contract only | Internal helpers |
| **internal** | Same contract + derived contracts | Inherited functionality |
| **external** | Outside only (other contracts, EOAs) | API functions |
| **public** | Anywhere (inside + outside) | General purpose |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Visibilities {
    // Only this contract
    function privateFunc() private pure returns (string memory) {
        return "private";
    }
    
    // This contract + children
    function internalFunc() internal pure returns (string memory) {
        return "internal";
    }
    
    // Outside only - cannot be called internally without `this.`
    function externalFunc() external pure returns (string memory) {
        return "external";
    }
    
    // Anywhere
    function publicFunc() public pure returns (string memory) {
        return "public";
    }
    
    function testCalls() public view {
        privateFunc();  // OK
        internalFunc(); // OK
        // externalFunc(); // ERROR - can't call directly
        this.externalFunc(); // OK - but costs more gas
        publicFunc();   // OK
    }
}
```

### Q39: How to conditionally throw an error with a message?
**Answer:**
Use the `require` statement:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract RequireExample {
    mapping(address => uint256) public balances;
    
    function withdraw(uint256 _amount) public {
        // First argument: condition to test
        // Second argument: error message if condition fails
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        require(_amount > 0, "Amount must be positive");
        
        balances[msg.sender] -= _amount;
        payable(msg.sender).transfer(_amount);
    }
}
```

**Other Error Handling:**
- `require(condition, message)` - Input validation, external conditions
- `revert(message)` - Manual revert with message
- `assert(condition)` - Internal invariants (no message in older versions)

### Q40: What are the two artifacts produced by the Solidity compiler?
**Answer:**
The **ABI** and the **Bytecode**:

| Artifact | Purpose |
|----------|---------|
| **ABI (Application Binary Interface)** | JSON describing the contract interface |
| **Bytecode** | Compiled code that runs on the EVM |

```json
// ABI Example (simplified)
[
  {
    "type": "function",
    "name": "transfer",
    "inputs": [
      {"name": "to", "type": "address"},
      {"name": "amount", "type": "uint256"}
    ],
    "outputs": [{"type": "bool"}]
  }
]
```

### Q41: What is the ABI of a smart contract?
**Answer:**
The ABI (Application Binary Interface) defines the **interface** of a smart contract - the set of functions that can be called from outside.

**ABI Contains:**
- Function names
- Function argument types
- Return types
- Event definitions

**ABI Does NOT Contain:**
- Function implementations
- Internal logic
- Private/internal functions

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ABIExample {
    event Transfer(address indexed from, address indexed to, uint256 value);
    
    function foo() public pure returns (uint256) { // In ABI
        return 42;
    }
    
    function bar(uint256 x) external view returns (bool) { // In ABI
        return x > 0;
    }
    
    function _internal() internal pure {} // NOT in ABI
    function _private() private pure {}   // NOT in ABI
}
```

### Q42: In the following contract, which functions will be part of the ABI?

```solidity
contract Example {
    function foo() public pure returns (uint256) { return 1; }
    function bar() external pure returns (uint256) { return 2; }
    function baz() internal pure returns (uint256) { return 3; }
}
```

**Answer:**
Only `foo()` and `bar()` will be in the ABI.

- `foo()` - **Yes** (public)
- `bar()` - **Yes** (external)
- `baz()` - **No** (internal - can only be called from inside)

### Q43: Does the EVM understand Solidity?
**Answer:**
**No!** The EVM only understands **bytecode**. Solidity must be compiled to bytecode outside the blockchain before deployment.

```
Solidity (.sol) --> Compiler (solc) --> Bytecode --> EVM
```

### Q44: What is EVM bytecode?
**Answer:**
EVM bytecode is a series of **elementary instructions called opcodes**. These opcodes define very simple operations.

**Examples of Opcodes:**
| Opcode | Description |
|--------|-------------|
| ADD | Add two numbers |
| MUL | Multiply two numbers |
| SSTORE | Store to storage |
| SLOAD | Load from storage |
| CALL | Call another contract |
| RETURN | Return data |

There are over 100 opcodes defined in the Ethereum Yellow Paper. Coding directly in opcodes would be extremely tedious, which is why high-level languages like Solidity exist.

### Q45: What are the two APIs used to interact with a smart contract?
**Answer:**

| API | Cost | Modifies Blockchain | Returns Value |
|-----|------|---------------------|---------------|
| **eth_sendTransaction** | Costs gas | Yes | Transaction hash only |
| **eth_call** | Free | No | Function return value |

```javascript
// Using ethers.js

// Transaction - costs gas, modifies state
await contract.transfer(recipient, amount); // Uses eth_sendTransaction

// Call - free, read-only
const balance = await contract.balanceOf(address); // Uses eth_call
```

---

## Memory Locations & Gas

### Q46: What is gas?
**Answer:**
Gas is an **abstract unit to measure transaction cost**. Every operation in the EVM has a gas cost.

**Key Points:**
- Gas measures computational effort
- Gas prevents infinite loops and spam
- Different operations cost different amounts of gas
- Storage operations are most expensive

### Q47: How is gas paid?
**Answer:**
Gas is paid in Ether using the formula:

```
Total Cost = Gas Used × Gas Price
```

| Component | Description |
|-----------|-------------|
| **Gas Used** | Actual computation cost of the transaction |
| **Gas Price** | Amount of Wei per gas unit (set by sender) |
| **Gas Limit** | Maximum gas the transaction can consume |

```solidity
// Example: If a function uses 50,000 gas
// And gas price is 20 Gwei (20 × 10^9 Wei)
// Cost = 50,000 × 20 × 10^9 = 1,000,000 Gwei = 0.001 ETH
```

### Q48: What happens if there is not enough gas in a transaction?
**Answer:**
The transaction is **stopped and all state changes are reverted**. However, the gas that was consumed up to that point is NOT refunded - the sender still pays for the failed attempt.

This is why setting an appropriate gas limit is important:
- Too low: Transaction fails, you lose gas
- Too high: Unused gas is refunded, but ties up funds temporarily

### Q49: Who pays for gas in a transaction?
**Answer:**
The **sender of the transaction** pays for gas. This is always an EOA (Externally Owned Account), not a contract.

When Contract A calls Contract B:
- The original transaction sender pays all gas
- Contract A doesn't pay; it doesn't have "gas"

### Q50: What are the four memory locations in Solidity?
**Answer:**

| Location | Persistence | Cost | Use Case |
|----------|-------------|------|----------|
| **storage** | Permanent (on-chain) | Expensive | State variables |
| **memory** | Temporary (function call) | Moderate | Function parameters, local variables |
| **stack** | Temporary (operation) | Cheap | Simple computations |
| **calldata** | Temporary (read-only) | Cheapest | External function inputs |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemoryLocations {
    // Storage - persists between calls
    uint256[] public storageArray;
    
    // Calldata - read-only, cheapest for external inputs
    function processData(uint256[] calldata _data) external {
        // Memory - temporary, modifiable
        uint256[] memory tempArray = new uint256[](_data.length);
        
        for (uint256 i = 0; i < _data.length; i++) {
            tempArray[i] = _data[i] * 2;
        }
    }
}
```

### Q51: What is the difference between address and address payable?
**Answer:**

| address | address payable |
|---------|-----------------|
| Cannot receive Ether | Can receive Ether |
| No `transfer` or `send` methods | Has `transfer` and `send` methods |
| Use for general addresses | Use when you need to send Ether |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AddressTypes {
    address public regularAddress;
    address payable public payableAddress;
    
    function sendEther(address payable _to) public payable {
        _to.transfer(msg.value); // Only works with payable
    }
    
    function convertToPayable(address _addr) public pure returns (address payable) {
        return payable(_addr); // Explicit conversion
    }
}
```

**Important**: This is only a Solidity distinction. On the Ethereum network, all addresses are the same.

### Q52: Do you need address payable to send ERC-20 tokens?
**Answer:**
**No!** You only need `address payable` to send **Ether**. ERC-20 tokens are transferred via contract function calls, not direct Ether transfers.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
}

contract TokenSender {
    function sendTokens(
        address _token,      // Regular address for token contract
        address _recipient,  // Regular address for recipient
        uint256 _amount
    ) external {
        IERC20(_token).transfer(_recipient, _amount);
    }
}
```

---

## Development Environment

### Q53: What is the most famous IDE for Solidity?
**Answer:**
**Remix** is the most famous IDE for Solidity development. It's a browser-based IDE that requires no installation.

**Features:**
- Browser-based (no installation)
- Built-in Solidity compiler
- In-memory blockchain for testing
- Debugger
- Static analysis tools
- Plugin system

**Access**: https://remix.ethereum.org

### Q54: Name two famous frameworks for Solidity smart contract development.
**Answer:**

| Framework | Description |
|-----------|-------------|
| **Truffle** | Most popular framework for developing Solidity contracts. Provides deployment, testing, and interaction tools |
| **Hardhat** | Modern alternative to Truffle with better debugging and TypeScript support |
| **Foundry** | Rust-based, extremely fast testing framework |

**OpenZeppelin** is also essential - not a framework but provides audited, reusable contract implementations for standards like ERC-20, ERC-721, etc.

### Q55: Which JavaScript Ethereum tool can you use to develop on a local blockchain?
**Answer:**
**Ganache** (formerly TestRPC) is a CLI tool that starts a local development blockchain.

**Features:**
- Instant transactions (no mining wait)
- 10 pre-funded accounts
- Configurable block times
- State snapshots
- Event and transaction logs

```bash
# Install and run
npm install -g ganache
ganache

# Outputs 10 accounts with 100 ETH each
# Listens on http://localhost:8545
```

### Q56: What do you need to deploy a smart contract to the Ethereum network?
**Answer:**

1. **Bytecode** of the smart contract (from compilation)
2. **Ethereum address** with enough Ether for gas costs
3. **Wallet** to sign the transaction
4. **Tool** to create and send the transaction

```javascript
// Example using ethers.js
const factory = new ethers.ContractFactory(abi, bytecode, wallet);
const contract = await factory.deploy(constructorArgs);
await contract.deployed();
```

### Q57: List some famous Ethereum wallets.
**Answer:**

| Wallet | Type | Description |
|--------|------|-------------|
| **MetaMask** | Browser Extension | Most popular, works with most dApps |
| **MyEtherWallet** | Website | Web-based wallet interface |
| **Ledger** | Hardware | Cold storage, very secure |
| **Trezor** | Hardware | Cold storage, very secure |

**Best Practice**: Use hardware wallets (Ledger/Trezor) for large amounts. Keep only small amounts in MetaMask for daily dApp use.

### Q58: List three networks where you can deploy a smart contract.
**Answer:**

| Network | Type | Description |
|---------|------|-------------|
| **Mainnet** | Production | Real Ethereum, real money |
| **Goerli** | Testnet | Public test network (replaces Ropsten) |
| **Sepolia** | Testnet | Newer public test network |
| **Local** | Development | Ganache or Hardhat Network |

**Deployment Flow:**
1. Local development (Ganache/Hardhat)
2. Testnet (Goerli/Sepolia)
3. Mainnet (Production)

---

## Wallets & Networks

*(Covered in Q57-Q58 above)*

---

## Inheritance & Code Reuse

### Q59: What mechanisms exist for code reuse in Solidity?
**Answer:**
Three main mechanisms:

1. **Functions**: Group common code into reusable functions
2. **Inheritance**: Child contracts inherit from parent contracts
3. **Libraries**: Reusable code that can be attached to types

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// 1. Functions
contract Utils {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }
}

// 2. Inheritance
contract Parent {
    function foo() public virtual pure returns (string memory) {
        return "Parent";
    }
}

contract Child is Parent {
    function foo() public pure override returns (string memory) {
        return "Child";
    }
}

// 3. Libraries
library Math {
    function max(uint256 a, uint256 b) internal pure returns (uint256) {
        return a > b ? a : b;
    }
}
```

### Q60: How to make contract A inherit from contract B?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// 1. Import the parent contract
import "./ContractB.sol";

// 2. Use 'is' keyword to inherit
contract ContractA is ContractB {
    
    // 3. Call parent constructor if needed
    constructor(uint256 _value) ContractB(_value) {
        // Child constructor code
    }
}
```

**Complete Example:**

```solidity
// Parent contract
contract Ownable {
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
}

// Child contract inherits from Ownable
contract Token is Ownable {
    mapping(address => uint256) public balances;
    
    function mint(uint256 amount) public onlyOwner {
        balances[owner] += amount;
    }
}
```

### Q61: If contract A inherits from contract B and both define function foo(), which one is resolved?
**Answer:**

**Case 1: Same signature** - The child's implementation overrides the parent:
```solidity
contract B {
    function foo() public virtual pure returns (uint256) { return 1; }
}

contract A is B {
    function foo() public pure override returns (uint256) { return 2; }
}
// A.foo() returns 2
```

**Case 2: Different signatures** - Both functions exist (function overloading):
```solidity
contract B {
    function foo(uint256 x) public pure returns (uint256) { return x; }
}

contract A is B {
    function foo() public pure returns (uint256) { return 0; }
}
// A.foo() returns 0
// A.foo(5) returns 5 (inherited from B)
```

---

## Advanced Types & Solidity Changes

### Q62: How to manage dates in Solidity?
**Answer:**
Solidity doesn't have a native date type. Dates are managed using **Unix timestamps** (seconds since Jan 1, 1970).

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DateExample {
    uint256 public deploymentTime;
    
    constructor() {
        deploymentTime = block.timestamp; // Current timestamp
    }
    
    function isExpired(uint256 deadline) public view returns (bool) {
        return block.timestamp > deadline;
    }
    
    // Helper: 1 day in seconds
    function oneDay() public pure returns (uint256) {
        return 1 days; // Solidity time unit = 86400
    }
}
```

### Q63: How to get the current timestamp in Solidity?
**Answer:**
Use `block.timestamp` (formerly `now` in older versions):

```solidity
uint256 currentTime = block.timestamp; // Seconds since Unix epoch
```

**Time Units in Solidity:**
- `1 seconds` = 1
- `1 minutes` = 60
- `1 hours` = 3600
- `1 days` = 86400
- `1 weeks` = 604800

### Q64: How to build a timestamp for one day in the future?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract FutureTime {
    function oneDayFromNow() public view returns (uint256) {
        return block.timestamp + 1 days;
        // Or: block.timestamp + 86400
    }
    
    function daysFromNow(uint256 _days) public view returns (uint256) {
        return block.timestamp + (_days * 1 days);
    }
}
```

### Q65: What are the main changes in Solidity 0.5 compared to 0.4?
**Answer:**

| Feature | Solidity 0.4 | Solidity 0.5+ |
|---------|--------------|---------------|
| Constructor | `function ContractName()` | `constructor()` |
| Address types | Only `address` | `address` and `address payable` |
| `var` keyword | Allowed | Removed |
| Events | `EventName()` | `emit EventName()` |
| Memory location | Implicit | Must be explicit for complex types |
| Function visibility | Implicit (public) | Must be explicit |
| `calldata` | N/A | Available for external functions |

```solidity
// Solidity 0.4 style (OLD)
// function MyContract() public { } // Constructor
// event Transfer(address to); Transfer(to); // Event without emit

// Solidity 0.5+ style (CURRENT)
constructor() { }
emit Transfer(to);
function foo(string memory _name) public { }
```

---

## Gas Optimization

### Q66: Give three ways to save gas.
**Answer:**

1. **Put less data on-chain**: Only store essential data; use events or off-chain storage for the rest

2. **Use events instead of storage**: Events are much cheaper than storage but can't be read by contracts

3. **Optimize variable ordering**: Pack variables to use fewer storage slots

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GasOptimized {
    // BAD - uses 3 storage slots
    // uint128 a;  // slot 0 (only uses half)
    // uint256 b;  // slot 1 (full)
    // uint128 c;  // slot 2 (only uses half)
    
    // GOOD - uses 2 storage slots
    uint128 a;  // slot 0
    uint128 c;  // slot 0 (packed with a)
    uint256 b;  // slot 1
    
    // Use events for data you don't need to read on-chain
    event DataStored(uint256 indexed id, string data);
    
    function storeData(uint256 _id, string calldata _data) external {
        emit DataStored(_id, _data); // Cheaper than storage!
    }
}
```

### Q67: How would you optimally order uint128, bytes32, and uint128 to save gas?
**Answer:**
Put the two `uint128` variables **next to each other**:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract OptimalOrder {
    // OPTIMAL - 2 storage slots
    uint128 a;    // Slot 0 (128 bits)
    uint128 b;    // Slot 0 (128 bits) - PACKED!
    bytes32 c;    // Slot 1 (256 bits)
    
    // SUB-OPTIMAL - 3 storage slots
    // uint128 a;  // Slot 0
    // bytes32 b;  // Slot 1
    // uint128 c;  // Slot 2
}
```

**Why?** The EVM stores variables in 32-byte (256-bit) slots. Solidity packs consecutive variables into the same slot if they fit. Two `uint128` (16 bytes each) fit in one 32-byte slot.

### Q68: How to concatenate two strings in Solidity?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StringConcat {
    function concat(string memory a, string memory b) 
        public pure returns (string memory) 
    {
        return string(abi.encodePacked(a, b));
    }
    
    function concatMultiple(string memory a, string memory b, string memory c) 
        public pure returns (string memory) 
    {
        return string(abi.encodePacked(a, b, c));
    }
}
```

**Note**: Since Solidity 0.8.12, you can also use `string.concat(a, b)`.

### Q69: How to get the length of a string in Solidity?
**Answer:**
Cast the string to `bytes` and access the `length` property:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StringLength {
    function getLength(string memory s) public pure returns (uint256) {
        return bytes(s).length;
    }
}
```

**Note**: This returns byte length, not character count. For UTF-8 strings with multi-byte characters, the result may differ from the character count.

---

## Contract Interactions

### Q70: How to create a smart contract from another smart contract?
**Answer:**
Use the `new` keyword:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Child {
    uint256 public value;
    
    constructor(uint256 _value) {
        value = _value;
    }
}

contract Factory {
    Child[] public children;
    
    function createChild(uint256 _value) public returns (Child) {
        Child child = new Child(_value);
        children.push(child);
        return child;
    }
    
    // Create with Ether
    function createChildWithEther(uint256 _value) public payable returns (Child) {
        Child child = new Child{value: msg.value}(_value);
        return child;
    }
}
```

### Q71: How to call a function of another smart contract?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Define interface or import contract
interface IExternalContract {
    function getValue() external view returns (uint256);
    function setValue(uint256 _value) external;
}

contract Caller {
    function callExternal(address _contractAddress) public view returns (uint256) {
        // Reference contract by its address
        IExternalContract externalContract = IExternalContract(_contractAddress);
        
        // Call the function
        return externalContract.getValue();
    }
}
```

### Q72: How to get the address of a contract created from another contract?
**Answer:**
Cast the new contract to `address`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Child {
    constructor() {}
}

contract Factory {
    function createAndGetAddress() public returns (address) {
        Child child = new Child();
        return address(child); // Cast to address
    }
}
```

### Q73: What is the value of msg.sender if a contract calls another contract?
**Answer:**
`msg.sender` will be the **address of the calling contract**, not the original EOA.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Inner {
    function getSender() public view returns (address) {
        return msg.sender; // Will be Outer's address!
    }
}

contract Outer {
    Inner public inner;
    
    constructor() {
        inner = new Inner();
    }
    
    function callInner() public view returns (address) {
        // msg.sender here is the EOA
        // but inside inner.getSender(), msg.sender is Outer's address
        return inner.getSender();
    }
}
```

**To get original sender**: Use `tx.origin` (but be careful - it's a security risk!)

### Q74: How to transfer ERC-20 tokens?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    function approve(address spender, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
    function allowance(address owner, address spender) external view returns (uint256);
}

contract TokenTransfer {
    // Direct transfer (from caller's balance)
    function sendTokens(
        address _token,
        address _to,
        uint256 _amount
    ) external {
        IERC20(_token).transfer(_to, _amount);
    }
    
    // Delegated transfer (requires prior approval)
    function sendTokensFrom(
        address _token,
        address _from,
        address _to,
        uint256 _amount
    ) external {
        // _from must have called approve(this contract, amount) first
        IERC20(_token).transferFrom(_from, _to, _amount);
    }
}
```

---

## Events & Logging

### Q75: How to declare and emit an event?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract EventExample {
    // 1. Declare event at contract level
    event Transfer(
        address indexed from,
        address indexed to,
        uint256 value
    );
    
    event Log(string message);
    
    function transfer(address _to, uint256 _amount) public {
        // 2. Emit event inside function
        emit Transfer(msg.sender, _to, _amount);
        emit Log("Transfer completed");
    }
}
```

### Q76: What is the indexed keyword in event definitions?
**Answer:**
`indexed` allows external entities to **filter events** by that field's value.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract IndexedEvents {
    // 'from' and 'to' are indexed - can filter by these
    // 'value' is not indexed - cannot filter by this
    event Transfer(
        address indexed from,  // Can filter: "all transfers FROM this address"
        address indexed to,    // Can filter: "all transfers TO this address"
        uint256 value          // Cannot filter
    );
}
```

```javascript
// JavaScript - filter events by indexed parameters
const filter = contract.filters.Transfer(fromAddress, null); // All from specific address
const events = await contract.queryFilter(filter);
```

### Q77: How many event fields can be marked as indexed?
**Answer:**
**Maximum 3 indexed fields** per event. This is an EVM limitation.

```solidity
event MyEvent(
    address indexed a,  // 1
    address indexed b,  // 2
    uint256 indexed c,  // 3
    uint256 d           // Not indexed (limit reached)
);
```

### Q78: Can a smart contract read events emitted before?
**Answer:**
**No!** Only external entities (off-chain code) can query events. Smart contracts cannot read event logs.

Events are stored in transaction logs, which are separate from contract storage and inaccessible to the EVM.

### Q79: Can you delete or modify a past event?
**Answer:**
**No!** Events are **immutable** once emitted. They become part of the blockchain's permanent history.

### Q80: How to do console.log debugging in Solidity?
**Answer:**
There's no direct equivalent, but you can use events:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Debug {
    event Log(string message);
    event LogUint(string message, uint256 value);
    event LogAddress(string message, address value);
    
    function debugExample(uint256 _value) public {
        emit Log("Function called");
        emit LogUint("Value received", _value);
        emit LogAddress("Caller", msg.sender);
        
        // Continue with actual logic...
    }
}
```

**Better Option**: Use Hardhat's `console.log`:
```solidity
import "hardhat/console.sol";

contract Debug {
    function test() public view {
        console.log("Value:", 123);
    }
}
```

---

## Access Control Patterns

### Q81: How to implement access control without using a modifier?
**Answer:**
Use `require` statement directly in the function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AccessControlBasic {
    address public admin;
    
    constructor() {
        admin = msg.sender;
    }
    
    function adminOnlyFunction() public {
        require(msg.sender == admin, "Only admin can call this");
        // Function logic here
    }
}
```

### Q82: How to implement access control with a modifier?
**Answer:**
Create a modifier and attach it to functions:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AccessControlModifier {
    address public admin;
    
    constructor() {
        admin = msg.sender;
    }
    
    // Define modifier
    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can call this");
        _; // Placeholder for function body
    }
    
    // Use modifier in function signature
    function adminOnlyFunction() public onlyAdmin {
        // Function logic - only executes if modifier passes
    }
    
    function anotherAdminFunction() public onlyAdmin {
        // Reusable!
    }
}
```

### Q83: How to cancel a transaction?
**Answer:**
Once sent, you can't cancel a transaction directly. But you can **replace it** with another transaction that:

1. Has the **same nonce** as the original
2. Has a **higher gas price** (so miners prefer it)
3. Does something harmless (like sending 0 ETH to yourself)

```javascript
// Using ethers.js to cancel a pending transaction
const cancelTx = {
    to: wallet.address,      // Send to yourself
    value: 0,                // No value
    nonce: originalNonce,    // Same nonce!
    gasPrice: higherGasPrice // Higher than original
};

await wallet.sendTransaction(cancelTx);
```

If the replacement transaction gets mined first, the original becomes invalid (same nonce can't be used twice).

---

## Advanced Security

### Q84: What is the ABI encoder v2 pragma statement?
**Answer:**
It enables experimental features not yet part of standard Solidity:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
pragma abicoder v2; // Enable advanced encoding

contract ABIEncoderV2Example {
    struct User {
        string name;
        uint256 age;
    }
    
    // Can return structs from external functions (not possible in v1)
    function getUser() external pure returns (User memory) {
        return User("Alice", 30);
    }
}
```

**Note**: In Solidity 0.8+, ABI coder v2 is the default.

### Q85: Is it safe to use ABI encoder v2 in production?
**Answer:**
**Yes, in Solidity 0.8+** it's the default and considered stable.

In earlier versions (0.5-0.7), it was experimental and should be used with caution. The main risks were around edge cases with complex nested types.

### Q86: Is it possible to send a transaction without requiring users to pay gas?
**Answer:**
**Yes!** This is called a **gasless transaction** or **meta-transaction**.

**How it works:**
1. User signs a message (not a transaction) on the frontend
2. Message + signature sent to your backend (off-chain)
3. Your backend creates a real transaction with the user's data
4. Your backend pays the gas
5. Smart contract verifies the signature and acts on user's behalf

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GaslessExample {
    mapping(address => uint256) public nonces;
    
    function executeMetaTx(
        address user,
        uint256 value,
        uint256 nonce,
        bytes memory signature
    ) external {
        // Verify nonce
        require(nonces[user] == nonce, "Invalid nonce");
        
        // Recreate signed message
        bytes32 hash = keccak256(abi.encodePacked(user, value, nonce));
        bytes32 ethSignedHash = keccak256(
            abi.encodePacked("\x19Ethereum Signed Message:\n32", hash)
        );
        
        // Recover signer
        address signer = recoverSigner(ethSignedHash, signature);
        require(signer == user, "Invalid signature");
        
        // Execute action on behalf of user
        nonces[user]++;
        // ... do something with value
    }
    
    function recoverSigner(bytes32 hash, bytes memory sig) 
        internal pure returns (address) 
    {
        (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);
        return ecrecover(hash, v, r, s);
    }
    
    function splitSignature(bytes memory sig) 
        internal pure returns (uint8 v, bytes32 r, bytes32 s) 
    {
        require(sig.length == 65, "Invalid signature length");
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
    }
}
```

### Q87: Which Solidity function verifies a signature?
**Answer:**
`ecrecover` - it recovers the address that signed a message.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SignatureVerification {
    function verify(
        address _signer,
        bytes32 _messageHash,
        bytes memory _signature
    ) public pure returns (bool) {
        bytes32 ethSignedHash = keccak256(
            abi.encodePacked("\x19Ethereum Signed Message:\n32", _messageHash)
        );
        
        (uint8 v, bytes32 r, bytes32 s) = splitSignature(_signature);
        address recovered = ecrecover(ethSignedHash, v, r, s);
        
        return recovered == _signer;
    }
    
    function splitSignature(bytes memory sig) 
        internal pure returns (uint8, bytes32, bytes32) 
    {
        require(sig.length == 65);
        bytes32 r;
        bytes32 s;
        uint8 v;
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
        return (v, r, s);
    }
}
```

### Q88: What is a re-entrancy attack?
**Answer:**
A re-entrancy attack occurs when a malicious contract exploits the order of operations in a victim contract, calling back into it before state updates are complete.

**The DAO Hack Example:**

```solidity
// VULNERABLE Contract
contract Vulnerable {
    mapping(address => uint256) public balances;
    
    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount);
        
        // 1. Send Ether BEFORE updating balance (BUG!)
        (bool success,) = msg.sender.call{value: _amount}("");
        require(success);
        
        // 2. Update balance AFTER sending
        balances[msg.sender] -= _amount;
    }
}

// ATTACKER Contract
contract Attacker {
    Vulnerable public target;
    
    constructor(address _target) {
        target = Vulnerable(_target);
    }
    
    // This function is called when receiving Ether
    receive() external payable {
        if (address(target).balance >= 1 ether) {
            // Re-enter before balance is updated!
            target.withdraw(1 ether);
        }
    }
    
    function attack() external payable {
        target.withdraw(1 ether); // Start the attack
    }
}
```

**Attack Flow:**
1. Attacker calls `withdraw(1 ether)`
2. Vulnerable contract sends 1 ETH to Attacker
3. Attacker's `receive()` triggers, calls `withdraw()` again
4. Balance hasn't been updated yet, so check passes!
5. Loop continues until Vulnerable is drained

### Q89: How to prevent a re-entrancy attack?
**Answer:**
Three main solutions:

**1. Checks-Effects-Interactions Pattern (Recommended)**
```solidity
function withdraw(uint256 _amount) public {
    // CHECKS
    require(balances[msg.sender] >= _amount);
    
    // EFFECTS (update state BEFORE external call)
    balances[msg.sender] -= _amount;
    
    // INTERACTIONS (external call LAST)
    (bool success,) = msg.sender.call{value: _amount}("");
    require(success);
}
```

**2. Reentrancy Guard (Mutex)**
```solidity
bool private locked;

modifier noReentrant() {
    require(!locked, "No re-entrancy");
    locked = true;
    _;
    locked = false;
}

function withdraw(uint256 _amount) public noReentrant {
    // Protected from re-entrancy
}
```

**3. Limit Gas (for transfer)**
```solidity
// transfer() only forwards 2300 gas - not enough for re-entrancy
payable(msg.sender).transfer(_amount);
```

---

## Libraries

### Q90: What is a library in Solidity?
**Answer:**
A library is reusable code that can be used by other contracts. Libraries:

- Don't have their own storage
- Can't hold Ether
- Can't inherit or be inherited
- Provide only functions

**Two Types:**
| Type | Address | Deployment |
|------|---------|------------|
| **Embedded** | No own address | Compiled into contract |
| **Deployed** | Has own address | Linked at deployment |

### Q91: Give an example of how to use a library in a smart contract.
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Define the library
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "Addition overflow");
        return c;
    }
    
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "Subtraction underflow");
        return a - b;
    }
}

// Use the library
contract Calculator {
    using SafeMath for uint256; // Attach to type
    
    function calculate(uint256 a, uint256 b) public pure returns (uint256) {
        // Method 1: Dot notation (a is first argument)
        uint256 sum = a.add(b);
        
        // Method 2: Direct call
        uint256 diff = SafeMath.sub(a, b);
        
        return sum + diff;
    }
}
```

**Note**: With `using SafeMath for uint256`, the variable before the dot becomes the first argument.

### Q92: When is a library embedded vs deployed?
**Answer:**

| Library Has | Result |
|-------------|--------|
| Only `internal` functions | **Embedded** - code copied into contract |
| `public`/`external` functions | **Deployed** - separate contract, linked |

```solidity
// EMBEDDED - only internal functions
library EmbeddedLib {
    function internalFunc(uint256 a) internal pure returns (uint256) {
        return a * 2;
    }
}

// DEPLOYED - has public function
library DeployedLib {
    function publicFunc(uint256 a) public pure returns (uint256) {
        return a * 2;
    }
}
```

---

## Hashing & Randomness

### Q93: How to produce a hash of multiple values in Solidity?
**Answer:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Hashing {
    function hashValues(
        string memory _text,
        uint256 _num,
        address _addr
    ) public pure returns (bytes32) {
        // 1. Pack values together
        bytes memory packed = abi.encodePacked(_text, _num, _addr);
        
        // 2. Hash the packed data
        return keccak256(packed);
    }
    
    // Shorter version
    function hashDirect(string memory _a, uint256 _b) 
        public pure returns (bytes32) 
    {
        return keccak256(abi.encodePacked(_a, _b));
    }
}
```

**Hash Functions Available:**
- `keccak256` - Most common, returns bytes32
- `sha256` - SHA-256 hash
- `ripemd160` - RIPEMD-160 hash

### Q94: How to generate a random number in Solidity?
**Answer:**
Use block variables as a source of randomness (with caveats):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract RandomNumber {
    function getRandomNumber() public view returns (uint256) {
        return uint256(
            keccak256(
                abi.encodePacked(
                    block.timestamp,
                    block.prevrandao, // Was block.difficulty before merge
                    msg.sender
                )
            )
        );
    }
    
    // Random number in range [0, max)
    function getRandomInRange(uint256 max) public view returns (uint256) {
        return getRandomNumber() % max;
    }
}
```

**⚠️ Security Warning:**
This method is **NOT secure** for high-stakes applications:
- Miners can manipulate `block.timestamp` and `block.prevrandao`
- Transaction ordering can be exploited

**For secure randomness, use:**
- Chainlink VRF (Verifiable Random Function)
- Commit-reveal schemes
- Other oracle solutions

---

## Assembly in Solidity

### Q95: What are the two kinds of assembly in Solidity?
**Answer:**

| Type | Description |
|------|-------------|
| **Functional** | Uses function-like syntax, more readable |
| **Instructional** | Raw opcodes, very low-level |

Most developers use **functional** style:

```solidity
assembly {
    // Functional style (common)
    let x := add(1, 2)
    let y := mload(0x40)
    
    // Instructional would be raw opcodes
    // ADD, MLOAD, etc.
}
```

### Q96: How to declare assembly code in Solidity?
**Answer:**
Use the `assembly` keyword with curly braces:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AssemblyExample {
    function addNumbers(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := add(a, b)
        }
    }
    
    function getBalance(address addr) public view returns (uint256 result) {
        assembly {
            result := balance(addr)
        }
    }
}
```

### Q97: Create a function to determine if an address is a contract or EOA.
**Answer:**
Check if the address has code using `extcodesize`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AddressChecker {
    function isContract(address _addr) public view returns (bool) {
        uint256 codeSize;
        
        assembly {
            codeSize := extcodesize(_addr)
        }
        
        return codeSize > 0;
    }
    
    // Inverse: check if EOA (human)
    function isHuman(address _addr) public view returns (bool) {
        uint256 codeSize;
        
        assembly {
            codeSize := extcodesize(_addr)
        }
        
        return codeSize == 0;
    }
}
```

**⚠️ Caveat:** During a contract's constructor, `extcodesize` returns 0. So a contract being created can bypass this check.

### Q98: What are common use cases for assembly in Solidity?
**Answer:**

1. **Gas optimization** - Assembly can be more efficient
2. **Low-level operations** - Memory manipulation, direct storage access
3. **Features not in Solidity** - Some operations only available in assembly
4. **Return data handling** - Complex return data parsing

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AssemblyUseCases {
    // Gas-efficient batch transfer
    function efficientTransfer(address payable[] memory recipients) external payable {
        uint256 amount = msg.value / recipients.length;
        
        for (uint256 i = 0; i < recipients.length; i++) {
            assembly {
                let success := call(
                    gas(),
                    mload(add(add(recipients, 0x20), mul(i, 0x20))),
                    amount,
                    0, 0, 0, 0
                )
            }
        }
    }
    
    // Direct storage access
    function readStorageSlot(uint256 slot) public view returns (bytes32 result) {
        assembly {
            result := sload(slot)
        }
    }
}
```

### Q99: What is the purpose of the free memory pointer in Solidity?
**Answer:**
The free memory pointer (at `0x40`) tracks the next available memory location.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemoryPointer {
    function getFreeMemoryPointer() public pure returns (uint256 ptr) {
        assembly {
            ptr := mload(0x40) // Load free memory pointer
        }
    }
    
    function allocateMemory(uint256 size) public pure returns (uint256 ptr) {
        assembly {
            ptr := mload(0x40)                    // Get current pointer
            mstore(0x40, add(ptr, size))         // Update pointer
        }
    }
}
```

### Q100: What are some important EVM opcodes to know?
**Answer:**

| Opcode | Description | Example Use |
|--------|-------------|-------------|
| `add`, `sub`, `mul`, `div` | Arithmetic | Math operations |
| `mload`, `mstore` | Memory read/write | Temporary data |
| `sload`, `sstore` | Storage read/write | Persistent data |
| `call`, `delegatecall` | External calls | Contract interactions |
| `balance` | Get ETH balance | Check account balance |
| `extcodesize` | Code size at address | Check if contract |
| `revert` | Revert execution | Error handling |
| `return` | Return data | Function returns |
| `keccak256` | Hash data | Hashing |
| `caller` | msg.sender | Access control |

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract OpcodeExamples {
    function examples() public view returns (
        uint256 sum,
        uint256 bal,
        address sender
    ) {
        assembly {
            sum := add(10, 20)           // Arithmetic
            bal := balance(address())    // Get contract balance
            sender := caller()           // Get msg.sender
        }
    }
}
```

---

## Summary

This guide covered 100 essential Solidity and blockchain interview questions across difficulty levels:

**Easy (Q1-Q26)**: Fundamentals of Ethereum, Solidity syntax, basic data types, collections

**Intermediate (Q27-Q83)**: Structs, enums, memory locations, gas, development tools, contract interactions, events, access control

**Difficult (Q84-Q100)**: Security patterns, meta-transactions, libraries, hashing, randomness, assembly

**Key Takeaways:**
1. Smart contracts are immutable - test thoroughly before deployment
2. Security is paramount - understand re-entrancy and other attack vectors
3. Gas optimization matters - every operation costs money
4. Use established patterns - leverage OpenZeppelin and audited libraries
5. Test on testnets before mainnet deployment

**Recommended Resources:**
- [Solidity Documentation](https://docs.soliditylang.org/)
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Consensys Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
