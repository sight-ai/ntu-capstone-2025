# NTU Blockchain Capstone: TrustChain for AI App Platform
## Executive Summary for Academic Review

### Project Overview
**Title**: TrustChain - Decentralized Authorization and Usage Control System for AI Application Platform

**Duration**: 8 weeks (Half semester)

**Team Size**: 1-2 students

**Prerequisites**: 
- Solid understanding of blockchain fundamentals
- Experience with Solidity and Web3 development
- Basic knowledge of cryptography

### The Ecosystem: AI App Marketplace

This project involves building blockchain infrastructure for an AI application marketplace - think "App Store for AI" or "Stripe for AI services." The ecosystem connects:
- **Users** who have AI credits/budgets
- **Applications** that need AI capabilities  
- **AI Providers** (OpenAI, Anthropic, Google, etc.)

Students will build the trust layer that makes this ecosystem possible, addressing the fundamental challenge: **How do we enable users to safely authorize apps to spend their AI credits without fear of abuse?**

### Problem Statement

The SightAI app platform faces a fundamental trust challenge: users are reluctant to authorize third-party applications to use their AI credits due to concerns about:
- Lack of transparency in usage tracking
- Potential for unauthorized charges
- No immutable audit trail
- Centralized control over disputes

While the main platform efficiently handles operations through traditional infrastructure, there's a clear need for a **complementary blockchain layer** that provides transparency, immutability, and trustless verification.

### Key Stakeholders & Their Journeys

**Users (End Consumers)**
- Want predictable costs, transparency, and control
- Fear unexpected charges (the "AWS horror story")
- Journey: Discover app → Hesitate about costs → Set conservative limits → Build trust → Increase usage

**Application Developers**
- Need simple integration with normal API key experience
- Want to focus on features, not billing infrastructure
- Journey: Register app → Sandbox testing → Get verified → Launch → Scale

**Applications (The Products)**
- Need user trust to grow
- Compete on features, not infrastructure
- Lifecycle: Development → Launch → Growth → Maturity

**AI Gateway (Intelligence Layer)**
- Routes to optimal AI provider (OpenAI, Anthropic, Google, etc.)
- Tracks usage in real-time

**The Platform (Trust Broker)**
- Manages authorizations and billing
- Verifies applications
- Handles disputes
- Needs blockchain for immutable trust layer

### Proposed Solution

Students will build a blockchain-based trust layer that works alongside the traditional platform, focusing on areas where blockchain adds genuine value:

```
Traditional Platform (Fast)     ←→     Blockchain Layer (Trustless)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━          ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• OAuth authorization flows            • Immutable authorization records
• High-frequency operations            • Usage limit enforcement
• Centralized databases                • Scope control verification
• Payment processing                   • Public audit trail
```

## Core Project Components

### Component 1: Smart Contract Authorization Registry (40% of work)

The heart of the project - a comprehensive authorization management system on blockchain.

#### Key Features to Implement:

**Authorization Management**
```solidity
contract AuthorizationRegistry {
    struct Authorization {
        address userWallet;
        bytes32 appId;
        uint256 monthlyTokenLimit;    // Token usage limit
        uint256 dailyRequestLimit;     // API request limit
        bytes32 scopeHash;            // Authorized permissions
        uint256 timestamp;
        uint256 expiresAt;
        bool isActive;
    }
    
    // Core functions students will implement
    function grantAuthorization(...) external;
    function revokeAuthorization(...) external;
    function updateLimits(...) external;
    function checkAuthorization(...) external view returns (bool);
    function enforceTokenLimit(...) external returns (bool);
    function enforceDailyLimit(...) external returns (bool);
}
```

**Scope Management System**
- Implement merkle tree for efficient scope storage
- Scope verification without revealing all permissions
- Dynamic scope updates with user consent

**Usage Limit Controls**
- Token consumption limits (per request, daily, monthly)
- Request count limits
- Model-specific restrictions (e.g., GPT-4 vs GPT-3.5)
- Cost caps in USD equivalent

#### Deliverables:
1. Smart contract with comprehensive authorization logic
2. Multi-level limit enforcement (token, request, cost)
3. Scope management via merkle trees
4. Emergency pause and revocation mechanisms
5. Gas-optimized storage patterns

### Component 2: App Verification System (30% of work)

A simplified verification system focused on app reputation and basic governance.

#### Key Features to Implement:

**App Registry**
```solidity
contract AppRegistry {
    struct App {
        bytes32 appId;
        address developer;
        uint256 trustScore;
        bool isVerified;
        bool isBlacklisted;
        uint256 totalAuthorizations;
        uint256 totalUsage;
    }
    
    // Verification levels
    enum VerificationLevel {
        Unverified,     // New apps
        BasicVerified,  // Passed basic checks
        TrustedApp      // Community trusted
    }
}
```

**Malicious App Detection**
- Implement slashing for apps that exceed user limits
- Blacklist mechanism for repeated violations
- Appeal process for disputed blacklisting

**Simple Governance**
- Report mechanism for users
- Threshold-based automatic flagging
- Manual review trigger system

#### Deliverables:
1. App registration and verification contract
2. Trust scoring algorithm
3. Blacklist and appeal system
4. Basic governance for disputes

### Component 3: Usage Tracking and Audit (30% of work)

Focus on transparent usage recording and verification.

#### Key Features to Implement:

**Usage Commitment System**
```solidity
contract UsageAudit {
    struct UsageRecord {
        bytes32 authId;
        uint256 period;  // Daily or monthly
        uint256 tokensUsed;
        uint256 requestCount;
        uint256 costUSD;
        bytes32 merkleRoot;  // Proof of detailed usage
    }
    
    // Commit usage data on-chain
    function commitUsage(...) external;
    
    // Verify usage against limits
    function verifyWithinLimits(...) external view returns (bool);
    
    // Challenge suspicious usage
    function challengeUsage(...) external;
}
```

**Limit Enforcement**
- Check token consumption against authorized limits
- Verify request counts don't exceed daily caps
- Alert when approaching limits (emit events)

**Basic Audit Trail**
- Immutable usage records
- Period-based aggregation (daily/monthly)
- Simple dispute mechanism

#### Deliverables:
1. Usage tracking smart contract
2. Limit verification logic
3. Basic merkle proof system for detailed records
4. Event emission for limit warnings

### Advanced Features (Bonus - if time permits)

These are optional enhancements for exceptional students:

1. **Zero-Knowledge Proofs** (+10%)
   - Private usage verification without revealing details
   - Implement using Circom/SnarkJS

2. **Cross-Chain Support** (+10%)
   - Deploy on multiple chains (Ethereum, Polygon)
   - Implement basic bridge for authorization sync

3. **Batch Operations** (+10%)
   - Gas optimization through batching
   - Merkle tree batch updates

## Adjusted Timeline (8 Weeks)

### Week 1-2: Foundation
**Goals:**
- Environment setup (Hardhat, testing framework)
- Basic Authorization Registry contract
- Core data structures
- Initial unit tests

**Deliverables:**
- [ ] Development environment ready
- [ ] Basic contract skeleton
- [ ] Authorization struct and mappings
- [ ] Grant/revoke functions

### Week 3-4: Authorization System
**Goals:**
- Complete authorization management
- Implement limit enforcement
- Scope management with merkle trees
- Comprehensive testing

**Deliverables:**
- [ ] Full authorization lifecycle
- [ ] Token and request limit checks
- [ ] Scope verification system
- [ ] 80% test coverage

### Week 5-6: App Verification & Usage Tracking
**Goals:**
- App registry and verification
- Usage commitment system
- Basic governance mechanisms
- Integration between contracts

**Deliverables:**
- [ ] App verification contract
- [ ] Usage audit contract
- [ ] Blacklist mechanism
- [ ] Contract interactions working

### Week 7: Integration & Optimization
**Goals:**
- Platform API integration
- Gas optimization
- Security review
- Documentation

**Deliverables:**
- [ ] Mock platform integration
- [ ] Gas usage under targets
- [ ] Security checklist completed
- [ ] Technical documentation

### Week 8: Testing & Presentation
**Goals:**
- End-to-end testing
- Demo application
- Final presentation preparation
- Code cleanup

**Deliverables:**
- [ ] Live demo ready
- [ ] Presentation slides
- [ ] Final code submission
- [ ] Project report

## Technical Requirements

### Smart Contract Requirements
- Solidity ^0.8.19
- OpenZeppelin contracts for security
- Upgradeable proxy pattern
- Gas optimization techniques

### Development Stack
- **Framework**: Hardhat or Foundry
- **Testing**: Mocha/Chai or Forge tests
- **Frontend**: Basic React app for demo
- **Network**: Sepolia/Mumbai testnet

### Integration Points
Students will create mock integrations to demonstrate:
```javascript
// Platform integration example
class PlatformIntegration {
    async recordAuthorization(user, app, limits) {
        // Call smart contract
        const tx = await contract.grantAuthorization(
            user,
            app,
            limits.tokenLimit,
            limits.requestLimit,
            scopeHash
        );
        return tx.hash;
    }
    
    async checkLimits(authId, currentUsage) {
        // Verify against blockchain
        return await contract.verifyWithinLimits(
            authId,
            currentUsage.tokens,
            currentUsage.requests
        );
    }
}
```

## Evaluation Criteria (Adjusted for 8 weeks)

### Core Implementation (50%)
- Authorization Registry completeness
- Limit enforcement accuracy  
- Scope management functionality
- Code quality and structure

### Security & Testing (25%)
- No critical vulnerabilities
- Comprehensive test coverage (>80%)
- Proper access controls
- Input validation

### Innovation (15%)
- Creative solutions to problems
- Gas optimization techniques
- Additional features beyond requirements

### Documentation & Presentation (10%)
- Clear code documentation
- Architecture diagrams
- Demo quality
- Presentation clarity

## Success Metrics

### Minimum Viable Product
- [ ] Authorization lifecycle working (grant, update, revoke)
- [ ] Token and request limits enforced
- [ ] Basic app verification
- [ ] Usage tracking functional
- [ ] Gas cost < 200k for authorization

### Good Implementation
- [ ] All MVP features plus:
- [ ] Merkle tree scope management
- [ ] Batch operations
- [ ] Comprehensive testing
- [ ] Gas cost < 150k for authorization

### Excellent Implementation
- [ ] All above plus:
- [ ] Advanced features (ZK proofs or cross-chain)
- [ ] Production-ready security
- [ ] Innovative solutions
- [ ] Gas cost < 100k for authorization

## Resources Provided

### From SightAI Team
- API documentation for integration points
- Test data and scenarios
- Weekly office hours
- Code review sessions

### Development Resources
- Testnet tokens (ETH, MATIC)
- Example contracts for reference
- Integration test suite
- Security checklist

### Learning Materials
- Solidity best practices guide
- Gas optimization techniques
- Merkle tree implementation examples
- ZK proof tutorials (for advanced features)

## Key Learning Outcomes

By completing this project, students will gain practical experience in:

### Blockchain Fundamentals
- Smart contract design and development
- Gas optimization strategies
- Security best practices
- Testing methodologies

### Advanced Concepts
- Merkle trees for efficient data storage
- Authorization and access control patterns
- Limit enforcement mechanisms
- Basic governance systems

### System Design
- Hybrid Web2/Web3 architecture
- Trust minimization principles
- Scalability considerations
- Integration patterns

### Real-World Skills
- Problem decomposition
- Trade-off analysis
- Documentation
- Presentation skills

## Project Uniqueness

This project stands out because it:

1. **Solves Real Problems**: Addresses actual trust issues in AI service consumption
2. **Teaches Restraint**: Shows when NOT to use blockchain (payments stay off-chain)
3. **Focuses on Core Value**: Authorization and limits are perfect blockchain use cases
4. **Manageable Scope**: Achievable in 8 weeks by 1-2 students
5. **Industry Relevant**: AI + Blockchain intersection is highly valuable

## Final Notes

### For Students
- Start with the Authorization Registry - it's the foundation
- Test extensively - security is paramount
- Focus on core features before attempting bonuses
- Ask for help during office hours

### For Instructors
- Week 4 checkpoint: Authorization system should be complete
- Week 6 checkpoint: All contracts deployed to testnet
- Encourage iterative development over perfection
- Use provided test scenarios for evaluation

### Success Tips
1. **Week 1**: Don't skip environment setup - it pays off later
2. **Week 2-4**: Focus entirely on Authorization Registry
3. **Week 5-6**: App verification can be simple but functional
4. **Week 7**: Integration is more important than optimization
5. **Week 8**: Practice the demo multiple times

---

*"The best blockchain solution is often the simplest one that solves the actual problem."*