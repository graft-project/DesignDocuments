# Disqualification Flow
## Draft, version 0.2

### Change Log
| Date       | RFC Version |
|------------|-------------|
| 03/26/2019 | 0.1         |
| 04/03/2019 | 0.2         |
| 04/04/2019 | 0.3         |

***

## Related Documents

 * [[RFC-002-SLS] Supernode List Selection](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-002-SLS%5D-Supernode-List-Selection.md)
 * [[RFC-003-RTVF] RTA Transaction Validation Flow](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-003-RTVF%5D-RTA-Transaction-Validation-Flow.md)

***

## Qualification Set

**Qualification Set** is a set of SNs that is a subset of another set **Blockchain-based List (BBL)**. BBL is is a list of all SNs with valid stakes for particular height of the blockchain. Qualification Set is generated randomly, but deterministically from another subset such that any participant can generate the same subset and check that entities really in the subset.

## Qualification Sample

**Qualification Sample** is a Qualification Set, it is a special sample of supernodes which collectively make a decision whether or not a supernode (SN) is qualified to participate in an authorization sample and submits a disqualification transaction to the blockchain in case the supernode is not qualified.

There are two types of SN qualifications:

*   **Authorization Qualification**. It based on **Authorization Qualification Sample (AuthS)**, a Qualification Sample that effectively is the currently active authorization sample. Its purpose is to filter out supernodes from the authorization sample that did not satisfy the authorization sample requirements. AuthS is exactly the same as Auth Sample that participate in a RTA transaction. AuthS is generated basing on RTA payment ID, and BBL which itself depends on current blockchain height and state of stakes for that height. SNs that make decisions of qualifications and those to be qualified, both mush be in AuthS.

*   **Blockchain-based Qualification**. It uses two Qualification Sets: **Qualification Candidate List (QCL)** SNs that are checked to be qualified, and **Blockchain-based Qualification Sample (BBQS)** SNs that make descisions about qualifications. Both QCL and BBQS are generated from current BBL. The decision is driven by activity proof from a SN in QCL. Supernodes that haven't sent the proof to those in BBQS should be disqualified from participation in authorization samples over a number of blocks.

## Disqualification Transaction

**Disqualification Transaction** is a special type of transaction which reports a supernode that isn't meeting its criteria for 
participation in authorizations. The **disqualification transaction** has the following attributes:

1. The transaction cannot contain any key images.
2. The transaction has zero mining fee.
3. Field **transaction_extra** of the transaction must contain: 
    1. At least one value or list of public identification keys of supernodes to be disqualified.
    2. Data for validating qualification sample such as current blockchain height and hash. This data can vary depending on the qualification sample which generates **_disqualification transaction_**.
    3. List of public identification keys of supernodes from the qualification sample voted for disqualification and their signatures of the data in 1. and 2.. The signature signs by supernode private identification key.

## Disqualification Transaction Validation

### Disqualification Transaction Validation by the Member of Authorization Qualification Sample

To validate the disqualification transaction, each supernode which participates in the **Authorization Qualification Sample** should:

*   check if the disqualification transaction contains key images. If the disqualification transaction contains key images, it 
should be rejected.
*   check if the disqualification transaction is the zero-fee transaction. If the disqualification transaction isn't a zero-fee 
transaction, it should be rejected.
*   check correctness of the data for validation qualification sample. If data is incorrect, the disqualification transaction should 
be rejected.
*   check correctness of qualification supernodes added to the transaction. For this, supernode should
    *   select the qualification sample using qualification sample data
    *   validate public identification keys of qualification supernodes, added to the disqualification transaction, with the selected 
    qualification sample.
*   check correctness of the list of supernodes to be disqualified.

### Blockchain Disqualification Transaction Validation

To validate a disqualification transaction, graftnode should:

*   check if the disqualification transaction contains key images. If the disqualification transaction contains key images, it 
should be rejected.
*   check if the disqualification transaction is the zero-fee transaction. If the disqualification transaction isn't a zero-fee 
transaction, it should be rejected.
*   check correctness of the data for validation qualification sample (the data depends on the type of qualification sample). If 
data is incorrect, the disqualification transaction should be rejected.
*   check correctness of qualification supernodes if they were added to the transaction. For this, graftnode should
    *   select the qualification sample using qualification sample data,
    *   validate public identification keys of qualification supernodes, which added to the disqualification transaction, with 
    the selected qualification sample.
*   check correctness of the list of supernodes to be disqualified. Depending on the type of the disqualification transaction, 
this validation can differ:
    *   for the disqualification transaction from **Authorization Qualification Sample**, disqualified supernodes should be 
    part of the authorization sample. To validate them, graftnode should
        *   select the qualification sample using qualification sample data,
        *   validate if disqualified-to-be supernodes are a part of the selected qualification sample.
    *   for the disqualification transaction from **_Blockchain-based Qualification Sample,_** disqualified supernodes should 
    be part of the **Full Blockchain-based List** or next **Blockchain-based List**. To validate them, graftnode should
        *   select the **Full Blockchain-based List** or next **Blockchain-based List** using qualification sample data,
        *   validate if disqualified-to-be supernodes are a part of the selected list.
*   validate signatures of qualification supernodes. If some of these signatures are invalid, the disqualification transaction 
should be rejected.

## Authorization Sample Qualification Flow

**Authorization Sample Qualification Flow** is used by **Authorization Sample** to provide authorization supernode qualification 
after RTA Transaction validation and consists of the following steps:

1. **Authorization Qualification Sample Initialization.**
    1. After a supernode from the authorization sample successfully submits RTA transaction to the blockchain, it checks signatures
    in the RTA Transaction and adds all public identification keys of unresponsive supernodes to **Disqualification List**, and all
    public identification keys of active supernodes to **Authorization Qualification Sample**.
    2. In case a supernode in the authorization sample reaches validation timeout state, the supernode creates a "status fail" 
    message, adds signatures in the RTA Transaction, as well as collects all public identification keys of authorization sample 
    supernodes that missed validation to **Disqualification List**.
2. **Disqualification Transaction Creation.** To create a disqualification transaction, supernode should add the following data to 
the transaction:
    1. Qualification sample validation data: payment block number, payment block hash and RTA payment ID.
    2. The list of public identification keys of active supernodes from Authorization Qualification Sample. In this case, active 
    supernode means an authorization supernode sent its signature for RTA Transaction validation.
    3. The list of public identification keys of supernodes from **Disqualification List**.
3. **Disqualification Transaction Signing.**
    1. After disqualification transaction creation, supernode signs it as a regular transaction, using empty randomly created wallet.
    2. Then supernode encrypts transaction for supernodes from **Authorization Qualification Sample** using 
    [Multiple Recipients Message Encryption](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md#multiple-recipients-message-encryption) 
    and multicasts it over the network.
    3. When each supernode in the **Authorization Qualification Sample** receives disqualification transaction, it
        1. validates the transaction,
        2. signs if everything is correct,
        3. encrypts it for supernodes from **Authorization Qualification Sample** using [Multiple Recipients Message Encryption]
        (https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md#multiple-recipients-message-encryption), and
        4. multicasts to the network.
    4. Supernode issued disqualification transaction collects the signatures; and, upon receiving signatures from all supernodes 
    from **Authorization Qualification Sample**, sends disqualification transaction to the blockchain.

## Blockchain-based List Qualification Flow

**Blockchain-based List Qualification Flow** is used by current **Blockchain-based List** to provide supernode disqualification 
for supernodes from **Full Blockchain-based List**. **Blockchain-based List Qualification Flow** consists of the following steps:

1. Blockchain-based List Qualification Sample Initialization
    1. Since **Blockchain-based Qualification Sample** is always based on the current **Blockchain-based List**, the qualification 
    process starts upon a new active **Blockchain-based List** is selected.
    2. **Blockchain-based Qualification Sample** is selected, using a random and fixed subset of supernodes from the current 
    **Blockchain-based List**, based on the block hash of the next (or few next) **Blockchain-based List(s)** to be validated.
    3. **The Qualification Candidates List** - the list of the supernodes to be checked for disqualification - is selected randomly, 
    using a fixed subset of supernodes from the next (or few next)**Blockchain-based List(s)** based on block hash(es) corresponding 
    to the **Blockchain-based List(s)**.
2. Qualification Poll
    1. To vote for disqualification of a supernode, qualification supernode should poll all supernodes, validated by the current 
    qualification sample. For this, qualification supernode:
        1. creates "ping" request including qualification supernode public identification key and data for validation qualification 
        sample,
        2. signs message using qualification supernode private identification key,
        3. encrypts the message for all supernodes, validated by the current qualification sample, using 
        [Multiple Recipients Message Encryption](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md#multiple-recipients-message-encryption), and
        4. multicasts "ping" request over the network.
    5. When supernode receives "ping" request, it:
        1. decrypts "ping" request,
        2. selects a qualification sample,
        3. validates if qualification supernode which sent "ping" request may participate in the qualification sample.
    6. If "ping" request validation passes, the supernode:
        1. creates a "ping" response with its public identification key, current blockchain height and the hash of the last block 
        in the blockchain,
        2. signs it using private identification key,
        3. encrypts it for the qualification supernode, which sent "ping" request, using 
        [Multiple Recipients Message Encryption](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md#multiple-recipients-message-encryption), and
        4. unicasts it to this qualification supernode.
3. Disqualification Voting
    1. Qualification supernode waits for "ping" responses from supernodes to-be-validated during some period of time, after that the 
    qualification supernode:
        1. collects all responses from supernodes,
        2. validates responses by checking the public identification key, current blockchain height and hash of the last block in 
        the blockchain that come in the response, as well as a signature that signed this response,
        3. gets the list of supernodes missed sending "ping" response or failed response validation,
        4. creates disqualification request; the request contains data for validation qualification sample, creator's public 
        identification key, and the list of public identification keys of supernodes to be disqualified (**Disqualification List**),
        5. signs it using qualification supernode private identification key,
        6. encrypts a request for all supernodes in the qualification sample, using 
        [Multiple Recipients Message Encryption](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md#multiple-recipients-message-encryption), and
        7. multicasts it over the network.
    2. When qualification supernode receives disqualification request, it:
        1. decrypts disqualification requests,
        2. validates data for selection qualification sample,
        3. validates that the requester is eligible to participate in a qualification sample,
        4. validates the signature of the requester,
        5. validates that supernodes from the list of disqualified supernodes in the disqualification request are in both 
        **Disqualification CandidatesList** and **Disqualification List** of this supernode,
        6. if all validations passed, supernode accepts disqualification request and stores it.
4. Disqualification Transaction Processing
    1. After a supernode in the qualification sample receives disqualification requests from all other supernodes in the qualification 
    sample, it:
    	  1. collects all supernodes, voted for disqualification by all supernodes in the qualification sample, into 
        **Disqualification List**,
        2. creates disqualification transaction: block number and block hash for **Blockchain-based List** selection, 
        (if needed) block number and block hash for validated **Blockchain-based List**, and the list of public identification 
        keys of supernodes from **Disqualification List**,
        3. signs transaction and
        4. sends it to the blockchain.

## Supernode List Selection Extension

> SDP - Supernode Disqualification Period - The number of blocks during which disqualification rules affecting at the supernode 
if it was added to the disqualification transaction.
>
> ADS - Average Disqualification Score - The value computed as an average of all disqualification scores of some supernode in the 
Blockchain-based Qualification List.
>
> AAoS - Accumulated Age of stake - The value determines the reliability of the stake, based on the stake amount, number of blocks, 
passed after stake activation (as usual AoS) and average disqualification score (ADS), 
`AoS = StakeAmount * StakeTxBlockNumber * (1 - ADS)`.

### Selecting Blockchain-based Disqualification List

To form a blockchain-based disqualification list at a  block, graftnode performs the following operations:

1. Gets three block numbers: current block height (called **BlockHeight**), a block number for the block we build Blockchain-based 
Disqualification List for (called **BDListBlockNumber**), and a block number which is less than **BDListBlockNumber** on Supernode 
Disqualification Period (SDP) (called **SDPBlockNumber**).
2. Scans all blocks in the blockchain from **SDPBlockNumber** to **BDListBlockNumber**, and
    1. collects all disqualification transactions,
    2. checks all disqualified supernodes in all disqualification transactions,
    3. checks type of disqualification transaction,
    4. computes the disqualification score for each disqualified supernode as
    ```
    (DTBlockNumber - SDPBlockNumber) / (2 * (BDListBlockNumber - SDPBlockNumber)),
    ```
    where `DTBlockNumber` is block number of the block that contains corresponding disqualification transaction, and 
    5. adds each disqualified supernode to the blockchain-based disqualification list with the following data:
        1. supernode public identification key,
        2. supernode disqualification score,
        3. type of disqualification transaction,
        4. block number of the block which includes corresponding disqualification transaction. 
    6. If disqualified supernode already presents in the blockchain-based disqualification list, simply adds new data to the related 
    item in the list without duplicating the supernode entry itself.
3. Scans all blocks in the blockchain from **BDListBlockNumber** to **BlockHeight**, and
    1. collects all disqualification transactions,
    2. checks all disqualified supernodes in all disqualification transactions,
    3. checks type of disqualification transaction,
    4. computes the disqualification score for each disqualified supernode as
    ```
    0.5 + (DTBlockNumber - BDListBlockNumber) / (2 * (BlockHeight - BDListBlockNumber)),
    ```
    where `DTBlockNumber` is block number of the block which includes corresponding disqualification transaction, and 
    5. adds each disqualified supernode to the blockchain-based disqualification list with the following data:
        1. supernode public identification key,
        2. supernode disqualification score,
        3. type of disqualification transaction,
        4. block number of the block which includes corresponding disqualification transaction. 
    6. If disqualified supernode already presents in the blockchain-based disqualification list, simply adds new data to the related item in the list without duplicating the supernode entry itself.

### Selecting current Blockchain-based List
##### (After the discussion this section will be moved to [Supernode List Selection](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-002-SLS%5D-Supernode-List-Selection.md) document)
**Blockchain-based List** is selected for each supernode tier. At a given block and tier, graftnode forms current **Blockchain-based List** from the previous **Blockchain-based List** and **Full Blockchain-based List** by:

1. Getting block hash (hexadecimal encoded) of the block, we build Blockchain-based List for, and initializing seed for pseudo-random generator (for example, Mersenne Twister) using this block hash.
2. Getting **Blockchain-based List** for the current supernode tier and the previous block hash. The following steps describe the process:
    1. Select supernodes from the previous **Blockchain-based List** that have stakes and belong to the tier. Hereafter this list is called **Active Blockchain-based List**.
    2. Sort the supernodes by accumulated age of stake (AAoS). If the accumulated ages of the stake are equal, the corresponding supernodes are sorted by their public identification keys.
    3. Get a random index, normalized by the size of **Active Blockchain-based List**, and select a supernode by this index from **Active Blockchain-based List**. If the selected supernode is already on the list, it will be skipped. The process continues until the graftnode selects **PBLSize** supernodes or no available supernodes remains.
3. Getting **Full Blockchain-based List**. The following steps describe the process:
    1. Select all supernodes from **Full Blockchain-based List** that have stakes and belong to the current supernode tier. Hereafter this list is called **Active Full Blockchain-based List**.
    2. Sort the selected supernodes from **Active Full Blockchain-based List** by accumulated age of stake (AAoS). If the accumulated ages of the stake are equal, the corresponding supernodes are sorted by their public identification keys.
    3. Get a random index, normalized it by the size of **Active Full Blockchain-based List**, and select a supernode by this index from **Active Full Blockchain-based List**. If the selected supernode is already on the list, it will be skipped. The process continues until graftnode selects **RBLSize** supernodes or the number of selected supernodes from previous **Active Blockchain-based List** and **Active Full Blockchain-based List** is equal to **BSLSize**.
