# Multichain Quick Start - Safe

This page explains how to create an multi-chain indexer for [Safe](https://safe.global/), a system that makes secure wallets requiring multiple authorisations. This boosts security and lowers the risk of unauthorised use.

After reading this guide, you'll understand the protocol, know about multi-signature setups, and learn how to set up a SubQuery indexer to monitor and track signed message events on different EVM blockchains.

<!-- @include: ./snippets/multi-chain-quickstart-reference.md -->

Safe factory contracts have been deployed on various blockchain networks, sometimes using different contract addresses. Nevertheless, as the same smart contract was utilised, every instance retains the same collection of functions and events.

<!-- @include: ../snippets/evm-quickstart-reference.md -->

As a prerequisite, you will need to generate types from the ABI files of each smart contract. You can obtain these ABI files by searching for the ABIs of the mentioned smart contract addresses on blockchain scanners.

For instance, you can locate the ABI for the Safe Ethereum smart contract at the bottom of [this page](https://etherscan.io/address/0x12302fE9c02ff50939BaAaaf415fc226C078613C#code). Additionally, you can kickstart your project by using the EVM Scaffolding approach (detailed [here](../quickstart.md#evm-project-scaffolding)). You'll find all the relevant events to be scaffolded in the documentation for each type of smart contract.

::: tip Note
Check the final code repository [here](https://github.com/subquery/ethereum-subql-starter/tree/main/Multi-Chain/safe) to observe the integration of all previously mentioned configurations into a unified codebase.
:::

In this Safe indexing project, our main focus is on configuring the indexer to exclusively capture logs generated by two specific types of Safe smart contracts:

In this Safe indexing project, our primary focus lies in configuring the indexer to selectively capture logs generated by two specific types of Safe smart contracts:

1. **`SafeProxy`** and **`SafeProxyFactory`** (contract address on Ethereum: `0x12302fE9c02ff50939BaAaaf415fc226C078613C`): These contracts are utilised for cost-efficiency of the deployment of individual safe smart contracts. Safe adopts the [proxy pattern](https://blog.openzeppelin.com/proxy-patterns/) to reduce deployment expenses and enable contract upgradability. The ProxyFactory is employed to create a new safe that links to the proxy.

2. **Individual Safe Smart Contracts**: These contracts encompass all the essential functionality needed for establishing and executing Safe transactions.

<!-- @include: ./snippets/multi-chain-evm-manifest-intro.md#level2 -->

To begin, we will establish an Ethereum indexer. As Safe proxies have undergone multiple updates, the indexing process necessitates the configuration of three handlers. In this illustration, we introduce specific smart contracts along with their respective addresses and logs:

::: code-tabs

@tab project.yaml

```yaml
dataSources:
  - kind: ethereum/Runtime
    startBlock: 7450116
    options:
      abi: GnosisSafeProxyFactory
      address: "0x12302fE9c02ff50939BaAaaf415fc226C078613C"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.0.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_0_0
          filter:
            topics:
              - ProxyCreation(address)
  - kind: ethereum/Runtime
    startBlock: 9084508
    options:
      abi: GnosisSafeProxyFactory
      address: "0x76E2cFc1F5Fa8F6a5b3fC4c8F4788F0116861F9B"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.1.1.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_1_0
          filter:
            topics:
              - ProxyCreation(address)
  - kind: ethereum/Runtime
    startBlock: 12504126
    options:
      abi: GnosisSafeProxyFactory
      address: "0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.3.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_3_0
          filter:
            topics:
              - ProxyCreation(address,address)
```

:::

The factory smart contracts mentioned above create new contract instances for every new safe. As a result, we must employ [dynamic data sources](../../build/dynamicdatasources.md) to establish indexers for each of these new contracts. To integrate the dynamic data sources, simply add the following code to the manifest file:

::: code-tabs

@tab project.yaml

```yaml
templates:
  - kind: ethereum/Runtime
    name: GnosisSafe
    options:
      abi: GnosisSafe
    assets:
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleEthSignMsg
          filter:
            topics:
              - SignMsg(bytes32)
```

:::

<!-- @include: ../snippets/ethereum-manifest-note.md -->

Next, change the name of the file mentioned above to `ethereum.yaml` to indicate that this file holds the Ethereum configuration.

<!-- @include: ./snippets/multi-chain-creation.md -->

::: code-tabs

@tab subquery-multichain.yaml

```yaml
specVersion: 1.0.0
query:
  name: "@subql/query"
  version: "*"
projects:
  - ethereum.yaml
  - matic.yaml
  - op.yaml
```

:::

Also, you will end up with the individual chains' manifest files like those:

::: code-tabs

@tab op.yaml

```yaml
specVersion: 1.0.0
version: 0.0.1
name: ethereum-safe
description: This project indexes the Safe signature data from various chains
runner:
  node:
    name: "@subql/node-ethereum"
    version: ">=3.0.0"
  query:
    name: "@subql/query"
    version: "*"
schema:
  file: ./schema.graphql
network:
  chainId: "10"
  endpoint:
    - https://optimism.llamarpc.com
  # dictionary: https://dict-tyk.subquery.network/query/optimism-mainnet
dataSources:
  - kind: ethereum/Runtime
    startBlock: 110991101
    options:
      abi: GnosisSafeProxyFactory
      address: "0xC22834581EbC8527d974F8a1c97E1bEA4EF910BC"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.3.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_3_0
          filter:
            topics:
              - ProxyCreation(address,address)
templates:
  - kind: ethereum/Runtime
    name: GnosisSafe
    options:
      abi: GnosisSafe
    assets:
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleOpSignMsg
          filter:
            topics:
              - SignMsg(indexed bytes32)
repository: https://github.com/subquery/ethereum-subql-starter
```

@tab matic.yaml

```yaml
specVersion: 1.0.0
version: 0.0.1
name: ethereum-safe
description: This project indexes the Safe signature data from various chains
runner:
  node:
    name: "@subql/node-ethereum"
    version: ">=3.0.0"
  query:
    name: "@subql/query"
    version: "*"
schema:
  file: ./schema.graphql
network:
  chainId: "137"
  endpoint:
    - https://polygon.llamarpc.com
  # dictionary: https://gx.api.subquery.network/sq/subquery/polygon-dictionary
dataSources:
  - kind: ethereum/Runtime
    startBlock: 45222934
    options:
      abi: GnosisSafeProxyFactory
      address: "0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.3.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_3_0
          filter:
            topics:
              - ProxyCreation(address,address)
templates:
  - kind: ethereum/Runtime
    name: GnosisSafe
    options:
      abi: GnosisSafe
    assets:
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleMaticSignMsg
          filter:
            topics:
              - SignMsg(indexed bytes32)
repository: https://github.com/subquery/ethereum-subql-starter
```

@tab ethereum.yaml

```yaml
specVersion: 1.0.0
version: 0.0.1
name: ethereum-safe
description: This project indexes the Safe signature data from various chains
runner:
  node:
    name: "@subql/node-ethereum"
    version: ">=3.0.0"
  query:
    name: "@subql/query"
    version: "*"
schema:
  file: ./schema.graphql
network:
  chainId: "1"
  endpoint:
    - https://eth.llamarpc.com
  dictionary: https://gx.api.subquery.network/sq/subquery/eth-dictionary
dataSources:
  - kind: ethereum/Runtime
    startBlock: 7450116
    options:
      abi: GnosisSafeProxyFactory
      address: "0x12302fE9c02ff50939BaAaaf415fc226C078613C"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.0.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_0_0
          filter:
            topics:
              - ProxyCreation(address)
  - kind: ethereum/Runtime
    startBlock: 9084508
    options:
      abi: GnosisSafeProxyFactory
      address: "0x76E2cFc1F5Fa8F6a5b3fC4c8F4788F0116861F9B"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.1.1.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_1_0
          filter:
            topics:
              - ProxyCreation(address)
  - kind: ethereum/Runtime
    startBlock: 12504126
    options:
      abi: GnosisSafeProxyFactory
      address: "0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2"
    assets:
      GnosisSafeProxyFactory:
        file: ./abis/GnosisSafeProxyFactory_v1.3.0.json
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleProxyCreation_1_3_0
          filter:
            topics:
              - ProxyCreation(address,address)
templates:
  - kind: ethereum/Runtime
    name: GnosisSafe
    options:
      abi: GnosisSafe
    assets:
      GnosisSafe:
        file: ./abis/GnosisSafe.json
    mapping:
      file: ./dist/index.js
      handlers:
        - kind: ethereum/LogHandler
          handler: handleEthSignMsg
          filter:
            topics:
              - SignMsg(bytes32)
repository: https://github.com/subquery/ethereum-subql-starter
```

:::

<!-- @include: ./snippets/multi-chain-network-origin-note.md -->

<!-- @include: ../snippets/schema-intro.md#level2 -->

```graphql
type Sig @entity {
  id: ID!
  account: String!
  msgHash: String!
  timestamp: BigInt!
  network: String!
}
```

This single object is `Sig`, containing several parameters to be filled from on-chain data. Additionally, it will include a `network` attribute explicitly provided through mapping logic.

<!-- @include: ../snippets/evm-codegen.md -->

```ts
import { Sig } from "../types";
import { ProxyCreationLog as ProxyCreation_v1_0_0 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v100";
import { ProxyCreationLog as ProxyCreation_v1_1_1 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v111";
import { ProxyCreationLog as ProxyCreation_v1_3_0 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v130";

import { SignMsgLog } from "../types/abi-interfaces/GnosisSafe";
```

<!-- @include: ../snippets/evm-mapping-intro.md#level2 -->

Setting up mappings for this smart contract is straightforward. In this instance, the mappings are stored within the `src/mappings` directory, with the sole mapping file being `factory.ts`. Now, let's take a closer look at it:

```ts
import { ProxyCreationLog as ProxyCreation_v1_0_0 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v100";
import { ProxyCreationLog as ProxyCreation_v1_1_1 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v111";
import { ProxyCreationLog as ProxyCreation_v1_3_0 } from "../types/abi-interfaces/GnosisSafeProxyFactory_v130";

import { SignMsgLog } from "../types/abi-interfaces/GnosisSafe";
import { createGnosisSafeDatasource as GnosisSafeContract } from "../types";
import { GnosisSafe__factory } from "../types/contracts";
import { Sig } from "../types";
import assert from "assert";

async function handleProxyCreation(proxyAddress: string): Promise<void> {
  let safeInstance = GnosisSafe__factory.connect(proxyAddress, api);
  let callGetOwnerResult = await safeInstance.getOwners();
  if (callGetOwnerResult) GnosisSafeContract({ proxyAddress });
  logger.warn(`Created a datasource for ${proxyAddress}`);
}

export async function handleProxyCreation_1_0_0(
  event: ProxyCreation_v1_0_0
): Promise<void> {
  assert(event.args, "No args in log");
  logger.warn("handleProxyCreation_1_0_0 is tiggered");
  await handleProxyCreation(event.args.proxy);
}

export async function handleProxyCreation_1_1_1(
  event: ProxyCreation_v1_1_1
): Promise<void> {
  assert(event.args, "No args in log");
  logger.warn("handleProxyCreation_1_1_0 is tiggered");
  await handleProxyCreation(event.args.proxy);
}

export async function handleProxyCreation_1_3_0(
  event: ProxyCreation_v1_3_0
): Promise<void> {
  assert(event.args, "No args in log");
  logger.warn("handleProxyCreation_1_3_0 is tiggered");
  await handleProxyCreation(event.args.proxy);
}

async function createSig(event: SignMsgLog, network: string): Promise<void> {
  logger.warn("createSig is tiggered");
  let sig = await Sig.create({
    id: event.transaction.hash,
    account: event.address,
    msgHash: event.topics[1].slice(2),
    timestamp: event.block.timestamp,
    network: network,
  });
  sig.save();
}

export async function handleEthSignMsg(event: SignMsgLog): Promise<void> {
  await createSig(event, "ethereum");
}

export async function handleMaticSignMsg(event: SignMsgLog): Promise<void> {
  await createSig(event, "matic");
}

export async function handleOpSignMsg(event: SignMsgLog): Promise<void> {
  await createSig(event, "op");
}
```

This code appears to be a TypeScript script for handling events and creating data sources for Gnosis Safe contracts on Ethereum and other networks. Here's a brief explanation of the key components:

This code begins by importing various modules, interfaces, and contract factories required for interacting with Safe contracts and handling events.

Then, there are several event handling functions defined in this code, each corresponding to a specific version of the `ProxyCreationLog` event. These functions receive event data, ensure that event arguments are present, log messages, and then call the `handleProxyCreation` function.

`handleProxyCreation` function handles the creation of a data source for a Safe proxy contract. It connects to the contract, retrieves the owners, and then creates a Safe data source. Subsequently, every subsequent `SignMsg` event generated in each newly created data source will be processed.

And there are three handling functions (`handleEthSignMsg`, `handleMaticSignMsg`, and `handleOpSignMsg`) that are triggered by this `SignMsg` event. Those functions utilise the `createSig` function to create signature objects for Ethereum, Matic, and Op networks, respectively. These functions specify the network and call `createSig` to handle the event and create the signature.

Finally, `createSig` function is responsible for creating a signature object based on the provided event data. It extracts relevant information from the event, such as the transaction hash, account, message hash, timestamp, and network, and then saves this signature data.

This code essentially centralises the handling of `SignMsg` events for various networks and ensures that they are correctly recorded in the `Sig` object with network-specific attributes, facilitating data tracking and analysis for each network.

🎉 At this stage, we have successfully incorporated all the desired entities and mappings that can be retrieved from Safe smart contracts. For each of these entities, we've a single mapping handler to structure and store the data in a queryable format.

::: tip Note
Check the final code repository [here](https://github.com/subquery/ethereum-subql-starter/tree/main/Multi-Chain/safe) to observe the integration of all previously mentioned configurations into a unified codebase.
:::

<!-- @include: ../snippets/build.md -->

<!-- @include: ../snippets/run-locally.md -->

<!-- @include: ../snippets/query-intro.md -->

::: details Sigs

#### Request

```graphql
{
  query {
    sigs {
      nodes {
        id
        msgHash
        timestamp
        account
        network
      }
    }
  }
}
```

#### Response

```json
{
  "data": {
    "query": {
      "sigs": {
        "nodes": [
          {
            "id": "0x00049ea38f5d4330503fc3a3aec6b38bfd99a4740592846604bec866d8b846f7",
            "msgHash": "3d033a2acc018a468f69d3ed53241bac0ae569eaac4859b26cb3a803d8d2dd21",
            "timestamp": "1646297276",
            "account": "0x00f10F0fD39533bd8567c763B2671cDa00Da7872",
            "network": "ethereum"
          },
          {
            "id": "0x689449b9d3ec424f6272e47f6601bde91086add7f37554e878697403dc6113cc",
            "msgHash": "e621b182c5cf3b806d87cd08d924e832300e149b97aaf0ad9e28c58dde94a479",
            "timestamp": "1646296933",
            "account": "0x00f10F0fD39533bd8567c763B2671cDa00Da7872",
            "network": "ethereum"
          }
        ]
      }
    }
  }
}
```

:::

::: details Network Metadatas

#### Request

```graphql
{
  _metadatas {
    totalCount
    nodes {
      chain
      lastProcessedHeight
    }
  }
}
```

#### Response

```json
{
  "data": {
    "_metadatas": {
      "totalCount": 3,
      "nodes": [
        {
          "chain": "137",
          "lastProcessedHeight": 45222964
        },
        {
          "chain": "10",
          "lastProcessedHeight": 110991253
        },
        {
          "chain": "1",
          "lastProcessedHeight": 14312934
        }
      ]
    }
  }
}
```

:::

::: details Dictionaries

#### Request

```graphql
{
  _metadata {
    dynamicDatasources
  }
}
```

#### Response

```json
{
  "data": {
    "_metadata": {
      "dynamicDatasources": "[{\"templateName\":\"GnosisSafe\",\"args\":{\"proxyAddress\":\"0xF56F29E3fe941FDFb48859d46bB24425Fd648e55\"},\"startBlock\":110991101}]"
    }
  }
}
```

:::

<!-- @include: ../snippets/whats-next.md -->
