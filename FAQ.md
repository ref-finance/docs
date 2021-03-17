# FAQ

### Why not combine swap and fungible token transfer into one action?

Because NEAR is an async system, any cross contract call will take extra block to process.

Let review an example where User A swaps Token X to AMM in async environment:
 1. Token X to AMM transfer takes first block
 2. AMM swap to Token B and scheduling call to transfer token Y from AMM to the user
 3. Token Y gets transferred... OR it fails due to lack of registration of User A in Token Y
 4. Callback AMM to indicate if it was successful or not
 5. If it was unsuccessful, need to return User A their token X

 Right when step 1 happen everyone observed that User A wants to swap given amount of token and can try to front run their AMM swap to make some money.

 Also users can maliciously do swap that will bounce from token Y - to change the price of the swap for 1 blocks. Which means that others will get dramatically different price at no cost to the attacker (their full funds must return).

Even harder if user want to swap between token A -> token B -> token C. This will be even more complex and include even more vectors of attack in the middle.

Instead current approach allows to deposit funds first, do all the trading, including multi-hop swap safely and then safely withdraw.

This also means that expanding one contract with logic of CDPs, leverage and anything else becomes easier as well.

