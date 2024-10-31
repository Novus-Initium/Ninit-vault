1. install necessary dependencies for Allo

npm install @openzeppelin/contracts-upgradeable
npm install solady

2. Add submodule to lib/superfluid-protocol-monorepo
git submodule add https://github.com/superfluid-finance/protocol-monorepo.git lib/superfluid-protocol-monorepo
git submodule update --init --recursive

add work space in package.json: 

"workspaces": {

"packages": [

"packages/hardhat",

"packages/nextjs",

"packages/hardhat/contracts/external/allo-v2"

]

},
