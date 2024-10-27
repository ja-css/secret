
Hardware	Specification
CPU	8-core processor
CPU Support	Models with support for Intel SHA Extensions (AMD since Zen microarchitecture or Intel since Ice Lake) will significantly speed up the processes.
RAM	256 GiB RAM + Swap
GPU	Nvidia GPU with at least 11GB VRAM
Disk 2 TB NVMe disk

----------

1) Purchase miners from Bitmain
2) Host miners with Antpool
3) Check daily earnings on antpool

CPU: Intel_Xeon(R)_Silver4210R*1
Power: 280W
Memory: DDR4_32G_2933MHz_RDIMM*3
System Disk: 480G_SSD_SATA3*2
Data Disk: 16T_HDD_SATA_3*36
Network: 2 ports, 10Gb/s Fiber Optic Interface with 1 module
Power Supply: 1200W*2

----------

https://github.com/ipld/specs/blob/master/block-layer/graphsync/graphsync.md

----------

https://filecoin.io/blog/posts/how-storage-and-retrieval-deals-work-on-filecoin/

Introduction
The Filecoin network achieves economies of scale by allowing anyone to participate as a storage provider. Currently the network is made up of hundreds of storage providers spread across the globe. Content addressing and cryptographic storage proofs verify that data is being stored correctly and securely over time on miners’ hardware, which creates a robust and reliable service.

This blog post covers the basic stages of the two types of deals in Filecoin, namely storage deals and retrieval deals, and details their lifecycle. The cryptographic proofs used to verify that participants in the system are performing their responsibilities according to their commitments are also explained.

Data on Filecoin
In order to store files on Filecoin, clients must first import them in their local Filecoin node. This step produces a data CID - a content identifier, an ID that uniquely describes the content. Later, the data is transferred to a miner. Another way to store files on Filecoin is via offline deals, which is not cover in this post.

Importing data locally to a Filecoin node can be done with the lotus client import command. It’s important to remember the resulting data CID (it is also available on the local node later on), because it must be used to later retrieve data from miners.

After importing data into the local node, users have to initiate a deal. This can be done with the lotus client deal command. The command takes a data CID as input, produces a Filecoin Piece, and interactively takes the user through the storage deal flow detailed below.

The Filecoin Piece is the main unit of negotiation for data that users store on the Filecoin network. The Filecoin Piece is not of a specific size, but is upper-bounded by the size of the Sector, governed by the parameters of the network. If a Filecoin Piece is larger than the size of a Sector that the miner supports, it has to be split into more pieces so that each piece fits into a sector.

Filecoin Piece

A Filecoin Piece is a CAR file containing an IPLD DAG with its own data/payload CID and piece CID.

CAR stands for Content Addressable aRchives - a CAR file is a serialised representation of any IPLD DAG as the concatenation of its blocks, plus a header describing the graph in the file (with the root CID).

When a client wants to store a file in the Filecoin network, they start by producing the IPLD DAG of the file with UnixFS (this is what the lotus client import command does). The hash that represents the root node of the DAG is an IPFS-style CID, called data/payload CID.

UnixFS is a protobuf-based format for describing files, directories, and symlinks in IPFS. UnixFS is used in Filecoin as a file formatting guideline for files submitted to the Filecoin network.

The resulting CAR file is padded with extra zero bits in order for the file to make a binary merkle tree.

The Storage Deal Flow
Users can store data in and retrieve data from the Filecoin network via deals. Participants in the network, miners (supply-side) and clients (demand-side), interact with each other via storage deals and retrieval deals.

The lifecycle of a storage deal is as follows:

1. Discovery
   The client identifies miners and determines their current asks, i.e. the price per GiB per epoch (30sec.) in attoFIL (1 attoFIL is equal to 10^-18 * FIL) that miners want to receive in order to accept a deal. Currently a deal in Filecoin has a minimum duration of 180 days.

You can list all currently active miners by querying the JSON RPC API of a synced node (for test purposes the https://api.node.glif.io public endpoint is used), with the Filecoin.StateListMiners method:

curl -X POST -H "Content-Type: application/json" \
--data '{ "jsonrpc": "2.0", "method": "Filecoin.StateListMiners", "params": [ null ], "id": 1 }' \
'https://api.node.glif.io' | jq

{
"jsonrpc": "2.0",
"result": [
"f011303",
"f011092",
...
You might want to decide on a specific provider based on their reputation or power in the network. Reputation metrics for miners are not part of the Filecoin protocol yet, and not covered in this post.

Once you have chosen a specific miner, you need to fetch its PeerID, for example with the Filecoin.StateMinerInfo method, to establish a secure connection to them via libp2p protocol:

curl -X POST -H "Content-Type: application/json" \
--data '{ "jsonrpc": "2.0", "method": "Filecoin.StateMinerInfo", "params": [ "f03274", null ], "id": 1 }' \
'https://api.node.glif.io' | jq

{
"jsonrpc": "2.0",
"result": {
"Owner": "f03261",
"PeerId": "12D3KooWP5D9TmqC45i6L2e2qQHYcuxaUwPdYo6CzqUMVmFEH3N9",
...
You can then query for a signed StorageAsk with the Filecoin.ClientQueryAsk method. This will establish a direct libp2p connection to the selected miner, and ask for a storage quote:

curl -X POST https://api.node.glif.io \
-H "Content-Type: application/json" --data-binary @- << EOF
{
"jsonrpc": "2.0", "method": "Filecoin.ClientQueryAsk", "id": 1,
"params": [
"12D3KooWP5D9TmqC45i6L2e2qQHYcuxaUwPdYo6CzqUMVmFEH3N9",
"f03274"
]
}
EOF

{
"jsonrpc": "2.0",
"result": {
"Price": "100000000000",
"VerifiedPrice": "100000000000",
"MinPieceSize": 256,
"MaxPieceSize": 34359738368,
"Miner": "f03274",
"Timestamp": 148031,
"Expiry": 1199231,
"SeqNo": 14
},
"id": 1
}
The result includes details about deals that this miner is willing to accept, such as range for the admitted Filecoin Piece sizes and price per GiB per epoch. Note that making a storage deal proposal which matches the miner’s storage ask is a precondition, but not sufficient to ensure the deal is accepted - the storage provider may run its own decision logic later on.

2. Negotiation and data transfer
   During this phase both parties come to an agreement about the terms of the deal, such as the cost of deal, the duration of deal, the starting epoch of the deal, etc.

Then data is transferred from the client to the miner.

3. Publishing
   The deal is published on-chain, via the PublishStorageDeals message, making the storage provider publicly accountable for the deal.

4. Handoff
   Once the deal is published on-chain, it is handed over to the Storage Mining subsystem, to pack into a sector which is later sealed, and subsequently proven continuously.

The Storage Mining subsystem
The Storage Mining subsystem ensures miners can effectively commit storage to the Filecoin network and:

Participate in the Filecoin Storage Market by taking on client data and participating in storage deals.
Participate in Filecoin Storage Power Consensus, verifying and generating blocks to grow the Filecoin blockchain and earning block rewards and fees for doing so.
It oversees the following processes:

Committing new storage and registering new sectors
In order to register a sector in Filecoin, a miner must seal the sector. Sealing is a computation-heavy process that produces a unique representation of the data in the form of a proof, called Proof-of-Replication or PoRep. Once the proof has been generated, the miner compresses it and submits the result to the blockchain. This is a certification that the miner has indeed replicated a copy of the data they agreed to store.

Continuously proving storage (see WindowPoSt)
Every storage miner must continuously submit proofs on-chain to prove that they continue to store their sectors.

Declaring storage faults and recovering from them (see Faults)
Failing to submit the proofs mentioned above for a given sector will result in a fault, and the miner will be subject to penalties.

Storage miner and client considerations
As covered above, storage deals are published on-chain before they are active and sealed. This is important because publishing a deal locks clients’ funds in escrow on-chain. Thus, the miner has assurance that if they do indeed seal the data in a sector, they will get paid for it.

It helps to think of publishing a deal on-chain as signing a contract, and of sealing and activating a deal as starting to do the work the miner committed to.

From the perspective of a client who wants to store data on Filecoin, the deal passes roughly through the following stages:

Fund a deal - client locks funds in escrow
Propose a deal to the miner
Check for intent to accept deal
Transfer data for the deal to the miner - this is done via the GraphSync protocol. GraphSync is a protocol for synchronising IPLD graphs among peers. It allows a host to make a single request to a remote peer for all of the results of traversing an IPLD selector on the remote peer’s local IPLD graph. Lotus uses the ipfs/go-graphsync implementation of the GraphSync protocol.
Check for acceptance - make sure that the miner has accepted the deal and published it on-chain.
Sealing - deal is on-chain, and miner is currently sealing a sector that contains the deal.
Active - deal has been sealed and is active. From here on, the storage provider/miner should periodically prove that they continue to store the data. More details in the Proof-of-Spacetime section below.
From the perspective of a miner who provides a service to the client by storing their data, the deal passes roughly through the following stages:

Verify a deal - receive a proposal for a deal, and check it parameters (size, price, etc.)
Check for locked funds - make sure that client has locked funds and can pay for the deal.
Wait for data - receive data from client for the deal.
Pledge collateral for the deal on chain
Publish the deal on-chain
Seal the sector
Activate deal - from here on, the storage provider/miner periodically submits WindowPoSt proofs, proving that they are continuously storing the data.
The Retrieval Deal Flow
Retrieval deals, unlike storage deals, happen mostly off-chain facilitated by payment channels. Data transmission is metered, and the client pays the miner incrementally as the data is being transferred. Creating payment channels, and redeeming vouchers, is the only part of the process which involves interacting with the Filecoin blockchain.

This is the overall process:

Discovery - the client identifies miners who have the data that it needs and requests a retrieval quote from them - price per byte, unseal price, payment interval.
Payment channel setup - the client sets up a payment channel between them and the miner (if one doesn’t already exist).
Data transfer with payment - miner sends data to client until payment is required. Payment processing is requested when a certain threshold is reached, and data transfer continues after that. Depending on whether the miner has the data in their block-store or not, they might need to first unseal it - a non-trivial and non-instantaneous operation, which is the opposite of sealing described in the section about storage deals.
The client has not successfully retrieved a full copy of the data.

Proof-of-Spacetime
The sections above glanced over many details that make Filecoin unique and provide probabilistic guarantees on data to users. This section covers two of the proofs that Filecoin utilises, and also explains how they fit into the protocol and the problems they address.

Proof-of-Spacetime (PoSt) is a procedure by which a storage miner can prove to the Filecoin network they continue to store a unique copy of some data on behalf of the network.

Proof-of-Spacetime manifests in two distinct varieties in Filecoin today:

Window Proof-of-Spacetime (WindowPoSt) and
Winning Proof-of-Spacetime (WinningPoSt)
Winning Proof-of-Spacetime
Winning Proof-of-Spacetime (WinningPoSt) is the mechanism by which storage miners are rewarded for their contributions to the Filecoin network. At the beginning of each epoch, a small number of storage miners are elected to each mine a new block. As a requirement for doing so, each miner is tasked with submitting a compressed Proof-of-Storage for a specified sector. Each elected miner who successfully creates a block is granted FIL (a block reward), as well as the opportunity to charge other Filecoin participants fees to include messages in the block.

Storage miners who fail to do this in the necessary window will forfeit their opportunity to mine a block, but will not otherwise incur penalties for their failure to do so.

Window Proof-of-Spacetime
Window Proof-of-Spacetime (WindowPoSt) is the mechanism by which the commitments made by storage miners are audited by the Filecoin blockchain.

Every storage miner should maintain their pledged sectors. These sectors contain deals made with clients or empty sectors. The latter are called committed capacity, that is, miners can make capacity commitments, filling a sector with arbitrary data, rather than with client data. Maintaining these sectors allows the storage miner to provably demonstrate that they are reserving space on behalf of the network.

Every day is broken down into an array of windows, currently 48 windows, with a duration of 30 minutes (60 epochs, since 1 epoch is equal to 30sec)

Each storage miner’s set of pledged sectors is partitioned into subsets, one subset for each window.

Within a given window (30min), each storage miner must submit a Proof-of-Spacetime for each sector in their respective subset. This requires ready access to each of the challenged sectors, and will result in a zk-SNARK proof published to the Filecoin blockchain as a message in a block. In this way, every sector of pledged storage is audited at least once in any 24-hour period, and a permanent, verifiable, and public record attesting to each storage miner’s continued commitment is kept.

WindowPoSt sample deadlines

In the diagram above, you can see that a sample miner should submit WindowPoSt proofs in deadlines 0 (> 16TB), deadline 1 (< 8TB) and deadline 2 (< 8TB), with the majority of their sectors falling in deadline 0. Deadlines for each miner are randomised, and for this specific miner, start respectively in epoch 1635, in epoch 1695, and in epoch 1755. You can inspect these deadlines and more details about miners on the SpaceGap tool.

The Filecoin network expects constant availability of stored data. Failing to submit WindowPoSt for a sector will result in a fault, and the storage miner supplying the sector will be slashed. This incentives storage miners for healthy behaviour.

Faults
A fault occurs when a proof is not included in the Filecoin blockchain within the proving period, as a result of loss of network connectivity, storage malfunction, or malicious behaviour.

When a fault is registered for a sector, the Filecoin network will slash the storage miner that is supposed to be storing the sector; that is, it will assess penalties to the miner (to be paid out of the collateral fronted by the miner) for their failure to uphold their pledge of storage.

There are three types of sector fault fees:

Sector fault fee - This fee is paid per sector per day while the sector is in a faulty state. The size of the fee is slightly more than the amount the sector is expected to earn per day in block rewards. If a sector remains faulty for more than 2 consecutive weeks, the sector will pay a termination fee and be removed from the chain state.
Sector fault detection fee - This is a one-time fee paid in the event of a failure if the miner does not report it honestly and instead the unreported failure is caught by the blockchain. Given the probabilistic nature of PoSt checks, this is set to a few days worth of block reward that would be expected to be earned by a particular sector.
Sector termination fee - A sector can be terminated before its expiration date through automatic faults or miner decisions. A termination fee is charged that is, in principle, equivalent to how much a sector has earned so far, up to a limit in order to avoid discouraging long sector lifetimes.
Read more about faults and economics around them at the Filecoin Spec website

Conclusion
This post covered some of the concepts related to storing and retrieving data on Filecoin, the protocols that clients and miners engage in to make that happen, and the different proofs and guarantees involved in the process.

It detailed the flow for storage and retrieval deals from the perspective of clients and miners, as well as the penalties that the Filecoin protocol would enforce in case one of the parties misbehaves.

To conclude, it outlined some of the foundations of how the Filecoin protocol governs the Filecoin network resulting in a reliable and trustless decentralised storage network.