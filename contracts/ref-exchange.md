# Ref Exchange

The Exchange contract is the main contract facilitating the trading. All pools are managed inside one contract, allowing for atomic trades between them.

There are next main interactions:
 - Create a new pool with set of tokens
    - Currently there is only equal weight constant product of 2 tokens, called `simple_pool`.
    - More pool types will be added over time via governance.
 - Call to register user's account with the exchange.
 - Deposit funds into the contract -- as a user you first need to deposit the funds into the contracts. There is a list of globally whitelisted tokens, or user can also separately register for tokens if they are not whitelisted (this is done to avoid malicious token contracts filling user's space).
 - Deposited funds can be put as liquidity into one of the pools.
 - Depoisted funds can be used to swap, including multihop swap between multiple pools.
 - Withdraw funds back to the user's account from the contract.

## Constructor

Create new pool with given `owner_id` account that will have priviliaged access (like upgrades and whitelists) and `exchange_fee` in bps and `referral_fee` in bps that will be charged from pools on top of their LP pools.

```
    #[init]
    pub fn new(owner_id: ValidAccountId, exchange_fee: u32, referral_fee: u32) -> Self;
```

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
 - `registration_only` if passed, only attached $NEAR to cover minimum balance for account creation. The rest gets refunded.
 - stores the amount of $NEAR passed at deposit to cover future deposits usage.

## Register tokens for given user

There is global whitelist, but users can register tokens they want to use separately. This makes sure that malicious token contracts can't spam user's storage.

```
    /// Registers given token in the user's account deposit.
    /// Fails if not enough balance on this account to cover storage.
    pub fn register_tokens(&mut self, token_ids: Vec<ValidAccountId>);
```

## Unregister token for given user

```
    /// Unregister given token from user's account deposit.
    /// Fails if the balance of any given token is non 0.
    pub fn unregister_tokens(&mut self, token_ids: Vec<ValidAccountId>);
```

## Deposit

To deposit funds, user should send the token to the contract via callback.

```
token.ft_transfer_call(<exchange contract>, <amount>, "");
```

Notes:
- make sure that `exchange contract` account has storage deposit in `token`. User code can call the `token.storage_deposit(Some(<exchange contract>))` covering exactly 125 bytes of storage.
- `msg` must be empty.

This will call `ft_on_transfer` callback which will record the deposits.

If token is not in the global whitelist or user haven't registered this token for themself - this will fail.

## Withdraw

To withdraw funds, the request is made per token with required amount.

```
    /// Withdraws given token from the deposits of given user.
    /// Optional unregister will try to remove record of this token from AccountDeposit for given user.
    /// Unregister will fail if the left over balance is non 0.
    #[payable]
    pub fn withdraw(&mut self, token_id: ValidAccountId, amount: U128, unregister: Option<bool>);
```

Notes:
 - if `unregister` is `true` and `amount` is full amount on the account of the given user, can automatically call `unregister_tokens` on `token_id`. Allowing to free up some of the $NEAR locked for storage for this token.

## Add simple pool

To create "Simple pool" (equal weighted constant product), need to specify list of tokens (up to 10).
Must attach enough $NEAR to cover storage for the new pool (~300 bytes).

```
#[payable]
pub fn add_simple_pool(&mut self, tokens: Vec<ValidAccountId>, fee: u32) -> u32;
```

## Add liquidity

Add given amounts from deposited into given pool. Amounts map to the tokens in the given pool.

The amounts must be deposited before calling this contract first.
Fails if there is not enough tokens.

```
pub fn add_liquidity(&mut self, pool_id: u64, amounts: Vec<U128>);
```

## Remove liquidity

Remove liquidity from given pool. Pass how many shares to withdraw and `min_amounts` of tokens of given pool that should be received (to prevent front running).

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
pub fn swap(&mut self, actions: Vec<SwapAction>, referral_id: Option<ValidAccountId>) -> U128
```

Notes:
 - `referral_id` is optional argument that allows frontends to get paid for facilitating transactions. If not provided, the `referral_fee` doesn't get paid out and goes to LPs.

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

## Get global token whitelist

```
pub fn get_whitelisted_tokens(&self) -> Vec<AccountId>;
```

## Get user token whitelist

Returns tokens that are specifically allowed for given user (doesn't include global whitelist).

```
pub fn get_user_whitelisted_tokens(&self, account_id: &AccountId);
```

## Get owner

```
/// Get the owner of this account.
pub fn get_owner(&self);
```
