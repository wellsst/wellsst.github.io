---
layout: post
title:  "Blockchain and its non-Bitcoin application"
date:   2019-04-15 15:18:27 +1000
categories: blockchain bitcoin
---

## Blockchain basics - What is it?

Blockchain emerged from the Bitcoin project back in around 2008.  Broadly defined, it is an underlying technology that ensures a distributed, de-centralised system in which all nodes of the network (or all parties however you want to term this) have access to all records/transactions in the system/ledger.  These transactions are immutable, once added a block they cannot be changed.  

It has the attributes of being:

* Persistent
* Transparent
* Public
* Auditable
* Append only ledger - no changes possible

The aim of any blockchain based system should be that all actors involved in the system should act/make decisions to improve the system for the greater good.

So what is the use in this and as someone once said:

> "Blockchain is the answer, but what is the question?"

I aim to collect many of those questions here.

Key components:

* Public/private keys
* Hashing - blocks have hashcodes
* Merkel trees - A tree of hashes that form the relationships between blocks.  Ensuring then that no underlying transactions can be tampered with.  

Peer-to-peer trading (cut out the middle man like banks).  Validated by all in the network.  Not all users can see the all content of the transactions but you know a transaction has occurred.

How do we take any asset and trade it - use blockchain

Trust is a major issue moving this technology forward.  We have moved from a centralised trust (eg a bank) to a distributed trust (everyone in the chain)


## What is a block

A block can be built in various ways depending on the solution required which typically resolves also around the level of trust required.

Traditionally (from bitcoin) a block comprises:

* Nonce - a string that will alter the blocks hash.  Certifies that a block is legitimate.
* Hash of the previous block
* Transactions
* Hash of the block

Blocks are mined which essentially means the value for the Nonce is solved for the hash of the block.

## Some use cases

Banks and Fin's are adopting BC first as this seems the most obvious applications

### Electricity trading, eg cars with batteries distributing power

Solar/wind and can sell excess and use IoT to facilitate the transfer of power.  Big cities could donate excess power to poorer areas

### IoT

Connected devices

### Procurement of goods between companies

### Insurance

### Contracts

https://aeternity.com/

### Supply chain improvements

Transactions are documented in a permanents decentralised ledger and can be monitored securely and transparently cutting.  Can monitor entire supply chain to reduce time, cost, labour, errors and allow improvements of the process.

Rigid contracts as bad actors in the chain could be boycotted.

### Agriculture

Fractional ownership - eg multiple farmers can part own a tractor/land - record keeping for each item used and time spent.  Input and output from the land can be tracked with blockchain

### Forecasting

### Gambling

Place bets on a race in a decentralised way, see: https://www.augur.net/

### Voting

Prevent fraud used with processes in registrations, verification and vote counting

Immutable, publicly viewable ledgers of recorded votes would make elections more fair and democratic

### News

Can we prevent fake news?

### Uber v2 - decentralised ride sharing

Uber is not peer to peer as Uber is the middle man

Arcade CiIty  & LaZooz

Automatically pay for parking, tolls and fuel - good for driverless cars.

ewallets being developed by innogy, zf and ubs

### Data storage

Decentralised data storage

### Charity

Can track where donations are actually going

see: BitGive

### Chit funds ??  Kitty party

Parallel banking

## Blockchain platforms



## Disclaimer

I am not (yet) a blockchain expert

## Additional Points

## For further investigation

## Deeper reading - references

* [Original whitepaper](https://bitcoin.org/bitcoin.pdf)
* [Consensus overview](https://www.coindesk.com/short-guide-blockchain-consensus-protocols)

* [edX Berkeley course](https://courses.edx.org/courses/course-v1:BerkeleyX+CS198.2x+1T2019)

* [The Blockchain & Bitcoin - Computerphile](https://www.youtube.com/watch?v=qcuc3rgwZAE)

* [Bitcoin how miner adds transactions to the blockchain](https://blog.goodaudience.com/how-a-miner-adds-transactions-to-the-blockchain-in-seven-steps-856053271476)
