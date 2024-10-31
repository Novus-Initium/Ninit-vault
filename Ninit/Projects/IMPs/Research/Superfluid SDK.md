
### Key Features

1. **Real-Time Streaming**:
    
    - Allows continuous token streams rather than discrete transactions.
    - Ideal for use cases requiring ongoing payments like subscriptions or salaries.
2. **Programmable Cashflows**:
    
    - Enables the creation of programmable, composable cash flows.
    - Provides flexibility and automation for financial operations.
3. **Versatility**:
    
    - Compatible with various EVM-based networks.
    - Developer-friendly and adaptable to multiple use cases.

### Capabilities and Use Cases

1. **Continuous Payments**:
    
    - Stream tokens for subscriptions, salaries, rent, etc.
    - Payments can be made per second instead of monthly.
2. **Reward Distributions**:
    
    - Stream rewards to multiple participants in real-time.
    - Ensures fair and transparent distribution systems.
3. **Continuous Dollar-Cost Averaging (DCA)**:
    
    - Automate DCA into preferred assets every second.
    - Reduces the complexity and cost of traditional DCA transactions.

### Getting Started with the SDK

1. **Installation**:
    
    - Install the SDK core with `npm install @superfluid-finance/sdk-core`.
    - Install peer dependencies `graphql` and `ethers.js` with `npm install graphql ethers`.
2. **Initialization**:
    
    - Create a Framework instance, which serves as the entry point for interacting with Superfluid features.
    - Requires `chainId` and `provider` options.
    - Example:

```
import { Framework } from "@superfluid-finance/sdk-core"; import { ethers } from "ethers"; const sf = await Framework.create({ chainId: 137, // e.g., Polygon provider: ethersProvider, // ethers provider });
```

**3. Signer Creation**:

- Signers can be created using Web3Provider, MetaMask, Hardhat, or Node.js environments.
- Example with Web3Modal:

```
import Web3Modal from "web3modal"; import { Web3Provider } from "@ethersproject/providers"; const web3ModalRawProvider = await web3Modal.connect(); const web3ModalProvider = new Web3Provider(web3ModalRawProvider, "any"); const sf = await Framework.create({ chainId: 137, provider: web3ModalProvider, }); const web3ModalSigner = sf.createSigner({ web3Provider: web3ModalProvider });
```

### Operations

1. **Token Operations**:
    
    - Manage Super Tokens (e.g., USDCx, DAIx).
    - Perform upgrades, downgrades, and transfers.
2. **Agreement Operations**:
    
    - Utilize ConstantFlowAgreementV1 (CFA) and InstantDistributionAgreementV1 (IDA) for streaming and distributing tokens.
    - Example to get a flow:

```
const flowInfo = await sf.cfaV1.getFlow({ superToken: "0x...", // Super Token address sender: "0x...", // Sender's address receiver: "0x...", // Receiver's address providerOrSigner: /* Provider or signer */ });
```

### Advanced Usage

1. **Creating, Updating, and Deleting Flows**:
    
    - Create, update, or delete token streams between accounts.
    - Example to create a flow:

```
const createFlowOperation = daix.createFlow({ sender: "0x...", receiver: "0x...", flowRate: "1000000000" }); const txnResponse = await createFlowOperation.exec(signer); const txnReceipt = await txnResponse.wait();
```

**ACL Permissions**:

- Manage Access Control List (ACL) permissions for streams.
- Example to set ACL permissions:

```
const setPermissionsOperation = sf.cfaV1.setACLPermissions({ flowOperator: "0x...", // Operator address permissions: 7, // Permissions bitmask flowRateAllowance: "1000000000" }); const txnResponse = await setPermissionsOperation.exec(signer); const txnReceipt = await txnResponse.wait();
```