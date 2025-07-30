# บทที่ 3: Solidity Fundamentals

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Solidity syntax และ language features
- ทำความเข้าใจ Smart Contract structure
- เรียนรู้ Data types และ Control structures
- ฝึกเขียน Functions และ Modifiers

## 📚 Solidity Overview

**Solidity** เป็น high-level programming language สำหรับเขียน Smart Contracts บน Ethereum Virtual Machine (EVM) รวมถึง Taraxa Network

### **🔧 Language Characteristics**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Contract structure overview
contract SolidityFundamentals {
    // State variables
    uint256 public totalSupply;
    mapping(address => uint256) public balances;
    
    // Events
    event Transfer(address indexed from, address indexed to, uint256 value);
    
    // Modifiers
    modifier onlyPositive(uint256 amount) {
        require(amount > 0, "Amount must be positive");
        _;
    }
    
    // Constructor
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply;
        balances[msg.sender] = _initialSupply;
    }
    
    // Functions
    function transfer(address to, uint256 amount) 
        external 
        onlyPositive(amount) 
        returns (bool) 
    {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[to] += amount;
        
        emit Transfer(msg.sender, to, amount);
        return true;
    }
}
```

## 🎯 Data Types

### **🔢 Value Types**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract DataTypes {
    // Boolean
    bool public isActive = true;
    bool public isComplete = false;
    
    // Integers
    uint8 public smallNumber = 255;        // 0 to 255
    uint256 public largeNumber = 1000000;  // 0 to 2^256-1
    int256 public signedNumber = -1000;    // -2^255 to 2^255-1
    
    // Address types
    address public owner;
    address payable public recipient;
    
    // Bytes
    bytes1 public singleByte = 0xff;
    bytes32 public hash = keccak256("Hello");
    bytes public dynamicBytes = "Hello World";
    
    // String
    string public name = "Taraxa Contract";
    
    // Enums
    enum Status { Pending, Active, Inactive, Completed }
    Status public currentStatus = Status.Pending;
    
    constructor() {
        owner = msg.sender;
        recipient = payable(msg.sender);
    }
    
    // Type conversions and operations
    function typeOperations() external pure returns (
        uint256 sum,
        bytes32 nameHash,
        address converted
    ) {
        // Arithmetic operations
        uint256 a = 10;
        uint256 b = 20;
        sum = a + b;
        
        // Hash operations
        nameHash = keccak256(abi.encodePacked("Taraxa"));
        
        // Address conversion
        converted = address(uint160(123456789));
        
        return (sum, nameHash, converted);
    }
    
    // Working with enums
    function updateStatus(Status _newStatus) external {
        require(msg.sender == owner, "Only owner can update");
        currentStatus = _newStatus;
    }
    
    function getStatusName() external view returns (string memory) {
        if (currentStatus == Status.Pending) return "Pending";
        if (currentStatus == Status.Active) return "Active";
        if (currentStatus == Status.Inactive) return "Inactive";
        return "Completed";
    }
}
```

### **📦 Reference Types**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract ReferenceTypes {
    // Arrays
    uint256[] public dynamicArray;
    uint256[5] public fixedArray;
    
    // Mappings
    mapping(address => uint256) public balances;
    mapping(address => mapping(address => uint256)) public allowances;
    mapping(string => bool) public nameExists;
    
    // Structs
    struct User {
        string name;
        uint256 age;
        bool isActive;
        uint256[] favoriteNumbers;
    }
    
    mapping(address => User) public users;
    User[] public allUsers;
    
    // Events for tracking changes
    event UserCreated(address indexed userAddress, string name, uint256 age);
    event ArrayUpdated(uint256 newLength);
    event BalanceUpdated(address indexed user, uint256 newBalance);
    
    // Array operations
    function addToArray(uint256 _number) external {
        dynamicArray.push(_number);
        emit ArrayUpdated(dynamicArray.length);
    }
    
    function removeFromArray(uint256 _index) external {
        require(_index < dynamicArray.length, "Index out of bounds");
        
        // Replace with last element and pop
        dynamicArray[_index] = dynamicArray[dynamicArray.length - 1];
        dynamicArray.pop();
        
        emit ArrayUpdated(dynamicArray.length);
    }
    
    function getArrayLength() external view returns (uint256) {
        return dynamicArray.length;
    }
    
    function getAllArrayElements() external view returns (uint256[] memory) {
        return dynamicArray;
    }
    
    // Mapping operations
    function setBalance(address _user, uint256 _amount) external {
        balances[_user] = _amount;
        emit BalanceUpdated(_user, _amount);
    }
    
    function approve(address _spender, uint256 _amount) external {
        allowances[msg.sender][_spender] = _amount;
    }
    
    function getAllowance(address _owner, address _spender) 
        external 
        view 
        returns (uint256) 
    {
        return allowances[_owner][_spender];
    }
    
    // Struct operations
    function createUser(
        string memory _name, 
        uint256 _age,
        uint256[] memory _favoriteNumbers
    ) external {
        require(!nameExists[_name], "Name already exists");
        
        User storage newUser = users[msg.sender];
        newUser.name = _name;
        newUser.age = _age;
        newUser.isActive = true;
        newUser.favoriteNumbers = _favoriteNumbers;
        
        allUsers.push(newUser);
        nameExists[_name] = true;
        
        emit UserCreated(msg.sender, _name, _age);
    }
    
    function updateUserAge(uint256 _newAge) external {
        require(bytes(users[msg.sender].name).length > 0, "User does not exist");
        users[msg.sender].age = _newAge;
    }
    
    function addFavoriteNumber(uint256 _number) external {
        require(bytes(users[msg.sender].name).length > 0, "User does not exist");
        users[msg.sender].favoriteNumbers.push(_number);
    }
    
    function getUserInfo(address _userAddress) 
        external 
        view 
        returns (
            string memory name,
            uint256 age,
            bool isActive,
            uint256[] memory favoriteNumbers
        ) 
    {
        User storage user = users[_userAddress];
        return (user.name, user.age, user.isActive, user.favoriteNumbers);
    }
    
    function getTotalUsers() external view returns (uint256) {
        return allUsers.length;
    }
}
```

## 🔧 Functions และ Modifiers

### **⚡ Function Types และ Visibility**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract FunctionTypes {
    uint256 private _value;
    address public owner;
    bool private _paused;
    
    // Events
    event ValueChanged(uint256 oldValue, uint256 newValue);
    event OwnershipTransferred(address oldOwner, address newOwner);
    event PauseStateChanged(bool isPaused);
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    modifier notPaused() {
        require(!_paused, "Contract is paused");
        _;
    }
    
    modifier validValue(uint256 _amount) {
        require(_amount > 0, "Value must be positive");
        require(_amount <= 1000000, "Value too large");
        _;
    }
    
    constructor(uint256 _initialValue) {
        _value = _initialValue;
        owner = msg.sender;
        _paused = false;
    }
    
    // Pure function - no state read/write
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
    
    function calculateHash(string memory _text) 
        external 
        pure 
        returns (bytes32) 
    {
        return keccak256(abi.encodePacked(_text));
    }
    
    // View function - reads state but doesn't modify
    function getValue() external view returns (uint256) {
        return _value;
    }
    
    function getContractInfo() 
        external 
        view 
        returns (
            uint256 value,
            address contractOwner,
            bool isPaused,
            uint256 blockNumber,
            uint256 timestamp
        ) 
    {
        return (_value, owner, _paused, block.number, block.timestamp);
    }
    
    // State-changing functions
    function setValue(uint256 _newValue) 
        external 
        onlyOwner 
        notPaused 
        validValue(_newValue) 
    {
        uint256 oldValue = _value;
        _value = _newValue;
        emit ValueChanged(oldValue, _newValue);
    }
    
    function incrementValue() external notPaused {
        _value += 1;
        emit ValueChanged(_value - 1, _value);
    }
    
    function transferOwnership(address _newOwner) external onlyOwner {
        require(_newOwner != address(0), "Invalid address");
        require(_newOwner != owner, "Already the owner");
        
        address oldOwner = owner;
        owner = _newOwner;
        emit OwnershipTransferred(oldOwner, _newOwner);
    }
    
    function pause() external onlyOwner {
        _paused = true;
        emit PauseStateChanged(true);
    }
    
    function unpause() external onlyOwner {
        _paused = false;
        emit PauseStateChanged(false);
    }
    
    // Payable function - can receive Ether
    function deposit() external payable notPaused {
        require(msg.value > 0, "Must send some TARA");
        // Contract now holds the sent TARA
    }
    
    function withdraw(uint256 _amount) external onlyOwner {
        require(_amount <= address(this).balance, "Insufficient balance");
        payable(owner).transfer(_amount);
    }
    
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
    
    // Function overloading
    function processData(uint256 _number) external pure returns (string memory) {
        return string(abi.encodePacked("Number: ", _number));
    }
    
    function processData(string memory _text) external pure returns (string memory) {
        return string(abi.encodePacked("Text: ", _text));
    }
    
    // Function with multiple return values
    function getMultipleValues() 
        external 
        view 
        returns (
            uint256 currentValue,
            address currentOwner,
            uint256 contractBalance,
            bool contractPaused
        ) 
    {
        return (_value, owner, address(this).balance, _paused);
    }
    
    // Function with named returns
    function calculateValues(uint256 _input) 
        external 
        pure 
        returns (
            uint256 doubled,
            uint256 squared,
            uint256 cubed
        ) 
    {
        doubled = _input * 2;
        squared = _input * _input;
        cubed = _input * _input * _input;
        // No need for explicit return when using named returns
    }
}
```

### **🔒 Advanced Modifiers**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract AdvancedModifiers {
    mapping(address => bool) public admins;
    mapping(string => bool) public lockedFunctions;
    uint256 public lastExecutionTime;
    uint256 public constant COOLDOWN_PERIOD = 1 hours;
    
    // Events
    event AdminAdded(address admin);
    event AdminRemoved(address admin);
    event FunctionLocked(string functionName);
    event FunctionUnlocked(string functionName);
    
    modifier onlyAdmin() {
        require(admins[msg.sender], "Not an admin");
        _;
    }
    
    modifier notLocked(string memory _functionName) {
        require(!lockedFunctions[_functionName], "Function is locked");
        _;
    }
    
    modifier cooldown() {
        require(
            block.timestamp >= lastExecutionTime + COOLDOWN_PERIOD,
            "Function is in cooldown"
        );
        _;
        lastExecutionTime = block.timestamp;
    }
    
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Invalid address");
        require(_addr != address(this), "Cannot be contract address");
        _;
    }
    
    modifier onlyEOA() {
        require(tx.origin == msg.sender, "Only EOA allowed");
        _;
    }
    
    modifier withinRange(uint256 _value, uint256 _min, uint256 _max) {
        require(_value >= _min && _value <= _max, "Value out of range");
        _;
    }
    
    modifier nonReentrant() {
        require(!_locked, "Reentrant call");
        _locked = true;
        _;
        _locked = false;
    }
    
    bool private _locked = false;
    
    constructor() {
        admins[msg.sender] = true;
        lastExecutionTime = block.timestamp;
    }
    
    // Admin management
    function addAdmin(address _newAdmin) 
        external 
        onlyAdmin 
        validAddress(_newAdmin) 
    {
        require(!admins[_newAdmin], "Already an admin");
        admins[_newAdmin] = true;
        emit AdminAdded(_newAdmin);
    }
    
    function removeAdmin(address _admin) 
        external 
        onlyAdmin 
        validAddress(_admin) 
    {
        require(admins[_admin], "Not an admin");
        require(_admin != msg.sender, "Cannot remove yourself");
        
        admins[_admin] = false;
        emit AdminRemoved(_admin);
    }
    
    // Function locking
    function lockFunction(string memory _functionName) external onlyAdmin {
        lockedFunctions[_functionName] = true;
        emit FunctionLocked(_functionName);
    }
    
    function unlockFunction(string memory _functionName) external onlyAdmin {
        lockedFunctions[_functionName] = false;
        emit FunctionUnlocked(_functionName);
    }
    
    // Protected functions
    function sensitiveOperation() 
        external 
        onlyAdmin 
        notLocked("sensitiveOperation") 
        cooldown 
        onlyEOA 
    {
        // Perform sensitive operation
    }
    
    function setValue(uint256 _value) 
        external 
        onlyAdmin 
        withinRange(_value, 1, 1000) 
        notLocked("setValue") 
    {
        // Set value within valid range
    }
    
    // Non-reentrant function example
    function withdraw() external nonReentrant {
        uint256 amount = address(this).balance;
        require(amount > 0, "No balance to withdraw");
        
        // This could potentially be called recursively
        // but nonReentrant modifier prevents it
        payable(msg.sender).transfer(amount);
    }
    
    // Complex modifier combination
    function complexOperation(uint256 _value, address _target) 
        external 
        onlyAdmin 
        validAddress(_target) 
        withinRange(_value, 100, 10000) 
        notLocked("complexOperation") 
        cooldown 
        nonReentrant 
    {
        // Multiple security checks applied
        // Perform complex operation here
    }
}
```

## 🔍 Control Structures

### **🔄 Conditional และ Loops**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract ControlStructures {
    uint256[] public numbers;
    mapping(address => uint256[]) public userNumbers;
    
    event NumberProcessed(uint256 number, string category);
    event ArraySorted(uint256[] sortedArray);
    
    // Conditional statements
    function categorizeNumber(uint256 _number) external returns (string memory) {
        string memory category;
        
        if (_number == 0) {
            category = "Zero";
        } else if (_number % 2 == 0) {
            category = "Even";
        } else {
            category = "Odd";
        }
        
        // Ternary operator
        string memory size = _number > 100 ? "Large" : "Small";
        
        emit NumberProcessed(_number, category);
        return string(abi.encodePacked(category, " and ", size));
    }
    
    function gradeToLetter(uint256 _score) external pure returns (string memory) {
        if (_score >= 90) {
            return "A";
        } else if (_score >= 80) {
            return "B";
        } else if (_score >= 70) {
            return "C";
        } else if (_score >= 60) {
            return "D";
        } else {
            return "F";
        }
    }
    
    // For loops
    function addMultipleNumbers(uint256[] memory _numbers) external {
        for (uint256 i = 0; i < _numbers.length; i++) {
            numbers.push(_numbers[i]);
        }
    }
    
    function calculateSum() external view returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < numbers.length; i++) {
            sum += numbers[i];
        }
        return sum;
    }
    
    function findEvens() external view returns (uint256[] memory) {
        // First pass to count evens
        uint256 evenCount = 0;
        for (uint256 i = 0; i < numbers.length; i++) {
            if (numbers[i] % 2 == 0) {
                evenCount++;
            }
        }
        
        // Create array with exact size
        uint256[] memory evens = new uint256[](evenCount);
        uint256 evenIndex = 0;
        
        // Second pass to populate array
        for (uint256 i = 0; i < numbers.length; i++) {
            if (numbers[i] % 2 == 0) {
                evens[evenIndex] = numbers[i];
                evenIndex++;
            }
        }
        
        return evens;
    }
    
    // While loops (careful with gas limits)
    function findFirstMultiple(uint256 _number, uint256 _multiple) 
        external 
        view 
        returns (uint256) 
    {
        uint256 current = _number;
        uint256 iterations = 0;
        
        while (current % _multiple != 0 && iterations < 1000) {
            current++;
            iterations++;
        }
        
        require(iterations < 1000, "No multiple found within limit");
        return current;
    }
    
    // Nested loops
    function createMultiplicationTable(uint256 _size) 
        external 
        pure 
        returns (uint256[][] memory) 
    {
        require(_size <= 10, "Size too large");
        
        uint256[][] memory table = new uint256[][](_size);
        
        for (uint256 i = 0; i < _size; i++) {
            table[i] = new uint256[](_size);
            for (uint256 j = 0; j < _size; j++) {
                table[i][j] = (i + 1) * (j + 1);
            }
        }
        
        return table;
    }
    
    // Continue and break simulation (using return/early exit)
    function processNumbers() external view returns (uint256) {
        uint256 sum = 0;
        
        for (uint256 i = 0; i < numbers.length; i++) {
            // Skip zeros (simulate continue)
            if (numbers[i] == 0) {
                continue;
            }
            
            // Stop at first number > 1000 (simulate break)
            if (numbers[i] > 1000) {
                break;
            }
            
            sum += numbers[i];
        }
        
        return sum;
    }
    
    // Complex control flow
    function complexProcessing(uint256[] memory _data) 
        external 
        pure 
        returns (
            uint256 processedCount,
            uint256 sum,
            uint256 max,
            uint256 min
        ) 
    {
        require(_data.length > 0, "Empty array");
        
        processedCount = 0;
        sum = 0;
        max = 0;
        min = type(uint256).max;
        
        for (uint256 i = 0; i < _data.length; i++) {
            uint256 value = _data[i];
            
            // Skip invalid values
            if (value == 0) {
                continue;
            }
            
            // Process value
            processedCount++;
            sum += value;
            
            // Update max
            if (value > max) {
                max = value;
            }
            
            // Update min
            if (value < min) {
                min = value;
            }
            
            // Early termination condition
            if (processedCount >= 100) {
                break;
            }
        }
        
        return (processedCount, sum, max, min);
    }
    
    // Switch-like behavior using mapping
    mapping(uint256 => string) private dayNames;
    
    constructor() {
        dayNames[1] = "Monday";
        dayNames[2] = "Tuesday";
        dayNames[3] = "Wednesday";
        dayNames[4] = "Thursday";
        dayNames[5] = "Friday";
        dayNames[6] = "Saturday";
        dayNames[7] = "Sunday";
    }
    
    function getDayName(uint256 _dayNumber) external view returns (string memory) {
        require(_dayNumber >= 1 && _dayNumber <= 7, "Invalid day number");
        return dayNames[_dayNumber];
    }
    
    // Utility functions
    function addNumber(uint256 _number) external {
        numbers.push(_number);
    }
    
    function addUserNumber(uint256 _number) external {
        userNumbers[msg.sender].push(_number);
    }
    
    function getNumbers() external view returns (uint256[] memory) {
        return numbers;
    }
    
    function getUserNumbers() external view returns (uint256[] memory) {
        return userNumbers[msg.sender];
    }
    
    function clearNumbers() external {
        delete numbers;
    }
}
```

## 🚨 Error Handling

### **⚠️ Require, Assert, และ Revert**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Custom error definitions (more gas efficient)
error InsufficientBalance(uint256 requested, uint256 available);
error UnauthorizedAccess(address caller, address required);
error InvalidInput(string parameter, uint256 value);
error ContractPaused();

contract ErrorHandling {
    uint256 public totalSupply;
    mapping(address => uint256) public balances;
    address public owner;
    bool public paused = false;
    
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event ErrorOccurred(string errorType, string message);
    
    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert UnauthorizedAccess(msg.sender, owner);
        }
        _;
    }
    
    modifier notPaused() {
        if (paused) {
            revert ContractPaused();
        }
        _;
    }
    
    constructor(uint256 _initialSupply) {
        require(_initialSupply > 0, "Initial supply must be positive");
        
        totalSupply = _initialSupply;
        balances[msg.sender] = _initialSupply;
        owner = msg.sender;
    }
    
    // Using require for input validation
    function transfer(address _to, uint256 _amount) 
        external 
        notPaused 
        returns (bool) 
    {
        require(_to != address(0), "Cannot transfer to zero address");
        require(_amount > 0, "Amount must be positive");
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        
        balances[msg.sender] -= _amount;
        balances[_to] += _amount;
        
        emit Transfer(msg.sender, _to, _amount);
        return true;
    }
    
    // Using custom errors (more gas efficient)
    function transferWithCustomErrors(address _to, uint256 _amount) 
        external 
        notPaused 
        returns (bool) 
    {
        if (_to == address(0)) {
            revert InvalidInput("recipient", uint256(uint160(_to)));
        }
        
        if (_amount == 0) {
            revert InvalidInput("amount", _amount);
        }
        
        if (balances[msg.sender] < _amount) {
            revert InsufficientBalance(_amount, balances[msg.sender]);
        }
        
        balances[msg.sender] -= _amount;
        balances[_to] += _amount;
        
        emit Transfer(msg.sender, _to, _amount);
        return true;
    }
    
    // Using assert for internal consistency checks
    function mint(address _to, uint256 _amount) external onlyOwner notPaused {
        require(_to != address(0), "Cannot mint to zero address");
        require(_amount > 0, "Amount must be positive");
        
        uint256 oldTotalSupply = totalSupply;
        uint256 oldBalance = balances[_to];
        
        totalSupply += _amount;
        balances[_to] += _amount;
        
        // Assert for internal consistency
        assert(totalSupply == oldTotalSupply + _amount);
        assert(balances[_to] == oldBalance + _amount);
        
        emit Transfer(address(0), _to, _amount);
    }
    
    // Try-catch for external calls
    function safeExternalCall(address _target, bytes memory _data) 
        external 
        onlyOwner 
        returns (bool success, bytes memory result) 
    {
        try this.externalCall(_target, _data) returns (bytes memory returnData) {
            return (true, returnData);
        } catch Error(string memory reason) {
            emit ErrorOccurred("Error", reason);
            return (false, bytes(reason));
        } catch Panic(uint errorCode) {
            emit ErrorOccurred("Panic", string(abi.encodePacked("Code: ", errorCode)));
            return (false, abi.encodePacked("Panic code: ", errorCode));
        } catch (bytes memory lowLevelData) {
            emit ErrorOccurred("LowLevel", "Unknown error");
            return (false, lowLevelData);
        }
    }
    
    function externalCall(address _target, bytes memory _data) 
        external 
        returns (bytes memory) 
    {
        (bool success, bytes memory result) = _target.call(_data);
        require(success, "External call failed");
        return result;
    }
    
    // Error recovery patterns
    function batchTransfer(
        address[] memory _recipients, 
        uint256[] memory _amounts
    ) external notPaused returns (bool[] memory results) {
        require(_recipients.length == _amounts.length, "Array length mismatch");
        
        results = new bool[](_recipients.length);
        
        for (uint256 i = 0; i < _recipients.length; i++) {
            try this.transferInternal(msg.sender, _recipients[i], _amounts[i]) {
                results[i] = true;
            } catch {
                results[i] = false;
                emit ErrorOccurred("BatchTransfer", "Transfer failed");
            }
        }
        
        return results;
    }
    
    function transferInternal(address _from, address _to, uint256 _amount) 
        external 
        returns (bool) 
    {
        require(msg.sender == address(this), "Internal function");
        require(_to != address(0), "Invalid recipient");
        require(_amount > 0, "Invalid amount");
        require(balances[_from] >= _amount, "Insufficient balance");
        
        balances[_from] -= _amount;
        balances[_to] += _amount;
        
        emit Transfer(_from, _to, _amount);
        return true;
    }
    
    // Graceful degradation
    function getBalanceWithFallback(address _user) 
        external 
        view 
        returns (uint256 balance, bool isValid) 
    {
        if (_user == address(0)) {
            return (0, false);
        }
        
        return (balances[_user], true);
    }
    
    // Administrative functions
    function pause() external onlyOwner {
        paused = true;
    }
    
    function unpause() external onlyOwner {
        paused = false;
    }
    
    function emergencyWithdraw() external onlyOwner {
        require(paused, "Must be paused for emergency");
        payable(owner).transfer(address(this).balance);
    }
    
    // Validation helpers
    function validateTransferParams(address _to, uint256 _amount) 
        external 
        pure 
        returns (bool isValid, string memory errorMsg) 
    {
        if (_to == address(0)) {
            return (false, "Invalid recipient address");
        }
        
        if (_amount == 0) {
            return (false, "Amount must be positive");
        }
        
        return (true, "");
    }
    
    function checkBalance(address _user, uint256 _required) 
        external 
        view 
        returns (bool sufficient, uint256 shortfall) 
    {
        uint256 balance = balances[_user];
        
        if (balance >= _required) {
            return (true, 0);
        } else {
            return (false, _required - balance);
        }
    }
}
```

## 🎯 แบบฝึกหัด

### **📝 แบบฝึกหัดที่ 1: Basic Contract**
สร้าง contract ที่มี:
- State variables ทุกประเภท
- Functions ด้วย visibility ต่างๆ
- Events และ modifiers

### **🔧 แบบฝึกหัดที่ 2: Calculator Contract**
สร้าง calculator ที่มี:
- Basic arithmetic operations
- Error handling
- History tracking
- Access control

### **🎯 แบบฝึกหัดที่ 3: Voting System**
สร้าง voting system ที่มี:
- Candidate registration
- Voting mechanisms
- Result calculation
- Security measures

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 2: Development Environment Setup](./02-development-environment.md)  
**บทถัดไป**: [บทที่ 4: First Smart Contract](./04-first-smart-contract.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Solidity Documentation](https://docs.soliditylang.org/)
- [Solidity by Example](https://solidity-by-example.org/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)

---

ตอนนี้คุณมีความรู้พื้นฐาน Solidity แล้ว! ในบทถัดไป เราจะเขียน Smart Contract แรกบน Taraxa 🚀
