# Supernode RTA and Cryptonode RTA APIs
[1. Supernode Core Interfaces](#supernode-core-interfaces)

   1.1 GetPaymentData - return payment data for given payment id, block number, block hash
   
   1.2 StorePaymentData - handles payment multicast and stores payment data. Called by Cryptonode to it's connected supernode.
   
   1.3 AuthorizeRtaTx - process incoming RTA Tx authorization  (handled by auth sample, PoS Proxy and Wallet Proxy)

   1.4 UpdatePaymentStatus - handles broadcast with payment status update

   1.5 GetAuthSample - returns auth sample for given payment id and block number

   1.6 GetSupernodeList - returns list of valid or all known supernodes

2. Pos/Wallet Interfaces
   
   2.1 Presale - supernode returns auth sample for given payment id. Called by PoS

   2.2 Sale - process sale. Сalled by PoS

   2.3 GetPaymentData - returns payment data for given payment id, block number, block hash. Called by PoS or Wallet

   2.4 Pay - process payment. Called by Wallet
   
   2.5 GetPaymentStatus - returns payment status for given payment id

3. JSON-RPC interfaces on cryptonode side to communicate with supernode

   3.1 Broadcast - broadcasts message  to all the network

   3.2 Multicast - sends message to the group of supernodes

   3.3 Unicast - sends direct message to the specific supernode
   
   3.4 SendSupernodeAnnounce - supernode announces itself, to be broadcasted on p2p network

4. P2P messages
   
   4.1 COMMAND_SUPERNODE_ANNOUNCE - broadcase message, supernode announces itself for the network
   
   4.2 COMMAND_BROADCAST - broadcasted message
   
   4.3 COMMAND_MULTICAST - message to the group of destinations
   
   4.4 COMMAND_UNICAST - direct message to the specific destination

## 1. Supernode Core Interfaces


GetPaymentData - return payment data for given payment id, block number, block hash
Input:

PaymentID - globally unique payment id
BlockNumber - number of block auth sample built for;
BlockHash  - hash of this block in blockchain (probably redudant);
CallbackURI -  optional, if set - do not return response to caller but send it as separate unicast message
Output: 

PaymentData - serialized encrypted payment data
AuthSampeKeys - array of N (N=8) public supernode ids and it's message keys. It's possible to retrieve a wallet address by public id
Notes:

call can be either synchronous or asynchronous
Request:
```
POST /core/get_payment_data
```
Request body:
```
{
    "PaymentId": "payment_id string",
    "BlockNumber": 123456,
    "BlockHash" : "08600e7b9bbcce6ace8dcc786cff8804cd5b4623abc636b11446dea9ab18bfa5"
}
```
Normal response (sync call):
```
HTTP code: 200
{
     "PaymentData": {
           "EncryptedPayment":  "08600e7b9bb...08600e7b9bb", // encrypted serialized payment (incl amount and payment details)
           "AuthSampleKeys" : [  // 8 keys for auth sample members
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            ]
            "PosProxy" : {
                "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805",
                "WalletAddress" : "F8F6UxPemGyfyxxU1yN4Yc52obSq4zboNC47iGSg5YnmSZB1cCHbAUQ47eXJ4xGBQoAMKC7JLhPbSBJrPRQ6tx8C1nUVs78"
            }
     }
}
```
Normal response (async call, from remote peer via unicast message)

HTTP code: 202
Response body: `N/A`

Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
StorePaymentData - handles payment multicast and stores payment data. Called by Cryptonode to it's connected supernode.
PaymentID - globally unique payment id
PaymentData - serialized encrypted payment data (incl amount and purchase details)
MessageKeys - array of encrypted message keys corresponding to each auth sample members. Keys needed to decrypt PaymentData and Amount
Output: 

None
Notes:

Caller doesn't need any response 
Request:
```
POST /core/store_payment_data
```
Request body:
```
{
    "PaymentId": "payment_id string",
 
     "PaymentData": {
           "EncryptedPayment":  "08600e7b9bb...08600e7b9bb", // encrypted serialized payment (incl amount and payment details)
           "AuthSampleKeys" : [  // 8 keys for auth sample members
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            ]
            "PosProxy" : {
                "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805",
                "WalletAddress" : "F8F6UxPemGyfyxxU1yN4Yc52obSq4zboNC47iGSg5YnmSZB1cCHbAUQ47eXJ4xGBQoAMKC7JLhPbSBJrPRQ6tx8C1nUVs78"
            }
     }
}
```
Normal response:
```
HTTP code: 202
```
Response body: `N/A`

Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
AuthorizeRtaTx - process incoming RTA Tx authorization  (handled by auth sample, PoS Proxy and Wallet Proxy)
Input:

TxBlob - encrypted transaction blob (encrypted transaction blob, serialized as hexadecimal)
TxKey  - encrypted transaction private key, serialized as hexadecimal;
MessageKeys - Map of  Auth sample ids and corresponding encrypted message keys to decrypt tx key
Output: 

None (this is async call)

Request
```
POST: /core/authorize_rta_tx
```
Request body:
```
{
    "TxBlob": "08600e7b9bb...08600e7b9bb", // encrypted serialized transaction as hexadecimal string. Includes payment id;
    "TxKey" : "08600....a9ab18bfa5", // encrypted tx key.
    "MessageKeys" : [  // 8 keys for auth sample members + 1 key for pos proxy. There's no wallet proxy here as it the one who originates this message so tx is already signed by wallet proxy
        { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
        { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
        { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
        { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
        { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
        { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
        { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
        { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
        { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
    ]
}
```
Normal response:
```
HTTP code: 202
```
Response body: N/A

Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
UpdatePaymentStatus - handles broadcast with payment status update
Input:

PaymentID - globally unique payment id
Status - payment status, integer constant
PubKey - public id key of supernode who updates status
Signature - message signature (signed message is concatenated PaymentID, Status, PubKey)
Output: 

None 
Request
```
POST: /core/update_payment_status
```
Normal response:
```
HTTP code: 202
```
Response body: `N/A`

Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
GetAuthSample - returns auth sample for given payment id and block number
Input:

PaymentID - globally unique payment id
BlockNumber - block number to build sample on
Output: 

Array of N (N=8) auth sample members
Request
```
POST: /core/get_auth_sample
```
Normal response:
```
HTTP code: 200
```
Response body:
```
{
    "AuthSample" : [
            "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805",
            "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f",
            "55ae4cf3d6b1d0bf3b1e57b1aa17a42871b401261cc50b26dc8f9cd34c40e9cf",
            "4e62b5e4ece7ab7e0f1d4eaca224de6223e80af549137b3a955a9d7e20ee3402",
            "e20b656a844331c4e4fd29d7683a2a0a9b4d54aba050f7a7e9e6450ba89e86bc",
            "34947c1bacb74adfde4e69592aff0134bc9bca6762e65581e22c96ab872859c7",
            "ef433f37cdb08e668e74c978ee8bf8279dda5a4b1841d3d4258ae6b14f4ad91f",
            "179d4b2032d0fc01432b4b66f577d4a427a4aa27c5270a690b19cdf1ec030cb3"
    ]
}
```
Error response:

HTTP code: 500
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
GetSupernodeList - returns list of valid or all known supernodes
Input:

ReturnAll - if set  - all known supernodes returned; otherwise - only valid ones (valid stake and recently updated)
Output: 

Array of Objects representing supernode
Request
```
GET: /core/get_supernode_list/<return_all> 
```
Normal response:
```
HTTP code: 200
```
Response body:
```
"items": [
            {
                "Address": "",
                "PublicId": "7f672501f62b98a56ce98b780b943766f6d8107a45fb1cdfc4eb383146beda34",
                "StakeAmount": 0,
                "StakeFirstValidBlock": 0,
                "StakeExpiringBlock": 0,
                "IsStakeValid": false,
                "BlockchainBasedListTier": 0,
                "AuthSampleBlockchainBasedListTier": 0,
                "IsAvailableForAuthSample": false,
                "LastUpdateAge": 55891
            },
         
            {
                "Address": "F52XRQoyr5KjaTtLnRVLVJLTZz1zsEcScaoa1voYSFWq9Nes1u3rP7vYwYcRiXxpu5VUGffM35G9pBYAPqj1yDa267zooyr",
                "PublicId": "d839513f06234034f7f857164cf8ab9469119d56d18559e9f9942f48a7cee71b",
                "StakeAmount": 500000000000000,
                "StakeFirstValidBlock": 300261,
                "StakeExpiringBlock": 305261,
                "IsStakeValid": true,
                "BlockchainBasedListTier": 1,
                "AuthSampleBlockchainBasedListTier": 1,
                "IsAvailableForAuthSample": true,
                "LastUpdateAge": 65
            }
]
```
Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
## 2. Pos/Wallet Interfaces


Presale - supernode returns auth sample for given payment id. Called by PoS
Input:

PaymentID - randomly generated string (TODO: define what is the form of the string - e.g. GUID, some fixed size hexadecimal/base64/base58 etc)
Output: 

BlockNumber - number of block auth sample built for;
BlockHash  - hash of this block in blockchain (probably redudant);
AuthSample - array of N (N=8) public supernode ids
Notes: 

call is synchronous, returns result to the client immediately


Request:
```
POST: /dapi/v3.0/presale
```
Request body:
```
{
    "PaymentId": "payment_id string"
}
```
Normal response:

HTTP code: 200 
Response body:
```
{
    "BlockNumber": 310173,
    "BlockHash": "66a740478c1d1b01fa90b4d70988a02c781f16470256d8104a71e1838d8038f1",
    "AuthSample": [
        "002fc53f231568a2ecd6387db83e1a0211d5060a91f0534fdf5474a7e9b556ae",
        "217c474a2394584a74051a7dd579c496dd3bc0768e8c56c69659ac93c8f78023",
        "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb",
        "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44",
        "914c13339fdfacdbbebe4c223d1900415432aab24f1f995823286104c7ac9eaa",
        "ba3990991be1db9f05f47179e16ddad7f0959f952be8aaea0ee4d19e8d82ce7f",
        "cdba49cbdece633266681b3c6f80f1085e7b3d3e0cca395d3986d10ab3ea0d6a",
        "ce7cf758df6f2eb7f74d28730078be733cb953bda37a5f6e54ab09140f40e712"
    ],
    "PosProxy": {
        "Id": "cdba49cbdece633266681b3c6f80f1085e7b3d3e0cca395d3986d10ab3ea0d6a",
        "WalletAddress": "F8sHVwypnjw4fVdMvf3iZ5bisEUJ9DcQoZDBuC7aMRMFPCTHrGtPy7CBe5r68qHdjVKgdggg5NGpAD3r6JoQQiMBAoSEo7x"
    }
}
```
Error response:

HTTP code: 500
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```

Sale - process sale. Сalled by PoS
Input:

PaymentData - encrypted payment data blob (list of items, amounts, any other info)
Amount - encrypted transaction amount in atomic units 
MessageKeys - array of encrypted message keys (to decrypt PaymentData and Amount)
Output: 

Accepted or error 

POST: /dapi/v3.0/sale/

Payload
```
{
    "PaymentID" : <globally unique payment id>
    "PaymentData": {
      "EncryptedPayment":  "08600e7b9bb...08600e7b9bb", // encrypted serialized payment (incl amount and payment details)
      "AuthSampleKeys" : [  // 8 keys for auth sample members
            { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
            { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
            { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
            { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
            { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
            { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
            { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
        ]
        "PosProxy" : {
            "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805",
            "WalletAddress" : "F8F6UxPemGyfyxxU1yN4Yc52obSq4zboNC47iGSg5YnmSZB1cCHbAUQ47eXJ4xGBQoAMKC7JLhPbSBJrPRQ6tx8C1nUVs78"
        }
     }
}
```
Normal response:

HTTP code: 202 accepted
Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
GetPaymentData - returns payment data for given payment id, block number, block hash. Called by PoS or Wallet
Input:

PaymentID - globally unique payment id
BlockNumber - number of block auth sample built for;
BlockHash  - hash of this block in blockchain (probably redudant);
Output: 

PaymentData - serialized encrypted payment data
AuthSampeKeys - array of N (N=8) public supernode ids and it's message keys. It's possible to retrieve a wallet address by public id
Notes:

call is synchronous
Request:
```
POST /dapi/v3.0/get_payment_data
```
Request body:
```
{
    "PaymentId": "payment_id string",
    "BlockNumber": 123456,
    "BlockHash" : "08600e7b9bbcce6ace8dcc786cff8804cd5b4623abc636b11446dea9ab18bfa5"
}
```
Normal response  (payment data found on the given supernode):
```
HTTP code: 200
{
     "PaymentData": {
           "EncryptedPayment":  "08600e7b9bb...08600e7b9bb", // encrypted serialized payment (incl amount and payment details)
           "AuthSampleKeys" : [  // 8 keys for auth sample members
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
                { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
                { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
                { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
                { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            ]
            "PosProxy" : {
                "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805",
                "WalletAddress" : "F8F6UxPemGyfyxxU1yN4Yc52obSq4zboNC47iGSg5YnmSZB1cCHbAUQ47eXJ4xGBQoAMKC7JLhPbSBJrPRQ6tx8C1nUVs78"
            }
     }
}
```
Pending response (payment data was not found on given supernode, supernode sends request to another supernode)
```
HTTP code: 202
```
Response body: N/A

Error response (payment id already known as failed):
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```

Pay - process payment. Called by Wallet
Input:

TxBlob - encrypted transaction blob
TxKey - encrypted transaction private key
MessageKeys - encrypted message keys (multiple recipients encryption, to decrypt transaction and tx private key)
Output: 

Accepted or error 

POST: /dapi/v3.0/pay/

Payload
```
{
    "TxBlob": "08600e7b9bb...08600e7b9bb", // encrypted serialized transaction as hexadecimal string. Includes payment id;
    "TxKey" : "08600....a9ab18bfa5", // encrypted tx private key.
    "MessageKeys" : [  // 8+2 keys for auth sample members and proxy supernodes
            { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
            { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
            { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
            { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            { "id" : "1f0a6a65fc768348f781b0ad58dcc910408c3bd85cfab9d451577f5a98261805", "key": "7683a2a0..4c40e9cf" },
            { "id" : "96afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
            { "id" : "54749bad9925d34e414062a4e3cf3991e1a1ee5c778fc6c44a53c11f55cafe44", "key": "7683a2a0..4c40e9cf" },
            { "id" : "25b316d25e6c2dd8dd60fd983de9fbd5a9bb1fcf96d65bbb1c295708bafa00cb", "key": "7683a2a0..4c40e9cf" },
            { "id" : "45afb7860aa4bb758aae24a5f182b4e1f641d782c1fc772d3228da5c6108e57f", "key": "7683a2a0..4c40e9cf" },
 
    ]
}
```
Normal response:
```
HTTP code: 202 accepted
```
Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
GetPaymentStatus - returns payment status for given payment id
Request:
```
POST: /dapi/v3.0/get_payment_status/
```
Payload
```
{
    "PaymentId" : "payment_id string"
}
```
Normal response:
```
HTTP code: 200
```
Response body:
```
{
    
   "Status" : 1 (Waiting) | 2 (InProgress) | 3 (Success) | 4 (Fail) | 5 (RejectedByWallet) | 6 (RejectedByPOS)
    
}
```
Error response:
```
HTTP code: 500
```
Response body:
```
{
    "code": <error_code>,
    "message": "Error description"
}
```
## 3. JSON-RPC interfaces on cryptonode side to communicate with supernode
Broadcast - broadcasts message  to all the network
This call will be called by supernode in case sale details with given payment id wasn't found in local supernode (supernode handling "sale_details" call from wallet)

Request URI:
```
/json_rpc/rta
```
Request payload:
```
{
    "json_rpc" : "2.0",
    "method" : "broadcast",
    "id" : "<unique_id>",
    "params": {
        "sender_address" : "public address of the sender supernode",
        "callback_uri" : "URI to call on remote supernode. We don't handle method names currently, all the URIs are unique"
        "data" : "serialized payload to pass to remote URI"
    }
}
```

Normal response:
```
{
    "jsonrpc": "2.0",
    "id": "0",
    "result": {
        "status" : <integer value, 0 is OK>
    }
}
Error response:
```
{
    "id": "0",
    "jsonrpc": "2.0",
    "error": {
        "code": "code",
        "message": "message"
    }
}
```
Multicast - sends message to the group of supernodes
Request URI:
```
/json_rpc/rta
```
Request payload:
```
{
    "json_rpc" : "2.0",
    "method" : "multicast",
    "id" : "<unique_id>",
    "params": {
        "receiver_addresses" : [ "address1", "address2", ... , "addressN" ],
        "sender_address" : "address of sending supernode",
        "callback_uri" : "URI to call on remote supernode. We don't handle method names currently, all the URIs are unique",
        "data" : "serialized payload to pass to remote URI"
    }
}
```
Normal Response payload:
```
{
    "jsonrpc": "2.0",
    "id": "0",
    "result": {
        "status" : <integer value, 0 is OK>
    }
}
```
Error response payload:
```
{
    "id": "0",
    "jsonrpc": "2.0",
    "error": {
        "code": "code",
        "message": "message"
    }
}
```
Unicast - sends direct message to the specific supernode
Request URI:
```
/json_rpc/rta
```
Request payload:
```
{
    "json_rpc" : "2.0",
    "method" : "unicast",
    "id" : "<unique_id>",
    "params": {
        "receiver_address" : "destination address"
        "sender_address" : "address of sending supernode",
        "callback_uri" : "URI to call on remote supernode. We don't handle method names currently, all the URIs are unique",
        "data" : "serialized payload to pass to remote URI"
    }
}
```

Normal Response payload:
```
{
    "jsonrpc": "2.0",
    "id": "0",
    "result": {
        "status" : <integer value, 0 is OK>
    }
}
```
Error response payload:
```
{
    "id": "0",
    "jsonrpc": "2.0",
    "error": {
        "code": "code",
        "message": "message"
    }
}
```
SendSupernodeAnnounce - supernode announces itself, to be broadcasted on p2p network
Request URI:
```
/json_rpc/rta
```
Request payload:
```
{
    "json_rpc" : "2.0",
    "method" : "send_supernode_announce",
    "id" : "<unique_id>",
    "params": {
         
        "supernode_public_id" : "<hexadecimal public key>"
        "height"  : <blockchain height at the moment announce was generated>,
        "signature" : "<hexadecimal signature>",
        "network_address" : <network address (ip_address:port) of the supernode>
    }
}
```
Normal Response payload:
```
{
    "jsonrpc": "2.0",
    "id": "0",
    "result": {
        "status" : <integer value, 0 is OK>
    }
}
```
Error response payload:
```
{
    "id": "0",
    "jsonrpc": "2.0",
    "error": {
        "code": "code",
        "message": "message"
    }
}

## 4. P2P messages
COMMAND_SUPERNODE_ANNOUNCE - broadcase message, supernode announces itself for the network

struct COMMAND_SUPERNODE_ANNOUNCE
{
      const static int ID = P2P_COMMANDS_POOL_BASE + 20;
 
      struct request : public cryptonote::COMMAND_RPC_SUPERNODE_ANNOUNCE::request
      {
          uint64_t hop;
 
          BEGIN_KV_SERIALIZE_MAP()
            KV_SERIALIZE(supernode_public_id)
            KV_SERIALIZE(height)
            KV_SERIALIZE(signature)
            KV_SERIALIZE(network_address)
            KV_SERIALIZE(hop)
          END_KV_SERIALIZE_MAP()
      };
 
      struct response : public cryptonote::COMMAND_RPC_SUPERNODE_ANNOUNCE::response { };
};
```

COMMAND_BROADCAST - broadcasted message
```
struct COMMAND_BROADCAST
{
      const static int ID = P2P_COMMANDS_POOL_BASE + 21;
 
      struct request : public cryptonote::COMMAND_RPC_BROADCAST::request
      {
          uint64_t hop;
          std::string message_id;
 
          BEGIN_KV_SERIALIZE_MAP()
            KV_SERIALIZE(sender_address)
            KV_SERIALIZE(callback_uri)
            KV_SERIALIZE(data)
            KV_SERIALIZE(wait_answer)
            KV_SERIALIZE(hop)
            KV_SERIALIZE(message_id)
          END_KV_SERIALIZE_MAP()
      };
 
      struct response : public cryptonote::COMMAND_RPC_BROADCAST::response { };
};
```

COMMAND_MULTICAST - message to the group of destinations
```
struct COMMAND_MULTICAST
{
      const static int ID = P2P_COMMANDS_POOL_BASE + 22;
 
      struct request : public cryptonote::COMMAND_RPC_MULTICAST::request
      {
          uint64_t hop;
          std::string message_id;
 
          BEGIN_KV_SERIALIZE_MAP()
            KV_SERIALIZE(receiver_addresses)
            KV_SERIALIZE(sender_address)
            KV_SERIALIZE(callback_uri)
            KV_SERIALIZE(data)
            KV_SERIALIZE(wait_answer)
            KV_SERIALIZE(hop)
            KV_SERIALIZE(message_id)
          END_KV_SERIALIZE_MAP()
      };
 
      struct response : public cryptonote::COMMAND_RPC_MULTICAST::response { };
};
```
COMMAND_UNICAST - direct message to the specific destination
```
struct COMMAND_UNICAST
{
      const static int ID = P2P_COMMANDS_POOL_BASE + 23;
 
      struct request : public cryptonote::COMMAND_RPC_UNICAST::request
      {
          uint64_t hop;
          std::string message_id;
 
          BEGIN_KV_SERIALIZE_MAP()
            KV_SERIALIZE(receiver_address)
            KV_SERIALIZE(sender_address)
            KV_SERIALIZE(callback_uri)
            KV_SERIALIZE(data)
            KV_SERIALIZE(wait_answer)
            KV_SERIALIZE(hop)
            KV_SERIALIZE(message_id)
          END_KV_SERIALIZE_MAP()
      };
 
      struct response : public cryptonote::COMMAND_RPC_UNICAST::response { };
};
```
