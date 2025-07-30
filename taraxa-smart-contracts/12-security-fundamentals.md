# บทที่ 12: Security Fundamentals

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ช่องโหว่ความปลอดภัยใน Smart Contracts
- เข้าใจหลักการ Security Best Practices
- ป้องกัน Common Attack Vectors
- ใช้เครื่องมือ Security Analysis และ Auditing

## 🛡️ Common Security Vulnerabilities

### **⚡ Reentrancy Attack**

```solidity
// ❌ Vulnerable to reentrancy
contract VulnerableBank {
    mapping(address => uint256) public balances;
    
    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount);
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
        balances[msg.sender] -= amount; // State change after external call
    }
}

// ✅ Protected against reentrancy
contract SecureBank {
    mapping(address => uint256) public balances;
    bool private locked;
    
    modifier nonReentrant() {
        require(!locked, "Reentrant call");
        locked = true;
        _;
        locked = false;
    }
    
    function withdraw(uint256 amount) public nonReentrant {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount; // State change first
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
    }
}
```

### **💸 Integer Overflow/Underflow**

```solidity
// ✅ Safe math with Solidity 0.8+
contract SafeMath {
    uint256 public totalSupply;
    
    function mint(uint256 amount) public {
        totalSupply += amount; // Automatic overflow check
    }
    
    function burn(uint256 amount) public {
        totalSupply -= amount; // Automatic underflow check
    }
}
```

### **🔐 Access Control Issues**

```solidity
// contracts/security/AccessControlExample.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureContract is AccessControl, ReentrancyGuard {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
    
    bool public paused = false;
    mapping(address => uint256) public balances;
    
    event Paused(address account);
    event Unpaused(address account);
    
    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }
    
    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }
    
    function pause() public onlyRole(PAUSER_ROLE) {
        paused = true;
        emit Paused(msg.sender);
    }
    
    function unpause() public onlyRole(PAUSER_ROLE) {
        paused = false;
        emit Unpaused(msg.sender);
    }
    
    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) whenNotPaused {
        balances[to] += amount;
    }
}
```

## 🛠️ Security Best Practices

### **1. Input Validation**

```solidity
contract InputValidation {
    mapping(address => uint256) public balances;
    uint256 public constant MAX_SUPPLY = 1000000 * 10**18;
    
    function transfer(address to, uint256 amount) public {
        require(to != address(0), "Transfer to zero address");
        require(to != address(this), "Transfer to contract");
        require(amount > 0, "Amount must be positive");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

### **2. State Machine Security**

```solidity
contract SecureAuction {
    enum AuctionState { Created, Started, Ended, Cancelled }
    
    AuctionState public state;
    address public highestBidder;
    uint256 public highestBid;
    uint256 public endTime;
    
    modifier inState(AuctionState _state) {
        require(state == _state, "Invalid state");
        _;
    }
    
    modifier onlyBefore(uint256 _time) {
        require(block.timestamp < _time, "Too late");
        _;
    }
    
    function placeBid() public payable inState(AuctionState.Started) onlyBefore(endTime) {
        require(msg.value > highestBid, "Bid too low");
        
        if (highestBidder != address(0)) {
            payable(highestBidder).transfer(highestBid);
        }
        
        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}
```

### **3. Oracle Security**

```solidity
// contracts/security/SecureOracle.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract SecureOracle {
    AggregatorV3Interface internal priceFeed;
    uint256 public constant PRICE_PRECISION = 1e8;
    uint256 public constant MAX_PRICE_AGE = 3600; // 1 hour
    
    constructor(address _priceFeed) {
        priceFeed = AggregatorV3Interface(_priceFeed);
    }
    
    function getLatestPrice() public view returns (uint256, uint256) {
        (
            uint80 roundId,
            int256 price,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        
        require(price > 0, "Invalid price");
        require(updatedAt > 0, "Round not complete");
        require(block.timestamp - updatedAt <= MAX_PRICE_AGE, "Price too old");
        require(answeredInRound >= roundId, "Stale price");
        
        return (uint256(price), updatedAt);
    }
    
    function getPriceWithValidation(uint256 maxAge) public view returns (uint256) {
        (uint256 price, uint256 updatedAt) = getLatestPrice();
        require(block.timestamp - updatedAt <= maxAge, "Price exceeds max age");
        return price;
    }
}
```

## 🔍 Advanced Security Patterns

### **🛡️ Circuit Breaker Pattern**

```solidity
// contracts/security/CircuitBreaker.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract CircuitBreaker {
    bool public emergencyStop = false;
    address public owner;
    
    uint256 public withdrawalLimit = 1000 ether;
    uint256 public dailyWithdrawn = 0;
    uint256 public lastResetTime = block.timestamp;
    
    mapping(address => uint256) public balances;
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    modifier stopInEmergency() {
        require(!emergencyStop, "Emergency stop activated");
        _;
    }
    
    modifier onlyInEmergency() {
        require(emergencyStop, "Not in emergency");
        _;
    }
    
    constructor() {
        owner = msg.sender;
    }
    
    function toggleEmergency() public onlyOwner {
        emergencyStop = !emergencyStop;
    }
    
    function withdraw(uint256 amount) public stopInEmergency {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        // Reset daily limit if needed
        if (block.timestamp > lastResetTime + 1 days) {
            dailyWithdrawn = 0;
            lastResetTime = block.timestamp;
        }
        
        require(dailyWithdrawn + amount <= withdrawalLimit, "Daily limit exceeded");
        
        balances[msg.sender] -= amount;
        dailyWithdrawn += amount;
        
        payable(msg.sender).transfer(amount);
    }
    
    function emergencyWithdraw() public onlyOwner onlyInEmergency {
        payable(owner).transfer(address(this).balance);
    }
}
```

### **⏱️ Time-Based Security**

```solidity
// contracts/security/TimeLock.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract TimeLock {
    uint256 public constant DELAY = 2 days;
    
    struct Transaction {
        address target;
        uint256 value;
        bytes data;
        uint256 eta; // Estimated time of arrival
        bool executed;
    }
    
    mapping(bytes32 => Transaction) public transactions;
    address public admin;
    
    event TransactionQueued(bytes32 indexed txHash, address target, uint256 value, bytes data, uint256 eta);
    event TransactionExecuted(bytes32 indexed txHash);
    event TransactionCancelled(bytes32 indexed txHash);
    
    modifier onlyAdmin() {
        require(msg.sender == admin, "Not admin");
        _;
    }
    
    constructor() {
        admin = msg.sender;
    }
    
    function queueTransaction(
        address target,
        uint256 value,
        bytes memory data
    ) public onlyAdmin returns (bytes32) {
        uint256 eta = block.timestamp + DELAY;
        bytes32 txHash = keccak256(abi.encode(target, value, data, eta));
        
        transactions[txHash] = Transaction({
            target: target,
            value: value,
            data: data,
            eta: eta,
            executed: false
        });
        
        emit TransactionQueued(txHash, target, value, data, eta);
        return txHash;
    }
    
    function executeTransaction(bytes32 txHash) public onlyAdmin {
        Transaction storage tx = transactions[txHash];
        require(tx.eta != 0, "Transaction not queued");
        require(block.timestamp >= tx.eta, "Transaction not ready");
        require(!tx.executed, "Transaction already executed");
        
        tx.executed = true;
        
        (bool success, ) = tx.target.call{value: tx.value}(tx.data);
        require(success, "Transaction execution failed");
        
        emit TransactionExecuted(txHash);
    }
    
    function cancelTransaction(bytes32 txHash) public onlyAdmin {
        delete transactions[txHash];
        emit TransactionCancelled(txHash);
    }
}
```

### **🔐 Multi-Signature Wallet**

```solidity
// contracts/security/MultiSigWallet.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract MultiSigWallet {
    event Deposit(address indexed sender, uint256 amount, uint256 balance);
    event SubmitTransaction(
        address indexed owner,
        uint256 indexed txIndex,
        address indexed to,
        uint256 value,
        bytes data
    );
    event ConfirmTransaction(address indexed owner, uint256 indexed txIndex);
    event RevokeConfirmation(address indexed owner, uint256 indexed txIndex);
    event ExecuteTransaction(address indexed owner, uint256 indexed txIndex);
    
    address[] public owners;
    mapping(address => bool) public isOwner;
    uint256 public numConfirmationsRequired;
    
    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool executed;
        uint256 numConfirmations;
    }
    
    mapping(uint256 => mapping(address => bool)) public isConfirmed;
    Transaction[] public transactions;
    
    modifier onlyOwner() {
        require(isOwner[msg.sender], "Not owner");
        _;
    }
    
    modifier txExists(uint256 _txIndex) {
        require(_txIndex < transactions.length, "Transaction does not exist");
        _;
    }
    
    modifier notExecuted(uint256 _txIndex) {
        require(!transactions[_txIndex].executed, "Transaction already executed");
        _;
    }
    
    modifier notConfirmed(uint256 _txIndex) {
        require(!isConfirmed[_txIndex][msg.sender], "Transaction already confirmed");
        _;
    }
    
    constructor(address[] memory _owners, uint256 _numConfirmationsRequired) {
        require(_owners.length > 0, "Owners required");
        require(
            _numConfirmationsRequired > 0 && _numConfirmationsRequired <= _owners.length,
            "Invalid number of required confirmations"
        );
        
        for (uint256 i = 0; i < _owners.length; i++) {
            address owner = _owners[i];
            require(owner != address(0), "Invalid owner");
            require(!isOwner[owner], "Owner not unique");
            
            isOwner[owner] = true;
            owners.push(owner);
        }
        
        numConfirmationsRequired = _numConfirmationsRequired;
    }
    
    receive() external payable {
        emit Deposit(msg.sender, msg.value, address(this).balance);
    }
    
    function submitTransaction(
        address _to,
        uint256 _value,
        bytes memory _data
    ) public onlyOwner {
        uint256 txIndex = transactions.length;
        
        transactions.push(Transaction({
            to: _to,
            value: _value,
            data: _data,
            executed: false,
            numConfirmations: 0
        }));
        
        emit SubmitTransaction(msg.sender, txIndex, _to, _value, _data);
    }
    
    function confirmTransaction(uint256 _txIndex)
        public
        onlyOwner
        txExists(_txIndex)
        notExecuted(_txIndex)
        notConfirmed(_txIndex)
    {
        Transaction storage transaction = transactions[_txIndex];
        transaction.numConfirmations += 1;
        isConfirmed[_txIndex][msg.sender] = true;
        
        emit ConfirmTransaction(msg.sender, _txIndex);
    }
    
    function executeTransaction(uint256 _txIndex)
        public
        onlyOwner
        txExists(_txIndex)
        notExecuted(_txIndex)
    {
        Transaction storage transaction = transactions[_txIndex];
        
        require(
            transaction.numConfirmations >= numConfirmationsRequired,
            "Cannot execute transaction"
        );
        
        transaction.executed = true;
        
        (bool success, ) = transaction.to.call{value: transaction.value}(transaction.data);
        require(success, "Transaction failed");
        
        emit ExecuteTransaction(msg.sender, _txIndex);
    }
}
```

## 🧪 Security Testing & Auditing

### **🔍 Security Testing Framework**

```javascript
// test/security-tests.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Security Tests", function() {
    let secureBank, attacker, owner, user;
    
    beforeEach(async function() {
        [owner, user, attacker] = await ethers.getSigners();
        
        const SecureBank = await ethers.getContractFactory("SecureBank");
        secureBank = await SecureBank.deploy();
        
        // Setup initial state
        await secureBank.connect(user).deposit({value: ethers.utils.parseEther("1")});
    });
    
    describe("Reentrancy Protection", function() {
        it("Should prevent reentrancy attacks", async function() {
            const AttackerContract = await ethers.getContractFactory("ReentrancyAttacker");
            const attackerContract = await AttackerContract.deploy(secureBank.address);
            
            await expect(
                attackerContract.attack({value: ethers.utils.parseEther("0.1")})
            ).to.be.revertedWith("Reentrant call");
        });
    });
    
    describe("Access Control", function() {
        it("Should only allow authorized users", async function() {
            await expect(
                secureBank.connect(attacker).adminFunction()
            ).to.be.revertedWith("AccessControl: account is missing role");
        });
    });
    
    describe("Input Validation", function() {
        it("Should reject invalid inputs", async function() {
            await expect(
                secureBank.transfer(ethers.constants.AddressZero, 100)
            ).to.be.revertedWith("Transfer to zero address");
        });
    });
});
```

### **📊 Gas Analysis for Security**

```javascript
// scripts/security-gas-analysis.js
async function analyzeSecurity() {
    const contracts = ["SecureBank", "CircuitBreaker", "MultiSigWallet"];
    
    for (const contractName of contracts) {
        console.log(`\n=== ${contractName} Security Analysis ===`);
        
        const Contract = await ethers.getContractFactory(contractName);
        const contract = await Contract.deploy();
        
        // Test critical functions
        const criticalFunctions = [
            "withdraw",
            "emergencyStop", 
            "executeTransaction"
        ];
        
        for (const func of criticalFunctions) {
            if (contract[func]) {
                try {
                    const tx = await contract[func]();
                    const receipt = await tx.wait();
                    console.log(`${func}: ${receipt.gasUsed} gas`);
                } catch (error) {
                    console.log(`${func}: Protected (${error.message})`);
                }
            }
        }
    }
}
```

## 📋 Security Checklist

### **🛡️ Pre-Deployment Security Audit**

```markdown
## Smart Contract Security Checklist

### Access Control
- [ ] Proper role-based access control
- [ ] Owner functions are protected
- [ ] No unauthorized function access
- [ ] Multi-signature for critical operations

### Input Validation
- [ ] All inputs are validated
- [ ] No integer overflow/underflow
- [ ] Address zero checks
- [ ] Array bounds checking

### External Calls
- [ ] Reentrancy protection
- [ ] Check-Effects-Interactions pattern
- [ ] Proper error handling
- [ ] Gas limit considerations

### State Management
- [ ] Proper state transitions
- [ ] No race conditions
- [ ] Atomic operations
- [ ] Event logging

### Business Logic
- [ ] Economic attacks prevention
- [ ] Flash loan attack resistance
- [ ] Oracle manipulation protection
- [ ] MEV resistance

### Emergency Procedures
- [ ] Circuit breaker mechanism
- [ ] Emergency pause functionality
- [ ] Upgrade procedures
- [ ] Recovery mechanisms
```

## 🎯 แบบฝึกหัด

### **🔐 แบบฝึกหัดที่ 1: Fix Vulnerabilities**
ตรวจหาและแก้ไขช่องโหว่ในโค้ดนี้:

```solidity
contract VulnerableContract {
    mapping(address => uint256) public balances;
    address public owner;
    
    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount);
        msg.sender.call{value: amount}("");
        balances[msg.sender] -= amount;
    }
    
    function changeOwner(address newOwner) public {
        owner = newOwner;
    }
}
```

### **🧪 แบบฝึกหัดที่ 2: Security Testing**
สร้าง comprehensive security test suite สำหรับ:
1. Reentrancy attacks
2. Access control bypasses  
3. Integer overflow/underflow
4. Front-running attacks

### **🛡️ แบบฝึกหัดที่ 3: Secure DeFi Protocol**
ออกแบบและสร้าง secure DeFi lending protocol ที่มี:
1. Flash loan protection
2. Oracle manipulation resistance
3. Liquidation safeguards
4. Emergency procedures

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 11: Gas Optimization](./11-gas-optimization.md)  
**บทถัดไป**: [บทที่ 13: Frontend Integration](./13-frontend-integration.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [OpenZeppelin Security](https://docs.openzeppelin.com/contracts/4.x/security)
- [Ethereum Security](https://ethereum.org/en/developers/docs/smart-contracts/security/)
- [SWC Registry](https://swcregistry.io/)
- [MythX Security Tools](https://mythx.io/)

---

ตอนนี้คุณมีความรู้ด้าน Security ที่จำเป็นสำหรับการสร้าง Smart Contract ที่ปลอดภัย! 🛡️✨
