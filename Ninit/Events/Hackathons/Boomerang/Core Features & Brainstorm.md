

1. Every quarter, a round from RoundFactory.sol is created by the Boomerang round operator 
	1. The application period and round period will last the length of each quarter
	3. Organizations who create a profile on ProjectRegistry.sol can apply for the funds that are distributed for that quarter
	4. Boomerang round operator accepts applications on a rolling basis
2. Throughout the quarter, users gamble, when they win they take from the Treasury, when they lose their loss goes to the Treasury.
3. 2 weeks before the round ends, the losers get to vote their losses to the Projects they care about (how do they vote if their losses go to the treasury?)
4. At the end the QF round gets distributed to the public goods projects


Trying to figure out the payout strategy for allo protocol, it seems a lil complicated so i wonder if we just use our own payout function where we pull matching pool funds from the round operator and calculate the distributions from the votes on a project through events.

Steps to take:

- Create a component that queries the ended rounds of the Round Manager
- Retrieve Vote event data for all projects in a that ended round
- Calculate the distributions through QF
- Distribute the amounts based on the votes


```
"axios": "^1.7.2",
"react-datepicker": "^4.8.0",
"@types/react-datepicker": "^6.2.0"
```
