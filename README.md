# Uniswap V2 Subgraph

The goal is to index emitted events of the uniswap factory contract with a subgraph in a local graph node. This is a good example for a [new chain integration](https://thegraph.com/docs/en/new-chain-integration/) if a [network is not supported](https://thegraph.com/docs/en/developing/supported-networks/) by [The graph](https://thegraph.com/);

This subgraph dynamically tracks any pair created by the uniswap factory. It tracks of the current state of Uniswap contracts, and contains derived stats for things like historical data and USD prices.

- aggregated data across pairs and tokens,
- data on individual pairs and tokens,
- data on transactions
- data on liquidity providers
- historical data on Uniswap, pairs or tokens, aggregated by day

## Set up

### ENV Vars
The Graph CLI does not natively support environment variable interpolation.
That's why we must perform the replacements manually
* subgraph.yaml: replace dataSources.source.address prop value by the factory contract address
* src/mappings/helpers.ts: Replace const FACTORY_ADDRESS by the factory contract address
* graph-node/docker-compose.yml:
  * Replace services.graph-node.environment.(postgres_user | postgres_pass | postgres_db) props values with a custom DB user & pwd
  * Replace services.graph-node.environment.ethereum with your custom RPC URL. Replace it like this: 'mainnet:REPLACE_THIS_PLACEHOLDER_WITH_THE_RPC_URL'
  * replace services.postgres.environment.(POSTGRES_USER | POSTGRES_PASSWORD) with the same user & pwd of the first point.

### Start a dockerized local node
First, let's check if everything is installed before proceeding with docker-compose
```bash
./graph-node/check.sh
```
Install images & launch the node container (IPFS + DB + Graph Protocol)
```bash
docker-compose -f ./graph-node/docker-compose.yml up
```

Wait till ipfs, postgres & graph-node containers are deployed before executing the next queries.


### Generate types based on schema
```bash
yarn run codegen
```

### Create the subgraph
```bash
yarn run create-local
```

Expected result from docker
```
graph-node-1  | Dec 14 21:10:56.167 INFO Received subgraph_create request, params: SubgraphCreateParams { name: SubgraphName("uniswap-v2") }, component: JsonRpcServe
```

### Deploy the subgraph in the local node
```bash
yarn run deploy-local
```

Expected result from command
```
Build completed: QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1

Deployed to http://localhost:8000/subgraphs/name/uniswap-v2/graphql

Subgraph endpoints:
Queries (HTTP):     http://localhost:8000/subgraphs/name/uniswap-v2
```

Expected result from docker
```
graph-node-1  | Dec 14 21:48:17.291 INFO Received subgraph_deploy request, params: SubgraphDeployParams { name: SubgraphName("uniswap-v2"), ipfs_hash: DeploymentHash("QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1"), node_id: None, debug_fork: None, history_blocks: None }, component: JsonRpcServer
graph-node-1  | Dec 14 21:48:17.988 INFO Set subgraph start block, block: Some(#429834 (4d7f61f146762bd0ee03f68c4e8c3393f4950eded2b745d0f2ea3f9ca4e22ece)), sgd: 0, subgraph_id: QmQKjuhi2ott8Yjn1BRrYFBR5iw8A72dGMZrKHkGdjoTo1, component: SubgraphRegistrar
```


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

## Sources

This repo uses sources from the following repos [the graph-node](https://github.com/graphprotocol/graph-node) & [uniswap v2 subgraph](https://github.com/Uniswap/v2-subgraph). 

* [The graph new chain integration](https://thegraph.com/docs/en/new-chain-integration/)
* [graph-node repo](https://github.com/graphprotocol/graph-node)
* [graph-cli doc](https://thegraph.com/docs/en/quick-start/)
* [uniswap v2 subgraph repo](https://github.com/Uniswap/v2-subgraph)
