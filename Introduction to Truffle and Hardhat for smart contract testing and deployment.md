## Building a Voting Contract: Development, Testing, and Deployment  

This tutorial provides a complete workflow for creating, testing, and deploying a voting smart contract on Binance Smart Chain (BSC) Testnet using Truffle and Hardhat. All interactions are executed via command line.  

### 1. Project Setup  
**Install dependencies**:  
```bash  
npm install -g truffle hardhat  
mkdir voting-dapp && cd voting-dapp  
npm init -y  
npm install @truffle/hdwallet-provider @nomiclabs/hardhat-ethers ethers  
```

### 2. Contract Development  
**File**: `contracts/Voting.sol`  
```solidity  
// SPDX-License-Identifier: MIT  
pragma solidity ^0.8.0;  

contract Voting {  
    struct Proposal {  
        bytes32 name;  
        uint voteCount;  
    }  

    address public chairperson;  
    mapping(address => bool) public hasVoted;  
    Proposal[] public proposals;  

    // Events  
    event Voted(address indexed voter, uint proposalIndex);  

    constructor(bytes32[] memory proposalNames) {  
        chairperson = msg.sender;  
        for (uint i = 0; i  winningVotes) {  
                winningVotes = proposals[i].voteCount;  
                winningIndex = i;  
            }  
        }  
    }  
}  
```

### 3. Testing with Truffle  
**File**: `test/Voting.test.js`  
```javascript  
const Voting = artifacts.require("Voting");  

contract("Voting", (accounts) => {  
    let voting;  
    const chairperson = accounts[0];  
    const voter = accounts[1];  
    const proposals = [web3.utils.asciiToHex("ProposalA"), web3.utils.asciiToHex("ProposalB")];  

    beforeEach(async () => {  
        voting = await Voting.new(proposals, { from: chairperson });  
    });  

    it("initializes proposals", async () => {  
        const prop0 = await voting.proposals(0);  
        assert.equal(web3.utils.hexToUtf8(prop0.name), "ProposalA");  
    });  

    it("allows voting", async () => {  
        await voting.vote(0, { from: voter });  
        const hasVoted = await voting.hasVoted(voter);  
        assert.isTrue(hasVoted);  
    });  

    it("calculates winner", async () => {  
        await voting.vote(0, { from: voter });  
        const winner = await voting.winningProposal();  
        assert.equal(winner, 0);  
    });  
});  
```
**Run tests**:  
```bash  
truffle test  
```

### 4. Deployment with Truffle  
**File**: `truffle-config.js`  
```javascript  
const HDWalletProvider = require('@truffle/hdwallet-provider');  
require('dotenv').config();  

module.exports = {  
    networks: {  
        bsc_testnet: {  
            provider: () => new HDWalletProvider(  
                process.env.MNEMONIC,  
                "https://data-seed-prebsc-1-s1.binance.org:8545"  
            ),  
            network_id: 97,  
            skipDryRun: true  
        }  
    }  
};  
```
**Deploy**:  
```bash  
truffle migrate --network bsc_testnet  
```

### 5. Deployment with Hardhat  
**File**: `hardhat.config.js`  
```javascript  
require("@nomiclabs/hardhat-waffle");  
require('dotenv').config();  

module.exports = {  
    solidity: "0.8.0",  
    networks: {  
        bsc_testnet: {  
            url: "https://data-seed-prebsc-1-s1.binance.org:8545",  
            accounts: [process.env.PRIVATE_KEY],  
            chainId: 97  
        }  
    }  
};  
```
**Deploy script**: `scripts/deploy.js`  
```javascript  
async function main() {  
    const proposals = ["ProposalA", "ProposalB"].map(p => ethers.utils.formatBytes32String(p));  
    const Voting = await ethers.getContractFactory("Voting");  
    const voting = await Voting.deploy(proposals);  
    console.log("Contract deployed to:", voting.address);  
}  
```
**Run deployment**:  
```bash  
npx hardhat run scripts/deploy.js --network bsc_testnet  
```

### 6. Command-Line Interaction  
**Truffle console**:  
```bash  
truffle console --network bsc_testnet  
```
```javascript  
// Get contract instance  
const voting = await Voting.deployed();  

// Vote  
await voting.vote(0, { from: accounts[1] });  

// Check results  
const winner = await voting.winningProposal();  
console.log("Winning proposal index:", winner.toString());  
```

**Hardhat interaction**:  
```javascript  
const hre = require("hardhat");  

async function interact() {  
    const contractAddress = "DEPLOYED_ADDRESS";  
    const Voting = await hre.ethers.getContractFactory("Voting");  
    const voting = Voting.attach(contractAddress);  

    // Vote  
    await voting.vote(0);  
    console.log("Vote recorded");  

    // Get winner  
    const winner = await voting.winningProposal();  
    console.log("Winning proposal:", winner.toString());  
}  
```
Run with: `npx hardhat run scripts/interact.js --network bsc_testnet`  

### Key Workflow Summary  
1. **Development**: Write contract with clear state management and events  
2. **Testing**: Validate functionality with Truffle test cases  
3. **Deployment**:  
   - Truffle: Use migrations with HDWalletProvider  
   - Hardhat: Script-based deployment with ethers.js  
4. **Interaction**:  
   - Truffle console for direct contract calls  
   - Hardhat scripts for programmatic interaction  

> **Pro Tip**: Always test contracts locally before deploying to testnets. Use events (`Voted` in our contract) for transparent transaction tracking.[1][2]  

For production readiness:  
- Add access controls (e.g., `onlyChairperson` modifier)  
- Implement time-based voting windows  
- Include vote delegation features[3][4]

[1] https://docs.soliditylang.org/en/latest/solidity-by-example.html

[2] https://docs.soliditylang.org/en/v0.6.5/solidity-by-example.html

[3] https://akhil.sh/tutorials/solidity/solidity/build_a_voting_contract/

[4] https://www.bitdegree.org/learn/best-code-editor/solidity-voting-example-1

[5] https://archive.trufflesuite.com/docs/truffle/quickstart/

[6] https://www.opcito.com/blogs/a-step-by-step-guide-for-smart-contract-deployment-using-hardhat

[7] https://archive.trufflesuite.com/docs/truffle/how-to/contracts/interact-with-your-contracts/

[8] https://ethereum.stackexchange.com/questions/95023/hardhat-how-to-interact-with-a-deployed-contract

[9] https://dev.to/mbogan/smart-contracts-step-by-step-a-beginners-guide-to-debugging-and-deploying-smart-contracts-with-infura-and-truffle-5af3

[10] https://www.innoq.com/en/blog/2019/02/ethereum-truffle-framework-tutorial/

[11] https://www.grizzlypeaksoftware.com/articles?id=5vWBWo4Zpi02FSVCmQunxk

[12] https://github.com/wilfreddenton/truffle-voting

[13] https://github.com/amolsr/voting-truffle

[14] https://www.youtube.com/watch?v=nDvawexSqSM

[15] https://gist.github.com/maheshmurthy/3da385a42678c3e36a8328cbe47cae5b

[16] https://coinsbench.com/smart-contract-a-beginners-guide-to-writing-your-first-solidity-contract-1a025eb3758c

[17] https://dev.to/independencedev/hardhat-deployment-and-upgradable-contract-3aij

[18] https://www.xdc.dev/jay_kulkarni_842b41d81b23/a-guide-to-migrating-from-truffle-to-hardhat-on-the-xdc-network-evm-smart-contract-development-520p

[19] https://github.com/jscriptcoder/smart-terminal

[20] https://www.youtube.com/watch?v=GB3hiiNNDjk

[21] https://dev.to/karthikaaax/how-i-built-my-first-smart-contract-a-simple-voting-app-in-solidity-4508

[22] https://www.bitdegree.org/learn/solidity-examples

[23] https://archive.trufflesuite.com/docs/truffle/reference/command-line-options/

[24] https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-truffle5

[25] https://livebook.manning.com/book/building-ethereum-dapps/chapter-12/

[26] https://www.mdpi.com/2571-5577/6/4/70

[27] https://hedera.com/learning/smart-contracts/how-to-call-a-smart-contract

[28] https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/compile/compile-a-contract-via-cli/

[29] https://www.npmjs.com/package/hardhat-interact
