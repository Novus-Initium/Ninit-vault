# What?

Inco is a modular confidential computing network that leverages Fully Homomorphic Encryption (FHE) to enable the development of decentralized applications (dApps) with on-chain confidentiality. Some key features of Inco that are relevant for building an online casino game with random number generation include:

1. **Encrypted State On-Chain**: Inco allows you to directly store encrypted data on the blockchain, eliminating the need for off-chain storage or coordination.**Composable Encrypted State**: Inco enables you to perform state transitions on top of encrypted data, fully on-chain. You can add, divide, and compare hidden values without having to decrypt them.

2. **On-Chain Randomness**: Inco provides the ability to generate random numbers on-chain for your applications, removing the need for external randomness services.To get started with using Inco for your online casino game:****Familiar Smart Contract Language and Tools****  

Inco is fully compatible with Ethereum development tools like Metamask, Hardhat, and Remix. You can build your dApp in Solidity using the same tools you're already familiar with.

****Confidentiality-as-a-Service (CaaS)****  

Inco's CaaS enables you to add customizable confidential state, computation, and randomness to your dApp, which can be deployed on any blockchain network like Ethereum, Arbitrum, Base, Polygon, or Optimism.****Encrypted Battleship Example****  

Inco provides a sample Solidity contract for an encrypted Battleship game, which demonstrates how to use their FHE-powered primitives to build games with hidden player boards and encrypted attacks.

To integrate Inco into your online casino game:

1. Set up your development environment with Inco-compatible tools like Hardhat and Metamask.
2. Use Inco's encrypted state and on-chain randomness primitives to build the core gameplay mechanics of your casino game, such as hiding player hands, shuffling decks, and generating random outcomes.
3. Leverage Inco's CaaS to deploy your confidential casino game dApp on the blockchain network of your choice.

By using Inco's confidentiality layer, you can build an online casino game that keeps player information and game state encrypted on-chain, while still allowing for interactive gameplay and provably fair random number generation.

#### **Encrypted State and Randomness On-Chain**  

Inco allows you to directly store encrypted game state on the blockchain, including player hands, deck shuffling, and random outcomes. This eliminates the need for off-chain storage or coordination. Inco's on-chain randomness primitives enable you to generate provably fair random numbers that are verifiable by players, without revealing the underlying values.

#### **Composable Encrypted Primitives**  

Inco's confidentiality-as-a-service (CaaS) provides Solidity-compatible primitives that let you perform state transitions on encrypted data, such as adding, dividing, and comparing hidden values. This allows you to build the core gameplay mechanics of your casino game entirely on-chain, while keeping player information confidential.[](https://www.blocmates.com/articles/fareplay-the-on-chain-casino-with-no-house)**Battleship Example**  
Inco provides a sample Solidity contract for an encrypted Battleship game, which demonstrates how to use their FHE-powered primitives to build games with hidden player boards and encrypted attacks. You can reference this example to understand how to structure your casino game's smart contract logic.[](https://www.blocmates.com/articles/fareplay-the-on-chain-casino-with-no-house)