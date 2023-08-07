## A practical example on how to perform V2/V3 sandwich attacks written using Rust and Huff.
## [`Artemis`](https://github.com/paradigmxyz/artemis) framework.

## Brief Explanation
Anytime that a transaction interacts with a Uniswap V2/V3 pool and its forks, there is some slippage introduced (routers, aggregators, other MEV bots). Sandwich bots, like this one, are a toxic form of MEV as they profit off this slippage by frontrunning the transaction pushing the price of an asset up to the slippage limit, and then immediately selling the asset through a backrun transaction.

**Bot Logic Breakdown** can be found under [bot/README.md]
**Contract Logic Breakdown** can be found under [contract/README.md]

## Features
- **Fully Generalized**: Sandwich any tx that introduces slippage.
- **V2 and V3 Logic**: Logic to handle Uniswap V2/V3 pools.
- **Multi-Meat**: Build and send multi-meat sandwiches.
- **Gas Optimized**: Contract written in Huff using unconventional gas optimizations.
- **Local Simulations**: Fast concurrent EVM simulations to find sandwich opportunities. 
- **Token Dust**: Stores dust at the end of every bundle for lower gas usage the next time the token is traded. 
- **Salmonella Checks**: Detect if erc20's transfer function uses any unusual opcodes that may produce different mainnet results.

## Notice
I'm just trying to get rich.
