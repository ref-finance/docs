# dev guide for ref-farming

Here we using near-cli as tools to demonstrate how to interact with ref-farming contract. 

It's important for front-end developers to understand the interface logic from FE perspective.

## views
---
### global info
**contract metadata**
```bash

% near view $FARMING get_metadata
# return statics
{
  version: '0.4.2',
  owner_id: 'ref_finance_owner.testnet',
  farmer_count: '4',
  farm_count: '7',
  seed_count: '6',
  reward_count: '1'
}

```

**all farms with pagination**
```bash
near view $FARMING list_farms '{"from_index": 0, "limit": 100}'
# return list
[
  {
    farm_id: 'ref-finance.testnet@20#0',
    farm_kind: 'SIMPLE_FARM',
    farm_status: 'Ended',
    seed_id: 'ref-finance.testnet@20',
    reward_token: 'rft.tokenfactory.testnet',
    start_at: '51012253',
    reward_per_session: '10000000000',
    session_interval: '3600',
    total_reward: '6000000000000',
    cur_round: '600',
    last_round: '600',
    claimed_reward: '4490147960435',
    unclaimed_reward: '1509852039565'
  },
  ...
]
```
Note: 
* There are three `farm_status` in contract, they are Created, Running, Ended;
* `start_at` is in block height;
* `session_interval` is in block num;

**all seeds with pagination**
```bash
near view $FARMING list_seeds_info '{"from_index": 0, "limit": 100}'
# return dict
{
  'ref-finance.testnet@9': {
    seed_id: 'ref-finance.testnet@9',
    seed_type: 'MFT',
    farms: [
      'ref-finance.testnet@9#2',
      'ref-finance.testnet@9#0',
      'ref-finance.testnet@9#1'
    ],
    next_index: 3,
    amount: '0',
    min_deposit: '1000000000000000000'
  },
  ...
}
```

**all rewards with pagination**

```bash
near view $FARMING list_rewards_info '{"from_index": 0, "limit": 100}'
# return dict
{
  'rft.tokenfactory.testnet': '42012000000000',
  'token.ref-finance.testnet': '12000000000000000000000',
  'wrap.testnet': '200000000000000000000000000'
}
```

### per seed info

**get single seed info**
```bash
near view $FARMING get_seed_info '{"seed_id": "ref-finance.testnet@9"}'
# return dict
{
  seed_id: 'ref-finance.testnet@9',
  seed_type: 'MFT',
  farms: [
    'ref-finance.testnet@9#2',
    'ref-finance.testnet@9#0',
    'ref-finance.testnet@9#1'
  ],
  next_index: 3,
  amount: '0',
  min_deposit: '1000000000000000000'
}
```

**farms in a seed**
```bash
near view $FARMING list_farms_by_seed '{"seed_id": "ref-finance.testnet@9"}'
# return list
[
  {
    farm_id: 'ref-finance.testnet@9#2',
    farm_kind: 'SIMPLE_FARM',
    farm_status: 'Running',
    seed_id: 'ref-finance.testnet@9',
    reward_token: 'wrap.testnet',
    start_at: '55736403',
    reward_per_session: '1000000000000000000000000',
    session_interval: '3600',
    total_reward: '100000000000000000000000000',
    cur_round: '38',
    last_round: '0',
    claimed_reward: '0',
    unclaimed_reward: '38000000000000000000000000'
  },
  ...
]
```


### per farm info

**single farm info**

```bash
near view $FARMING get_farm '{"farm_id": "ref-finance.testnet@11#0"}'
# return dict
{
  farm_id: 'ref-finance.testnet@11#0',
  farm_kind: 'SIMPLE_FARM',
  farm_status: 'Running',
  seed_id: 'ref-finance.testnet@11',
  reward_token: 'token.ref-finance.testnet',
  start_at: '55735800',
  reward_per_session: '10000000000000000000',
  session_interval: '3600',
  total_reward: '2400000000000000000000',
  cur_round: '38',
  last_round: '0',
  claimed_reward: '0',
  unclaimed_reward: '380000000000000000000'
}
```

### per user

**list rewards**
```bash
near view $FARMING list_rewards '{"account_id": "pika456.testnet"}'
#return dict
{ 'rft.tokenfactory.testnet': '281999999999' }

near view $FARMING get_reward '{"account_id": "pika456.testnet", "token_id": "rft.tokenfactory.testnet"}'
#return number string
'281999999999'
```

**get unclaimed**
```bash
near view $FARMING get_unclaimed_reward '{"account_id": "pika456.testnet", "farm_id": "ref-finance.testnet@30#0"}'
#return number string
'2429999999999'
```

**list user seeds**
```bash
near view $FARMING list_user_seeds '{"account_id": "pika456.testnet"}'
#return dict
{
  'ref-finance.testnet@31': '1000000000000000000000000',
  'ref-finance.testnet@30': '1100000000000000000000000'
}
```

## stake/unstake seeds


## create farm