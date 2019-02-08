# RTA Payment ID Generation
**RTA Payment ID** is unique RTA payment identifier for payment during Real-Time Authorization. Also, RTA payment ID is used in Auth Sample selection algorithm as a unique and randomized vector of indexes on the Blockchain-based List. For more details see "Selecting Auth Sample List" section.
In order to ensure unique and random RTA Payment ID, and, therefore, prevent possible fraud on the side of PoS Proxy supernode, it's calculated by other supernodes as a strong hash of PoS Proxy public identification key, from a pair, generated for each payment. Such an RTA Payment ID generation algorithm has several advantages:

* RTA Payment ID cannot be counterfeit using, for example, indexes of supernodes known by PoS Proxy Supernode;
* RTA Payment ID is unique since PoS Proxy needs a new one-time identification key, as well as an RTA payment ID, for each RTA payment;
* RTA Payment ID is random and unpredictable, therefore, an auth sample will be unique for each RTA;
* The correctness of RTA Payment ID can be always easily checked by generating it from one-time public identification key, which included into RTA transaction. 

# Supernode List Updating
> **AoS** - Age of stake: the value determines reliability of stake, based on stake amount and number of blocks, passed after stake activation, `AoS = StakeAmount * StakeTxBlockNumber`.
>
> **BSLSize** - Blockchain-based List Size: number of supernodes in Blockchain-based List.
>
> **PBLSize** - Previous Blockchain-based List Size: number of supernodes selected from previous Blockchain-based List.
>
> **RBLSize** - Random Blockchain-based List Size: number of supernodes randomly selected from Full Blockchain-based List, `RBLSize = BSLSize - PBLSize`.
>
> **ASSize** - Auth Sample Size: number of supernodes in Authorization Sample.
>
> **SASSize** - Sequential Auth Sample Size: number of supernodes selected from Blockchain-based List sequentially.
>
> **RASSize** - Random Auth Sample Size: number of supernodes selected from Blockchain-based List randomly, `RASSize = ASSize - SASSize`.

Active supernode lists are defined on several levels:
* **Full Blockchain-based List** is hosted by a graftnode, and includes all supernodes with valid stakes.
* **Blockchain-based** List is hosted by a graftnode, and consists of supernodes with valid stakes, taken from Full Blockchain-based List. This list is completely consistent over the network and can be verified by any graftnode or supernode. To validate an RTA transaction, we select a fixed number of supernodes from Blockchain-based List (this number must be large enough since it will be decreased on the next levels.)
* **Auth Sample List** is the list of supernodes, selected for RTA transaction validation. The current number of supernode in the auth sample is 8.
* Supernode keeps track of other supernodes in the network, using **Announcement List**.

An update of Supernode List is triggered by two events: receiving a new block from graftnode and receiving announce from some supernode.

When a graftnode receives a new block, it must perform the following operations to update Blockchain-based List:
1. Check all transaction in the block and update the supernode list if the block contains new stake transactions.
2. Check trusted re-staking period (TRP) of supernodes in the supernode list and remove supernodes which TRP ended.
3. Check supernode list and start TRP period for supernodes with stake transactions unlocked.

# Selecting Blockchain-based List
To form a Blockchain-based List from Full Blockchain-based List for the current block, graftnode performs the following operations:

1. Gets Full Blockchain-based List and Blockchain-based List for previous block hash.
2. Sorts previous Blockchain-based List by age of stake and randomly selects PBLSize supernodes from it.
3. Sorts Full Blockchain-based List by age of stake and randomly selects RBLSize supernodes from it.

# Selecting Blockchain-based List

To form the blockchain-based list from the full blockchain-based list for some block, graftnode performs the following operations:

1. Gets block hash (hexadecimal encoded) for the block, for which we build Blockchain-based List, and splits it into bytes.
2. Gets Blockchain-based List for the previous block hash:
    1. Sorts previous Blockchain-based List by age of stake.
    2. Gets first PBLSize bytes from splited block hash and selects PBLSize supernodes from it, using these one-byte numbers as indexes. If selected supernode is already selected, increase the index by one and check next supernode in the list until it found unselected supernode. Then graftnode moves to the next one-byte number and searches for next supernode. 
3. Gets Full Blockchain-based List:
    1. Sorts Full Blockchain-based List by age of stake.
    2. Gets next RBLSize bytes from the splited block hash and selects RBLSize supernodes from it, using these one-byte numbers as indexes. If selected supernode is already selected, increase the index by one and check next supernode in the list until it found unselected supernode. Then graftnode moves to the next one-byte number and searches for next supernode.
4. Joins both sublists into one list.

# Selecting Auth Sample List

To form an Auth Sample list, supernode performs the following operations:

1. Retrieves Blockchain-based List per required blockchain height from graftnode.
2. Using the retrieved Blockchain-based List and Announcement List, supernode selects Auth Sample list by:
    1. Sorting Blockchain-based List by age of stake (alternatively, can be based on supernode public identification keys or block number with supernode stake transaction).
    2. Selecting the first SASSize of supernodes from sorted Blockchain-based List. If some of this supernodes are inactive, supernode skips them and gets next until it selects SASSize supernodes.
    3. Extending Blockchain-based List by duplicating supernodes to get at least 256 supernodes in the list.
    4. Retrieving payment id (hexadecimal encoded), splitting it into byte.
    5. Using these one-byte numbers as indexes to select RASSize supernode from extended supernode list. If selected supernode is inactive or has already been selected, supernode increases the index by one and check next supernode in the list until it found active and unselected supernode. Then it moves to the next one-byte number and searches for next supernode.

    Supernode continues this process until it selects the number of supernodes required.
3. The created list of supernodes gets as auth sample supernode list and used for RTA transaction validation.

This algorithm has the following advantages:

1. Consistency, since it based on consistent Blockchain-based List.
2. Auth Sample is unique for each payment since it depends from payment id.
3. Auth Sample can be always validated by any graftnode or supernode using a sequential selection algorithm.