# Ref Farming
The farming contract is an independent lock&mining contract. It was built to support liquidity mining that the LPs of Ref Exchange may need. An outstanding feature of this farming contract is that it supports multiple mining rewards for one farming token (the farming token is also called seed in this contract).  

## Terminology

|word|meaning|notes|
|-|-|-|
|Seed|Farming-Token|User stakes seed to this contract for various rewards token back|
|SeedId|Id of seed, String|Token contract_id for ft token, token contract_id + "@" + inner_id for mft token|
|Farm||A place to get reward according to farmer's seed|
|FarmId|Id of farm, String|SeedId + "#" + farm_index in that seed|
|Farmer||Those who deposit seed to this contract|
|RPS|Reward-Per-Seed|The key concept to distribute rewards between farmers in a farm|
|RR|Reward Round in block num|the reward are released by round|

## Basic Logic

### Core Concept
---
**Farmer** deposit/stake **Seed** token to farming on all **Farm** that accept that seed, and gains **Reward** token back. different farm can (not must) grows different reward token.  

* A **Farmer** can deposit multiple **Seed**, 
* A **Seed** can be accepted by multiple **Farm**, 
* A **Farm** can only give out one type of **Reward**,
* **Reward** of different **Farm** can be identical or not,

### The Farm
---
To create a farm, we need deliver this info:  
```rust
pub struct HRSimpleFarmTerms {
    pub seed_id: SeedId,  // the seed this farm accepts.
    pub reward_token: ValidAccountId,  // the reward token contract_id.
    pub start_at: U64,  // the block height this farm plans to start running
    pub reward_per_session: U128,  // reward is released session by session
    pub session_interval: U64,  // session duration in block heights
}
#[payable]
pub fn create_simple_farm(&mut self, terms: HRSimpleFarmTerms) -> FarmId;
```

Then, deposit reward token to this farm using reward token's ft_transfer_call interface:  
```shell
token.ft_transfer_call("receiver_id": "<farming contract>", "amount": "<amount>", "msg": "<farm_id>");
```

A farm may have following status:
```rust
pub enum SimpleFarmStatus {
    Created,  // the init status of a farm
    Running,  // when start_at has passed and at least one shot of reward token has been deposited.
    Ended,  // when all reward token has been distributed.
    Cleared,  // when all reward token has been claimed out by farmers.
}
```

To distribute reward, a farm updates following info as latest status after each distribution:
```rust
pub struct SimpleFarmRewardDistribution {
    /// unreleased reward
    pub undistributed: Balance,
    /// the total rewards distributed but not yet claimed by farmers.
    pub unclaimed: Balance,
    /// Reward_Per_Seed
    /// rps(cur) = rps(prev) + distributing_reward / total_seed_staked
    pub rps: RPS,
    /// Reward_Round
    /// rr = (cur_block_height - start_at) / session_interval
    pub rr: u64,
}
```


## The Organization of Farm
`Farm` is organized by `Seed`:
```rust
pub struct FarmSeed {
    /// The Farming Token this FarmSeed represented for
    pub seed_id: SeedId,
    /// The seed is a FT or MFT
    pub seed_type: SeedType,
    /// all farms that accepted this seed
    /// Future Work: may change to HashMap<GlobalIndex, Farm> 
    /// to enable whole life-circle (especially for removing of farm). 
    pub farms: Vec<Farm>,
    /// total (staked) balance of this seed (Farming Token)
    pub amount: Balance,
}
```

## The Farmer
For a `farmer`, there are 3 main attributes:
* the `reward` he claims,
* the `seed` he deposits/stakes,
* the `user_rps` for each farm he involves.

```rust
pub struct Farmer {
    /// Native NEAR amount sent to this contract.
    /// Used for storage.
    pub amount: Balance,
    /// Amounts of various reward tokens the farmer claimed.
    pub rewards: HashMap<AccountId, Balance>,
    /// Amounts of various seed tokens the farmer staked.
    pub seeds: HashMap<SeedId, Balance>,
    /// record user_last_rps of farms
    pub user_rps: HashMap<FarmId, RPS>,
}
```

## Reward Distribution Logic
As designed that way, we can calculate farmers unclaimed reward like this pseudo code:  

```rust
// 1. get current reward round CRR
let crr = (env::block_index() - self.terms.start_at) / self.terms.session_interval;
// 2. get reward to distribute this time
let reward_added = (crr - self.last_distribution.rr) as u128 * self.terms.reward_per_session;
// 3. get current RPS
let crps = self.last_distribution.rps + reward_added / total_seeds;
// 4. get user unclaimed by multiple user_staked_seed with rps diff.
let unclaimed_reward = user_staked_seed * (crps - user_last_rps);
```
This logic is sealed in 
```rust
pub(crate) fn view_farmer_unclaimed_reward(
        &self,
        user_rps: &RPS,
        user_seeds: &Balance,
        total_seeds: &Balance,
    ) -> Balance
```
which, based on 
```rust
pub(crate) fn try_distribute(&self, total_seeds: &Balance) -> Option<SimpleFarmRewardDistribution>
```
to calculate cur RPS and RR of the farm without modifying the storage (means not really update the farm)

And when farmer actually claims his reward, the whole logic is sealed in 
```rust
pub(crate) fn claim_user_reward(
        &mut self, 
        user_rps: &RPS,
        user_seeds: &Balance, 
        total_seeds: &Balance
    ) -> Option<(Balance, Balance)>
```
which, based on 
```rust
pub(crate) fn distribute(&mut self, total_seeds: &Balance)
```
to calculate and update the farm.

### When to update the farm  
---

It's worth to noticed, 
each time a farmer deposit seed, withdraw seed, claim reward,  
All relevant farm woulds be invoke with distribute to update themselves, and  
furthermore, in deposit and withdraw seed actions, the farmer's claim_reward action would be automatically invoked to keep this RPS logic correctly.

## The Storage Fee
---
As each ```farmer``` would have a place to record his ```user_rps``` in each ```farm``` he involved, the storage belongs to a farmer may increase out of his notice.  

For example, when a new farm establishes and starts to run, which accepts a farmer's seed that has been staked in the contract, then at the action such as claim_reward, or deposit/withdraw seeds invoked by the farmer, his storage would expand to record the new rps related to that farm.  

Consider that, and also to improve farmer's user-experience, we have a `suggested_min_storage_usage()` which covers 5 `seed`, 5 `reward` and 10 `farm` as one shot. When farmer registers for the first time, we will force him to deposit more or equal to that amount, which is about 1,852 bytes, 0.01852 near. 

And when a farmer owes storage fee, then before he storage_deposit more fee,  
all changeable method would fail with ERR11_INSUFFICIENT_STORAGE.

