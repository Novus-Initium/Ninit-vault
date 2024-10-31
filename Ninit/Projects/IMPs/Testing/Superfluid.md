
This is a step by step developer guide to deploying FlowSender:

1. Template Scaffold-ETH-2
2. Add the superfluid and openzeppelin libraries
	 ```yarn add @superfluid-finance/ethereum-contracts @openzeppelin/contracts```
3. Add FlowSender .sol in packages/hardhat/contracts/ directory:
```
//SPDX-License-Identifier: Unlicensed
pragma solidity ^0.8.14;

import { ISuperfluid, ISuperToken } from "@superfluid-finance/ethereum-contracts/contracts/interfaces/superfluid/ISuperfluid.sol";
import { SuperTokenV1Library } from "@superfluid-finance/ethereum-contracts/contracts/apps/SuperTokenV1Library.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface IFakeDAI is IERC20 {
    function mint(address account, uint256 amount) external;
}

contract FlowSender {
    using SuperTokenV1Library for ISuperToken;
    mapping (address => bool) public accountList;
    ISuperToken public daix;

    // fDAIx address on Polygon Mumbai = 0x5D8B4C2554aeB7e86F387B4d6c00Ac33499Ed01f
    constructor(ISuperToken _daix) {
        daix = _daix;
    }

    function gainDaiX() external {
        IFakeDAI fdai = IFakeDAI(daix.getUnderlyingToken());
        fdai.mint(address(this), 10000e18);
        fdai.approve(address(daix), 20000e18);
        daix.upgrade(10000e18);
    }

    function createStream(int96 flowRate, address receiver) external {
        daix.createFlow(receiver, flowRate);
    }

    function updateStream(int96 flowRate, address receiver) external {
        daix.updateFlow(receiver, flowRate);
    }

    function deleteStream(address receiver) external {
        daix.deleteFlow(address(this), receiver);
    }

    function readFlowRate(address receiver) external view returns (int96 flowRate) {
        return daix.getFlowRate(address(this), receiver);
    }
}
```

4. Change deploy script to deploy FlowSender in packages/hardhat/deploy/00_deploy_your_contract.ts

```
import { HardhatRuntimeEnvironment } from "hardhat/types";

import { DeployFunction } from "hardhat-deploy/types";

import { Contract } from "ethers";

  

/**

* Deploys a contract named "YourContract" using the deployer account and

* constructor arguments set to the deployer address

*

* @param hre HardhatRuntimeEnvironment object.

*/

const deployYourContract: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {

/*

On localhost, the deployer account is the one that comes with Hardhat, which is already funded.

  

When deploying to live networks (e.g `yarn deploy --network sepolia`), the deployer account

should have sufficient balance to pay for the gas fees for contract creation.

  

You can generate a random account with `yarn generate` which will fill DEPLOYER_PRIVATE_KEY

with a random private key in the .env file (then used on hardhat.config.ts)

You can run the `yarn account` command to check your balance in every network.

*/

const { deployer } = await hre.getNamedAccounts();

const { deploy } = hre.deployments;

  

await deploy("FlowSender", {

from: deployer,

// Contract constructor arguments

args: [deployer],

log: true,

// autoMine: can be passed to the deploy function to make the deployment process faster on local networks by

// automatically mining the contract deployment transaction. There is no effect on live networks.

autoMine: true,

});

  

// Get the deployed contract to interact with it after deploying.

const yourContract = await hre.ethers.getContract<Contract>("FlowSender", deployer);

};

  

export default deployYourContract;

  

// Tags are useful if you have multiple deploy files and only want to run one of them.

// e.g. yarn deploy --tags YourContract

deployYourContract.tags = ["FlowSender"];
```


5. Configure hardhat to optimism sepolia testnet in hardhat/hardhat.config.js and nextjs/scaffold.congig.ts
	
6. Grab Funds [optimism sepolia faucet](https://www.alchemy.com/faucets/optimism-sepolia)
	1. Requires .001 ETH in mainnet to pull from faucet
8. Deployed FlowSender CA: 0xbF54Dd38d6480EF908f33Ace05Fd5eD2237A5CE1

Now time to test with UI!

1. Add to packages/nextjs/components/superfluid/
	1. FlowSender.tsx

FakeDAI

- Create FakeDAI 
- CA: 0x0234E25474772BD5724632Ccf3ADE843fD5446EC

```
import React, { useState, useEffect } from 'react';

import { ethers } from 'ethers';

import FlowSenderArtifact from '../../../hardhat/artifacts/contracts/FlowSender.sol/FlowSender.json';

  

interface FlowSenderComponentProps {

contractAddress: string;

}

  

const FlowSenderComponent: React.FC<FlowSenderComponentProps> = ({ contractAddress }) => {

const [flowRate, setFlowRate] = useState('');

const [receiver, setReceiver] = useState('');

const [flowRateRead, setFlowRateRead] = useState<string | null>(null);

const [provider, setProvider] = useState<ethers.providers.Web3Provider | null>(null);

const [signer, setSigner] = useState<ethers.Signer | null>(null);

const [flowSenderContract, setFlowSenderContract] = useState<ethers.Contract | null>(null);

  

useEffect(() => {

if (typeof window !== 'undefined' && typeof window.ethereum !== 'undefined') {

const web3Provider = new ethers.providers.Web3Provider(window.ethereum);

setProvider(web3Provider);

setSigner(web3Provider.getSigner());

setFlowSenderContract(new ethers.Contract(contractAddress, FlowSenderArtifact.abi, web3Provider.getSigner()));

}

}, [contractAddress]);

  

const createStream = async () => {

if (flowSenderContract) {

await flowSenderContract.createStream(ethers.utils.parseUnits(flowRate, 18), receiver);

}

};

  

const readFlowRate = async () => {

if (flowSenderContract) {

const rate = await flowSenderContract.readFlowRate(receiver);

setFlowRateRead(ethers.utils.formatUnits(rate, 18));

}

};

  

const containerStyle = {

display: 'flex',

flexDirection: 'column' as 'column',

alignItems: 'center',

padding: '20px',

border: '1px solid #ccc',

borderRadius: '8px',

maxWidth: '400px',

margin: '0 auto',

};

  

const inputStyle = {

margin: '10px 0',

padding: '10px',

width: '100%',

borderRadius: '4px',

border: '1px solid #ccc',

};

  

const buttonStyle = {

margin: '10px 0',

padding: '10px 20px',

width: '100%',

backgroundColor: '#4CAF50',

color: 'white',

border: 'none',

borderRadius: '4px',

cursor: 'pointer',

};

  

const buttonSecondaryStyle = {

...buttonStyle,

backgroundColor: '#008CBA',

};

  

return (

<div style={containerStyle}>

<input

type="text"

placeholder="Receiver Address"

value={receiver}

onChange={(e) => setReceiver(e.target.value)}

style={inputStyle}

/>

<input

type="text"

placeholder="Flow Rate"

value={flowRate}

onChange={(e) => setFlowRate(e.target.value)}

style={inputStyle}

/>

<button onClick={createStream} style={buttonStyle}>

Create Stream

</button>

<button onClick={readFlowRate} style={buttonSecondaryStyle}>

Read Flow Rate

</button>

{flowRateRead && <p>Current Flow Rate: {flowRateRead}</p>}

</div>

);

};

  

export default FlowSenderComponent;
```

2. Add FlowSenderPlayground.tsx in same folder
```
import React from 'react';

import FlowSenderComponent from './FlowSender';

  

function FlowSenderPlayground() {

const yourAddress = '0xbF54Dd38d6480EF908f33Ace05Fd5eD2237A5CE1'; // Replace with your actual deployed contract address

return (

<div>

<FlowSenderComponent contractAddress={yourAddress} />

</div>

);

}

  

export default FlowSenderPlayground;
```

