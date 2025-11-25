# Smart Contract Specification: Simplified Authorization & Usage Control

## Overview
A streamlined smart contract system focused on authorization management and usage limit enforcement for the AI app platform. This specification is designed for 1-2 students to implement in 8 weeks.

## Core Contract: `AuthorizationRegistry.sol`

### Design Principles
1. **Simplicity First**: Avoid over-engineering
2. **Gas Efficiency**: Optimize for common operations
3. **Clear Limits**: Focus on token and request limits (not payments)
4. **Security**: Basic but robust access controls

### Main Contract Structure

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract AuthorizationRegistry is Ownable, Pausable, ReentrancyGuard {
    
    // ============ State Variables ============
    
    uint256 public authorizationCounter;
    uint256 public constant MAX_DAILY_REQUESTS = 10000;
    uint256 public constant MAX_MONTHLY_TOKENS = 10_000_000;
    
    // ============ Core Structures ============
    
    struct Authorization {
        bytes32 authId;
        address userAddress;
        bytes32 appId;
        bool isActive;
        uint256 createdAt;
        uint256 expiresAt;  // 0 means no expiry
        LimitConfig limits;
        UsageTracking usage;
    }
    
    struct LimitConfig {
        uint256 maxTokensPerRequest;   // Max tokens in single request
        uint256 dailyTokenLimit;       // Daily token limit
        uint256 monthlyTokenLimit;     // Monthly token limit
        uint256 dailyRequestLimit;     // Daily request count limit
        string[] allowedModels;        // Allowed AI models
        bytes32 scopeHash;            // Hash of authorized scopes
    }
    
    struct UsageTracking {
        uint256 currentDayTokens;      // Tokens used today
        uint256 currentDayRequests;    // Requests made today
        uint256 currentMonthTokens;    // Tokens used this month
        uint256 lastResetDay;          // Last daily reset timestamp
        uint256 lastResetMonth;        // Last monthly reset timestamp
        uint256 totalTokensUsed;       // Lifetime tokens used
        uint256 totalRequests;         // Lifetime requests
    }
    
    struct AppInfo {
        bytes32 appId;
        address developer;
        bool isVerified;
        bool isBlacklisted;
        uint256 trustScore;    // 0-100 score
        uint256 totalUsers;
        uint256 violations;     // Count of limit violations
    }
    
    // ============ Mappings ============
    
    mapping(bytes32 => Authorization) public authorizations;
    mapping(address => bytes32[]) public userAuthorizations;
    mapping(bytes32 => AppInfo) public registeredApps;
    mapping(bytes32 => bytes32[]) public appUserAuthorizations;
    
    // For quick lookup
    mapping(address => mapping(bytes32 => bytes32)) public activeAuthLookup;
    
    // ============ Events ============
    
    event AuthorizationGranted(
        bytes32 indexed authId,
        address indexed user,
        bytes32 indexed appId,
        uint256 monthlyTokenLimit,
        uint256 dailyRequestLimit
    );
    
    event AuthorizationRevoked(
        bytes32 indexed authId,
        address indexed user,
        bytes32 indexed appId,
        string reason
    );
    
    event LimitsUpdated(
        bytes32 indexed authId,
        uint256 newMonthlyTokenLimit,
        uint256 newDailyRequestLimit
    );
    
    event UsageRecorded(
        bytes32 indexed authId,
        uint256 tokensUsed,
        uint256 requestCount,
        uint256 timestamp
    );
    
    event LimitExceeded(
        bytes32 indexed authId,
        string limitType,  // "daily_tokens", "monthly_tokens", "daily_requests"
        uint256 attempted,
        uint256 limit
    );
    
    event AppBlacklisted(
        bytes32 indexed appId,
        uint256 violations,
        uint256 timestamp
    );
}
```

## Core Functions Implementation

### 1. Authorization Management

```solidity
/**
 * @notice Grant authorization for an app to use AI services on behalf of user
 * @param appId The application identifier
 * @param monthlyTokenLimit Maximum tokens per month
 * @param dailyRequestLimit Maximum requests per day
 * @param allowedModels Array of allowed model names
 * @param scopeHash Hash of authorized scopes
 */
function grantAuthorization(
    bytes32 appId,
    uint256 monthlyTokenLimit,
    uint256 dailyRequestLimit,
    string[] memory allowedModels,
    bytes32 scopeHash
) external whenNotPaused returns (bytes32 authId) {
    
    // Input validation
    require(appId != bytes32(0), "Invalid app ID");
    require(monthlyTokenLimit > 0, "Monthly limit must be positive");
    require(dailyRequestLimit > 0, "Daily limit must be positive");
    require(monthlyTokenLimit <= MAX_MONTHLY_TOKENS, "Exceeds max monthly limit");
    require(dailyRequestLimit <= MAX_DAILY_REQUESTS, "Exceeds max daily limit");
    
    // Check app status
    AppInfo memory app = registeredApps[appId];
    require(app.appId != bytes32(0), "App not registered");
    require(!app.isBlacklisted, "App is blacklisted");
    
    // Check for existing active authorization
    bytes32 existingAuth = activeAuthLookup[msg.sender][appId];
    require(existingAuth == bytes32(0), "Active authorization exists");
    
    // Generate unique authorization ID
    authId = keccak256(
        abi.encodePacked(
            msg.sender,
            appId,
            authorizationCounter++,
            block.timestamp
        )
    );
    
    // Create authorization
    Authorization storage auth = authorizations[authId];
    auth.authId = authId;
    auth.userAddress = msg.sender;
    auth.appId = appId;
    auth.isActive = true;
    auth.createdAt = block.timestamp;
    auth.expiresAt = 0; // No expiry by default
    
    // Set limits
    auth.limits.maxTokensPerRequest = monthlyTokenLimit / 100; // Simple heuristic
    auth.limits.dailyTokenLimit = monthlyTokenLimit / 30;     // Distribute evenly
    auth.limits.monthlyTokenLimit = monthlyTokenLimit;
    auth.limits.dailyRequestLimit = dailyRequestLimit;
    auth.limits.allowedModels = allowedModels;
    auth.limits.scopeHash = scopeHash;
    
    // Initialize usage tracking
    auth.usage.lastResetDay = block.timestamp;
    auth.usage.lastResetMonth = block.timestamp;
    
    // Update mappings
    userAuthorizations[msg.sender].push(authId);
    appUserAuthorizations[appId].push(authId);
    activeAuthLookup[msg.sender][appId] = authId;
    
    // Update app stats
    registeredApps[appId].totalUsers++;
    
    emit AuthorizationGranted(
        authId,
        msg.sender,
        appId,
        monthlyTokenLimit,
        dailyRequestLimit
    );
}

/**
 * @notice Revoke an authorization
 */
function revokeAuthorization(bytes32 authId, string memory reason) external {
    Authorization storage auth = authorizations[authId];
    
    require(auth.authId != bytes32(0), "Authorization not found");
    require(auth.isActive, "Already revoked");
    require(auth.userAddress == msg.sender || owner() == msg.sender, "Unauthorized");
    
    auth.isActive = false;
    delete activeAuthLookup[auth.userAddress][auth.appId];
    
    if (registeredApps[auth.appId].totalUsers > 0) {
        registeredApps[auth.appId].totalUsers--;
    }
    
    emit AuthorizationRevoked(authId, auth.userAddress, auth.appId, reason);
}

/**
 * @notice Update limits for an existing authorization
 */
function updateLimits(
    bytes32 authId,
    uint256 newMonthlyTokenLimit,
    uint256 newDailyRequestLimit
) external {
    Authorization storage auth = authorizations[authId];
    
    require(auth.userAddress == msg.sender, "Not your authorization");
    require(auth.isActive, "Authorization not active");
    require(newMonthlyTokenLimit > 0, "Invalid token limit");
    require(newDailyRequestLimit > 0, "Invalid request limit");
    
    auth.limits.monthlyTokenLimit = newMonthlyTokenLimit;
    auth.limits.dailyTokenLimit = newMonthlyTokenLimit / 30;
    auth.limits.dailyRequestLimit = newDailyRequestLimit;
    auth.limits.maxTokensPerRequest = newMonthlyTokenLimit / 100;
    
    emit LimitsUpdated(authId, newMonthlyTokenLimit, newDailyRequestLimit);
}
```

### 2. Usage Tracking and Limit Enforcement

```solidity
/**
 * @notice Check if a request is within limits
 * @param authId Authorization identifier
 * @param requestTokens Tokens needed for this request
 * @return allowed Whether the request is allowed
 * @return reason If not allowed, the reason
 */
function checkRequestAllowed(
    bytes32 authId,
    uint256 requestTokens
) external view returns (bool allowed, string memory reason) {
    
    Authorization memory auth = authorizations[authId];
    
    if (!auth.isActive) {
        return (false, "Authorization not active");
    }
    
    if (auth.expiresAt > 0 && block.timestamp > auth.expiresAt) {
        return (false, "Authorization expired");
    }
    
    // Check per-request limit
    if (requestTokens > auth.limits.maxTokensPerRequest) {
        return (false, "Exceeds per-request token limit");
    }
    
    // Check daily token limit
    if (_isNewDay(auth.usage.lastResetDay)) {
        // Would reset, so check against fresh limit
        if (requestTokens > auth.limits.dailyTokenLimit) {
            return (false, "Exceeds daily token limit");
        }
    } else {
        if (auth.usage.currentDayTokens + requestTokens > auth.limits.dailyTokenLimit) {
            return (false, "Would exceed daily token limit");
        }
    }
    
    // Check monthly token limit
    if (_isNewMonth(auth.usage.lastResetMonth)) {
        // Would reset, so check against fresh limit
        if (requestTokens > auth.limits.monthlyTokenLimit) {
            return (false, "Exceeds monthly token limit");
        }
    } else {
        if (auth.usage.currentMonthTokens + requestTokens > auth.limits.monthlyTokenLimit) {
            return (false, "Would exceed monthly token limit");
        }
    }
    
    // Check daily request limit
    if (_isNewDay(auth.usage.lastResetDay)) {
        // Would reset to 1
        if (1 > auth.limits.dailyRequestLimit) {
            return (false, "Exceeds daily request limit");
        }
    } else {
        if (auth.usage.currentDayRequests + 1 > auth.limits.dailyRequestLimit) {
            return (false, "Would exceed daily request limit");
        }
    }
    
    return (true, "");
}

/**
 * @notice Record usage after a successful API call
 * @param authId Authorization identifier
 * @param tokensUsed Actual tokens consumed
 */
function recordUsage(
    bytes32 authId,
    uint256 tokensUsed
) external onlyOwner {  // In production, this would be called by the platform
    
    Authorization storage auth = authorizations[authId];
    require(auth.isActive, "Authorization not active");
    
    // Reset daily counters if new day
    if (_isNewDay(auth.usage.lastResetDay)) {
        auth.usage.currentDayTokens = 0;
        auth.usage.currentDayRequests = 0;
        auth.usage.lastResetDay = block.timestamp;
    }
    
    // Reset monthly counters if new month  
    if (_isNewMonth(auth.usage.lastResetMonth)) {
        auth.usage.currentMonthTokens = 0;
        auth.usage.lastResetMonth = block.timestamp;
    }
    
    // Update usage
    auth.usage.currentDayTokens += tokensUsed;
    auth.usage.currentDayRequests += 1;
    auth.usage.currentMonthTokens += tokensUsed;
    auth.usage.totalTokensUsed += tokensUsed;
    auth.usage.totalRequests += 1;
    
    // Check for violations
    if (auth.usage.currentDayTokens > auth.limits.dailyTokenLimit) {
        emit LimitExceeded(authId, "daily_tokens", auth.usage.currentDayTokens, auth.limits.dailyTokenLimit);
        _recordViolation(auth.appId);
    }
    
    if (auth.usage.currentMonthTokens > auth.limits.monthlyTokenLimit) {
        emit LimitExceeded(authId, "monthly_tokens", auth.usage.currentMonthTokens, auth.limits.monthlyTokenLimit);
        _recordViolation(auth.appId);
    }
    
    emit UsageRecorded(authId, tokensUsed, 1, block.timestamp);
}

// Helper functions
function _isNewDay(uint256 lastReset) private view returns (bool) {
    return (block.timestamp - lastReset) >= 1 days;
}

function _isNewMonth(uint256 lastReset) private view returns (bool) {
    return (block.timestamp - lastReset) >= 30 days;
}

function _recordViolation(bytes32 appId) private {
    registeredApps[appId].violations++;
    
    // Auto-blacklist after 10 violations
    if (registeredApps[appId].violations >= 10) {
        registeredApps[appId].isBlacklisted = true;
        emit AppBlacklisted(appId, registeredApps[appId].violations, block.timestamp);
    }
}
```

### 3. App Management

```solidity
/**
 * @notice Register a new app (simplified version)
 */
function registerApp(
    bytes32 appId,
    address developer
) external onlyOwner {
    require(appId != bytes32(0), "Invalid app ID");
    require(developer != address(0), "Invalid developer");
    require(registeredApps[appId].appId == bytes32(0), "App already registered");
    
    registeredApps[appId] = AppInfo({
        appId: appId,
        developer: developer,
        isVerified: false,
        isBlacklisted: false,
        trustScore: 50,  // Start with neutral score
        totalUsers: 0,
        violations: 0
    });
}

/**
 * @notice Verify an app after review
 */
function verifyApp(bytes32 appId) external onlyOwner {
    require(registeredApps[appId].appId != bytes32(0), "App not found");
    registeredApps[appId].isVerified = true;
    registeredApps[appId].trustScore = 75;  // Boost trust score
}

/**
 * @notice Blacklist a malicious app
 */
function blacklistApp(bytes32 appId) external onlyOwner {
    require(registeredApps[appId].appId != bytes32(0), "App not found");
    registeredApps[appId].isBlacklisted = true;
    
    emit AppBlacklisted(appId, registeredApps[appId].violations, block.timestamp);
}
```

### 4. Query Functions

```solidity
/**
 * @notice Get user's active authorizations
 */
function getUserActiveAuthorizations(address user) 
    external 
    view 
    returns (bytes32[] memory) 
{
    bytes32[] memory allAuths = userAuthorizations[user];
    uint256 activeCount = 0;
    
    // Count active
    for (uint256 i = 0; i < allAuths.length; i++) {
        if (authorizations[allAuths[i]].isActive) {
            activeCount++;
        }
    }
    
    // Collect active
    bytes32[] memory activeAuths = new bytes32[](activeCount);
    uint256 index = 0;
    for (uint256 i = 0; i < allAuths.length; i++) {
        if (authorizations[allAuths[i]].isActive) {
            activeAuths[index++] = allAuths[i];
        }
    }
    
    return activeAuths;
}

/**
 * @notice Get current usage for an authorization
 */
function getCurrentUsage(bytes32 authId) 
    external 
    view 
    returns (
        uint256 dailyTokensUsed,
        uint256 dailyRequestsUsed,
        uint256 monthlyTokensUsed,
        uint256 totalTokensUsed
    ) 
{
    UsageTracking memory usage = authorizations[authId].usage;
    
    // Check if counters should be reset
    if (_isNewDay(usage.lastResetDay)) {
        dailyTokensUsed = 0;
        dailyRequestsUsed = 0;
    } else {
        dailyTokensUsed = usage.currentDayTokens;
        dailyRequestsUsed = usage.currentDayRequests;
    }
    
    if (_isNewMonth(usage.lastResetMonth)) {
        monthlyTokensUsed = 0;
    } else {
        monthlyTokensUsed = usage.currentMonthTokens;
    }
    
    totalTokensUsed = usage.totalTokensUsed;
}

/**
 * @notice Check if user has authorized an app
 */
function hasActiveAuthorization(address user, bytes32 appId) 
    external 
    view 
    returns (bool) 
{
    bytes32 authId = activeAuthLookup[user][appId];
    return authId != bytes32(0) && authorizations[authId].isActive;
}
```

## Gas Optimization Techniques

### Storage Packing
```solidity
struct OptimizedAuthorization {
    address userAddress;        // 20 bytes
    uint32 createdAt;           // 4 bytes (timestamp)
    uint32 expiresAt;           // 4 bytes (timestamp)
    bool isActive;              // 1 byte
    // Packs into 1 storage slot (32 bytes)
    
    uint128 monthlyTokenLimit;  // 16 bytes
    uint128 dailyTokenLimit;    // 16 bytes
    // Packs into 1 storage slot
    
    uint64 dailyRequestLimit;   // 8 bytes
    uint64 currentDayRequests;  // 8 bytes
    uint64 totalRequests;       // 8 bytes
    uint32 lastResetDay;        // 4 bytes
    // Packs into 1 storage slot
}
```

### Batch Operations
```solidity
function batchRecordUsage(
    bytes32[] calldata authIds,
    uint256[] calldata tokensUsed
) external onlyOwner {
    require(authIds.length == tokensUsed.length, "Length mismatch");
    
    for (uint256 i = 0; i < authIds.length; i++) {
        recordUsage(authIds[i], tokensUsed[i]);
    }
}
```

## Testing Strategy

### Core Test Cases
```javascript
describe("AuthorizationRegistry", function() {
    describe("Authorization Management", function() {
        it("Should grant authorization with correct limits");
        it("Should prevent duplicate active authorizations");
        it("Should revoke authorization");
        it("Should update limits");
        it("Should expire authorizations");
    });
    
    describe("Usage Tracking", function() {
        it("Should track daily usage correctly");
        it("Should track monthly usage correctly");
        it("Should reset daily counters");
        it("Should reset monthly counters");
        it("Should emit LimitExceeded events");
    });
    
    describe("Limit Enforcement", function() {
        it("Should reject requests over per-request limit");
        it("Should reject requests over daily token limit");
        it("Should reject requests over monthly token limit");
        it("Should reject requests over daily request limit");
    });
    
    describe("App Management", function() {
        it("Should register new apps");
        it("Should verify apps");
        it("Should blacklist after violations");
        it("Should prevent blacklisted apps from getting authorizations");
    });
});
```

## Deployment Script

```javascript
const hre = require("hardhat");

async function main() {
    // Deploy Authorization Registry
    const AuthRegistry = await hre.ethers.getContractFactory("AuthorizationRegistry");
    const authRegistry = await AuthRegistry.deploy();
    await authRegistry.deployed();
    
    console.log("AuthorizationRegistry deployed to:", authRegistry.address);
    
    // Register some test apps
    const testApps = [
        { id: ethers.utils.id("TestApp1"), developer: "0x..." },
        { id: ethers.utils.id("TestApp2"), developer: "0x..." }
    ];
    
    for (const app of testApps) {
        await authRegistry.registerApp(app.id, app.developer);
        console.log(`Registered app: ${app.id}`);
    }
    
    // Verify on Etherscan
    if (network.name !== "hardhat") {
        await hre.run("verify:verify", {
            address: authRegistry.address,
            constructorArguments: [],
        });
    }
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

## Integration Example

```javascript
// Platform backend integration
class BlockchainAuthService {
    constructor(contractAddress, signer) {
        this.contract = new ethers.Contract(
            contractAddress,
            AUTH_REGISTRY_ABI,
            signer
        );
    }
    
    async authorizeApp(appId, limits) {
        const tx = await this.contract.grantAuthorization(
            appId,
            limits.monthlyTokens,
            limits.dailyRequests,
            limits.allowedModels || [],
            ethers.utils.id(JSON.stringify(limits.scopes))
        );
        
        const receipt = await tx.wait();
        const event = receipt.events.find(e => e.event === 'AuthorizationGranted');
        return event.args.authId;
    }
    
    async checkLimit(authId, tokensNeeded) {
        const [allowed, reason] = await this.contract.checkRequestAllowed(
            authId,
            tokensNeeded
        );
        return { allowed, reason };
    }
    
    async recordUsage(authId, tokensUsed) {
        // This would be called by the platform after successful API call
        const tx = await this.contract.recordUsage(authId, tokensUsed);
        await tx.wait();
    }
}
```

## Security Considerations

1. **Access Control**: Only platform can record usage, only users can grant/revoke their authorizations
2. **Reentrancy**: Use ReentrancyGuard for all state-changing functions
3. **Integer Overflow**: Solidity 0.8+ has built-in overflow protection
4. **Pausable**: Emergency pause mechanism for critical issues
5. **Input Validation**: Check all inputs for validity

## Future Enhancements (Post-MVP)

1. **Merkle Tree Scopes**: More efficient scope management
2. **Zero-Knowledge Proofs**: Private usage verification
3. **Cross-Chain Support**: Deploy on multiple chains
4. **Delegation**: Allow users to delegate authorization management
5. **Time-based Limits**: Hourly limits, custom periods

---

This simplified specification focuses on the core authorization and usage limit features, making it achievable for 1-2 students in 8 weeks while still providing valuable blockchain learning experiences.