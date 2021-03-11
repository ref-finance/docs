# CLI Usage

Describes how to use the contract via CLI.

# Deploy contract

TestNet:

> near deploy ref-finance.testnet --wasmFile=res/ref_exchange.wasm
> near call ref-finance.testnet new '' --accountId ref-finance.testnet

# Add a simple pool

Add simple pool with 2 tokens and 0.3% fee.

> near call ref-finance.testnet add_simple_pool '{"tokens": ["token1.ref-finance.testnet", "token2.ref-finance.testnet"], "fee": 30}' --accountId testmewell.testnet --amount 0.1

# Query pools

To query first 10 pools:

> near view ref-finance.testnet get_pools '{"from_index": 0, "limit": 10}'

# Register account in the exchange

> near call ref-finance.testnet storage_deposit '' --accountId testmewell.testnet --amount 0.1

# Deposit funds

Before sending funds for token X, make sure that exchange is registered for token X.

> near call token1.ref-finance.testnet storage_deposit '{"account_id": "ref-finance.testnet"}' --accountId testmewell.testnet --amount 0.0125

Actually deposit funds to the exchange (attaching 1yN for security):

> near call token1.ref-finance.testnet ft_transfer_call '{"receiver_id": "ref-finance.testnet", "amount": "1000000000000", "msg": ""}' --accountId testmewell.testnet --amount 0.000000000000000000000001

# Query deposit balances in the exchange

> near view ref-finance.testnet get_deposits '{"account_id": "testmewell.testnet"}'

# Add liquidity to a pool

> near call ref-finance.testnet add_liquidity '{"pool_id": 0, "amounts": ["10000", "10000"]}' --accountId testmewell.testnet

# Get pool's information

> near view ref-finance.testnet get_pool '{"pool_id": 0}'

# Get number of liquidity shares in the pool

> near view ref-finance.testnet get_pool_shares '{"pool_id": 0, "account_id": "testmewell.testnet"}'

# Remove liquidity from a pool

> near call ref-finance.testnet remove_liquidity '{"pool_id": 0, "shares": "1000000000000000000000000", "min_amounts": ["1", "1"]}' --accountId testmewell.testnet

# Output amount after swap

> near view ref-finance.testnet get_return '{"pool_id": 0, "token_in": "token1.ref-finance.testnet", "amount_in": "10000", "token_out": "token2.ref-finance.testnet"}'

# Swap

Swap via a single pool:

> near call ref-finance.testnet swap '{"actions": [{"pool_id": 0, "token_in": "token1.ref-finance.testnet", "amount_in": "10000", "token_out": "token2.ref-finance.testnet", "min_amount_out": "1"}]}' --accountId testmewell.testnet
