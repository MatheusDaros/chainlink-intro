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

![image](https://github.com/MatheusDaros/chainlink-intro/assets/17483282/53cd17f4-6fdf-4d44-b0b4-8907c41d4de6)

## Benefits of Chainlink VRF

Chainlink VRF offers several benefits for smart contract developers and users who need verifiable randomness:

- It provides a source of randomness that is provably fair and unbiased, as it cannot be influenced by any external factors or parties.
- It allows smart contracts to access randomness on-demand, without relying on pre-determined or scheduled events.
- It enables smart contracts to verify the authenticity of the randomness on-chain, without needing any external oracles or trusted parties.
- It leverages the security and reliability of the Chainlink network, which consists of hundreds of independent node operators who are incentivized to provide high-quality data and services.
- It supports multiple blockchain platforms and networks, such as Ethereum, Polygon, Binance Smart Chain, Avalanche, Fantom, and more.

## VRF Experimentation

To use Chainlink VRF inside a smart contract we first need to fund the LINK tokens needed to carry out this processing.
There are two methods for requesting randomness with Chainlink VRF v2:

- [Subscription](https://docs.chain.link/vrf/v2/subscription): Create a subscription account and fund its balance with LINK tokens. Users can then connect multiple consuming contracts to the subscription account. When the consuming contracts request randomness, the transaction costs are calculated after the randomness requests are fulfilled and the subscription balance is deducted accordingly. This method allows you to fund requests for multiple consumer contracts from a single subscription.
- [Direct funding](https://docs.chain.link/vrf/v2/direct-funding): Consuming contracts directly pay with LINK when they request random values. You must directly fund your consumer contracts and ensure that there are enough LINK tokens to pay for randomness requests. Were going to use this method in this article.

Direct funding method doesn't require a subscription and is optimal for one-off requests for randomness. This method also works best for applications where your end-users must pay the fees for VRF because the cost of the request is determined at request time.

Unlike the subscription method, the direct funding method does not require you to create subscriptions and pre-fund them. Instead, you must directly fund consuming contracts with LINK tokens before they request randomness. Because the consuming contract directly pays the LINK for the request, the cost is calculated during the request and not during the callback when the randomness is fulfilled. If you need help estimating the costs, check [this documentation page.](https://docs.chain.link/vrf/v2/estimating-costs)

### Setting up the smart contract

To set up your consuming contract, you must take care of these steps:

- Your contract must inherit [VRFV2WrapperConsumerBase](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFV2WrapperConsumerBase.sol).

- Your contract must implement the `fulfillRandomWords` function, which is the _callback VRF function_. Here, you add logic to handle the random values after they are returned to your contract.
- Submit your VRF request by calling the `requestRandomness` function in the [VRFV2WrapperConsumerBase](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFV2WrapperConsumerBase.sol) contract. Include the following parameters in your request:
  - `requestConfirmations`: The number of block confirmations the VRF service will wait to respond. The minimum and maximum confirmations for your network can be found here.
  - `callbackGasLimit`: The maximum amount of gas to pay for completing the callback VRF function.
  - `numWords`: The number of random numbers to request. You can find the maximum number of random values per request for your network in the Supported networks page.
 
You can learn more about how Chainlink process the request in [this documentation page](https://docs.chain.link/vrf/v2/direct-funding#how-vrf-processes-your-request).

To test this process we're using this smart contract:

```solidity
// SPDX-License-Identifier: MIT
// An example of a consumer contract that directly pays for each request.
pragma solidity ^0.8.7;

import "@chainlink/contracts/src/v0.8/ConfirmedOwner.sol";
import "@chainlink/contracts/src/v0.8/VRFV2WrapperConsumerBase.sol";

/**
 * Request testnet LINK and ETH here: https://faucets.chain.link/
 * Find information on LINK Token Contracts and get the latest ETH and LINK faucets here: https://docs.chain.link/docs/link-token-contracts/
 */

/**
 * THIS IS AN EXAMPLE CONTRACT THAT USES HARDCODED VALUES FOR CLARITY.
 * THIS IS AN EXAMPLE CONTRACT THAT USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

contract VRFv2DirectFundingConsumer is
    VRFV2WrapperConsumerBase,
    ConfirmedOwner
{
    event RequestSent(uint256 requestId, uint32 numWords);
    event RequestFulfilled(
        uint256 requestId,
        uint256[] randomWords,
        uint256 payment
    );

    struct RequestStatus {
        uint256 paid; // amount paid in link
        bool fulfilled; // whether the request has been successfully fulfilled
        uint256[] randomWords;
    }
    mapping(uint256 => RequestStatus)
        public s_requests; /* requestId --> requestStatus */

    // past requests Id.
    uint256[] public requestIds;
    uint256 public lastRequestId;

    // Depends on the number of requested values that you want sent to the
    // fulfillRandomWords() function. Test and adjust
    // this limit based on the network that you select, the size of the request,
    // and the processing of the callback request in the fulfillRandomWords()
    // function.
    uint32 callbackGasLimit = 100000;

    // The default is 3, but you can set this higher.
    uint16 requestConfirmations = 3;

    // For this example, retrieve 2 random values in one request.
    // Cannot exceed VRFV2Wrapper.getConfig().maxNumWords.
    uint32 numWords = 2;

    // Address LINK - hardcoded for Sepolia
    address linkAddress = 0x779877A7B0D9E8603169DdbD7836e478b4624789;

    // address WRAPPER - hardcoded for Sepolia
    address wrapperAddress = 0xab18414CD93297B0d12ac29E63Ca20f515b3DB46;

    constructor()
        ConfirmedOwner(msg.sender)
        VRFV2WrapperConsumerBase(linkAddress, wrapperAddress)
    {}

    function requestRandomWords()
        external
        onlyOwner
        returns (uint256 requestId)
    {
        requestId = requestRandomness(
            callbackGasLimit,
            requestConfirmations,
            numWords
        );
        s_requests[requestId] = RequestStatus({
            paid: VRF_V2_WRAPPER.calculateRequestPrice(callbackGasLimit),
            randomWords: new uint256[](0),
            fulfilled: false
        });
        requestIds.push(requestId);
        lastRequestId = requestId;
        emit RequestSent(requestId, numWords);
        return requestId;
    }

    function fulfillRandomWords(
        uint256 _requestId,
        uint256[] memory _randomWords
    ) internal override {
        require(s_requests[_requestId].paid > 0, "request not found");
        s_requests[_requestId].fulfilled = true;
        s_requests[_requestId].randomWords = _randomWords;
        emit RequestFulfilled(
            _requestId,
            _randomWords,
            s_requests[_requestId].paid
        );
    }

    function getRequestStatus(
        uint256 _requestId
    )
        external
        view
        returns (uint256 paid, bool fulfilled, uint256[] memory randomWords)
    {
        require(s_requests[_requestId].paid > 0, "request not found");
        RequestStatus memory request = s_requests[_requestId];
        return (request.paid, request.fulfilled, request.randomWords);
    }

    /**
     * Allow withdraw of Link tokens from the contract
     */
    function withdrawLink() public onlyOwner {
        LinkTokenInterface link = LinkTokenInterface(linkAddress);
        require(
            link.transfer(msg.sender, link.balanceOf(address(this))),
            "Unable to transfer"
        );
    }
}
```

For this contract to work, we're going make sure that the consuming contract has enough LINK.
To do so, complete these two steps: 

1: Acquire [testnet LINK](https://docs.chain.link/resources/acquire-link);

2: [Fund your contract](https://docs.chain.link/resources/fund-your-contract) with testnet LINK. For this example, funding your contract with **2 LINK** should be sufficient.

Now that the contract is funded, we're going to test the randomness function with these steps:

- Call the `requestRandomWords()` function to send the request for random values to Chainlink VRF;

- To fetch the request ID of your request, call `lastRequestId()`;

- After the oracle returns the random values to your contract, the mapping s_requests is updated. The received random values are stored in `s_requests[_requestId].randomWords`.

- Call `getRequestStatus()` and specify the requestId to display the random words.

To complete these steps using Remix, you can follow [this tutorial](https://docs.chain.link/vrf/v2/direct-funding/examples/get-a-random-number#create-and-deploy-a-vrf-v2-compatible-contract).

## Learn more about Chainlink VRF

(1) Introduction to Chainlink VRF | Chainlink Documentation: <https://docs.chain.link/vrf/v2/introduction/>

(2) What is Chainlink VRF, and how does it work? - Cointelegraph: <https://cointelegraph.com/news/what-is-chainlink-vrf-and-how-does-it-work>

(3) How Chainlink VRF works - Smart Contract Research Forum: <https://www.smartcontractresearch.org/t/how-chainlink-vrf-works/2896>

(4) Security Considerations: <https://docs.chain.link/vrf/v2/security>

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
