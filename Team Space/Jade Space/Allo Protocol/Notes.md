
## I. [Pool](https://docs.allo.gitcoin.co/overview/pool)

- Distributes funds in pools, where each fund is owned by a profile in the #Registry and has an #AllocationStrategy
- Dependencies:
	- #Registry
	- #AllocationStrategy

---
## II. [Allocation Strategies](https://docs.allo.gitcoin.co/overview/allocation-strategy)

- Allocation Strategies are custom contracts that allow how a #Pool is distributed.

#### A. Components of AllocationStrategy

1. Recipients - Who is eligible to receive funding from the #Pool
	1. Grantees apply and are approved by admins
2. Allocation - who is eligible to decide on how the funds should be allocated and how will they express their opinion (grants, committee, token voting, SBTs)
	1. People donate to projects, casting votes for how much matching pool project should receive
3. Payouts - Based on the allocation inputs, how will payouts be determined?
	1. Quadratic Funding
4. Distribution (lump sum, milestone based, stream)
	1. Donations and lump sum: projects receive individual donations as they come, then receive a lump sum payment from the pool
#### B. Allocators

- Allocators are wallet addresses that use the allocate function
- Each strategy will need to determine what makes a wallet eligible to be an allocator
- Considerations:
	- Is there a certain time period allocation can occur?
	- Can an allocator allocate multiple times?
	- Does allocation require funds to be sent, if so where are they stored?
#### i. [Examples](https://docs.allo.gitcoin.co/strategies/allocation#examples)

|Strategy Contract|Allocate Description|Eligibility|Allocation Window|Multiple Allocations|Funds Transferred?|
|---|---|---|---|---|---|
|[Donation Voting(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/donation-voting)|Allocators donate directly to their chosen recipient|All addresses eligible|Pool manager sets window|Multiple allocations allowed|Yes, must be from token allowlist. Funds stored on strategy contract.|
|[Direct Grants Simple(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/direct-grants-simple)|Pool managers allocate grant money to accepted recipients|Only pool managers are eligible|No set allocation window|Multiple allocations allowed|Funds are not transferred on allocate|
|[RFP Committee(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/rfp-committee)|Committee members use allocate to vote on a recipient, one recipient wins by majority|Only committee members are eligible|No set allocation window|Only one allocation allowed per member, but they can change their vote|Funds are not transferred on allocate|

#### C. Distribution

###### i. Management & Permissions

- Strategies need to determine how payout are handled for recipients
- Distribution is resonsible for:
	- Calculating payout amounts
	- Determining when and how often payouts occur
	- Ensuring payout eligibility and accuracy
	- Managing who can initiate a payout

###### ii. Implementation

- Distribution strategies have flexibility in how funds get distributed from a pool:
	- Sending payout as lump sum
	- Creating set of milestones for when funds can be requested by recipients
	- Turning on stream with Sablier or Superfluid
- Payouts will be implemented by calling distribute() on #Allo contract
- Allo allows for experimenting with different distribution mechanisms (see examples above under Allocators)

---
## III. [Project Registry](https://docs.allo.gitcoin.co/overview/project-registry)

- On chain registry of projects used for reputation and eligibility
- once a project is registered, they can create pool of funding tied to that profile, or apply to receive funding from another pool in Allo
- Profiles can be like organizations and can distribute/receive funding
#### A. Project Profiles

- Project registrations are given a unique address to represent itself this address can be used to:
	- receive funding distributed through an allocation strategy
	- be the subject of an attestation (attesting to project managers having been kyc'ed)
	- hold a soul bound token demonstrating that the project is a member of a larger community
	- **allocation strategies can include automated elgibility checks**

#### B. Recipients

- A recipient is a group/individual eligible for allocation of a pool
- Recipient management is determined by the customizable #AllocationStrategy of the contract

**Example Strategy May Include**

- Application/review, recipients must apply have their application reviewed and approved by pool manager
- Token requirements - implementation of registerRecipient can check for the balance of a particular token (ERC20, ERC721, soul bound token)
- Automated elgibility checks - check recipient for token, is listed in curated registry of projects, holds a #Hypercert, has a particular #Attestation

---
## IV. Contracts

#### A. Allo.sol

- registerRecipient is located in Allo.sol which manages eligibility for recipients, where you also implement your strategy
- when desigining eligibility it is important to think through:
	- what data is needed to determine recipient eligibility and identify the recipient?
	- what are the eligibility requirements and how will it be determined that they are being met or not?
	- what does the registration process look like? are there any statuses a recipient would go through?

###### i. [Examples](https://docs.allo.gitcoin.co/strategies/recipients#examples)

|Strategy Contract|Eligibility Requirements|Identification Method|Who Can Use `registerRecipient`|Registration Process|
|---|---|---|---|---|
|[Donation Voting(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/donation-voting)|Pool manager may require recipients to be registred on Allo|Identified by Allo `anchorId` or wallet address|Prospective recipients use `registerRecipient` to complete an application|Pool managers review eligible applications and manually add them to the pool|
|[Direct Grants Simple(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/direct-grants-simple)|Pool manager may required recipients to be registred on Allo, may require additional information when applying|Identified by Allo `anchorId` or wallet address|Recipients use `registerRecipient` to be added to the pool|All eligible registrations are automatically added to the pool|
|[RFP Committee(opens in a new tab)](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies/rfp-committee)|Pool manager may required recipients to be registred on Allo, may require additional information when applying|Identified by Allo `anchorId` or wallet address|Prospective recipients use `registerRecipient` to register for the pool|All eligible registrations are automatically added to the pool|
#### B. [IStrategy interface](https://docs.allo.gitcoin.co/strategies/interface)

###### i. [Tutorial](https://docs.allo.gitcoin.co/strategies/writing-a-custom-strategy)

###### ii. Functions

 a. **initialize**

[`function initialize(uint256 _poolId, bytes memory _data) external`](https://docs.allo.gitcoin.co/strategies/interface#user-content-fn-1)

- used when createPool is called to set up the #AllocationStrategy
- function should emit #Initialized and #PoolActive event

###### [Parameters](https://docs.allo.gitcoin.co/strategies/interface#parameters)

| name    | type    | description                                                                              |
| ------- | ------- | ---------------------------------------------------------------------------------------- |
| _poolId | uint256 | poolId assigned by Allo when the pool is created                                         |
| _data   | bytes   | Any initial data required by the strategy. `BaseStrategy.sol` does not require any data. |
b. **registerRecipient**

[`function registerRecipient(bytes memory _data, address _sender) external payable returns (address)`](https://docs.allo.gitcoin.co/strategies/interface#user-content-fn-1)
- called by #Allo contract to add new recipients to the pool
- must return a #recipientId address
- should emit a #Registered event
###### Parameters

|name|type|description|
|---|---|---|
|_data|bytes|Any data required by the strategy to register the recipient.|
|_sender|address|Address that made the call to `Allo.sol`.|
c. [**allocate**](https://docs.allo.gitcoin.co/strategies/interface#allocate)

[`function allocate(bytes memory _data, address _sender) external payable`](https://docs.allo.gitcoin.co/strategies/interface#user-content-fn-1)

`allocate` is called by `Allo.sol` to make allocations to a recipient. This function should emit an `Allocated` event.
 
[Parameters](https://docs.allo.gitcoin.co/strategies/interface#parameters-2)

|name|type|description|
|---|---|---|
|_data|bytes|Any data required by the strategy to allocate to a recipient.|
|_sender|address|Address that made the call to `Allo.sol`.|

d. [**distribute**](https://docs.allo.gitcoin.co/strategies/interface#distribute)

[`function distribute(address[] memory _recipientIds, bytes memory _data, address _sender) external`](https://docs.allo.gitcoin.co/strategies/interface#user-content-fn-1)

`distribute` is used to distribute tokens to recipients. This function should emit a `Distributed` event.

[Parameters](https://docs.allo.gitcoin.co/strategies/interface#parameters-3)

|name|type|description|
|---|---|---|
|_recipientIds|address[]|Array of recipient addresses to distribute tokens to.|
|_data|bytes|Any data required by the strategy to distribute tokens.|
|_sender|address|Address that made the call to `Allo.sol`.|

e. [Views](https://docs.allo.gitcoin.co/strategies/interface#views)

`IStrategy.sol` exposes the following views:

|name|description|
|---|---|
|getAllo()|Returns the address of the Allo contract.|
|getPoolId()|Returns the PoolId for this strategy.|
|getStrategyId()|Returns the id of the strategy.|
|isValidAllocator(address _allocator)|Returns a boolean indicating whether provided address is a valid allocator.|
|isPoolActive()|Returns a boolean indicating whether pool is active.|
|getPoolAmount()|Returns the amount of funding in the pool.|
|getRecipientStatus()|Returns a the status of a recipient.|
|getPayouts(address[] memory _recipientIds, bytes[] memory _data)|Returns a PayoutSummary[] for the provided recipientIds.|
f. [Strategy Library](https://github.com/allo-protocol/allo-v2/tree/main/contracts/strategies)
