## Summary
Bot logic relies heavily on REVM simulations to detect sandwichable transactions. The simulations are done by injecting a modified router contract called [`crumbRouter.sol`] into a new EVM instance. Once injected, a concurrent binary search is performed to find an optimal input amount that results in the highest revenue. After sandwich calculations, the bot performs a [salmonella] check. If the sandwich is salmonella free, the bot then calculates gas bribes and sends the bundle to the fb relay. 

## Logic Breakdown
- At startup, index all pools from a specific factory by parsing the `PairCreated` event. And fetch all token dust stored on crumb addy.
- Read and decode tx from mempool.
- Send tx to [`trace_call`] to obtain `stateDiff`.
- Check if `statediff` contains keys that equal to indexed pool addresses.
- For each pool that tx touches:
  - Find the optimal amount in for a sandwich attack by performing a concurrent binary search.
  - Check for salmonella by checking if tx uses unconventional opcodes.
- If profitable after gas calculations, send the bundle to relays. 

## Usage

1. This repo requires you to run an [Erigon](https://github.com/ledgerwatch/erigon) archive node. The bot relies on the `newPendingTransactionsWithBody` subscription rpc endpoint which is a Erigon specific method. The node needs to be synced in archive mode to index all pools. 

2. [Install Rust](https://www.rust-lang.org/tools/install) if you haven't already. 

3. Fill in the searcher address in Huff contract and deploy either straight onchain or via create2 using a [metamorphic](https://github.com/0age/metamorphic) like factory.
> If you are using create2, you can easily mine for an address containing 7 zero bytes, saving 84 gas of calldata every time the contract address is used as an argument. [read more](https://medium.com/coinmonks/deploy-an-efficient-address-contract-a-walkthrough-cb4be4ffbc70).

4. Copy `.env.example` into `.env` and fill out values.

```console
cp .env.example .env
```

```
WSS_RPC=ws://localhost:8545
SEARCHER_PRIVATE_KEY=0000000000000000000000000000000000000000000000000000000000000001
FLASHBOTS_AUTH_KEY=0000000000000000000000000000000000000000000000000000000000000002
SANDWICH_CONTRACT=0xaAaAaAaaAaAaAaaAaAAAAAAAAaaaAaAaAaaAaaAa
SANDWICH_INCEPTION_BLOCK=...
```

5. Run the integration tests

```console
cargo test -p strategy --release --features debug
```

6. Run the bot in `debug mode`
Test bot's sandwich finding functionality without a deployed or funded contract (no bundles will be sent)

```
cargo run --release --features debug
```

7. Running the bot

```console
cargo run --release
```


## Improvements
- Token -> Weth sandwiches by using a 'flashswap' between two pools.
Normally we can only sandwich Weth -> Token swaps as the bot has Weth inventory, however you can use another pool's reserves as inventory to sandwich swaps in the other direction.
[example](https://eigenphi.io/mev/ethereum/tx/0x502b66ce1a8b71098decc3585c651745c1af55de19e8f29ec6fff4ed2fcd1589).
