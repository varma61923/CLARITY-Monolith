CLARITY: Censorship-Resistant Ledger for Anonymous Reporting, Integrity, and Truth for You

Welcome to CLARITY, a decentralized platform for journalists and whistleblowers to publish news anonymously, free from censorship. Built on Ethereum, IPFS, and DAO governance, CLARITY ensures truth and transparency with human-authored content.

This README guides you through setting up, deploying, and using CLARITY. It's designed for everyone—developers, journalists, or curious users—to get started easily.

Table of Contents
1. Project Overview
2. Getting Started
3. Deployment
4. User Manual
5. Troubleshooting
6. Contributing
7. License
8. Contact

Project Overview

Mission
CLARITY empowers anonymous, uncensored news publishing with blockchain and IPFS. It uses zero-knowledge proofs and DAO governance to protect free speech and ensure content authenticity.

Key Features
- Publish articles on IPFS with on-chain metadata.
- Sign in with Ethereum (SIWE) and anonymous zk-KYC.
- Anti-AI checks for human-authored content.
- Multilingual support (English, Spanish, Chinese, Arabic, Hindi).
- Stake-to-flag for community moderation.
- Gasless voting via Snapshot.
- On-chain subscriptions with automated keepers.
- Rich-text and Markdown editors.
- Reputation graph, light/dark themes, tooltips, and whitepaper page.

Tech Stack
- Frontend: React, TypeScript, Vite, Radix UI, Framer Motion, Tiptap
- Blockchain: Ethereum (Sepolia), Ethers.js, SIWE
- Storage: IPFS
- Indexing: The Graph (simulated)
- Governance: Snapshot (simulated)
- Smart Contracts: Solidity
- Utilities: Lunr (search), CryptoJS (encryption), ZK proofs (simulated)

Getting Started

Prerequisites
- Node.js (v18+): https://nodejs.org/
- npm (included with Node.js)
- MetaMask: https://metamask.io/
- Infura account: https://infura.io/ (for Ethereum RPC and IPFS)
- IPFS node or pinning service (e.g., Pinata)
- Code editor (e.g., VS Code)
- Git
- Optional: Hardhat/Foundry for contracts

Installation
1. Clone the repository:
   git clone https://github.com/your-org/clarity.git
   cd clarity
2. Install dependencies:
   npm install
3. Create a .env file:
   VITE_IPFS_API=https://ipfs.infura.io:5001
   VITE_ETHEREUM_RPC=https://sepolia.infura.io/v3/YOUR_INFURA_KEY
   VITE_SNAPSHOT_SPACE=clarity.eth
   VITE_ENS_NAME=clarity.eth
   - Replace YOUR_INFURA_KEY with your Infura key.
   - Set VITE_SNAPSHOT_SPACE to your Snapshot space.
   - Set VITE_ENS_NAME to your ENS domain.

Configuration
1. IPFS: Use Infura, Pinata, or a local IPFS node (ipfs daemon).
2. Ethereum RPC: Use Infura or Alchemy for Sepolia testnet.
3. Smart Contracts:
   - Copy Solidity contracts from src/App.tsx to contracts/ folder.
   - Update contract addresses in src/App.tsx:
     contracts: {
       JournalistSBT: '0xYourAddress',
       ArticleRegistry: '0xYourAddress',
       SubscriptionManager: '0xYourAddress',
       DAOController: '0xYourAddress',
       ReputationManager: '0xYourAddress',
       ZKDonationRelayer: '0xYourAddress',
       DisputeResolver: '0xYourAddress',
     }
4. The Graph: Set up a subgraph (see Deployment).
5. Snapshot: Create a Snapshot space at https://snapshot.org/.

Deployment

Local Development
1. Start the server:
   npm run dev
   Access at http://localhost:5173.
2. Test with MetaMask on Sepolia.
3. Build for production:
   npm run build

IPFS Deployment
1. Build the app:
   npm run build
2. Pin to IPFS:
   - Pinata:
     npm install -g pinata-cli
     pinata pinDir dist
   - IPFS CLI:
     ipfs add -r dist
   Copy the CID (e.g., QmYourCID).
3. Update ENS:
   - Set clarity.eth content hash to ipfs://QmYourCID at https://app.ens.domains/.
   - Access at https://clarity.eth.limo.

Smart Contracts Deployment
1. Set up Hardhat:
   npm install --save-dev hardhat
   npx hardhat init
2. Copy contracts from src/App.tsx to contracts/ folder.
3. Configure hardhat.config.ts:
   import { HardhatUserConfig } from 'hardhat/config';
   import '@nomicfoundation/hardhat-toolbox';

   const config: HardhatUserConfig = {
     solidity: '0.8.20',
     networks: {
       sepolia: {
         url: process.env.VITE_ETHEREUM_RPC,
         accounts: ['YOUR_PRIVATE_KEY'],
       },
     },
   };
   export default config;
4. Deploy:
   npx hardhat run scripts/deploy.ts --network sepolia
   Example deploy.ts:
   import { ethers } from 'hardhat';

   async function main() {
     const contracts = [
       'JournalistSBT',
       'ArticleRegistry',
       'SubscriptionManager',
       'DAOController',
       'ReputationManager',
       'ZKDonationRelayer',
       'DisputeResolver',
     ];
     for (const contract of contracts) {
       const ContractFactory = await ethers.getContractFactory(contract);
       const deployed = await ContractFactory.deploy();
       await deployed.deployed();
       console.log(`${contract} deployed to: ${deployed.address}`);
     }
   }
   main().catch(error => {
     console.error(error);
     process.exitCode = 1;
   });
5. Update contract addresses in src/App.tsx.

The Graph Subgraph
1. Create a subgraph at https://thegraph.com/.
2. Define subgraph.yaml:
   specVersion: 0.0.5
   schema:
     file: ./schema.graphql
   dataSources:
     - kind: ethereum
       name: ArticleRegistry
       network: sepolia
       source:
         address: '0xYourArticleRegistryAddress'
         abi: ArticleRegistry
       mapping:
         entities:
           - Article
         abis:
           - name: ArticleRegistry
             file: ./abis/ArticleRegistry.json
         eventHandlers:
           - event: ArticleRegistered(uint256,string,string,address)
             handler: handleArticleRegistered
3. Define schema.graphql:
   type Article @entity {
     id: ID!
     contentCid: String!
     metadataCid: String!
     author: Bytes!
     timestamp: BigInt!
   }
4. Implement mappings.ts:
   import { ArticleRegistered } from '../generated/ArticleRegistry/ArticleRegistry';
   import { Article } from '../generated/schema';

   export function handleArticleRegistered(event: ArticleRegistered): void {
     let article = new Article(event.params.id.toString());
     article.contentCid = event.params.contentCid;
     article.metadataCid = event.params.metadataCid;
     article.author = event.params.author;
     article.timestamp = event.block.timestamp;
     article.save();
   }
5. Deploy:
   graph auth --studio YOUR_ACCESS_TOKEN
   graph deploy --studio clarity-subgraph
6. Update queryTheGraph in src/App.tsx with GraphQL queries.

Snapshot Integration
1. Create a Snapshot space at https://snapshot.org/ (e.g., clarity.eth).
2. Update VITE_SNAPSHOT_SPACE in .env.
3. Install Snapshot.js:
   npm install @snapshot-labs/snapshot.js
4. Update snapshotVote in src/App.tsx:
   import { Web3Provider } from '@ethersproject/providers';
   import Snapshot from '@snapshot-labs/snapshot.js';

   const snapshotVote = async (proposalId: number, inFavor: boolean, address: string) => {
     const hub = 'https://hub.snapshot.org';
     const client = new Snapshot.Client712(hub);
     const web3 = new Web3Provider(window.ethereum);
     const vote = {
       space: ClarityConfig.snapshotSpace,
       proposal: `0x${proposalId.toString(16)}`,
       type: 'single-choice',
       choice: inFavor ? 1 : 2,
       app: 'clarity',
     };
     await client.vote(web3, address, vote);
   };

User Manual

Connecting a Wallet
1. Visit http://localhost:5173 or https://clarity.eth.limo.
2. Click "Connect Wallet" or "Sign In with Ethereum".
3. In MetaMask, connect your wallet and sign the message.

Browsing Articles
1. Click "Home" to see the article feed.
2. Search articles by title/content or filter by language/category.
3. Click an article to read it, view metadata, and see flags.
4. Donate (standard or anonymous) or subscribe to the author.

Publishing an Article
1. Click "Dashboard" (requires wallet).
2. Choose Markdown or Rich Text editor.
3. Enter title, select language/category, and write content.
4. Enable ZK Watermark (optional).
5. Click "Publish Article" and sign in MetaMask.

Subscribing to Authors
1. Click "Subscriptions".
2. Select an author, enter amount (e.g., 0.1 ETH), choose currency.
3. Click "Subscribe" and confirm in MetaMask.
4. View and cancel subscriptions in the same section.

Participating in Governance
1. Click "Governance".
2. Create a proposal with title and description.
3. Vote on proposals ("Vote For" or "Vote Against") via Snapshot.
4. Delegate voting power in the Dashboard’s "Delegate" tab.

Flagging and Resolving Disputes
1. In an article, click "Flag Article".
2. Enter reason and stake (e.g., 0.01 ETH), then confirm.
3. In Governance, resolve disputes by entering dispute ID and resolution.

Managing Your Profile
1. Click "Profile".
2. View pseudonym, reputation, SBT status, and activity.
3. See reputation trend graph.

Exploring the Whitepaper
1. Click "Whitepaper" in the header or footer.
2. Read CLARITY’s mission and principles.

Troubleshooting
- MetaMask not connecting: Ensure MetaMask is installed and set to Sepolia.
- IPFS errors: Verify VITE_IPFS_API and your pinning service credentials.
- Contract errors: Check contract addresses in ClarityConfig.contracts.
- Snapshot voting fails: Ensure VITE_SNAPSHOT_SPACE matches your space.
- Slow performance: Add pagination to article feeds (future improvement).

Contributing
1. Fork the repository.
2. Create a branch: git checkout -b feature/your-feature
3. Commit changes: git commit -m "Add your feature"
4. Push: git push origin feature/your-feature
5. Open a pull request.

License
MIT License. See LICENSE file for details.

Contact
- Website: https://clarity.eth.limo
- Twitter: @ClarityPlatform
- Email: support@clarity.eth
- Discord: https://discord.gg/clarity

Thank you for joining CLARITY’s mission to protect truth and free speech. Let’s build a decentralized future together!
