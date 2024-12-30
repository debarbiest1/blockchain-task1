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
```
Create a script compile.js to compile the contract:
```
// importing the required modules
const solc = require('solc'); // solidity compiler
const fs = require('fs'); // file system module to handle file operations
const path = require('path'); // module to handle and transform file paths

// define the path to the solidity contract file
const contractPath = path.resolve(__dirname, 'MyContract.sol'); // constructs the absolute path to 'MyContract.sol'

// read the solidity contract's source code from the file
const sourceCode = fs.readFileSync(contractPath, 'utf8'); // reads the content of 'MyContract.sol' as a utf-8 string

// create an input object for the solidity compiler
const input = {
    language: 'Solidity', // specifies the programming language
    sources: {
        'MyContract.sol': { content: sourceCode }, // provides the source code for the compiler
    },
    settings: {
        outputSelection: {
            '*': { // applies the selection to all contracts
                '*': ['abi', 'evm.bytecode'] // specifies that we need the abi and bytecode in the output
            }
        }
    }
};

// compile the contract using the solidity compiler
const output = JSON.parse(solc.compile(JSON.stringify(input))); // compiles the input and parses the output

// check if the compilation was successful
if (!output.contracts['MyContract.sol'] || !output.contracts['MyContract.sol'].MyContract) {
    console.error("compilation failed. check the input and solidity version."); // prints error if compilation fails
    process.exit(1); // exits the process with an error code
}

// extract the bytecode from the compiled contract
const bytecode = output.contracts['MyContract.sol'].MyContract.evm.bytecode.object; 
fs.writeFileSync('bytecode.bin', bytecode); // saves the bytecode to a file named 'bytecode.bin'

// extract the abi (application binary interface) from the compiled contract
const abi = output.contracts['MyContract.sol'].MyContract.abi; 
fs.writeFileSync('abi.json', JSON.stringify(abi, null, 2)); // saves the abi to a file named 'abi.json'

// print a success message to the console
console.log('compilation successful!');

```
Run the script:
```
node compile.js
```
3. Deploying the Contract
Install Web3.js and Hardhat:
```
npm install web3 hardhat
```
Create a deployment script deploy.js:
```
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
![image](https://github.com/user-attachments/assets/69ec5e62-03c5-4bf6-b9ee-722dc53b7feb)
![image](https://github.com/user-attachments/assets/07198ddc-4af2-4ee9-8084-9b2d0e838c14)
![image](https://github.com/user-attachments/assets/118974d5-c8fc-47e4-b7bf-5b00b573a1c8)
![image](https://github.com/user-attachments/assets/f49e9844-e6ec-4761-8c82-d7be160c05fc)
![image](https://github.com/user-attachments/assets/f891b11f-3d28-45d0-ba44-64a9e00e855a)
![image](https://github.com/user-attachments/assets/f07302db-909d-4d2d-92df-f6663c2e1bb0)
![image](https://github.com/user-attachments/assets/464ffb80-5793-45cc-b1aa-343f26680d2c)


PS C:\Users\ASUS\assignment-1-blockchain> npx hardhat node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

Account #2: 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (10000 ETH)
Private Key: 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a

Account #3: 0x90F79bf6EB2c4f870365E785982E1f101E93b906 (10000 ETH)
Private Key: 0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6

Account #4: 0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65 (10000 ETH)
Private Key: 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a

Account #5: 0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc (10000 ETH)
Private Key: 0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba

Account #6: 0x976EA74026E726554dB657fA54763abd0C3a0aa9 (10000 ETH)
Private Key: 0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e

Account #7: 0x14dC79964da2C08b23698B3D3cc7Ca32193d9955 (10000 ETH)
Private Key: 0x4bbbf85ce3377467afe5d46f804f221813b2bb87f24d81f60f1fcdbf7cbf4356

Account #8: 0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f (10000 ETH)
Private Key: 0xdbda1821b80551c9d65939329250298aa3472ba22feea921c0cf5d620ea67b97

Account #9: 0xa0Ee7A142d267C1f36714E4a8F75612F20a79720 (10000 ETH)
Private Key: 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6

Account #10: 0xBcd4042DE499D14e55001CcbB24a551F3b954096 (10000 ETH)
Private Key: 0xf214f2b2cd398c806f84e317254e0f0b801d0643303237d97a22a48e01628897

Account #11: 0x71bE63f3384f5fb98995898A86B02Fb2426c5788 (10000 ETH)
Private Key: 0x701b615bbdfb9de65240bc28bd21bbc0d996645a3dd57e7b12bc2bdf6f192c82

Account #12: 0xFABB0ac9d68B0B445fB7357272Ff202C5651694a (10000 ETH)
Private Key: 0xa267530f49f8280200edf313ee7af6b827f2a8bce2897751d06a843f644967b1

Account #13: 0x1CBd3b2770909D4e10f157cABC84C7264073C9Ec (10000 ETH)
Private Key: 0x47c99abed3324a2707c28affff1267e45918ec8c3f20b8aa892e8b065d2942dd

Account #14: 0xdF3e18d64BC6A983f673Ab319CCaE4f1a57C7097 (10000 ETH)
Private Key: 0xc526ee95bf44d8fc405a158bb884d9d1238d99f0612e9f33d006bb0789009aaa

Account #15: 0xcd3B766CCDd6AE721141F452C550Ca635964ce71 (10000 ETH)
Private Key: 0x8166f546bab6da521a8369cab06c5d2b9e46670292d85c875ee9ec20e84ffb61

Account #16: 0x2546BcD3c84621e976D8185a91A922aE77ECEc30 (10000 ETH)
Private Key: 0xea6c44ac03bff858b476bba40716402b03e41b8e97e276d1baec7c37d42484a0

Account #17: 0xbDA5747bFD65F08deb54cb465eB87D40e51B197E (10000 ETH)
Private Key: 0x689af8efa8c651a91ad287602527f3af2fe9f6501a7ac4b061667b5a93e037fd

Account #18: 0xdD2FD4581271e230360230F9337D5c0430Bf44C0 (10000 ETH)
Private Key: 0xde9be858da4a475276426320d5e9262ecfc3ba460bfac56360bfa6c4c28b4ee0

Account #19: 0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199 (10000 ETH)
Private Key: 0xdf57089febbacf7ba0bc227dafbffa9fc08a93fdc68e1e42411a14efcf23656e

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.`

## License
This project is licensed under the MIT License.
