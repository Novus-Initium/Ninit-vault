
packages/nextjs/components/allo/CreateGrantRound.tsx

```
"use client";
import React, { useState, useEffect } from 'react';
import DatePicker from 'react-datepicker';
import 'react-datepicker/dist/react-datepicker.css';
import { useScaffoldWriteContract } from '~~/hooks/scaffold-eth';
import { notification } from '~~/utils/scaffold-eth';
import { parseUnits, getAddress, isAddress, hexlify } from 'ethers';
import { AddressZero } from '@ethersproject/constants';
import { encodeRoundParameters } from '../../../hardhat/scripts/utils'; // Adjust the path to where your utils file is located
import { ethers } from 'ethers';
import { create } from 'ipfs-http-client';
import dotenv from 'dotenv';

// Load environment variables

dotenv.config();

// Configure IPFS client with Infura

const projectId = process.env.NEXT_PUBLIC_INFURA_PROJECT_ID;
const apiKey = process.env.NEXT_PUBLIC_INFURA_API_KEY;

const ipfs = create({
host: 'ipfs.infura.io',
port: 5001,
protocol: 'https',
headers: {
authorization: `Basic ${Buffer.from(`${projectId}:${apiKey}`).toString('base64')}`,

},

});

  

const chainIdToNetworkName: { [key: string]: string } = {

"1": 'mainnet',

"4": 'rinkeby',

"42": 'kovan',

"137": 'polygon',

"31337": 'localhost', // Localhost Hardhat Network

// Add other networks as needed

};

  

const preloadedABIs: { [network: string]: { [contract: string]: any } } = {

localhost: {

QuadraticFundingVotingStrategyFactory: require('../../../hardhat/deployments/localhost/QuadraticFundingVotingStrategyFactory.json'),

MerklePayoutStrategyFactory: require('../../../hardhat/deployments/localhost/MerklePayoutStrategyFactory.json'),

},

// Add other networks as needed

};

  

const getABI = (networkName: string, contractName: string) => {

if (!preloadedABIs[networkName] || !preloadedABIs[networkName][contractName]) {

throw new Error(`ABI for ${contractName} on network ${networkName} not found`);

}

return preloadedABIs[networkName][contractName];

};

  

const CreateRoundForm = () => {

const [ownerAddress, setOwnerAddress] = useState('');

const [matchAmount, setMatchAmount] = useState('');

const [token, setToken] = useState(AddressZero);

const [roundFeePercentage, setRoundFeePercentage] = useState('');

const [roundFeeAddress, setRoundFeeAddress] = useState('');

const [applicationsStartTime, setApplicationsStartTime] = useState<Date | null>(null);

const [applicationsEndTime, setApplicationsEndTime] = useState<Date | null>(null);

const [roundStartTime, setRoundStartTime] = useState<Date | null>(null);

const [roundEndTime, setRoundEndTime] = useState<Date | null>(null);

const [ownedBy, setOwnedBy] = useState(''); // New state for ownedBy address

const [networkName, setNetworkName] = useState<string | null>(null); // Updated type

const { writeContractAsync, isMining } = useScaffoldWriteContract('RoundFactory');

  

useEffect(() => {

const fetchOwnerAddress = async () => {

try {

const provider = new ethers.BrowserProvider((window as any).ethereum);

const signer = await provider.getSigner();

const address = await signer.getAddress();

setOwnerAddress(address);

  

const network = await provider.getNetwork();

const chainId = network.chainId.toString(); // Convert bigint to string

const networkName = chainIdToNetworkName[chainId] || 'localhost';

  

setNetworkName(networkName);

} catch (error) {

console.error("Failed to get owner address or network ID:", error);

}

};

  

setRoundFeePercentage(process.env.ROUND_FEE_PERCENTAGE || '2');

setRoundFeeAddress(process.env.ROUND_FEE_ADDRESS || '0x891D071510EdAC606519237f88C2a9F531fbFAbE');

  

fetchOwnerAddress();

}, []);

  

const uploadToIPFS = async (metadata: any) => {

try {

const result = await ipfs.add(JSON.stringify(metadata));

return result.path; // This is the IPFS hash (CID)

} catch (error) {

console.error('Failed to upload to IPFS', error);

throw error;

}

};

  

const handleCreateRound = async (e: React.FormEvent) => {

e.preventDefault();

try {

// Validate addresses

const validateAddress = (address: string) => {

if (!isAddress(address)) {

throw new Error(`Invalid address: ${address}`);

}

return getAddress(address);

};

  

if (!networkName || networkName === 'unknown') {

throw new Error('Network name not detected');

}

  

const votingContract = getABI(networkName, 'QuadraticFundingVotingStrategyFactory');

const payoutContract = getABI(networkName, 'MerklePayoutStrategyFactory');

  

const initAddress = {

votingStrategy: validateAddress(votingContract.address),

payoutStrategy: validateAddress(payoutContract.address),

};

  

const initRoundTime = {

applicationsStartTime: applicationsStartTime ? Math.floor(applicationsStartTime.getTime() / 1000) : 0,

applicationsEndTime: applicationsEndTime ? Math.floor(applicationsEndTime.getTime() / 1000) : 0,

roundStartTime: roundStartTime ? Math.floor(roundStartTime.getTime() / 1000) : 0,

roundEndTime: roundEndTime ? Math.floor(roundEndTime.getTime() / 1000) : 0,

};

  

// Generate metadata content

const roundMetadata = {

name: 'Grant Round', // Example metadata

description: 'This is a grant round for funding projects', // Example metadata

// Add other metadata fields as needed

};

  

const applicationMetadata = {

name: 'Application', // Example metadata

description: 'This is an application for the grant round', // Example metadata

// Add other metadata fields as needed

};

  

// Upload metadata to IPFS

const roundMetaPtrCID = await uploadToIPFS(roundMetadata);

const applicationMetaPtrCID = await uploadToIPFS(applicationMetadata);

  

const initMetaPtr = [

{ protocol: 1, pointer: roundMetaPtrCID }, // Protocol 1 for IPFS

{ protocol: 1, pointer: applicationMetaPtrCID }, // Protocol 1 for IPFS

];

  

const initRoles = {

adminRoles: [validateAddress(ownerAddress)],

roundOperators: [validateAddress(ownerAddress)],

};

  

const matchAmountParsed = parseUnits(matchAmount, 18);

  

const params = [

initAddress,

initRoundTime,

matchAmountParsed,

validateAddress(token),

parseInt(roundFeePercentage),

validateAddress(roundFeeAddress),

initMetaPtr,

initRoles,

];

  

const encodedParameters = encodeRoundParameters(params);

  

await writeContractAsync({

functionName: 'create',

args: [`0x${hexlify(encodedParameters)}`, validateAddress(ownedBy)], // Ensure hex format

});

  

notification.success('Round created successfully');

} catch (error: any) {

notification.error(`Failed to create round: ${error.message}`);

}

};

  

return (

<form onSubmit={handleCreateRound} className="space-y-6">

<h1 className="text-2xl font-bold mb-6 text-center text-black">Create a Grant Round</h1>

<div>

<label htmlFor="matchAmount" className="block text-sm font-medium text-gray-700">

Match Amount

</label>

<input

id="matchAmount"

type="text"

value={matchAmount}

onChange={(e) => setMatchAmount(e.target.value)}

required

className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

<div>

<label htmlFor="token" className="block text-sm font-medium text-gray-700">

Token Address (use 0x000...000 for ETH)

</label>

<input

id="token"

type="text"

value={token}

onChange={(e) => setToken(e.target.value)}

required

className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

<div>

<label htmlFor="applicationsStartTime" className="block text-sm font-medium text-gray-700">

Applications Start Time

</label>

<div className="mt-1">

<DatePicker

selected={applicationsStartTime}

onChange={(date) => setApplicationsStartTime(date)}

showTimeSelect

dateFormat="Pp"

className="custom-datepicker-input mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

</div>

<div>

<label htmlFor="applicationsEndTime" className="block text-sm font-medium text-gray-700">

Applications End Time

</label>

<div className="mt-1">

<DatePicker

selected={applicationsEndTime}

onChange={(date) => setApplicationsEndTime(date)}

showTimeSelect

dateFormat="Pp"

className="custom-datepicker-input mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

</div>

<div>

<label htmlFor="roundStartTime" className="block text-sm font-medium text-gray-700">

Round Start Time

</label>

<div className="mt-1">

<DatePicker

selected={roundStartTime}

onChange={(date) => setRoundStartTime(date)}

showTimeSelect

dateFormat="Pp"

className="custom-datepicker-input mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

</div>

<div>

<label htmlFor="roundEndTime" className="block text-sm font-medium text-gray-700">

Round End Time

</label>

<div className="mt-1">

<DatePicker

selected={roundEndTime}

onChange={(date) => setRoundEndTime(date)}

showTimeSelect

dateFormat="Pp"

className="custom-datepicker-input mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

</div>

<div>

<label htmlFor="ownedBy" className="block text-sm font-medium text-gray-700">

Owned By Address

</label>

<input

id="ownedBy"

type="text"

value={ownedBy}

onChange={(e) => setOwnedBy(e.target.value)}

required

className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"

/>

</div>

<div className="flex justify-center">

<button

type="submit"

disabled={isMining}

className="w-full max-w-xs py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"

>

{isMining ? 'Creating Round...' : 'Create Round'}

</button>

</div>

</form>

);

};

  

export default CreateRoundForm;
```

/nextjs/app/grants/page.tsx

```
"use client";

import Head from 'next/head';

import Link from 'next/link';

  

const GrantsPage = () => {

return (

<div className="min-h-screen bg-gray-100 flex flex-col items-center justify-center">

<Head>

<title>Grant Rounds</title>

</Head>

<div className="bg-white shadow-md rounded-lg p-10 max-w-lg w-full text-center">

<h1 className="text-3xl font-extrabold mb-8 text-indigo-600">Rounds</h1>

<div>

<Link href="/grants/create" passHref>

<button className="w-full mb-6 py-3 px-6 border border-transparent rounded-lg shadow-lg text-lg font-semibold text-white bg-indigo-600 hover:bg-indigo-700 hover:shadow-xl focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transform transition-transform duration-200 hover:-translate-y-1 active:translate-y-0">

Create a Grant Round

</button>

</Link>

<Link href="/grants/manage" passHref>

<button className="w-full mb-6 py-3 px-6 border border-transparent rounded-lg shadow-lg text-lg font-semibold text-white bg-indigo-600 hover:bg-indigo-700 hover:shadow-xl focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transform transition-transform duration-200 hover:-translate-y-1 active:translate-y-0">

Manage Your Grant Rounds

</button>

</Link>

<Link href="/grants/explore" passHref>

<button className="w-full py-3 px-6 border border-transparent rounded-lg shadow-lg text-lg font-semibold text-white bg-indigo-600 hover:bg-indigo-700 hover:shadow-xl focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transform transition-transform duration-200 hover:-translate-y-1 active:translate-y-0">

Explore Grant Rounds

</button>

</Link>

</div>

</div>

</div>

);

}

  

export default GrantsPage;
```


packages/nextjs/app/grants/create/page.tsx

```
"use client";

import Head from 'next/head';

import CreateRoundForm from '../../../components/allo/CreateGrantRound';

  

function CreateProjectPage() {

return (

<div className="min-h-screen flex items-center justify-center">

<Head>

<title>Create a Grant Round</title>

</Head>

<div className="bg-white shadow-md rounded-lg p-8 max-w-lg w-full">

<h1 className="text-2xl font-bold mb-6 text-center text-black">Create a Grant Round</h1>

<CreateRoundForm />

</div>

</div>

);

}

  

export default CreateProjectPage;
```


nextjs/.env

```
# Template for NextJS environment variables.

  

# For local development, copy this file, rename it to .env.local, and fill in the values.

# When deploying live, you'll need to store the vars in Vercel/System config.

  

# If not set, we provide default values (check `scaffold.config.ts`) so developers can start prototyping out of the box,

# but we recommend getting your own API Keys for Production Apps.

  

# To access the values stored in this env file you can use: process.env.VARIABLENAME

# You'll need to prefix the variables names with NEXT_PUBLIC_ if you want to access them on the client side.

# More info: https://nextjs.org/docs/pages/building-your-application/configuring/environment-variables

NEXT_PUBLIC_INFURA_PROJECT_ID=b6e36f7cc1e4439c8eef67e17f43b03d

NEXT_PUBLIC_INFURA_API_KEY=O4ayb1nTZ6LHtvWRM+ZtSQU6rDmvi/uiTW83xMpYxVtv1knncPU6Ew

NEXT_PUBLIC_ROUND_FEE_PERCENTAGE=2

NEXT_PUBLIC_ROUND_FEE_ADDRESS=0x891D071510EdAC606519237f88C2a9F531fbFAbE
```

next.config.js

```
const NodePolyfillPlugin = require('node-polyfill-webpack-plugin');

  

/** @type {import('next').NextConfig} */

const nextConfig = {

reactStrictMode: true,

typescript: {

ignoreBuildErrors: process.env.NEXT_PUBLIC_IGNORE_BUILD_ERROR === "true",

},

eslint: {

ignoreDuringBuilds: process.env.NEXT_PUBLIC_IGNORE_BUILD_ERROR === "true",

},

webpack: (config, { isServer }) => {

// Externals to be excluded from the client-side bundle

if (!isServer) {

config.externals.push({

'fsevents': "require('fsevents')",

'chokidar': "require('chokidar')",

});

}

  

// Add NodePolyfillPlugin for client-side

if (!isServer) {

config.plugins.push(new NodePolyfillPlugin());

}

  

// Exclude .node files

config.module.rules.push({

test: /\.node$/,

use: 'ignore-loader',

});

  

return config;

},

};

  

module.exports = nextConfig;
```

Added react_datepicker, date-fns, @types/react-datepicker,

ipfs-http-client

Added ignore-loader, node-polyfill-webpack-plugin libraries


/grants

/grants/create
/grants/manage
/grants/explore

/projects

/projects/create
/projects/manage
/projects/explore


Pinata

```
import pinataSDK from '@pinata/sdk'; const pinata = pinataSDK('yourPinataApiKey', 'yourPinataSecretApiKey'); const uploadToIPFS = async (metadata) => { const result = await pinata.pinJSONToIPFS(metadata); return result.IpfsHash; };
```


// const projectId = "b6e36f7cc1e4439c8eef67e17f43b03d";

// const apiKey = "O4ayb1nTZ6LHtvWRM+ZtSQU6rDmvi/uiTW83xMpYxVtv1knncPU6Ew";

// const auth = 'Basic ' + Buffer.from(projectId + ':' + apiKey).toString('base64');


- do i create the server in my hardhat folder?
- do i create a route in my frontend folder?
- where should i store the api key and secret?


it's saying to install - `npm install express cors axios`
then to create a server.js file in hardhat, then run the server, then allow for next to call that and store the .env file in my 