---

# Merkle Airdrop System

This project showcases the implementation of an ERC20 token airdrop system that leverages Merkle trees for secure and efficient whitelisting. The repository includes everything needed to deploy the smart contract using Hardhat, generate Merkle trees and proofs with TypeScript, and manage the airdrop process.

## Table of Contents

- [Requirements](#requirements)
- [Project Initialization](#project-initialization)
- [Executing the Merkle Script](#executing-the-merkle-script)
- [Contract Deployment](#contract-deployment)
- [Proof Generation](#proof-generation)
- [Contract Interaction](#contract-interaction)
- [Common Issues](#common-issues)

## Requirements

Before you start, make sure you have the following installed:

- Node.js (version 16 or higher recommended)
- npm or Yarn package manager
- Git version control

## Project Initialization

1. Clone this repository to your local machine:
   ```bash
   git clone https://github.com/your-username/merkle-airdrop.git
   cd merkle-airdrop
   ```

2. Install the necessary dependencies:
   ```bash
   npm install
   ```
   or
   ```bash
   yarn install
   ```

3. Set up your environment variables by creating a `.env` file in the project root with the following content:
   ```
   ETHERSCAN_API_KEY=your_etherscan_project_id
   PRIVATE_KEY=your_private_key
   ```
   Replace `your_etherscan_project_id` with your Etherscan API key and `your_private_key` with your deployment account's private key.

4. Modify the `hardhat.config.ts` file to include the necessary network settings for deployment, such as:

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import * as dotenv from "dotenv";

dotenv.config();

const config: HardhatUserConfig = {
  solidity: "0.8.24",
  networks: {
    sepolia: {
      url: `https://sepolia.infura.io/v3/${process.env.INFURA_API_KEY}`,
      accounts: [process.env.PRIVATE_KEY!]
    }
  }
};

export default config;
```

## Executing the Merkle Script

The `merkle.ts` script is designed to build a Merkle tree and generate the necessary proofs from a CSV file listing the token recipients and amounts.

1. Prepare your input CSV file:
   Place a file named `addresses.csv` in the `feed-files` directory with the following format:
   ```
   user_address,amount
   0x1234...,100
   0x5678...,200
   ```

2. Run the Merkle script:
   ```bash
   npx ts-node scripts/merkle.ts
   ```

Upon execution, two output files will be created:
- `tree.json`: Contains the entire Merkle tree data.
- `feed-files/proofs.json`: Contains the generated Merkle proofs for each listed address.

## Contract Deployment

1. Compile the Solidity contract:
   ```bash
   npx hardhat compile
   ```

2. Deploy the contract to the Sepolia testnet:
   ```bash
   npx hardhat run scripts/deploy.ts --network sepolia
   ```

   Ensure that you have a `deploy.ts` script in your `scripts` folder that handles the deployment of the `MerkleAirdrop` contract.

3. Record the deployed contract address for future operations.

## Proof Generation

Merkle proofs are automatically generated when you run the `merkle.ts` script. The proofs will be located in the `feed-files/proofs.json` file.

If you need to generate proofs for specific addresses, you can modify the following section in `merkle.ts`:

```typescript
// Iterate over the entries in the loaded tree
for (const [i, v] of loadedTree.entries()) {
  // Get the proof for each address
  const proof = loadedTree.getProof(i);
  proofs[v[0]] = proof; // Store the proof with the address as the key

  // Check for a specific address and get the proof if found
  if (v[0] === '0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2') {
    const proof = loadedTree.getProof(i);
    console.log('Proof:', proof);
  }
}
```

Adjust the script to generate proofs for addresses of interest.

## Contract Interaction

You can interact with your deployed contract using Hardhat tasks or custom scripts. Here’s an example script for claiming tokens:

1. Create a script `claim.ts` in the `scripts` folder:

```typescript
import { ethers } from "hardhat";
import proofs from "../feed-files/proofs.json";

async function main() {
  const [signer] = await ethers.getSigners();
  const contractAddress = "YOUR_DEPLOYED_CONTRACT_ADDRESS";
  const contract = await ethers.getContractAt("MerkleAirdrop", contractAddress, signer);

  const address = signer.address;
  const proof = proofs[address];
  const amount = ethers.utils.parseEther("100"); // Adjust based on your airdrop allocation

  const tx = await contract.claim(amount, proof);
  await tx.wait();
  console.log("Tokens successfully claimed!");
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

2. Execute the claim script:
   ```bash
   npx hardhat run scripts/claim.ts --network sepolia
   ```

## Common Issues

- **Gas Price Issues**: If you face problems with gas prices, consider tweaking the gas settings in your Hardhat configuration or during transactions.
- **Environment Variables**: Ensure that your `.env` file is correctly set up and is not pushed to version control.
- **Merkle Proof Verification**: If proof verification fails, ensure that the proof generation in `merkle.ts` aligns with the verification logic in the smart contract.

For further details on specific components, refer to the inline comments within the project files.
---
