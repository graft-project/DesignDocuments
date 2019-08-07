# General Principles 
GRAFT Network DEX is intended to facilitate a decentralized exchange infrastructure for the GRAFT Payment Network as well as function on its own providing fast, binding, collateralized cross-chain exchange capabilities to the trading parties.

Parameters:
Non-custodial (alt-chain funds remain with the owners)
Cross-chain (trades are executed among different blockchains with no inherent limitations other than network support)
Atomic swap or equivalent (the trade transaction has to be executed as one unit or with ability to roll back / receive refund when the trade doesn’t conclude)
Fast execution (so it can be folded into an RTA transaction)
Can execute both spot (market) and strike (limit) transactions

Concept:
GRAFT DEX as decentralized exchange secured by using fast (RTA) $GRFT based collateral transactions deposited by both trading sides into a multi signature wallet, validated by the Supernode validators.

Workflow:





Client1/EB1->SN_network: Select an SN sample
Client1/EB1->SN_network: Sends a trade proposal (spot or strike), trade proposal fee and trade parameters (exchange terms, # confirmations, ttl, etc) via SN network
EB2/Client2->Client1/EB1: Sends proposal / agrees to a trade
Client1/EB1->Client1/EB1: Selects offer(s)
Client1/EB1>SN_network: Create MS wallet with SN sample as signatories
Client1/EB1->SN_network: Sends collateral to MS wallet (RTA) (alt-chain receive address in metadata)

Client1/EB1->EB2/Client2: Acknowledges offer, forwards MS wallet address
EB2/Client2->SN_network: Sends collateral to MS wallet (RTA) (alt-chain receive address in metadata)
Client1/EB1->SN_network: Verifies collateral (via SN sample)

Client1/EB1->EB2/Client2: Send alt-chain tx 
Client1/EB1->SN_network: Forward data about alt-chain tx to SN validator sample, 
SN_network: Verifies and records 1’s collateral tx with data (alt-chain txid, cross-collateral, trade id, trade params) */fee
EB2/Client2->SN_network: Verifies collateral (via SN sample)
EB2/Client2->Client1/EB1: Send alt-chain tx
EB2/Client2->SN_network: Forward data about alt-chain tx to SN validator sample
SN_network: Verifies and records 2’s collateral tx with data (alt-chain txid, cross-collateral, trade id, trade params) */fee


Settlement:
Client1/EB1->SN_network: Requests refund or forward of the collateral
SN_network->Client1/EB1: Validates and forward collateral */fee
EB2/Client2->SN_network: Requests refund or forward of the collateral, along with validation fee
SN_network->EB2/Client2: Validates and forward collateral */fee


Assumptions:
For this phase, client, exchange broker, and (proxy or full) SN maintain a trusted relationship (owned by the same entity).  In the future it will be possible to separate them.


Client UI



Client / Exchange Broker

Connects to the Exchange Broker (EB)
Forms transactions
Executes trade logic
Generates trade offers / responses
Monitors trade results
Posts collateral transactions and requests collateral refunds 

Exchange Broker
Is a Supernode module
Handles trading related communication (broadcast & receive)
Provides an RPC (HTTPs?) API for the client
Participates in distributed order book

Supernodes from the sample
Collect info about the trade transaction from the parties
Validate the trades on request by matching against agreed parameters
Process collateral forward and refund requests

Sample Selection:
M:N (TBD)
Based on current Auth sample selection (? TBD)
Has to survive 100+ blocks
Economics
Exchange Brokers earn money through the “spread” (the difference between the rate they buy and sell currency at)
The SN Sample is paid a fee for each validation
Miners get paid for recording collateral transactions
Small fee for trade proposals to avoid DDOS

Order Book
Order book is distributed across exchange brokers via DHT
