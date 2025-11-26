# NTU Blockchain Capstone: Week-by-Week Checklist

## Project: TrustChain Authorization System
**Duration**: 8 Weeks  
**Team Size**: 1-2 Students

---

## Week 1: Setup & Foundation
### Goals
Get development environment ready and understand the project scope

### Checklist
- [ ] **Environment Setup**
  - [ ] Install Node.js (v16+) and npm
  - [ ] Install VS Code with Solidity extension
  - [ ] Install Hardhat: `npm install --save-dev hardhat`
  - [ ] Create project: `npx hardhat init`
  - [ ] Install OpenZeppelin: `npm install @openzeppelin/contracts`

- [ ] **Project Understanding**
  - [ ] Read complete project specification
  - [ ] Understand the AI app platform ecosystem
  - [ ] Review smart contract specification
  - [ ] Set up GitHub repository

- [ ] **Initial Contract Setup**
  - [ ] Create `AuthorizationRegistry.sol`
  - [ ] Define basic contract structure
  - [ ] Import OpenZeppelin contracts
  - [ ] Write first test: `test/AuthorizationRegistry.test.js`

### Deliverables
- Working Hardhat project
- GitHub repo with initial commit
- Basic contract skeleton

---

## Week 2: Core Data Structures
### Goals
Implement the main data structures and basic functions

### Checklist
- [ ] **Data Structures**
  - [ ] Implement `Authorization` struct
  - [ ] Implement `LimitConfig` struct
  - [ ] Implement `UsageTracking` struct
  - [ ] Implement `AppInfo` struct
  - [ ] Set up necessary mappings

- [ ] **Basic Functions**
  - [ ] Constructor and initialization
  - [ ] `registerApp()` function
  - [ ] Basic getters
  - [ ] Owner functions

- [ ] **Testing**
  - [ ] Test struct creation
  - [ ] Test app registration
  - [ ] Test access controls
  - [ ] Achieve 50% test coverage

### Deliverables
- All data structures implemented
- Basic functions working
- Test suite running

---

## Week 3: Authorization Management
### Goals
Complete the authorization lifecycle (grant, revoke, update)

### Checklist
- [ ] **Grant Authorization**
  - [ ] Implement `grantAuthorization()`
  - [ ] Input validation
  - [ ] Unique auth ID generation
  - [ ] Event emission
  - [ ] Update all mappings correctly

- [ ] **Revoke Authorization**
  - [ ] Implement `revokeAuthorization()`
  - [ ] Permission checks (user or owner)
  - [ ] Clean up mappings
  - [ ] Event emission

- [ ] **Update Limits**
  - [ ] Implement `updateLimits()`
  - [ ] Validation logic
  - [ ] Event emission

- [ ] **Testing**
  - [ ] Test authorization grant
  - [ ] Test duplicate prevention
  - [ ] Test revocation
  - [ ] Test limit updates
  - [ ] Test edge cases

### Deliverables
- Complete authorization management
- Comprehensive tests for auth lifecycle
- 70% test coverage

---

## Week 4: Usage Tracking & Limits
### Goals
Implement usage tracking and limit enforcement

### Checklist
- [ ] **Limit Checking**
  - [ ] Implement `checkRequestAllowed()`
  - [ ] Per-request limit check
  - [ ] Daily token limit check
  - [ ] Monthly token limit check
  - [ ] Daily request count check
  - [ ] Expiration check

- [ ] **Usage Recording**
  - [ ] Implement `recordUsage()`
  - [ ] Daily counter reset logic
  - [ ] Monthly counter reset logic
  - [ ] Total usage tracking
  - [ ] Violation detection

- [ ] **Helper Functions**
  - [ ] `_isNewDay()` helper
  - [ ] `_isNewMonth()` helper
  - [ ] `_recordViolation()` helper

- [ ] **Testing**
  - [ ] Test limit enforcement
  - [ ] Test usage recording
  - [ ] Test counter resets
  - [ ] Test violation detection
  - [ ] Test time-based logic

### Deliverables
- Working usage tracking system
- Limit enforcement complete
- 80% test coverage

---

## Week 5: App Verification & Governance
### Goals
Implement app verification and basic governance

### Checklist
- [ ] **App Verification**
  - [ ] `verifyApp()` function
  - [ ] Trust score management
  - [ ] Verification status tracking

- [ ] **Blacklisting System**
  - [ ] `blacklistApp()` function
  - [ ] Auto-blacklist after violations
  - [ ] Prevent blacklisted apps from auth
  - [ ] Appeal mechanism (basic)

- [ ] **Query Functions**
  - [ ] `getUserActiveAuthorizations()`
  - [ ] `getCurrentUsage()`
  - [ ] `hasActiveAuthorization()`
  - [ ] `getAppInfo()`

- [ ] **Events & Logging**
  - [ ] Ensure all events are emitted
  - [ ] Add indexed parameters
  - [ ] Test event emission

### Deliverables
- App verification system working
- All query functions implemented
- Event system complete

---

## Week 6: Advanced Features & Optimization
### Goals
Add advanced features and optimize for gas

### Checklist
- [ ] **Scope Management** (Optional but recommended)
  - [ ] Implement scope hash verification
  - [ ] Basic merkle tree for scopes
  - [ ] Scope update mechanism

- [ ] **Gas Optimization**
  - [ ] Pack structs efficiently
  - [ ] Optimize storage usage
  - [ ] Implement batch operations
  - [ ] Measure gas costs

- [ ] **Security Enhancements**
  - [ ] Add ReentrancyGuard
  - [ ] Add Pausable functionality
  - [ ] Emergency stop mechanism
  - [ ] Additional input validation

- [ ] **Bonus Features** (If time permits)
  - [ ] Basic ZK proof concept
  - [ ] Cross-chain design (not implementation)
  - [ ] Delegation mechanism

### Deliverables
- Gas-optimized contract
- Security features implemented
- At least one bonus feature attempted

---

## Week 7: Integration & Frontend
### Goals
Create integration demo and basic frontend

### Checklist
- [ ] **Contract Deployment**
  - [ ] Deploy to testnet (Sepolia)
  - [ ] Verify contract on Etherscan
  - [ ] Document contract addresses
  - [ ] Create deployment script

- [ ] **Integration Layer**
  - [ ] Create JavaScript SDK
  - [ ] Mock platform integration
  - [ ] API simulation
  - [ ] Usage examples

- [ ] **Basic Frontend** (Simple React app)
  - [ ] Connect wallet (MetaMask)
  - [ ] Display user authorizations
  - [ ] Grant new authorization
  - [ ] Revoke authorization
  - [ ] Show usage stats

- [ ] **Documentation**
  - [ ] API documentation
  - [ ] Integration guide
  - [ ] README updates
  - [ ] Architecture diagrams

### Deliverables
- Deployed contracts on testnet
- Working frontend demo
- Complete documentation

---

## Week 8: Testing, Polish & Presentation
### Goals
Final testing, polish, and prepare presentation

### Checklist
- [ ] **Comprehensive Testing**
  - [ ] 90%+ test coverage
  - [ ] Edge case testing
  - [ ] Gas consumption tests
  - [ ] Load testing (simulate 100 users)
  - [ ] Security audit checklist

- [ ] **Code Polish**
  - [ ] Code cleanup and refactoring
  - [ ] Comment all functions
  - [ ] Remove debug code
  - [ ] Final optimization pass

- [ ] **Demo Preparation**
  - [ ] Create demo script
  - [ ] Test demo flow multiple times
  - [ ] Prepare backup plan
  - [ ] Create demo video (backup)

- [ ] **Presentation Materials**
  - [ ] Slides (15-20 slides)
    - [ ] Problem statement
    - [ ] Solution architecture
    - [ ] Technical implementation
    - [ ] Demo
    - [ ] Challenges & learnings
  - [ ] Technical report (10-15 pages)
  - [ ] Executive summary (1 page)

- [ ] **Final Submission**
  - [ ] GitHub repository organized
  - [ ] All code committed
  - [ ] Documentation complete
  - [ ] Deployment instructions
  - [ ] Video demo uploaded

### Deliverables
- Final presentation ready
- All code and documentation submitted
- Live demo prepared

---

## Daily Progress Tracking

### Week 1-2: Foundation Phase
```
Mon [  ] Environment setup
Tue [  ] Contract skeleton
Wed [  ] Data structures
Thu [  ] Basic functions
Fri [  ] Initial tests
```

### Week 3-4: Core Development
```
Mon [  ] Authorization grant
Tue [  ] Authorization revoke/update
Wed [  ] Usage tracking
Thu [  ] Limit enforcement  
Fri [  ] Integration tests
```

### Week 5-6: Features & Polish
```
Mon [  ] App verification
Tue [  ] Governance features
Wed [  ] Gas optimization
Thu [  ] Security features
Fri [  ] Advanced features
```

### Week 7-8: Integration & Demo
```
Mon [  ] Deployment
Tue [  ] Frontend development
Wed [  ] Integration testing
Thu [  ] Presentation prep
Fri [  ] Final demo
```

---

## Risk Management

### Common Pitfalls to Avoid
1. **Week 1**: Don't skip proper setup - it saves time later
2. **Week 2**: Don't over-engineer data structures
3. **Week 3**: Test authorization thoroughly before moving on
4. **Week 4**: Time-based logic is tricky - test extensively
5. **Week 5**: Keep governance simple
6. **Week 6**: Don't optimize prematurely
7. **Week 7**: Start deployment early - it takes time
8. **Week 8**: Practice demo multiple times

### Contingency Plans
- **Behind schedule**: Focus on core features, skip bonuses
- **Technical blockers**: Use office hours, ask for help early
- **Team issues** (if 2 students): Clearly divide responsibilities
- **Demo fails**: Have video backup ready

---

## Success Metrics

### Minimum Viable Product (Week 6)
- [  ] Authorization lifecycle works
- [  ] Usage tracking functional
- [  ] Basic app verification
- [  ] 80% test coverage
- [  ] Deployed to testnet

### Good Implementation (Week 7)
- [  ] All MVP features
- [  ] Gas optimized
- [  ] Frontend demo
- [  ] 90% test coverage
- [  ] Documentation complete

### Excellent Implementation (Week 8)
- [  ] All above features
- [  ] Bonus features implemented
- [  ] Professional presentation
- [  ] Production-ready code
- [  ] Innovative solutions

---

## Resources & Help

### Documentation
- [Hardhat Documentation](https://hardhat.org/docs)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- [Solidity Documentation](https://docs.soliditylang.org)
- [Ethers.js](https://docs.ethers.org)

### Getting Help
- **Office Hours**: Every Tuesday 3-5 PM
- **Discord/Slack**: #blockchain-capstone channel
- **Email**: instructor@ntu.edu.sg
- **Stack Overflow**: Tag with 'solidity'

### Tools
- [Remix IDE](https://remix.ethereum.org) - Quick testing
- [Tenderly](https://tenderly.co) - Debugging
- [Etherscan](https://etherscan.io) - Contract verification
- [Alchemy](https://www.alchemy.com) - Node provider

---

## Final Tips

1. **Start Simple**: Get basic functionality working first
2. **Test Often**: Write tests as you code, not after
3. **Commit Frequently**: Use Git properly, commit daily
4. **Ask Questions**: Use office hours, don't struggle alone
5. **Document as You Go**: Don't leave it for the end
6. **Practice Demo**: The demo is crucial - practice it
7. **Manage Time**: 8 weeks go fast, stay on schedule
8. **Have Fun**: This is real blockchain development!

---

**Remember**: The goal is to learn blockchain development through solving a real problem. Focus on understanding, not just completing tasks.

Good luck! 🚀