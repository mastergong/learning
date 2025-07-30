# บทที่ 7: Token Standards (ERC-20 & ERC-721)

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Token Standards ที่สำคัญ (ERC-20, ERC-721, ERC-1155)
- สร้าง Fungible และ Non-Fungible Tokens บน Taraxa
- ทำความเข้าใจ Token Economics และ Utility Design
- ผสานรวม Tokens เข้ากับ Applications

## 🪙 ERC-20: Fungible Tokens

### **📚 ERC-20 Standard Overview**

```mermaid
graph TD
    A[ERC-20 Token] --> B[Standard Functions]
    A --> C[Events]
    A --> D[Optional Functions]
    
    B --> E[totalSupply()]
    B --> F[balanceOf()]
    B --> G[transfer()]
    B --> H[approve()]
    B --> I[transferFrom()]
    B --> J[allowance()]
    
    C --> K[Transfer]
    C --> L[Approval]
    
    D --> M[name()]
    D --> N[symbol()]
    D --> O[decimals()]
```

### **💎 Advanced ERC-20 Implementation**

```solidity
// contracts/TaraxaCommunityToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

/**
 * @title TaraxaCommunityToken (TCT)
 * @dev Advanced ERC-20 token with multiple features for community governance
 * 
 * Features:
 * - Fixed supply with burn capability
 * - Pausable transfers for emergency situations
 * - Reward distribution system
 * - Anti-whale protection
 * - Governance voting power based on holdings
 */
contract TaraxaCommunityToken is ERC20, ERC20Burnable, ERC20Pausable, Ownable, ReentrancyGuard {
    using SafeMath for uint256;

    // Token Configuration
    uint256 public constant INITIAL_SUPPLY = 1_000_000 * 10**18; // 1M tokens
    uint256 public constant MAX_TX_AMOUNT = 10_000 * 10**18;     // Max transaction: 10K tokens
    uint256 public constant MAX_WALLET_AMOUNT = 50_000 * 10**18; // Max wallet: 50K tokens
    
    // Reward System
    uint256 public rewardPool;
    uint256 public lastRewardDistribution;
    uint256 public constant REWARD_DISTRIBUTION_INTERVAL = 30 days;
    uint256 public constant REWARD_RATE = 2; // 2% of pool per distribution
    
    // Anti-whale protection
    bool public antiWhaleEnabled = true;
    mapping(address => bool) public isExcludedFromLimits;
    
    // Community features
    mapping(address => uint256) public contributionPoints;
    mapping(address => bool) public isCommunityMember;
    uint256 public totalCommunityMembers;
    
    // Governance
    mapping(address => uint256) public votingPower;
    uint256 public totalVotingPower;
    
    // Events
    event RewardDistributed(uint256 amount, uint256 recipients);
    event ContributionAdded(address indexed member, uint256 points);
    event CommunityMemberAdded(address indexed member);
    event CommunityMemberRemoved(address indexed member);
    event AntiWhaleToggled(bool enabled);
    event VotingPowerUpdated(address indexed member, uint256 oldPower, uint256 newPower);

    constructor() ERC20("Taraxa Community Token", "TCT") {
        _mint(msg.sender, INITIAL_SUPPLY);
        
        // Exclude owner from limits
        isExcludedFromLimits[msg.sender] = true;
        isExcludedFromLimits[address(this)] = true;
        
        // Initialize reward system
        lastRewardDistribution = block.timestamp;
        
        // Set initial voting power
        _updateVotingPower(msg.sender, INITIAL_SUPPLY);
    }

    // ============= OVERRIDE FUNCTIONS =============

    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Pausable) {
        super._beforeTokenTransfer(from, to, amount);
        
        // Anti-whale protection
        if (antiWhaleEnabled && from != address(0) && to != address(0)) {
            _checkTransferLimits(from, to, amount);
        }
    }

    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override {
        super._afterTokenTransfer(from, to, amount);
        
        // Update voting power
        if (from != address(0)) {
            _updateVotingPower(from, balanceOf(from));
        }
        if (to != address(0)) {
            _updateVotingPower(to, balanceOf(to));
        }
    }

    // ============= ANTI-WHALE PROTECTION =============

    function _checkTransferLimits(address from, address to, uint256 amount) internal view {
        // Check transaction amount limit
        if (!isExcludedFromLimits[from] && !isExcludedFromLimits[to]) {
            require(amount <= MAX_TX_AMOUNT, "TCT: Transfer amount exceeds limit");
        }
        
        // Check wallet balance limit for recipient
        if (!isExcludedFromLimits[to]) {
            uint256 recipientBalance = balanceOf(to).add(amount);
            require(recipientBalance <= MAX_WALLET_AMOUNT, "TCT: Wallet balance would exceed limit");
        }
    }

    function setAntiWhaleEnabled(bool _enabled) external onlyOwner {
        antiWhaleEnabled = _enabled;
        emit AntiWhaleToggled(_enabled);
    }

    function excludeFromLimits(address account, bool excluded) external onlyOwner {
        isExcludedFromLimits[account] = excluded;
    }

    function setMaxTransactionAmount(uint256 _maxTxAmount) external onlyOwner {
        require(_maxTxAmount >= 1000 * 10**18, "TCT: Too restrictive");
        // Update constant would require contract upgrade
    }

    // ============= COMMUNITY SYSTEM =============

    function addCommunityMember(address member) external onlyOwner {
        require(!isCommunityMember[member], "TCT: Already a community member");
        
        isCommunityMember[member] = true;
        totalCommunityMembers++;
        
        emit CommunityMemberAdded(member);
    }

    function removeCommunityMember(address member) external onlyOwner {
        require(isCommunityMember[member], "TCT: Not a community member");
        
        isCommunityMember[member] = false;
        totalCommunityMembers--;
        contributionPoints[member] = 0; // Reset contribution points
        
        emit CommunityMemberRemoved(member);
    }

    function addContributionPoints(address member, uint256 points) external onlyOwner {
        require(isCommunityMember[member], "TCT: Not a community member");
        
        contributionPoints[member] = contributionPoints[member].add(points);
        emit ContributionAdded(member, points);
    }

    function getContributionLevel(address member) public view returns (string memory) {
        uint256 points = contributionPoints[member];
        
        if (points >= 10000) return "Legendary Contributor";
        if (points >= 5000) return "Epic Contributor";
        if (points >= 2500) return "Elite Contributor";
        if (points >= 1000) return "Advanced Contributor";
        if (points >= 500) return "Active Contributor";
        if (points >= 100) return "Contributor";
        return "New Member";
    }

    // ============= REWARD SYSTEM =============

    function depositRewards() external payable onlyOwner {
        require(msg.value > 0, "TCT: No rewards to deposit");
        rewardPool = rewardPool.add(msg.value);
    }

    function distributeRewards() external nonReentrant {
        require(
            block.timestamp >= lastRewardDistribution.add(REWARD_DISTRIBUTION_INTERVAL),
            "TCT: Too early for reward distribution"
        );
        require(rewardPool > 0, "TCT: No rewards available");
        require(totalCommunityMembers > 0, "TCT: No community members");
        
        uint256 rewardAmount = rewardPool.mul(REWARD_RATE).div(100);
        require(rewardAmount > 0, "TCT: Reward amount too small");
        
        uint256 individualReward = rewardAmount.div(totalCommunityMembers);
        uint256 distributedAmount = 0;
        uint256 recipients = 0;
        
        // Distribute to all community members
        address[] memory members = _getAllCommunityMembers();
        
        for (uint256 i = 0; i < members.length; i++) {
            if (isCommunityMember[members[i]]) {
                (bool success, ) = members[i].call{value: individualReward}("");
                if (success) {
                    distributedAmount = distributedAmount.add(individualReward);
                    recipients++;
                }
            }
        }
        
        rewardPool = rewardPool.sub(distributedAmount);
        lastRewardDistribution = block.timestamp;
        
        emit RewardDistributed(distributedAmount, recipients);
    }

    function _getAllCommunityMembers() internal view returns (address[] memory) {
        // This is a simplified version - in production, you'd maintain a members array
        address[] memory members = new address[](totalCommunityMembers);
        // Implementation would iterate through all addresses to find community members
        // For simplicity, we'll just return empty array here
        return members;
    }

    function claimRewards() external nonReentrant {
        require(isCommunityMember[msg.sender], "TCT: Not a community member");
        
        // Calculate individual reward based on contribution points and token holdings
        uint256 baseReward = _calculateBaseReward(msg.sender);
        uint256 contributionBonus = _calculateContributionBonus(msg.sender);
        uint256 holdingBonus = _calculateHoldingBonus(msg.sender);
        
        uint256 totalReward = baseReward.add(contributionBonus).add(holdingBonus);
        
        require(totalReward <= rewardPool, "TCT: Insufficient reward pool");
        require(totalReward > 0, "TCT: No rewards available");
        
        rewardPool = rewardPool.sub(totalReward);
        
        (bool success, ) = msg.sender.call{value: totalReward}("");
        require(success, "TCT: Reward transfer failed");
    }

    function _calculateBaseReward(address member) internal view returns (uint256) {
        if (!isCommunityMember[member]) return 0;
        return rewardPool.div(totalCommunityMembers).div(10); // 10% of equal distribution
    }

    function _calculateContributionBonus(address member) internal view returns (uint256) {
        uint256 points = contributionPoints[member];
        if (points == 0) return 0;
        
        // Bonus based on contribution points (up to 50% extra)
        uint256 bonusMultiplier = points > 5000 ? 50 : points.div(100);
        return _calculateBaseReward(member).mul(bonusMultiplier).div(100);
    }

    function _calculateHoldingBonus(address member) internal view returns (uint256) {
        uint256 balance = balanceOf(member);
        if (balance == 0) return 0;
        
        // Bonus based on token holdings (up to 30% extra)
        uint256 holdingPercent = balance.mul(100).div(totalSupply());
        uint256 bonusMultiplier = holdingPercent > 30 ? 30 : holdingPercent;
        return _calculateBaseReward(member).mul(bonusMultiplier).div(100);
    }

    // ============= GOVERNANCE SYSTEM =============

    function _updateVotingPower(address account, uint256 newBalance) internal {
        uint256 oldPower = votingPower[account];
        uint256 newPower = _calculateVotingPower(newBalance);
        
        if (oldPower != newPower) {
            totalVotingPower = totalVotingPower.sub(oldPower).add(newPower);
            votingPower[account] = newPower;
            
            emit VotingPowerUpdated(account, oldPower, newPower);
        }
    }

    function _calculateVotingPower(uint256 balance) internal pure returns (uint256) {
        // Square root of balance for fairer voting (prevents whale dominance)
        return _sqrt(balance);
    }

    function _sqrt(uint256 x) internal pure returns (uint256) {
        if (x == 0) return 0;
        uint256 z = (x + 1) / 2;
        uint256 y = x;
        while (z < y) {
            y = z;
            z = (x / z + z) / 2;
        }
        return y;
    }

    function getVotingPowerPercentage(address account) external view returns (uint256) {
        if (totalVotingPower == 0) return 0;
        return votingPower[account].mul(10000).div(totalVotingPower); // Basis points (0.01%)
    }

    // ============= UTILITY FUNCTIONS =============

    function mint(address to, uint256 amount) external onlyOwner {
        require(totalSupply().add(amount) <= INITIAL_SUPPLY.mul(2), "TCT: Max supply exceeded");
        _mint(to, amount);
    }

    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }

    function emergencyWithdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "TCT: No funds to withdraw");
        
        (bool success, ) = owner().call{value: balance}("");
        require(success, "TCT: Withdrawal failed");
    }

    // ============= VIEW FUNCTIONS =============

    function getTokenInfo() external view returns (
        uint256 _totalSupply,
        uint256 _rewardPool,
        uint256 _totalCommunityMembers,
        uint256 _totalVotingPower,
        bool _antiWhaleEnabled,
        bool _paused
    ) {
        return (
            totalSupply(),
            rewardPool,
            totalCommunityMembers,
            totalVotingPower,
            antiWhaleEnabled,
            paused()
        );
    }

    function getUserInfo(address user) external view returns (
        uint256 _balance,
        uint256 _contributionPoints,
        string memory _contributionLevel,
        bool _isCommunityMember,
        uint256 _votingPower,
        uint256 _votingPercentage,
        bool _isExcludedFromLimits
    ) {
        return (
            balanceOf(user),
            contributionPoints[user],
            getContributionLevel(user),
            isCommunityMember[user],
            votingPower[user],
            totalVotingPower > 0 ? votingPower[user].mul(10000).div(totalVotingPower) : 0,
            isExcludedFromLimits[user]
        );
    }

    function getRewardInfo() external view returns (
        uint256 _rewardPool,
        uint256 _lastDistribution,
        uint256 _nextDistribution,
        uint256 _distributionInterval,
        uint256 _rewardRate
    ) {
        return (
            rewardPool,
            lastRewardDistribution,
            lastRewardDistribution.add(REWARD_DISTRIBUTION_INTERVAL),
            REWARD_DISTRIBUTION_INTERVAL,
            REWARD_RATE
        );
    }

    // Allow contract to receive ETH for rewards
    receive() external payable {
        rewardPool = rewardPool.add(msg.value);
    }
}
```

## 🎨 ERC-721: Non-Fungible Tokens (NFTs)

### **🖼️ Advanced NFT Implementation**

```solidity
// contracts/TaraxaCommunityNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

/**
 * @title TaraxaCommunityNFT
 * @dev Advanced ERC-721 NFT collection for Taraxa community members
 * 
 * Features:
 * - Tiered membership system
 * - Dynamic metadata generation
 * - Whitelist minting with Merkle proofs
 * - Revenue sharing with holders
 * - Evolution/upgrade system
 * - Staking rewards
 */
contract TaraxaCommunityNFT is 
    ERC721, 
    ERC721Enumerable, 
    ERC721URIStorage, 
    ERC721Burnable, 
    Ownable, 
    ReentrancyGuard, 
    Pausable 
{
    using Counters for Counters.Counter;
    using Strings for uint256;

    // Token counter
    Counters.Counter private _tokenIdCounter;

    // Collection Configuration
    uint256 public constant MAX_SUPPLY = 10000;
    uint256 public constant MAX_MINT_PER_TX = 5;
    uint256 public constant MAX_MINT_PER_WALLET = 10;
    
    // Pricing
    uint256 public publicMintPrice = 0.1 ether;
    uint256 public whitelistMintPrice = 0.05 ether;
    
    // Minting phases
    bool public whitelistMintActive = false;
    bool public publicMintActive = false;
    bytes32 public merkleRoot;
    
    // NFT Tiers
    enum Tier { BRONZE, SILVER, GOLD, PLATINUM, DIAMOND }
    
    struct NFTMetadata {
        Tier tier;
        uint256 power;
        uint256 experience;
        uint256 createdAt;
        uint256 lastUpgrade;
        bool isStaked;
        string customAttributes;
    }
    
    // Token metadata
    mapping(uint256 => NFTMetadata) public tokenMetadata;
    mapping(address => uint256) public mintedCount;
    mapping(bytes32 => bool) public whitelistClaimed;
    
    // Staking system
    mapping(uint256 => uint256) public stakingStartTime;
    mapping(uint256 => uint256) public accumulatedRewards;
    uint256 public stakingRewardRate = 1 ether; // Rewards per day
    uint256 public totalStaked = 0;
    
    // Revenue sharing
    uint256 public revenuePool;
    uint256 public lastRevenueDistribution;
    mapping(address => uint256) public revenueShares;
    mapping(address => uint256) public claimedRevenue;
    
    // Base URI
    string private _baseTokenURI;
    
    // Events
    event NFTMinted(address indexed to, uint256 indexed tokenId, Tier tier);
    event NFTUpgraded(uint256 indexed tokenId, Tier oldTier, Tier newTier);
    event NFTStaked(uint256 indexed tokenId, address indexed owner);
    event NFTUnstaked(uint256 indexed tokenId, address indexed owner, uint256 rewards);
    event RevenueDistributed(uint256 amount, uint256 holders);
    event ExperienceGained(uint256 indexed tokenId, uint256 experience);

    constructor(
        string memory name,
        string memory symbol,
        string memory baseURI
    ) ERC721(name, symbol) {
        _baseTokenURI = baseURI;
        _tokenIdCounter.increment(); // Start from 1
    }

    // ============= MINTING FUNCTIONS =============

    function whitelistMint(
        bytes32[] calldata merkleProof,
        uint256 quantity
    ) external payable nonReentrant whenNotPaused {
        require(whitelistMintActive, "TCNFT: Whitelist mint not active");
        require(quantity > 0 && quantity <= MAX_MINT_PER_TX, "TCNFT: Invalid quantity");
        require(msg.value >= whitelistMintPrice * quantity, "TCNFT: Insufficient payment");
        
        // Verify whitelist
        bytes32 leaf = keccak256(abi.encodePacked(msg.sender));
        require(
            MerkleProof.verify(merkleProof, merkleRoot, leaf),
            "TCNFT: Invalid whitelist proof"
        );
        require(!whitelistClaimed[leaf], "TCNFT: Already claimed");
        
        // Check supply and wallet limits
        require(_tokenIdCounter.current() + quantity <= MAX_SUPPLY, "TCNFT: Exceeds max supply");
        require(mintedCount[msg.sender] + quantity <= MAX_MINT_PER_WALLET, "TCNFT: Exceeds wallet limit");
        
        whitelistClaimed[leaf] = true;
        mintedCount[msg.sender] += quantity;
        
        for (uint256 i = 0; i < quantity; i++) {
            _mintNFT(msg.sender);
        }
        
        // Add to revenue pool
        revenuePool += msg.value;
    }

    function publicMint(uint256 quantity) external payable nonReentrant whenNotPaused {
        require(publicMintActive, "TCNFT: Public mint not active");
        require(quantity > 0 && quantity <= MAX_MINT_PER_TX, "TCNFT: Invalid quantity");
        require(msg.value >= publicMintPrice * quantity, "TCNFT: Insufficient payment");
        require(_tokenIdCounter.current() + quantity <= MAX_SUPPLY, "TCNFT: Exceeds max supply");
        require(mintedCount[msg.sender] + quantity <= MAX_MINT_PER_WALLET, "TCNFT: Exceeds wallet limit");
        
        mintedCount[msg.sender] += quantity;
        
        for (uint256 i = 0; i < quantity; i++) {
            _mintNFT(msg.sender);
        }
        
        // Add to revenue pool
        revenuePool += msg.value;
    }

    function ownerMint(address to, uint256 quantity) external onlyOwner {
        require(_tokenIdCounter.current() + quantity <= MAX_SUPPLY, "TCNFT: Exceeds max supply");
        
        for (uint256 i = 0; i < quantity; i++) {
            _mintNFT(to);
        }
    }

    function _mintNFT(address to) internal {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        
        // Determine tier based on randomness and rarity
        Tier tier = _determineTier(tokenId, to);
        uint256 basePower = _calculateBasePower(tier);
        
        // Set initial metadata
        tokenMetadata[tokenId] = NFTMetadata({
            tier: tier,
            power: basePower,
            experience: 0,
            createdAt: block.timestamp,
            lastUpgrade: block.timestamp,
            isStaked: false,
            customAttributes: ""
        });
        
        _safeMint(to, tokenId);
        
        emit NFTMinted(to, tokenId, tier);
    }

    function _determineTier(uint256 tokenId, address to) internal view returns (Tier) {
        // Pseudo-random tier determination with rarity weights
        uint256 randomValue = uint256(keccak256(abi.encodePacked(
            block.timestamp,
            block.difficulty,
            tokenId,
            to
        ))) % 100;
        
        if (randomValue < 5) return Tier.DIAMOND;     // 5%
        if (randomValue < 15) return Tier.PLATINUM;   // 10%
        if (randomValue < 35) return Tier.GOLD;       // 20%
        if (randomValue < 65) return Tier.SILVER;     // 30%
        return Tier.BRONZE;                           // 35%
    }

    function _calculateBasePower(Tier tier) internal pure returns (uint256) {
        if (tier == Tier.DIAMOND) return 1000;
        if (tier == Tier.PLATINUM) return 750;
        if (tier == Tier.GOLD) return 500;
        if (tier == Tier.SILVER) return 300;
        return 100; // BRONZE
    }

    // ============= UPGRADE SYSTEM =============

    function upgradeNFT(uint256 tokenId) external nonReentrant {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "TCNFT: Not token owner");
        require(!tokenMetadata[tokenId].isStaked, "TCNFT: Cannot upgrade staked NFT");
        
        NFTMetadata storage metadata = tokenMetadata[tokenId];
        require(metadata.tier != Tier.DIAMOND, "TCNFT: Already max tier");
        require(metadata.experience >= _getUpgradeRequirement(metadata.tier), "TCNFT: Insufficient experience");
        
        // Calculate upgrade cost
        uint256 upgradeCost = _getUpgradeCost(metadata.tier);
        require(msg.value >= upgradeCost, "TCNFT: Insufficient payment for upgrade");
        
        Tier oldTier = metadata.tier;
        metadata.tier = Tier(uint256(metadata.tier) + 1);
        metadata.power = _calculateBasePower(metadata.tier);
        metadata.lastUpgrade = block.timestamp;
        metadata.experience = 0; // Reset experience after upgrade
        
        // Add upgrade cost to revenue pool
        revenuePool += msg.value;
        
        emit NFTUpgraded(tokenId, oldTier, metadata.tier);
    }

    function _getUpgradeRequirement(Tier tier) internal pure returns (uint256) {
        if (tier == Tier.BRONZE) return 1000;
        if (tier == Tier.SILVER) return 2500;
        if (tier == Tier.GOLD) return 5000;
        if (tier == Tier.PLATINUM) return 10000;
        return type(uint256).max; // DIAMOND cannot upgrade
    }

    function _getUpgradeCost(Tier tier) internal view returns (uint256) {
        if (tier == Tier.BRONZE) return publicMintPrice / 2;
        if (tier == Tier.SILVER) return publicMintPrice;
        if (tier == Tier.GOLD) return publicMintPrice * 2;
        if (tier == Tier.PLATINUM) return publicMintPrice * 4;
        return type(uint256).max; // DIAMOND cannot upgrade
    }

    function addExperience(uint256 tokenId, uint256 experience) external onlyOwner {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        
        tokenMetadata[tokenId].experience += experience;
        emit ExperienceGained(tokenId, experience);
    }

    // ============= STAKING SYSTEM =============

    function stakeNFT(uint256 tokenId) external nonReentrant {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "TCNFT: Not token owner");
        require(!tokenMetadata[tokenId].isStaked, "TCNFT: Already staked");
        
        tokenMetadata[tokenId].isStaked = true;
        stakingStartTime[tokenId] = block.timestamp;
        totalStaked++;
        
        emit NFTStaked(tokenId, msg.sender);
    }

    function unstakeNFT(uint256 tokenId) external nonReentrant {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "TCNFT: Not token owner");
        require(tokenMetadata[tokenId].isStaked, "TCNFT: Not staked");
        
        // Calculate rewards
        uint256 stakingDuration = block.timestamp - stakingStartTime[tokenId];
        uint256 rewards = _calculateStakingRewards(tokenId, stakingDuration);
        
        tokenMetadata[tokenId].isStaked = false;
        stakingStartTime[tokenId] = 0;
        accumulatedRewards[tokenId] += rewards;
        totalStaked--;
        
        // Add experience based on staking duration
        uint256 experienceGained = stakingDuration / 1 days; // 1 exp per day
        tokenMetadata[tokenId].experience += experienceGained;
        
        emit NFTUnstaked(tokenId, msg.sender, rewards);
        
        // Transfer rewards if any
        if (rewards > 0) {
            (bool success, ) = msg.sender.call{value: rewards}("");
            require(success, "TCNFT: Reward transfer failed");
        }
    }

    function _calculateStakingRewards(uint256 tokenId, uint256 duration) internal view returns (uint256) {
        NFTMetadata memory metadata = tokenMetadata[tokenId];
        uint256 days = duration / 1 days;
        uint256 baseReward = stakingRewardRate * days;
        
        // Multiply by tier bonus
        uint256 tierMultiplier = uint256(metadata.tier) + 1;
        return baseReward * tierMultiplier;
    }

    function getStakingInfo(uint256 tokenId) external view returns (
        bool isStaked,
        uint256 stakingTime,
        uint256 pendingRewards,
        uint256 accumulatedTotal
    ) {
        NFTMetadata memory metadata = tokenMetadata[tokenId];
        
        if (metadata.isStaked) {
            uint256 stakingDuration = block.timestamp - stakingStartTime[tokenId];
            pendingRewards = _calculateStakingRewards(tokenId, stakingDuration);
        }
        
        return (
            metadata.isStaked,
            stakingStartTime[tokenId],
            pendingRewards,
            accumulatedRewards[tokenId]
        );
    }

    // ============= REVENUE SHARING =============

    function distributeRevenue() external nonReentrant {
        require(revenuePool > 0, "TCNFT: No revenue to distribute");
        require(totalSupply() > 0, "TCNFT: No NFTs exist");
        
        uint256 revenuePerNFT = revenuePool / totalSupply();
        
        // Track distribution for each holder
        for (uint256 i = 1; i < _tokenIdCounter.current(); i++) {
            if (_exists(i)) {
                address owner = ownerOf(i);
                revenueShares[owner] += revenuePerNFT;
            }
        }
        
        lastRevenueDistribution = block.timestamp;
        emit RevenueDistributed(revenuePool, totalSupply());
        
        revenuePool = 0; // Reset pool after distribution
    }

    function claimRevenue() external nonReentrant {
        uint256 claimableAmount = revenueShares[msg.sender] - claimedRevenue[msg.sender];
        require(claimableAmount > 0, "TCNFT: No revenue to claim");
        
        claimedRevenue[msg.sender] += claimableAmount;
        
        (bool success, ) = msg.sender.call{value: claimableAmount}("");
        require(success, "TCNFT: Revenue transfer failed");
    }

    function getClaimableRevenue(address holder) external view returns (uint256) {
        return revenueShares[holder] - claimedRevenue[holder];
    }

    // ============= METADATA FUNCTIONS =============

    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        require(_exists(tokenId), "TCNFT: URI query for nonexistent token");
        
        // If there's a custom URI set, use it
        string memory _tokenURI = super.tokenURI(tokenId);
        if (bytes(_tokenURI).length > 0) {
            return _tokenURI;
        }
        
        // Otherwise, generate dynamic metadata
        return _generateDynamicMetadata(tokenId);
    }

    function _generateDynamicMetadata(uint256 tokenId) internal view returns (string memory) {
        NFTMetadata memory metadata = tokenMetadata[tokenId];
        
        string memory tierName = _getTierName(metadata.tier);
        
        // This would typically return a URL to your metadata API
        // For demonstration, we'll return a constructed URL
        return string(abi.encodePacked(
            _baseTokenURI,
            tokenId.toString(),
            "?tier=", tierName,
            "&power=", metadata.power.toString(),
            "&experience=", metadata.experience.toString()
        ));
    }

    function _getTierName(Tier tier) internal pure returns (string memory) {
        if (tier == Tier.DIAMOND) return "Diamond";
        if (tier == Tier.PLATINUM) return "Platinum";
        if (tier == Tier.GOLD) return "Gold";
        if (tier == Tier.SILVER) return "Silver";
        return "Bronze";
    }

    function setCustomAttributes(uint256 tokenId, string memory attributes) external {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "TCNFT: Not token owner");
        
        tokenMetadata[tokenId].customAttributes = attributes;
    }

    // ============= ADMIN FUNCTIONS =============

    function setMintPhases(bool _whitelistActive, bool _publicActive) external onlyOwner {
        whitelistMintActive = _whitelistActive;
        publicMintActive = _publicActive;
    }

    function setMintPrices(uint256 _publicPrice, uint256 _whitelistPrice) external onlyOwner {
        publicMintPrice = _publicPrice;
        whitelistMintPrice = _whitelistPrice;
    }

    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        merkleRoot = _merkleRoot;
    }

    function setBaseURI(string memory baseURI) external onlyOwner {
        _baseTokenURI = baseURI;
    }

    function setStakingRewardRate(uint256 _rewardRate) external onlyOwner {
        stakingRewardRate = _rewardRate;
    }

    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }

    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance - revenuePool;
        require(balance > 0, "TCNFT: No funds to withdraw");
        
        (bool success, ) = owner().call{value: balance}("");
        require(success, "TCNFT: Withdrawal failed");
    }

    // ============= OVERRIDE FUNCTIONS =============

    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal override(ERC721, ERC721Enumerable) whenNotPaused {
        // Prevent transfer of staked NFTs
        if (from != address(0)) {
            require(!tokenMetadata[tokenId].isStaked, "TCNFT: Cannot transfer staked NFT");
        }
        
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function _burn(uint256 tokenId) internal override(ERC721, ERC721URIStorage) {
        super._burn(tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }

    // ============= VIEW FUNCTIONS =============

    function getTokenMetadata(uint256 tokenId) external view returns (
        Tier tier,
        uint256 power,
        uint256 experience,
        uint256 createdAt,
        uint256 lastUpgrade,
        bool isStaked,
        string memory customAttributes
    ) {
        require(_exists(tokenId), "TCNFT: Token does not exist");
        NFTMetadata memory metadata = tokenMetadata[tokenId];
        
        return (
            metadata.tier,
            metadata.power,
            metadata.experience,
            metadata.createdAt,
            metadata.lastUpgrade,
            metadata.isStaked,
            metadata.customAttributes
        );
    }

    function getCollectionStats() external view returns (
        uint256 totalSupply_,
        uint256 totalStaked_,
        uint256 revenuePool_,
        uint256 nextTokenId,
        bool whitelistActive,
        bool publicActive
    ) {
        return (
            totalSupply(),
            totalStaked,
            revenuePool,
            _tokenIdCounter.current(),
            whitelistMintActive,
            publicMintActive
        );
    }

    function getUserStats(address user) external view returns (
        uint256 ownedCount,
        uint256 stakedCount,
        uint256 claimableRevenue,
        uint256 totalMinted
    ) {
        ownedCount = balanceOf(user);
        
        // Count staked NFTs
        for (uint256 i = 0; i < ownedCount; i++) {
            uint256 tokenId = tokenOfOwnerByIndex(user, i);
            if (tokenMetadata[tokenId].isStaked) {
                stakedCount++;
            }
        }
        
        claimableRevenue = revenueShares[user] - claimedRevenue[user];
        totalMinted = mintedCount[user];
        
        return (ownedCount, stakedCount, claimableRevenue, totalMinted);
    }

    // Allow contract to receive ETH
    receive() external payable {
        revenuePool += msg.value;
    }
}
```

## 🎯 แบบฝึกหัด

### **📝 แบบฝึกหัดที่ 1: Token Integration**
สร้าง system ที่ผสานรวม ERC-20 และ ERC-721:
1. ใช้ ERC-20 tokens เป็น currency สำหรับ NFT marketplace
2. Stake ERC-20 เพื่อเพิ่ม voting power ใน governance
3. Burn ERC-20 เพื่อ upgrade NFT

### **🔧 แบบฝึกหัดที่ 2: Advanced Features**
เพิ่มฟีเจอร์:
1. NFT lending/borrowing protocol
2. Fractionalized NFT ownership
3. Cross-chain bridge support

### **🎯 แบบฝึกหัดที่ 3: Marketplace Integration**
สร้าง:
1. Decentralized marketplace contract
2. Auction system with bidding
3. Royalty distribution system

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 6: Testing and Deployment](./06-testing-deployment.md)  
**บทถัดไป**: [บทที่ 8: DeFi Protocols](./08-defi-protocols.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [ERC-20 Token Standard](https://eips.ethereum.org/EIPS/eip-20)
- [ERC-721 NFT Standard](https://eips.ethereum.org/EIPS/eip-721)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [NFT Metadata Standards](https://docs.opensea.io/docs/metadata-standards)

---

ตอนนี้คุณเข้าใจ Token Standards แล้ว! ในบทถัดไป เราจะเรียนรู้เรื่อง DeFi Protocols 🚀
