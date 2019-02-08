# Supernode Modes
Each supernode can work in one of the modes:
* Proxy
* Authorization
* Full

To operate in any of the above modes, the supernode needs to communicate to a graftnode.
### Proxy Supernode
In the Proxy mode, the supernode is open to serve clients' requests, charging the clients an optional fee for this activity. Public Proxy supernode should have a public IP address, available to the clients. Proxy supernode does not need to prove its stake to the network, neither it participates in an authorization sample. It may, however, participate in RTA transaction validation if chosen to be a PoS proxy.

### Authorization Supernode
Authorization mode allows a supernode to participate in authorization samples and validate RTA transactions. The supernode charges the clients an optional fee for this activity. Authorization supernode should prove its stake to the network. The supernode is not required to be publicly accessible. The access, however, is required if supernode's owner wants to manage the supernode remotely.

### Full Supernode
Full mode is a combination of the two above modes.

# Supernode Prerequisites
Upon start, each supernode should be given a public wallet address that is used to collect service fees and may be a receiver of a stake transaction.

A supernode generates temporal Supernode Identification Keypair (SIK), which used for message encryption and signing RTA validation signature. The supernode must regenerate the key pair per each stake renewal. Public Supernode Identification Key is also used for creating communication channels (tunnels) between supernodes.

> SVP - Stake Validation Period: The number of blocks after block with the stake transaction, which needs to generate to activate stake.
>
> SLP - Stake Locking Period: The number of blocks on which stake transaction will be locked and stake will be active.
>
> TRP - Trusted Re-staking Period: The number of blocks during which supernode is allowed to participate in RTA validation even if its stake transaction already unlocked.

# New Supernode Registration
To register a supernode, an owner needs to run the supernode, providing the public wallet address as a configuration parameter. Not storing any wallet related private information on supernode is a more secure approach, but it doesn't allow automatic re-staking. For more details about the renewal of stake see "[Supernode Re-staking Process](https://github.com/graft-project/graft-ng/wiki/%5BRFC-001-GSD%5D-General-Supernode-Design#supernode-re-staking-process)" section.

To configure the supernode in authorization mode, the owner must generate a stake transaction and submit it to the blockchain. Stake transaction must include the following data:

* the receiver of this transaction must be supernode's public wallet address;
* the amount must be equal to or higher than a particular supernode tier;
* unlock_time must be set to the number of blocks it takes stake transaction to be unlocked;
* tx_extra must contain transaction private key, which will be used to validate the amount sent;
* tx_extra must contain supernode public wallet address;
* tx_extra must contain supernode public identification key.

# Supernode Re-staking Process
Since stake transaction is locked only for a fixed period of time, supernode owner needs to submit new stake transaction periodically. The owner must retrieve from the supernode a new identification key and submit new stake transaction with the required data. During stake transaction renewal, supernode owner can change supernode public address (in this case, the supernode needs to be updated), supernode tier (can be changed using another stake transaction amount) and stake locking period (currently we plan to support only a fixed locking period, in future our plan is to add an option to specify a locking range and allow dynamic locking period inside this range.)

# Stake Locking Period (SLP) and Trusted Re-staking Period (TRP)
Stake Locking Period (SLP) determines the number of blocks when a stake transaction is locked. SLP starts at the block that contains the stake transaction. So, if the stake transaction is included in the block H, and SLP is T blocks, then stake transaction will be unlocked at block H + T. Since the blockchain must guarantee the locking of stake transactions, we consider the stake becomes active only after a number of blocks. We call this number Stake Validation Period (SVP). SVP increases the chances the stake transaction will not be included into an alternative chain.

Since SVP creates a downtime period between unlocking previous and validation of a new stake transaction, for every supernode with the previous stake transaction unlocked, we introduce Trusted Re-staking Period (TRP). TRP determines the number of blocks during which supernode is allowed to participate in RTA validation even if it has no locked stake. This also gives supernode owners some time to submit a new stake transaction for their supernode. If during TRP supernode owner doesn't renew its stake transaction, the supernode will be removed from active supernode list and will not be able to participate in RTA validation.

[[images/graft-periods.jpg|Stack Periods]]

# Supernode Status Announcement
During its lifespan, each supernode must communicate its status other supernodes over the P2P network. Being selected to participate in an authorization sample, communication with other members of the sample adds on. Although Gossip protocol has no guaranty over timed message delivery and has less than perfect scalability, with some optimization the mechanism serves well broadcasting the status. For the second case, we want to have a better-optimized mechanism, while still benefit from reusing the original P2P network.

The mechanism of periodic announcements has, therefore, a two-fold purpose:

1. make the best effort to deliver current status to all supernodes in the network without releasing the sender's IP to the whole network;
2. build reliable communication channels between any two active supernodes in the network without releasing IPs of the participants, while producing minimal traffic overhead. 

An announce message has the following fields:

* supernode public identification key;
* current block height;
* signature, generated from all message fields, using supernode private identification key;
* hop index.

An announce message is constructed by a supernode and passed to a local graftnode for further broadcast over the P2P network. Upon receiving an announce from a neighbour, a graftnode passes it to a local supernode, increment announce's hop index by 1, and re-send the message to other neighbours (some optimization steps are omitted here in the sake of simplicity). Therefore, both supernode and graftnode are able to utilize the mechanism of periodic announcements.

Supernode uses the announce messages to update the list of active supernodes and, therefore, to build more actual authorization sample for RTA validation based on this list.

Graftnode uses announce messages to construct communication channels over the P2P network. For each neighbour, the graftnode keeps a list of supernode public identification keys it has seen in the announces coming from this neighbour, ordered by order of arrival. The lowest hop index per neighbour-key pair (effectively the length of the communication channel) is also recorded.

Whenever two supernodes need to communicate, the sender issues a message, setting recipient's public identification key, while its local graftnode sets the hop index to the highest for given recipient. Upon receiving the message, and after checking the recipient is not local supernode, a graftnode looks up top 3 neighbour peers that have recipient's key. Then it decreases the hop index in the message by 1 and forwards the message to the selected peers. In case the hop index becomes 0, the message is dropped.

This mechanism allows us to maximize chances for a message to be delivered while minimizing delivery time and traffic overhead.

# Announcement-based Messaging
Since supernodes don't know about the IP addresses of each other, they cannot communicate with each other directly. All the communication between supernodes happens oner P2P network, using communication channels, created during sending of announces. There are three types of communication messages that can be sent using announcement-based communication channels: broadcast, multicast and unicast.
To protect messages from the data change by unfair supernodes, each message contains a public identification key and be signed by supernode's private identification key. The created signature should be included in the message.

## Broadcast
The broadcast is the message which hasn't a direct receiver and sent to all supernodes in the network. The broadcast is the closest to regular P2P messages and send in the same way as the announce message - from one node to all its peers and so on. The broadcast message contains the following fields:

* sender_address - sender's supernode public identification key;
* wait_answer - a boolean flag which determines that each receiver of this message should send its answer to the sender or not (answer sends using unicast message);
* data - serialized message data, which can be read on receiver's supernode;
* callback_uri - receiver's supernode endpoint to which will be sent the message;
* signature - the signature byte array of the signed message (including sender_address, wait_answer, data and callback_uri fields), also used as message identifier;
* hop - the hop index, which automatically added by graftnode, when it determines communication tunnels for the message.

## Multicast
The multicast is the message which sent to multiple receivers. For sending the multicast message, graftnode chooses three tunnels for each receiver's supernode and sends the message through them only one time. There are no need to send a multicast message to the tunnel multiple times even if few receivers use same tunnels since a multicast message contains all data for all receivers. The multicast message contains the same fields as the broadcast message with one additional field which contains the list of message receivers:

* sender_address - sender's supernode public identification key;
* receiver_addresses - the list of receiver's supernode public identification keys;
* wait_answer - a boolean flag which determines that each receiver of this message should send its answer to the sender or not (answer sends using unicast message);
* data - serialized message data, which can be read on receiver's supernode;
* callback_uri - receiver's supernode endpoint to which will be sent the message;
* signature - the signature byte array of the signed message (including sender_address, receiver_addresses, wait_answer, data and callback_uri fields), also used as message identifier;
* hop - the hop index, which automatically added by graftnode, when it determines communication tunnels for the message.

## Unicast
The unicast is the message which sent to the single receiver. For sending the unicast message, graftnode chooses three tunnels for receiver's supernode and sends the message through them. The unicast message contains the same fields as the broadcast message with one additional field which contains message receiver:

* sender_address - sender's supernode public identification key;
* receiver_address - receiver's supernode public identification key;
* wait_answer - a boolean flag which determines that each receiver of this message should send its answer to the sender or not (answer sends using unicast message);
* data - serialized message data, which can be read on receiver's supernode;
* callback_uri - receiver's supernode endpoint to which will be sent the message;
* signature - the signature byte array of the signed message (including sender_address, receiver_address, wait_answer, data and callback_uri fields), also used as message identifier;
* hop - the hop index, which automatically added by graftnode, when it determines communication tunnels for the message.

# Secure Supernode Communication
To protect user data and prevent it from being read by recipients other than expected during RTA, RTA validation participants may use encrypted messages. To send such message, supernode or client needs to know the public identification keys of all receivers of this message.

## Single Recipient Message Encryption
To encrypt a message to the single recipient, supernode
* Forms the message and convert it into a byte array.
* Using public identification key of receiver's supernode asymmetrically encrypts the message.

## Multiple Recipients Message Encryption
To encrypt a message to the multiple recipients, supernode
* Forms the message and convert it into a byte array.
* Generates a symmetric encryption key called message key.
* Encrypts message using a message key.
* Asymmetrically encrypts the message key for each recipient using its public identification key.
* Joins encrypted message and encrypted message keys into the one byte array.

After encryption, the encrypted message added to the communication message (unicast or multicast) with additional data which are used for message transferring.