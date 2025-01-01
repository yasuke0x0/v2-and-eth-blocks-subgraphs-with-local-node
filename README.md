# Uniswap V2 Subgraph & ETH Blocks

The goal is to deploy and utilize both subgraphs within a local Graph Node setup: one for indexing events emitted by the Uniswap Factory contract and another for indexing all mined blocks of the
blockchain.
This repo serves as a great example of how to integrate a new blockchain network that isn't yet supported by The Graph.

The factory subgraph dynamically tracks any pair created by the uniswap factory. It tracks of the current state of Uniswap contracts, and contains derived stats for things like historical data and USD
prices.

- aggregated data across pairs and tokens,
- data on individual pairs and tokens,
- data on transactions
- data on liquidity providers
- historical data on Uniswap, pairs or tokens, aggregated by day

The ETH Blocks subgraph tracks all created blocks

## 🚀 Set Up

### 🔐 Environment Variables (ENV Vars)

The Graph CLI does not natively support environment variable interpolation. You'll need to manually update the required files:

1. **Update Factory Contract Address in graph-cli/v2-subgraph folder:**

- `subgraph.yaml`:
    - Replace `dataSources.source.address` with the factory contract address.
    - Replace `dataSources.source.startBlock` with the factory contract start block (creation block).
- `src/mappings/helpers.ts`: Replace the `FACTORY_ADDRESS` constant with the factory contract address.

2. **Graph Node Configuration:**

- In `graph-node/docker-compose.yml`, update:
    - `services.graph-node.environment.postgres_user`, `postgres_pass`, and `postgres_db`: Set custom database username and password.
    - `services.graph-node.environment.ethereum`: Replace `mainnet:REPLACE_THIS_PLACEHOLDER_WITH_THE_RPC_URL` with your custom RPC URL.
    - `services.postgres.environment.POSTGRES_USER` and `POSTGRES_PASSWORD`: Use the same database username and password as above.

---

### 🚒 Start a Dockerized Local Node

#### 1. Check Dependencies

Ensure everything is installed by running the script:

```bash
./graph-node/check.sh
```

#### 2. Start Docker Containers

Install Docker images and launch the containers for IPFS, Postgres, and the Graph Node:

```bash
docker-compose -f ./graph-node/docker-compose.yml up
```

Wait for the `ipfs`, `postgres`, and `graph-node` containers to deploy before proceeding to the next steps.

---

### Install graph-cli dependencies

```bash
yarn --cwd ./graph-cli/eth-blocks-subgraph
```

```bash
yarn --cwd ./graph-cli/v2-subgraph
```

### 🔧 Generate Types Based on Schema

Generate TypeScript types based on your GraphQL schema:

```bash
yarn --cwd ./graph-cli/eth-blocks-subgraph run codegen
```

```bash
yarn --cwd ./graph-cli/v2-subgraph run codegen
```

---

### 🎨 Create the Subgraph

Create the subgraph locally:

```bash
yarn --cwd ./graph-cli/eth-blocks-subgraph run create-local
```

```bash
yarn --cwd ./graph-cli/v2-subgraph run create-local
```

#### Expected Output from Docker:

```
graph-node-1  | Jan 01 19:54:32.885 INFO Received subgraph_create request, params: SubgraphCreateParams { name: SubgraphName("eth-blocks") }, component: JsonRpcServer
```

```
graph-node-1  | Dec 14 21:10:56.167 INFO Received subgraph_create request, params: SubgraphCreateParams { name: SubgraphName("uniswap-v2") }, component: JsonRpcServer
```

---

### 📢 Deploy the Subgraphs to the Local Node

Deploy the subgraphs:

```bash
yarn --cwd ./graph-cli/eth-blocks-subgraph run deploy-local
```

```bash
yarn --cwd ./graph-cli/v2-subgraph run deploy-local
```

#### Expected CLI Output:

```
Build completed: QmcTdzvj7FRkj3mGgAVgHbEk5RWget1F1ta9YdNZ3SWahK

Deployed to http://localhost:8000/subgraphs/name/eth-blocks/graphql

Subgraph endpoints:
Queries (HTTP):     http://localhost:8000/subgraphs/name/eth-blocks
```

```
Build completed: QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1

Deployed to http://localhost:8000/subgraphs/name/uniswap-v2/graphql

Subgraph endpoints:
Queries (HTTP):     http://localhost:8000/subgraphs/name/uniswap-v2
```

#### Expected Docker Logs:

```
graph-node-1  | Dec 14 21:48:17.291 INFO Received subgraph_deploy request, params: SubgraphDeployParams { name: SubgraphName("uniswap-v2"), ipfs_hash: DeploymentHash("QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1"), node_id: None, debug_fork: None, history_blocks: None }, component: JsonRpcServer
graph-node-1  | Dec 14 21:48:17.988 INFO Set subgraph start block, block: Some(#429834 (4d7f61f146762bd0ee03f68c4e8c3393f4950eded2b745d0f2ea3f9ca4e22ece)), sgd: 0, subgraph_id: QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1, component: SubgraphRegistrar
```

---

### ✨ You're All Set!

Your local Graph Node is now running and indexing the Uniswap Factory and ETH blocks. You can query your subgraphs at:

```plaintext
http://localhost:8000/subgraphs/name/uniswap-v2
```

```plaintext
http://localhost:8000/subgraphs/name/eth-blocks
```

Remember, after deployment, the data will not be available immediately. You must wait for blockchain indexing to complete. The Graph needs to process and insert all contract-emitted events into its
local database before the data becomes accessible.

Here’s an example of blockchain scan output in a Docker environment:

```
graph-node-1  | Dec 14 22:19:55.434 INFO Scanned blocks [558946, 560945], range_size: 2000, sgd: 1, subgraph_id: QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1, component: BlockStream
```

This output shows the progress of The Graph node syncing with the blockchain, including the number of processed blocks and events. You can monitor this log to track when the data is fully indexed and
ready for use.

## Key Entity Overviews

#### UniswapFactory

Contains data across all of Uniswap V2. This entity tracks important things like total liquidity (in ETH and USD, see below), all time volume, transaction count, number of pairs and more.

#### Token

Contains data on a specific token. This token specific data is aggregated across all pairs, and is updated whenever there is a transaction involving that token.

#### Pair

Contains data on a specific pair.

#### Transaction

Every transaction on Uniswap is stored. Each transaction contains an array of mints, burns, and swaps that occured within it.

#### Mint, Burn, Swap

These contain specifc information about a transaction. Things like which pair triggered the transaction, amounts, sender, recipient, and more. Each is linked to a parent Transaction entity.

#### Block

Every block is stored along with its hash, timestamp, parent hash ...

## Example Queries

### Querying Aggregated Uniswap Data

### Querying pair list

This query returns the 10 first pairs of the factory.

```graphql
{
    pairs(first: 10) {
        id
        token1 {
            id
            name
        }
        token0 {
            id
            name
            symbol
        }
    }
}
```

### Querying Aggregated Uniswap Data

This query fetches aggredated data from all uniswap pairs and tokens, to give a view into how much activity is happening within the whole protocol.

```graphql
{
    uniswapFactories(first: 1) {
        pairCount
        totalVolumeUSD
        totalLiquidityUSD
    }
}
```

This query get data about the first 10 blocks 
```graphql
{
    blocks(first: 10) {
        id
        timestamp
        number
    }
}
```

## Sources

This repo uses sources from the following repos [the graph-node](https://github.com/graphprotocol/graph-node) & [uniswap v2 subgraph](https://github.com/Uniswap/v2-subgraph).

* [The graph new chain integration](https://thegraph.com/docs/en/new-chain-integration/)
* [graph-node repo](https://github.com/graphprotocol/graph-node)
* [graph-cli doc](https://thegraph.com/docs/en/quick-start/)
* [uniswap v2 subgraph repo](https://github.com/Uniswap/v2-subgraph)
* [ETH-blocks subgraph repo](https://github.com/blocklytics/ethereum-blocks)
