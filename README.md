# Introduction

In this lesson, you will learn about two services that Chainlink provides for smart contract developers: Chainlink VRF and Chainlink Automations. These services enable smart contracts to access verifiable randomness and decentralized automation, which can enhance the functionality and security of your dApps.

## Chainlink VRF

Chainlink VRF is a service that provides verifiable and unpredictable random numbers for smart contracts. It uses a cryptographic function that takes a secret key and a seed as inputs, and produces a random number and a proof as outputs. The proof can be verified by anyone using the public key, but the random number cannot be predicted or manipulated by anyone, not even the oracle operators or miners. This ensures that the randomness is fair and secure for applications that need it, such as gaming, NFTs, or lotteries.

You can learn more about Chainlink VRF from the [official documentation](https://docs.chain.link/vrf/v2/introduction/).

## How Chainlink VRF works

The Chainlink VRF works by allowing a smart contract to request a random number from a Chainlink node. The node operator running the Chainlink node will then use the VRF to generate a random number and provide it to the smart contract. The process involves the following steps:

- The smart contract developer creates a public/private key pair for their contract and registers it with the Chainlink VRF Coordinator contract.
- The smart contract sends a request for randomness to the VRF Coordinator, along with some payment in LINK tokens and a seed value. The seed value can be any arbitrary input, such as a block hash or a user input.
- The VRF Coordinator assigns the request to one of the registered Chainlink nodes that have the matching public key.
- The assigned Chainlink node uses its private key and the seed value to compute the random number and the proof using the VRF function. The VRF function is based on elliptic curve cryptography and ensures that the output is deterministic and unique for each input.
- The Chainlink node returns the random number and the proof to the VRF Coordinator, which forwards them to the requesting smart contract.
- The smart contract verifies the proof using the public key and checks that the random number matches the seed value. If the verification passes, the smart contract accepts the random number and uses it for its logic. If the verification fails, the smart contract rejects the random number and can request a new one.

## Benefits of Chainlink VRF

Chainlink VRF offers several benefits for smart contract developers and users who need verifiable randomness:

- It provides a source of randomness that is provably fair and unbiased, as it cannot be influenced by any external factors or parties.
- It allows smart contracts to access randomness on-demand, without relying on pre-determined or scheduled events.
- It enables smart contracts to verify the authenticity of the randomness on-chain, without needing any external oracles or trusted parties.
- It leverages the security and reliability of the Chainlink network, which consists of hundreds of independent node operators who are incentivized to provide high-quality data and services.
- It supports multiple blockchain platforms and networks, such as Ethereum, Polygon, Binance Smart Chain, Avalanche, Fantom, and more.

## Learn more about Chainlink VRF

(1) Introduction to Chainlink VRF | Chainlink Documentation: <https://docs.chain.link/vrf/v2/introduction/>

(2) What is Chainlink VRF, and how does it work? - Cointelegraph: <https://cointelegraph.com/news/what-is-chainlink-vrf-and-how-does-it-work>

(3) How Chainlink VRF works - Smart Contract Research Forum: <https://www.smartcontractresearch.org/t/how-chainlink-vrf-works/2896>

## Chainlink Automations

Chainlink Automations is a service that enables smart contract developers to automate their contract functions in a decentralized manner. It uses a network of Chainlink nodes that monitor the contract state and trigger transactions when certain conditions are met. For example, a smart contract could use Chainlink Automations to automatically rebalance its portfolio, distribute rewards, or execute trades. Chainlink Automations saves developers time and resources, and provides reliable and scalable automation for their dApps.

You can learn more about Chainlink Automations from the [official website](https://chain.link/automation).

## How Chainlink Automations works

Chainlink Automations works by allowing a smart contract to register an upkeep with the Chainlink Automation Registry contract. An upkeep is a job or task that the smart contract wants to execute periodically or conditionally. The upkeep can be based on time (e.g., every hour) or custom logic (e.g., when the price of an asset reaches a certain level). The smart contract also needs to provide some payment in LINK tokens to fund the upkeep.

The Chainlink Automation Registry assigns the upkeep to one or more Chainlink nodes that have registered as Automation Nodes. These nodes are the same ones that provide data feeds and verifiable randomness for other Chainlink services. The nodes use a turn-taking algorithm to service the upkeeps in a fair and random manner.

The assigned Chainlink nodes monitor the smart contract state and evaluate the upkeep logic off-chain, using a local simulation of the blockchain. This reduces the gas costs and latency of the automation process. When the nodes detect that the upkeep condition is met, they initiate an on-chain transaction to execute the smart contract function.

The smart contract verifies that the transaction is coming from an authorized Chainlink node and that the upkeep logic is satisfied. If the verification passes, the smart contract accepts the transaction and performs the function. If the verification fails, the smart contract rejects the transaction and can request a new one.

## Benefits of Chainlink Automations

Chainlink Automations offers several benefits for smart contract developers and users who need decentralized automation:

- It provides a flexible and customizable way to automate any smart contract function based on time or custom logic.
- It allows smart contracts to access off-chain data and events through Chainlink data feeds and verifiable randomness within their automation logic.
- It leverages the security and reliability of the Chainlink network, which consists of hundreds of independent node operators who are incentivized to provide high-quality data and services.
- It supports multiple blockchain platforms and networks, such as Ethereum, Polygon, Binance Smart Chain, Avalanche, Fantom, and more.

## Learn more about Chainlink Automations

(1) Introduction to Chainlink Automation | Chainlink Documentation: <https://docs.chain.link/chainlink-automation/introduction/>

(2) Chainlink Automation Architecture | Chainlink Documentation: <https://docs.chain.link/chainlink-automation/overview/>

(3) Reliable, high-performance smart contract automation: <https://chain.link/automation>
