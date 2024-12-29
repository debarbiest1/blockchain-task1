# Smart contract creation instructions

# Solidity Smart Contract Project  

Welcome to my Solidity smart contract deployment guide! This README serves as a comprehensive guide to building, deploying, and interacting with Ethereum-based smart contracts using **Solidity**, **Web3.js**, and **Hardhat**.  

---

## Table of Contents  
1. [Contributor](#contributor)  
2. [Introduction](#introduction)  
3. [Features](#features)  
4. [System Requirements](#system-requirements)  
5. [Setup Instructions](#setup-instructions)  
6. [Step-by-Step Guide](#step-by-step-guide)  
    - [Writing the Contract](#1-writing-the-contract)  
    - [Compiling the Contract](#2-compiling-the-contract)  
    - [Deploying the Contract](#3-deploying-the-contract)  
    - [Interacting with the Contract](#4-interacting-with-the-contract)   
7. [License](#license)  

---

## Contributor  
- **Kumissay Zhalmagambetova**   

---

## Introduction  
This project provides an easy-to-follow guide to creating and deploying smart contracts on the Ethereum blockchain. The contract is written in **Solidity** and deployed to a local Ethereum network using **Hardhat**. Additionally, you will learn how to interact with the deployed contract using **Web3.js**.  

---

## Features  
1. A Solidity contract with the following functionalities:  
   - Setting and updating a public variable.  
   - Receiving and withdrawing funds.  
   - Managing ownership.  
2. Comprehensive scripts for:  
   - Compiling the Solidity code.  
   - Deploying the contract to a blockchain.  
   - Interacting with the deployed contract.  
3. Support for local Ethereum nodes with Hardhat.  

---

## System Requirements  
Ensure your system meets the following requirements:  
1. **Node.js** (v18.16.1 or higher)  
2. **npm** (v9.5.1 or higher)  
3. Basic understanding of JavaScript and blockchain concepts.  

Check your versions with:  
```bash
node -v  
npm -v

```
## Setup Instructions
1. Clone this repository:
```
git clone https://github.com/debarbiest1/blockchain-task1/
```
2. Install dependencies:
```
npm install
```
## Step-by-Step Guide
1. Writing the Contract
Create a Solidity file named MyContract.sol with the following code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {
    uint256 public myNumber;
    address public owner;

    constructor(uint256 _myNumber) {
        myNumber = _myNumber;
        owner = msg.sender;
    }

    receive() external payable {}

    fallback() external payable {}

    function withdraw() external {
        require(msg.sender == owner, "Access denied");
        payable(owner).transfer(address(this).balance);
    }

    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }

    function setMyNumber(uint256 _myNumber) public {
        myNumber = _myNumber;
    }
}
```
2. Compiling the Contract
Install the Solidity compiler:
```
npm install solc@^0.8.0
Create a script compile.js to compile the contract:
javascript
Copy code
const solc = require('solc');
const fs = require('fs');
const path = require('path');

const fileName = 'MyContract.sol';
const contractPath = path.resolve(__dirname, fileName);
const sourceCode = fs.readFileSync(contractPath, 'utf8');

const input = {
    language: 'Solidity',
    sources: {
        [fileName]: { content: sourceCode },
    },
    settings: { outputSelection: { '*': { '*': ['*'] } } },
};

const output = JSON.parse(solc.compile(JSON.stringify(input)));
const bytecode = output.contracts[fileName].MyContract.evm.bytecode.object;
fs.writeFileSync('bytecode.bin', bytecode);

const abi = output.contracts[fileName].MyContract.abi;
fs.writeFileSync('abi.json', JSON.stringify(abi, null, 2));

console.log('Compilation successful!');
```
Run the script:
```
node compile.js
```
3. Deploying the Contract
Install Web3.js and Hardhat:
```
npm install web3 hardhat
Create a deployment script deploy.js:
javascript
Copy code
const Web3 = require('web3');
const fs = require('fs');

const web3 = new Web3('http://127.0.0.1:8545/');
const bytecode = fs.readFileSync('./bytecode.bin', 'utf8');
const abi = JSON.parse(fs.readFileSync('./abi.json', 'utf8'));

const deployContract = async () => {
    const accounts = await web3.eth.getAccounts();
    const contract = new web3.eth.Contract(abi);

    const instance = await contract.deploy({ data: '0x' + bytecode, arguments: [42] })
        .send({ from: accounts[0], gas: 1500000 });

    console.log('Contract deployed at:', instance.options.address);
    fs.writeFileSync('contractAddress.txt', instance.options.address);
};

deployContract().catch(console.error);
```
Start a local Hardhat node and deploy the contract:
```
npx hardhat node  
node deploy.js
```
4. Interacting with the Contract
Create an interaction script interact.js:
```
const Web3 = require('web3');
const fs = require('fs');

const web3 = new Web3('http://127.0.0.1:8545/');
const contractAddress = fs.readFileSync('./contractAddress.txt', 'utf8');
const abi = JSON.parse(fs.readFileSync('./abi.json', 'utf8'));

const interact = async () => {
    const accounts = await web3.eth.getAccounts();
    const contract = new web3.eth.Contract(abi, contractAddress);

    console.log('Initial balance:', await web3.eth.getBalance(contractAddress));

    await web3.eth.sendTransaction({
        from: accounts[0],
        to: contractAddress,
        value: web3.utils.toWei('0.1', 'ether'),
    });

    console.log('New balance:', await web3.eth.getBalance(contractAddress));

    await contract.methods.withdraw().send({ from: accounts[0] });
    console.log('Balance after withdrawal:', await web3.eth.getBalance(contractAddress));
};

interact().catch(console.error);
```
Execute the script:
```
node interact.js
```

## License
This project is licensed under the MIT License.
