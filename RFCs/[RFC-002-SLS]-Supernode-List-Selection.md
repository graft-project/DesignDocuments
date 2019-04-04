# Supernode List Selection
## RFC, version 1.1

### Change Log
| Date       | RFC Version |
|------------|-------------|
| 02/08/2019 | 1.0         |
| 04/04/2019 | 1.1         |

***

## Related Documents

 * [[RFC-001-GSD] General Supernode Design](https://github.com/graft-project/DesignDocuments/blob/master/RFCs/%5BRFC-001-GSD%5D-General-Supernode-Design.md)

***

## RTA Payment ID Generation
**RTA Payment ID** is unique RTA payment identifier for payment during Real-Time Authorization. Also, RTA payment ID is used in Auth Sample selection algorithm as a unique and randomized vector of indexes on the Blockchain-based List. For more details see "Selecting Auth Sample List" section.
In order to ensure unique and random RTA Payment ID, and, therefore, prevent possible fraud on the side of PoS Proxy supernode, it's calculated by other supernodes as a strong hash of PoS Proxy public identification key, from a pair, generated for each payment. Such an RTA Payment ID generation algorithm has several advantages:

* RTA Payment ID cannot be counterfeit using, for example, indexes of supernodes known by PoS Proxy Supernode;
* RTA Payment ID is unique since PoS Proxy needs a new one-time identification key, as well as an RTA payment ID, for each RTA payment;
* RTA Payment ID is random and unpredictable, therefore, an auth sample will be unique for each RTA;
* The correctness of RTA Payment ID can be always easily checked by generating it from one-time public identification key, which included into RTA transaction. 

## Supernode List Updating
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
>
> **NoT** - Number of Tiers - The number of supernode tiers. 

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

## Selecting Blockchain-based List

**Blockchain-based List** is selected for each supernode tier. At a given block and tier, graftnode forms current **Blockchain-based List** from the previous **Blockchain-based List** and **Full Blockchain-based List** by:

1. Getting block hash (hexadecimal encoded) of the block, we build Blockchain-based List for, and initializing seed for pseudo-random generator (for example, Mersenne Twister) using this block hash.
2. Getting **Blockchain-based List** for the current supernode tier and the previous block hash. The following steps describe the process:
    1. Select supernodes from the previous **Blockchain-based List** that have stakes and belong to the tier. Hereafter this list is called **Active Blockchain-based List**.
    2. Sort the supernodes by age of stake (AoS). If ages of the stake are equal, the corresponding supernodes are sorted by their public identification keys.
    3. Get a random index, normalized by the size of **Active Blockchain-based List**, and select a supernode by this index from **Active Blockchain-based List**. If the selected supernode is already on the list, it will be skipped. The process continues until the graftnode selects **PBLSize** supernodes or no available supernodes remains.
3. Getting **Full Blockchain-based List**. The following steps describe the process:
    1. Select all supernodes from **Full Blockchain-based List** that have stakes and belong to the current supernode tier. Hereafter this list is called **Active Full Blockchain-based List**.
    2. Sort the selected supernodes from **Active Full Blockchain-based List** by age of stake (AoS). If ages of the stake are equal, the corresponding supernodes are sorted by their public identification keys.
    3. Get a random index, normalized it by the size of **Active Full Blockchain-based List**, and select a supernode by this index from **Active Full Blockchain-based List**. If the selected supernode is already on the list, it will be skipped. The process continues until graftnode selects **RBLSize** supernodes or the number of selected supernodes from previous **Active Blockchain-based List** and **Active Full Blockchain-based List** is equal to **BSLSize**.

## Selecting Auth Sample List

To form an **Auth Sample list**, supernode performs the following operations for each supernode tier:

1. Retrieves **Blockchain-based List** for lowest supernode tier per required blockchain height from graftnode.
2. Using the retrieved **Blockchain-based List** and **Announcement List**, supernode selects **Auth Sample List** by:
    1. Sorting **Blockchain-based List** by age of stake (AoS). If ages of the stake of some supernodes are equal, these supernodes sorted by their public identification keys.
    2. Selecting the first **SASSize / NoT** of supernodes from sorted **Blockchain-based List**. If some of these supernodes are inactive, supernode skips them and gets next until it selects **SASSize / NoT** supernodes.
    3. Retrieving **RTA Payment ID** (hexadecimal encoded) and initializing pseudo-random generator (for example, Mersenne Twister) with it.
    4. Getting a random index, normalized it by the size of **Blockchain-based List** and selecting supernode by this number from **Blockchain-based List**. If selected supernode is already present in the list, supernode skips it.
    5. Supernode continues this process until it selects **RASSize / NoT** supernodes.
3. Continues process until supernode selects auth sample supernodes from each supernode tiers.
    1. If **Blockchain-based List** for some supernode tier hasn't enough auth sample supernodes, they must be selected from higher supernode tier. 
    2. If this tier also hasn't enough supernodes, they must be selected from any supernode tier which has enough supernodes and has the highest level from all available tiers. 
4. The created list of supernodes gets as auth sample supernode list and used for RTA transaction validation.

This algorithm has the following advantages:

1. Consistency, since it based on consistent Blockchain-based List.
2. Auth Sample is unique for each payment since it depends from payment id.
3. Auth Sample can be always validated by any graftnode or supernode using a sequential selection algorithm.
