# Exchange Broker Network and Collateralized DEX

## General Principles 

**GRAFT Network DEX**  is intended to facilitate a decentralized exchange infrastructure for the GRAFT Payment Network as well as function on its own providing fast, binding, collateralized cross-chain exchange capabilities to the trading parties.

## Parameters:

- Non-custodial (alt-chain funds remain with the owners)
- Cross-chain (trades are executed among different blockchains with no inherent limitations other than network support)
- Atomic swap or equivalent (the trade transaction has to be executed as one unit or with ability to roll back / receive refund when the trade doesn’t conclude)
- Fast execution (so it can be folded into an RTA transaction)
- Can execute both spot (market) and strike (limit) transactions

## Concept:

**GRAFT DEX** as decentralized exchange secured by using fast (RTA) $GRFT based collateral transactions deposited by both trading sides into a multi signature wallet, validated by the Supernode validators.

## Workflow:

![DEX5_1](https://user-images.githubusercontent.com/45132833/62647271-d4cd5d80-b958-11e9-9706-8af2ef8a2f91.png)


1. _**Client1/EB1->SN_network**_: Select an SN sample
2. _**Client1/EB1->SN_network**_: Sends a trade proposal (spot or strike), trade proposal fee and trade parameters (exchange terms, # confirmations, ttl, etc) via SN network
3. _**EB2/Client2->Client1/EB1**_: Sends proposal / agrees to a trade
4. _**Client1/EB1->Client1/EB1**_: Selects offer(s)
5. _**Client1/EB1>SN_network**_: Create MS wallet with SN sample as signatories
6. _**Client1/EB1->SN_network**_: Sends collateral to MS wallet (RTA) (alt-chain receive address in metadata)

7. _**Client1/EB1->EB2/Client2**_: Acknowledges offer, forwards MS wallet address
8. _**EB2/Client2->SN_network**_: Sends collateral to MS wallet (RTA) (alt-chain receive address in metadata)
9. _**Client1/EB1->SN_network**_: Verifies collateral (via SN sample)

10. _**Client1/EB1->EB2/Client2**_: Send alt-chain tx 
11. _**Client1/EB1->SN_network**_: Forward data about alt-chain tx to SN validator sample, 
12. _**SN_network**_: Verifies and records 1’s collateral tx with data (alt-chain txid, cross-collateral, trade id, trade params) */fee
13. _**EB2/Client2->SN_network**_: Verifies collateral (via SN sample)
14. _**EB2/Client2->Client1/EB1**_: Send alt-chain tx
15. _**EB2/Client2->SN_network**_: Forward data about alt-chain tx to SN validator sample
16. _**SN_network**_: Verifies and records 2’s collateral tx with data (alt-chain txid, cross-collateral, trade id, trade params) */fee


## Settlement:

- _**Client1/EB1->SN_network**_: Requests refund or forward of the collateral
- _**SN_network->Client1/EB1**_: Validates and forward collateral */fee
- _**EB2/Client2->SN_network**_: Requests refund or forward of the collateral, along with validation fee
- _**SN_network->EB2/Client2**_: Validates and forward collateral */fee


## Assumptions:

For this phase, client, exchange broker, and (proxy or full) SN maintain a trusted relationship (owned by the same entity).  In the future it will be possible to separate them.


## Client UI

![image_2019_08_12T07_17_55_000Z](https://user-images.githubusercontent.com/45132833/62850418-73f0ad00-bceb-11e9-85be-975a8efc5501.png)


## Client / Exchange Broker

- Connects to the Exchange Broker (EB)
- Forms transactions
- Executes trade logic
- Generates trade offers / responses
- Monitors trade results
- Posts collateral transactions and requests collateral refunds 

## Exchange Broker

- Is a Supernode module
- Handles trading related communication (broadcast & receive)
- Provides an RPC (HTTPs?) API for the client
- Participates in distributed order book

## Supernodes from the sample

- Collect info about the trade transaction from the parties
- Validate the trades on request by matching against agreed parameters
- Process collateral forward and refund requests

## Sample Selection:

- M:N (TBD)
- Based on current Auth sample selection (? TBD)
- Has to survive 100+ blocks

## Economics

- Exchange Brokers earn money through the “spread” (the difference between the rate they buy and sell currency at)
- The SN Sample is paid a fee for each validation
- Miners get paid for recording collateral transactions
- Small fee for trade proposals to avoid DDOS

## Order Book

- Order book is distributed across exchange brokers via DHT.
