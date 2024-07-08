# pyth-coredao-dapp
A simple dApp on Core with Pyth Network integration

# Pyth Price Feeds dApp with Core Network

This guide will walk you through creating a decentralized application that integrates Pyth price feeds with the Core network using Hardhat for development, Solidity for smart contracts, and React for the frontend.

## Project Setup

### 1. Initialize the Project
Create a new directory for your project and navigate into it:

```sh
mkdir pyth-price-feeds-dapp
cd pyth-price-feeds-dapp
npm init --yes
```

### 2. Install Dependencies
Install Hardhat and other necessary dependencies:

```
npm install --save-dev hardhat
npm install --save-dev chai @nomiclabs/hardhat-waffle
npm install react react-dom ethers
```

### 3. Set Up Hardhat
Initialize a new Hardhat project:

```
npx hardhat
```

Select "Create an empty hardhat.config.js" and "no" for installing the project's sample dependencies.

### 4. Create secret.json File
Create a secret.json file in the root directory of your project to store your private key securely. Replace YOUR_PRIVATE_KEY with your actual private key.

```
{
  "PrivateKey": "YOUR_PRIVATE_KEY"
}
```

5. Update hardhat.config.js
Replace the contents of hardhat.config.js with the following configuration:

```
require('@nomiclabs/hardhat-ethers');
require("@nomiclabs/hardhat-waffle");

const { PrivateKey } = require('./secret.json');

module.exports = {
  defaultNetwork: 'testnet',
  networks: {
    hardhat: {},
    testnet: {
      url: 'https://rpc.test.btcs.network',
      accounts: [PrivateKey],
      chainId: 1115,
    }
  },
  solidity: {
    compilers: [
      {
        version: '0.8.24',
        settings: {
          evmVersion: 'paris',
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },
  paths: {
    sources: './contracts',
    cache: './cache',
    artifacts: './artifacts',
  },
  mocha: {
    timeout: 20000,
  },
};
```

## Writing the Smart Contract
### 1. Create the Price Feed Smart Contract
Create a file contracts/PythPriceFeed.sol with the following content:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@pythnetwork/pyth-sdk-solidity/Pyth.sol";

contract PythPriceFeed {
    Pyth public pyth;

    constructor(address pythContract) {
        pyth = Pyth(pythContract);
    }

    function getPrice(bytes32 priceFeedId) public view returns (PythStructs.Price memory price) {
        price = pyth.getPrice(priceFeedId);
    }
}
```

## Deploying the Smart Contract
### 1. Create Deployment Script
Create a file scripts/deploy.js with the following content:

```
const hre = require("hardhat");

async function main() {
  await hre.run('compile'); // Ensure the contracts are compiled

  const PythPriceFeed = await hre.ethers.getContractFactory("PythPriceFeed");
  const pythPriceFeed = await PythPriceFeed.deploy("PYTH_CONTRACT_ADDRESS"); // Replace with Pyth contract address

  await pythPriceFeed.deployed();
  console.log("PythPriceFeed contract deployed to:", pythPriceFeed.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### 2. Deploy the Contract
Deploy the contract to the Core network:

```
npx hardhat run scripts/deploy.js --network testnet
```

## Setting Up the React Frontend
### 1. Create the React App
Create the basic structure for a React frontend:

```
npx create-react-app frontend
cd frontend
```

### 2. Install Ethers.js
Install the Ethers.js library:

```
npm install ethers
```

### 3. Add PythPriceFeedAbi.json
Copy the PythPriceFeed.json file from artifacts/contracts/PythPriceFeed.sol/ to the frontend/src directory and rename it to PythPriceFeedAbi.json.

### 4. Update src/App.js
Create or update the src/App.js file with the following content:

```
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import PythPriceFeedAbi from './PythPriceFeedAbi.json';

const contractAddress = 'YOUR_CONTRACT_ADDRESS'; // Replace with your deployed contract address

function App() {
  const [provider, setProvider] = useState(null);
  const [contract, setContract] = useState(null);
  const [price, setPrice] = useState(null);
  const [priceFeedId, setPriceFeedId] = useState("PYTH_PRICE_FEED_ID"); // Replace with actual price feed ID

  useEffect(() => {
    const init = async () => {
      if (window.ethereum) {
        try {
          const tempProvider = new ethers.providers.Web3Provider(window.ethereum);
          await tempProvider.send("eth_requestAccounts", []); // Request account access if needed
          setProvider(tempProvider);

          const signer = tempProvider.getSigner();
          const tempContract = new ethers.Contract(contractAddress, PythPriceFeedAbi, signer);
          setContract(tempContract);
        } catch (error) {
          console.error("Error connecting to contract:", error);
        }
      } else {
        console.error("Please install MetaMask!");
      }
    };
    init();
  }, []);

  const fetchPrice = async () => {
    if (!contract) {
      console.error("Contract is not initialized!");
      return;
    }

    try {
      const price = await contract.getPrice(priceFeedId);
      setPrice(price);
    } catch (error) {
      console.error("Error fetching price:", error);
    }
  };

  return (
    <div>
      <h1>Pyth Price Feed dApp</h1>
      <button onClick={fetchPrice}>Fetch Price</button>
      {price && (
        <div>
          <p>Price: {price.price}</p>
          <p>Confidence: {price.conf}</p>
        </div>
      )}
    </div>
  );
}

export default App;
```

## Running the React App
Start the React app:

```
npm start
```



