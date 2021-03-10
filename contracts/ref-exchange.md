# Ref Exchange

The Exchange contract is the main contract facilitating the trading. All pools are managed inside one contract, allowing for atomic trades between them.

There are next main interactions:
 - Create a new pool with set of tokens
    - Currently there is only equal weight constant product pool, called `simple_pool`. It supports up to 10 tokens.
    - More pool types will be added over time via governance
 - Call to register user's account with the exchange.
 - Deposit funds into the contract -- as a user you first need to deposit the funds into the contracts.
    - User can have up to 10 different token balances (THIS IS GOING TO CHANGE)
 - Deposited funds can be put as liquidity into one of the pools.
 - Depoisted funds can be used to swap, including multihop between multiple pools.
 - Withdraw funds back to the user's account from the contract.

## Register or deposit more storage for user's account

When interacting with the contract first time, user must call `storage_deposit` with attached $NEAR to register the account.
Later, user can keep adding more storage to have more tokens in the portfolio.
This $NEAR can be retrieved back later, if user withdraws all it's deposits and can unregister.

```
    #[payable]
    fn storage_deposit(
        &mut self,
        account_id: Option<ValidAccountId>,
        registration_only: Option<bool>,
    ) -> StorageBalance;
```

Notes:
 - `registration_only` is unused and can be always `None`.

## Deposit

To deposit funds, user should send the token to the contract via callback.

```
token.ft_transfer_call(<exchange contract>, <amount>, "");
```

Notes:
- make sure that `exchange contract` account has storage deposit in `token`. user code can call the `token.storage_deposit(Some(<exchange contract>))` covering 128 bytes of storage.
- `msg` must be empty.

This will call `ft_on_transfer` callback which will record the deposits.

## Withdraw

To withdraw funds, the request is made per token with required amount.

```
pub fn withdraw(&mut self, token_id: AccountId, amount: U128);
```

## Add simple pool

To create "Simple pool" (equal weighted constant product), need to specify list of tokens (up to 10).
Must attach enough $NEAR to cover storage for the new pool (~300 bytes).

```
#[payable]
pub fn add_simple_pool(&mut self, tokens: Vec<ValidAccountId>, fee: u32) -> u32;
```

## Add liquidity

Add given amounts from depoisted into given pool. Amounts map to the tokens in the given pool.

The amounts must be deposited before calling this contract first.
Fails if there is not enough tokens.

```
pub fn add_liquidity(&mut self, pool_id: u64, amounts: Vec<U128>);
```

## Remove liquidity

Remove liquidity from given pool. Pass how many shares to withdraw and `min_amounts` of tokens of given pool that should be received (to prevent frontrunning).

```
pub fn remove_liquidity(&mut self, pool_id: u64, shares: U128, min_amounts: Vec<U128>);
```

## Swap

Swap some token via set of swap actions to the outcome tokens.
Swap action represents some `amount` of `token_in` being exchanged to `token_out`.

```
pub struct SwapAction {
    /// Pool which should be used for swapping.
    pub pool_id: u64,
    /// Token to swap from.
    pub token_in: ValidAccountId,
    /// Amount to exchange.
    /// If amount_in is None, it will take amount_out from previous step.
    /// Will fail if amount_in is None on the first step.
    pub amount_in: Option<U128>,
    /// Token to swap into.
    pub token_out: ValidAccountId,
    /// Required minimum amount of token_out.
    pub min_amount_out: U128,
}
```

```
pub fn swap(&mut self, actions: Vec<SwapAction>) -> U128
```

## Get return

To compute how much exchanging over a single pool would return:

```
    pub fn get_return(
        &self,
        pool_id: u64,
        token_in: ValidAccountId,
        amount_in: U128,
        token_out: ValidAccountId,
    ) -> U128;
```

## Number of pools

```
pub fn get_number_of_pools(&self) -> u64;
```

## Get pools

Get `limit` number of pools with ids starting from given index.

```
pub fn get_pools(&self, from_index: u64, limit: u64) -> Vec<PoolInfo>
```

## Shares in the pool

Get number of shares that given user has in the pool.

```
pub fn get_pool_shares(&self, pool_id: u64, account_id: ValidAccountId) -> U128;
```

## Total number of shares in the pool

Get total number of shares in the pool.

```
pub fn get_pool_total_shares(&self, pool_id: u64) -> U128;
```

## Get users deposits

Get deposits of the user in the contract across all tokens.

```
pub fn get_deposits(&self, account_id: &AccountId) -> HashMap<AccountId, U128>;
```

