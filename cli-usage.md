# CLI Usage

Describes how to use the contract via CLI.

We will use `$CONTRACT_ID` for contract account id and `$USER_ID` for user account id.
`$TOKEN1`, `$TOKEN2`, etc are accounts for tokens.

You can set these variables via CLI: `export CONTRACT_ID=ref-finance.testnet`

# Deploy contract

Deploy to TestNet, to an account `$CONTRACT_ID` you have access keys for:

```
export NEAR_ENV=default
near deploy $CONTRACT_ID --wasmFile=res/ref_exchange.wasm
near call $CONTRACT_ID new "{\"owner_id\": \"$USER_ID\", \"exchange_fee\": 4, \"referral_fee\": 1}" --accountId $CONTRACT_ID
```

# Add a simple pool

Add simple pool with 2 tokens and 0.3% total fee (0.04% goes to exchange and 0.01% goes to referral).

```
near call $CONTRACT_ID add_simple_pool "{\"tokens\": [\"$TOKEN1\", \"$TOKEN2\"], \"fee\": 25}" --accountId $USER_ID --amount 0.1
```

# Query pools

To query first 10 pools:

```
near view $CONTRACT_ID get_pools '{"from_index": 0, "limit": 10}'
```

# Register account in the exchange

```
near call $CONTRACT_ID storage_deposit '' --accountId $USER_ID --amount 0.1
```

# Check that account is registered and storage available

```
near view $CONTRACT_ID storage_balance_of "{\"account_id\": \"$USER_ID\"}"
```

# Deposit funds

Before sending funds for token X, make sure that exchange is registered for token X.

```
near call $TOKEN1 storage_deposit "{\"account_id\": \"$CONTRACT_ID\"}" --accountId $USER_ID --amount 0.0125
```

Actually deposit funds to the exchange (attaching 1yN for security):

```
near call $TOKEN1 ft_transfer_call "{\"receiver_id\": \"$CONTRACT_ID\", \"amount\": \"1000000000000\", \"msg\": \"\"}" --accountId $USER_ID --amount 0.000000000000000000000001
```

# Query deposit balances in the exchange

```
near view $CONTRACT_ID get_deposits "{\"account_id\": \"$USER_ID\"}"
```

# Add liquidity to a pool

```
near call $CONTRACT_ID add_liquidity '{"pool_id": 0, "amounts": ["10000", "10000"]}' --accountId $USER_ID
```

# Get pool's information

```
near view $CONTRACT_ID get_pool '{"pool_id": 0}'
```

# Get number of liquidity shares in the pool

```
near view $CONTRACT_ID get_pool_shares "{\"pool_id\": 0, \"account_id\": \"$USER_ID\"}"
```

# Remove liquidity from a pool

```
near call $CONTRACT_ID remove_liquidity '{"pool_id": 0, "shares": "1000000000000000000000000", "min_amounts": ["1", "1"]}' --accountId $USER_ID
```

# Output amount after swap

```
near view $CONTRACT_ID get_return "{\"pool_id\": 0, \"token_in\": \"$TOKEN1\", \"amount_in\": \"10000\", \"token_out\": \"$TOKEN2\"}"
```

# Swap

Swap via a single pool:

```
near call $CONTRACT_ID swap "{\"actions\": [{\"pool_id\": 0, \"token_in\": \"$TOKEN1\", \"amount_in\": \"10000\", \"token_out\": \"$TOKEN2\", \"min_amount_out\": \"1\"}]}" --accountId $USER_ID
```

# Withdraw funds

To withdraw specific token from exchange back to user's account:

```
near call $CONTRACT_ID withdraw "{\"token_id\": \"$TOKEN1\", \"amount\": \"900000000000\"}" --accountId $USER_ID --amount 0.000000000000000000000001
```

# Check owner

```
near view $CONTRACT_ID get_owner
```