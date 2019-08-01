## General Principles

DEX supports both “spot” (market) and “strike” (limit) orders from POS and general users. 
Collateral transaction is required prior to the order in order to avoid “bad faith” and DDOS orders.

At this stage trust is implied between client and their exchange broker.  In the future, this relationship will become trustless.


## Client UI

![2019-08-01_13-26-06](https://user-images.githubusercontent.com/45132833/62307369-28890400-b48c-11e9-84b9-ba19dc36cdd3.jpg)


## Client

- Connects to the Exchange Broker (EB)
- Forms transactions
- Executes trade logic
- Monitors trade results
- Posts collateral transactions and requests collateral refunds (?)



## Exchange Broker 
- Is a Supernode module
- Handles trading related communication (broadcast & receive)
- Sets up and holds multi-signature (MS) wallets (MS wallet requires M:N signatures, with N being the SN sample size and signatories being the SN’s)
- Provides an RPC API for the client

## Supernodes from the sample

- Each SN validates the transaction by scanning the alt chain


## Workflow:

![2019-08-01_14-14-06](https://user-images.githubusercontent.com/45132833/62307370-29219a80-b48c-11e9-8ac9-8bc6451ae67f.jpg)


1. Client 1 creates a new trade proposal and sends to the exchange broker (EB1) it’s connected to:
   - Parameters: buy or sell, pair, limit or market price, “good for” time, whether partial is ok, priority, valid to,  # of confirmations required
2. EB1 prepares to announce the transaction to the network
   - Forms a SN sample
   - Creates a MS wallet 1
   - Instructs the Client to deposit collateral
3. Client 1 deposits grft collateral for the transaction
4. EB1 broadcasts the trade across the network and listens for responses
   - Includes parameters: buy or sell, pair, limit or market price, “good for” time, whether partial is ok, # of confirmations required
5. On accept response, EB1
   - Confirms if exchange tx still valid or been cancelled
   - Forwards MS wallet address to EB1 (with view key?)
6. EB2 prepares to execute transaction
   - Finds SN sample
   - Creates MS wallet
   - Client 2 forwards the collateral to MS wallet
   - Forwards info about collateral to EB1
7. Client1 sends alt-chain tx to Wallet 2
   - Sends altchain txid to EB2
8. Client2 sends alt-chain tx to Wallet 1 after receiving alt-chain tx from Client1(?)  
   - Sends altchain txid to EB1
9. Settlement
   - Client 1 requests refund of collateral after n confirmations have passed, or
   - Client 1 requests collateral forward if they haven’t received the funds on alt chain and requests to return a refund of his collateral
     - EB 1 creates tx and coordinates with SN’s for validation and signing
   - Client 2 requests refund of collateral after n confirmations have passed, or
   - Client 2 requests collateral forward if they haven’t received the funds on alt chain
     - EB2 creates tx and coordinates with SN’s for validation and signing


## Sample Selection:

- M:N (N=8, M=5)
- Make ID for the Sample (for following searches)
- Each side’s broker selects their own sample to avoid sample manipulation (?)


## Economics
- Exchange Brokers earn money through the “spread” (the difference between the rate they buy and sell currency at)
- The SN Sample is paid a fee for each validation
- Miners get paid for recording collateral transactions
