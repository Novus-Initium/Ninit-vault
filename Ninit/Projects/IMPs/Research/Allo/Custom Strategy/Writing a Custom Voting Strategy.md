
- Overview
	- 5 components of a strategy
	- Use one of the library/pre-built components
	- implement at least one component in strategy
	- use tests from allo-v2

1. Fork the [contractss repo](https://github.com/allo-protocol/allo-contracts)
2. Follow the [dev steps](https://github.com/allo-protocol/allo-contracts/blob/main/docs/DEV.md)
3. Create the .env file
	
## Setting up Environment and Testing

- Install libraries 
	- pnpm install
- Alter .env file with ids and keys
	- 1. INFURA_ID: f2852a0f786e4b4e8ee9297c6d0d05df
	2. DEPLOYER_PRIVATE_KEY:
	3. ETHERSCAN_API_KEY: URDDI4GZQV6HWFE7AY7HZW35D414WFRK6N
1. Compile smart contracts, generate TypeChain artifacts
- pnpm clean
- pnpm compile
- pnpm lint:sol
	- checks for
- pnpm test
- pnpm coverage
	- code coverage report
	- 85 artifacts in dir: typechain for target: ethers-v5
- Generate ABI
	- can be found in abis/ folder
	- generate both human readable ABI

	```pnpm run clear-abi```
	```pnpm run expert-abi```
- Report Gas
	- REPORT_GAS=true
	- pnpm test
- Delete smart contracts artifacts, the coverage reports and the hardhat cache

## [Deploy Steps](https://github.com/allo-protocol/allo-contracts/blob/main/docs/DEPLOY_STEPS.md)

## 1. Registry

- Select Network
- Deploy ProjectRegistry contract
- I added the following to test on sepolia:

```
// Test Networks

sepolia: createTestnetConfig("sepolia-test", `https://eth-sepolia.g.alchemy.com/v2/${alchemyKey}`),

"sepolia-test": {

accounts: [deployPrivateKey],

chainId: chainIds["sepolia-test"], // Sepolia testnet chain ID

url: `https://eth-sepolia.g.alchemy.com/v2/${alchemyKey}`,

gasPrice: 20000000000,

},
```

- Deploy project registry: 

```pnpm run deploy-project-registry sepolia```
- CA: 0x6fe42aC1C893bC05C413F0C066532D4d5326F347

## 2. ProgramFactory

- CA: 0xA3b651822698d49065370C01c55D388F2380f9af

```
pnpm run deploy-program-factory sepolia
```

## 3. ProgramImplementation

- CA: 0x25be64Ce8F5DF44eB0Cdb354B34bdFCA60B0d62B

```
pnpm run deploy-program-implementation goerli
```

## 4. Update program.config.ts

```
export const params: DeployParams = {
  goerli: {
    programImplementationContract: 'DEPLOYED_PROGRAM_IMPLEMENTATION_CONTRACT',
    programFactoryContract: 'DEPLOYED_PROGRAM_FACTORY_CONTRACT',
    ...
  },
};
```

## 5. Update ProgramFactory to reference ProgramImplementation contract

```
pnpm run link-program-implementation sepolia
```

# VotingStrategy Setup

## 1. Deploy QuadraticFundingVotingStrategyFactory

```
pnpm run deploy-qf-factory sepolia
```

## 2.  Deploy QFVSImplementation

```
pnpm run deploy-qf-implementation goerli
```

## 3. Update votingStrategy.config.ts

```
export const QFVotingParams: DeployParams = {
  "goerli": {
    factory: 'DEPLOYED_QF_FACTORY_CONTRACT',
    implementation: 'DEPLOYED_QF_IMPLEMENTATION_CONTRACT',
    ...
  },
  ...
};
```


# Round Setup

## 1. Deploy Round Factory

```
pnpm run deploy-round-factory goerli
```

## 2. Deploy Round Implementation

```
pnpm run deploy-round-implementation goerli
```


## 3. Update round.config.ts

```
export const params: DeployParams = {
  goerli: {
    roundImplementationContract: 'DEPLOYED_ROUND_IMPLEMENTATION_CONTRACT',
    roundFactoryContract: 'DEPLOYED_ROUND_FACTORY_CONTRACT',
    ...
  },
  ...
};
```

```
pnpm run link-round-implementation goerli
```






| Contract Name                                | Contract Address                           | Network |
| -------------------------------------------- | ------------------------------------------ | ------- |
| Registry                                     | 0x6fe42aC1C893bC05C413F0C066532D4d5326F347 | Sepolia |
| ProgramFactory                               | 0xA3b651822698d49065370C01c55D388F2380f9af | Sepolia |
| Program Implementation                       | 0x25be64Ce8F5DF44eB0Cdb354B34bdFCA60B0d62B |         |
| QuadraticFundingVotingStrategyFactory        | 0xb70DACa95029dafCC83f59A41cDA1c48129004ec |         |
| QuadraticFundingVotingStrategyImplementation | 0xb4CFb6C5FFC8C03645db72C15aaFB610a5E85cCc |         |
| RoundFactory                                 | 0xa434989F21037645Fa06cc2bfBF4fcBD4820B2Ef |         |
| RoundImplementation                          | 0x24F1d2Cf5b82C395b7094434aa6Bd42A33D1F012 |         |
|                                              |                                            |         |
|                                              |                                            |         |

# Validation

```
pnpm hardhat clean
pnpm hardhat verify --network goerli <CONTRACT_ADDRESS>
```

# Deploy Script

```

# Program
pnpm run deploy-program-factory goerli
pnpm run deploy-program-implementation goerli
pnpm run link-program-implementation goerli

# QF
pnpm run deploy-qf-factory goerli
pnpm run deploy-qf-implementation goerli
pnpm run link-qf-implementation goerli

# Payout
pnpm run deploy-merkle-factory goerli
pnpm run deploy-merkle-implementation goerli
pnpm run link-merkle-implementation goerli

# direct grants
pnpm run deploy-direct-factory goerli
pnpm run deploy-direct-implementation goerli
pnpm run link-direct-implementation goerli

# AlloSettings
pnpm run deploy-allo-settings goerli
pnpm run set-protocol-fee goerli

# Round
pnpm run deploy-round-factory goerli
pnpm run deploy-round-implementation goerli
pnpm run link-round-implementation goerli
pnpm run link-allo-settings goerli
pnpm run deploy-dummy-voting-strategy goerli

# Project Registry
pnpm run deploy-project-registry goerli

# These scripts would be used to create a test round
pnpm run create-program goerli
pnpm run create-qf-contract goerli
pnpm run create-merkle-contract goerli
pnpm run create-round goerli
```


```
# Program
pnpm run deploy-program-factory goerli
pnpm run deploy-program-implementation goerli
pnpm run link-program-implementation goerli

# QF
pnpm run deploy-qf-factory goerli
pnpm run deploy-qf-implementation goerli
pnpm run link-qf-implementation goerli

# Payout
pnpm run deploy-merkle-factory goerli
pnpm run deploy-merkle-implementation goerli
pnpm run link-merkle-implementation goerli

# direct grants
pnpm run deploy-direct-factory goerli
pnpm run deploy-direct-implementation goerli
pnpm run link-direct-implementation goerli

# AlloSettings
pnpm run deploy-allo-settings goerli
pnpm run set-protocol-fee goerli

# Round
pnpm run deploy-round-factory goerli
pnpm run deploy-round-implementation goerli
pnpm run link-round-implementation goerli
pnpm run link-allo-settings goerli
pnpm run deploy-dummy-voting-strategy goerli

# Project Registry
pnpm run deploy-project-registry goerli

# These scripts would be used to create a test round
pnpm run create-program goerli
pnpm run create-qf-contract goerli
pnpm run create-merkle-contract goerli
pnpm run create-round goerli
```

# Copy over artifacts

1. ProjectRegistry
2. ProgramFactory
3. ProgramImplementation
4. QuadraticFundingVotingStrategyFactory
5. QuadraticFundingVotingStrategyFactoryImplementation
6. RoundFactory
7. RoundImplementation




## Index

TypeChain: 

- Helps address the issues that arise from using js libraries interacting with smart contracts when developing in typescript
- It uses provided ABI files to generate typed wrappers for smart contracts.
- Still uses web3js but on the surface it provides robust, type safe API

Notes

- When interacting with smart contracts on Ethereum, you use Web3js
	- Pass ABI and address, call methods, create transactions
	- The js library creates dynamic interfaces during runtime, so this cannot be expressed in the Typescript type system


### Deploy Checklist

https://github.com/allo-protocol/allo-v2/blob/main/docs/DEPLOY_CHECKLIST.md


Deploy Steps

https://github.com/allo-protocol/allo-v2/blob/main/docs/DEPLOY_STEPS.md