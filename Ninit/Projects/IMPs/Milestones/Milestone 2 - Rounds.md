### **Deploy the Contracts:**
    
    - **Step 1:** Deploy the `AlloSettings` contract to configure protocol settings.
    - **Step 2:** Deploy the `QuadraticFundingVotingStrategyFactory` contract.
    - **Step 3:** Deploy the `RoundFactory` contract, referencing the `AlloSettings` and `QuadraticFundingVotingStrategyFactory`.
    - **Step 4:** Deploy the `ProgramFactory` contract.
    - **Step 5:** Deploy the `ProjectRegistry` contract.
    - **Step 6:** Deploy the initial `ProgramImplementation` and `RoundImplementation` contracts.

### **Initialize the Contracts:**
    
    - Initialize `AlloSettings` with the protocol treasury address and fee percentage.
    - Initialize the `QuadraticFundingVotingStrategyFactory` and `RoundFactory`.
    - Update the `ProgramFactory` with the address of the `ProgramImplementation`.
    - Update the `RoundFactory` with the address of the `RoundImplementation`.

### **Frontend Design:**

1. **Program Creation:**

- **Component:** `CreateProgram`
- **Functionality:** Form to input program metadata, and create a program using the `ProgramFactory`.

```
const createProgram = async (metaPtr, adminRoles, programOperators) => { const encodedParameters = ethers.utils.defaultAbiCoder.encode( ['tuple(uint256,string)', 'address[]', 'address[]'], [metaPtr, adminRoles, programOperators] ); await programFactory.create(encodedParameters); };
```

2. **Round Creation:**

- **Component:** `CreateRound`
- **Functionality:** Form to input round details and create a round within a program using the `RoundFactory`.

```
const createRound = async (parameters, ownedBy) => {
    const encodedParameters = ethers.utils.defaultAbiCoder.encode(
        // encode your parameters accordingly
    );
    await roundFactory.create(encodedParameters, ownedBy);
};

```


3. **Project Registry:**

- **Component:** `CreateProject`
- **Functionality:** Form to input project metadata, and create a project using the `ProjectRegistry`.

```
const createProject = async (metaPtr) => {
    await projectRegistry.createProject(metaPtr);
};

```

4. **Application to Round:**

- **Component:** `ApplyToRound`
- **Functionality:** Form to submit an application to a round.

```
const applyToRound = async (projectID, metaPtr) => {
    await roundImplementation.applyToRound(projectID, metaPtr);
};
```

5. **Voting on Projects:**

- **Component:** `VoteOnProjects`
- **Functionality:** Interface for donors to cast votes on projects during a round.

```
const vote = async (votes) => {
    await votingStrategy.vote(votes, voterAddress);
};

```


### Hooks

- **useContract** - Custom hook to get contract instances:

```
import { useEffect, useState } from 'react';
import { ethers } from 'ethers';

const useContract = (address, ABI) => {
    const [contract, setContract] = useState(null);

    useEffect(() => {
        if (address && ABI) {
            const provider = new ethers.providers.Web3Provider(window.ethereum);
            const signer = provider.getSigner();
            const contractInstance = new ethers.Contract(address, ABI, signer);
            setContract(contractInstance);
        }
    }, [address, ABI]);

    return contract;
};

export default useContract;
```

- **usePrograms** - Hook to fetch and manage programs:

```
import { useState, useEffect } from 'react';
import useContract from './useContract';
import { PROGRAM_FACTORY_ADDRESS, PROGRAM_FACTORY_ABI } from './constants';

const usePrograms = () => {
    const [programs, setPrograms] = useState([]);
    const programFactory = useContract(PROGRAM_FACTORY_ADDRESS, PROGRAM_FACTORY_ABI);

    useEffect(() => {
        const fetchPrograms = async () => {
            if (programFactory) {
                // Fetch all programs
                // Update programs state
            }
        };
        fetchPrograms();
    }, [programFactory]);

    return programs;
};

export default usePrograms;

```

### Deployment Order and Testing

1. Deploy the `AlloSettings` contract.
2. Deploy the `QuadraticFundingVotingStrategyFactory` contract.
3. Deploy the `RoundFactory` contract.
4. Deploy the `ProgramFactory` contract.
5. Deploy the `ProjectRegistry` contract.
6. Deploy the `ProgramImplementation` and `RoundImplementation` contracts.
7. Initialize the contracts in the following order:
    - Initialize `AlloSettings`.
    - Initialize `QuadraticFundingVotingStrategyFactory`.
    - Initialize `RoundFactory`.
    - Update `ProgramFactory` with `ProgramImplementation`.
    - Update `RoundFactory` with `RoundImplementation`.

```
async function main() {
    // Deploy AlloSettings
    const alloSettings = await AlloSettings.deploy();
    await alloSettings.deployed();
    console.log("AlloSettings deployed to:", alloSettings.address);

    // Initialize AlloSettings
    await alloSettings.initialize();
    await alloSettings.updateProtocolFeePercentage(1000); // 1%
    await alloSettings.updateProtocolTreasury(treasuryAddress);

    // Deploy QuadraticFundingVotingStrategyFactory
    const votingStrategyFactory = await QuadraticFundingVotingStrategyFactory.deploy();
    await votingStrategyFactory.deployed();
    console.log("VotingStrategyFactory deployed to:", votingStrategyFactory.address);

    // Initialize QuadraticFundingVotingStrategyFactory
    await votingStrategyFactory.initialize();
    await votingStrategyFactory.updateVotingContract(quadraticVotingImplementationAddress);

    // Deploy RoundFactory
    const roundFactory = await RoundFactory.deploy();
    await roundFactory.deployed();
    console.log("RoundFactory deployed to:", roundFactory.address);

    // Initialize RoundFactory
    await roundFactory.initialize();
    await roundFactory.updateAlloSettings(alloSettings.address);
    await roundFactory.updateRoundImplementation(roundImplementationAddress);

    // Deploy ProgramFactory
    const programFactory = await ProgramFactory.deploy();
    await programFactory.deployed();
    console.log("ProgramFactory deployed to:", programFactory.address);

    // Initialize ProgramFactory
    await programFactory.initialize();
    await programFactory.updateProgramContract(programImplementationAddress);

    // Deploy ProjectRegistry
    const projectRegistry = await ProjectRegistry.deploy();
    await projectRegistry.deployed();
    console.log("ProjectRegistry deployed to:", projectRegistry.address);

    // Now you can start creating programs, rounds, and projects using the frontend.
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});

```


### Deploy Steps

1. Program Factory
2. Program Implementation
3. Link Program Implmenetation
4. Deploy QF factory
5. Deploy QF Implementation
6. Link QF Implementation
7. Deploy Allo Settings
8. Set Protocol fee
9. Deploy Round Factory
10. Deploy Round Implementation
11. Link Round Implementation
12. Link Allo Settings
13. Deploy dummy voting strategy
14. Run create program
15. Run create qf contract
16. run create merkle contract


