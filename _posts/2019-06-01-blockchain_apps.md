---
layout: post
title:  "Blockchain and its non-Bitcoin application"
date:   2019-06-01 15:18:27 +1000
categories: blockchain bitcoin
---

Still very much a Work in progress, will take months but getting it out there since time is up this month...

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

### Cross-industry

https://www.r3.com/

Ethereum

Quorum

### Electricity trading, eg cars with batteries distributing power

Solar/wind and can sell excess and use IoT to facilitate the transfer of power.  Big cities could donate excess power to poorer areas

### Finance

https://www.fnality.org/ 

Dharma

inveniam.io

Goldman Sachs and others are investing in their own crypto-currencies

Interbank transfewrs

Ripple Global Payments group

SWIFT using hyperledger fabric



### IoT

Connected devices, BC can bring security and trust.  Can quarantine unknown or misbehaving IoT devices



### Procurement of goods between companies

### Insurance

A clear history for insured products
Automatic payouts
Shorter wait times
Fewer human errors
Fraud detection

Tokio Marine Insurance

B3i

AiGang

### Contracts

https://aeternity.com/

### Supply chain improvements

Transactions are documented in a permanents decentralised ledger and can be monitored securely and transparently cutting.  Can monitor entire supply chain to reduce time, cost, labour, errors and allow improvements of the process.

Rigid contracts as bad actors in the chain could be boycotted.

Walmart, using Hyperledger

Alibaba: reducing counterfeiting


### Agriculture

Fractional ownership - eg multiple farmers can part own a tractor/land - record keeping for each item used and time spent.  Input and output from the land can be tracked with blockchain

### Forecasting

### Gambling

Place bets on a race in a decentralised way, see: https://www.augur.net/

### Voting

Prevent fraud used with processes in registrations, verification and vote counting

Immutable, publicly viewable ledgers of recorded votes would make elections more fair and democratic

### Travel and tourism

#### Disintermediated bookings

Winding tree
Beenest/Beetoken - like AirBNB but decentralised

#### Sharing econbomy

Beenest
TUI bedshare

#### Rewards

Trippki

### Identity management

SITA:
Face recognition
1 time se QR coeds

Biometrics: ObjectTech 

### Medical


MedicalChain

MedREC

SweatCoin - exercise for tokens

Mint Health

### News

Can we prevent fake news?

### Uber v2 - decentralised ride sharing

Uber is not peer to peer as Uber is the middle man

Arcade CiIty  & LaZooz

MOBI - non-profit, aiming to make mobility services efficient, green, affordable, safe, and free of congestion.

Automatically pay for parking, tolls and fuel - good for driverless cars.

Mining of cars data, metrics such as mileage, efficiency

ewallets being developed by innogy, zf and ubs

Turo - airbnb for cars

### Data storage

Decentralised data storage

### Charity

Can track where donations are actually going

Distribute crypto vouchers which can be exchanged for aid

BitGive
bifrost
disberse

### Real estate

Use a blockchain land registry

Propy
Zebi
Chromaway

### Chit funds ??  Kitty party

Parallel banking

## Blockchain platforms

## When to not use Blockchain

The whole thing hinges on the existing strengths of PKI but what if that can be broken quickly by using Quantum computers, then our existing chains will be useless and need to be rebuilt/thrown away? 
https://www.technologyreview.com/s/613596/how-a-quantum-computer-could-break-2048-bit-rsa-encryption-in-8-hours/?utm_campaign=site_visitor.unpaid.engagement&utm_source=twitter&utm_medium=tr_social

Efficiency!  In bitcoin buying  a coffee can take 10 minutes but not with card/cash but a foreign $ transfer in 10 mins is much more efficient than today taking possibly a few days

Businesses - loss of control and therefore revenue when they are no longer the central controlling entity

How to change a blockchain protocol with many actors already agreeing on it? 

How to resolve issues without a central authority?  eg A blockchain AirBNB, how do you prove and get compensation if a client destroys your house?

## Disclaimer

I am not (yet) a blockchain expert

## Additional Points

## For further investigation

## Deeper reading - references

* [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook)
* [Original Bitcoin whitepaper](https://bitcoin.org/bitcoin.pdf)
* [Consensus overview](https://www.coindesk.com/short-guide-blockchain-consensus-protocols)
* [edX Berkeley course](https://courses.edx.org/courses/course-v1:BerkeleyX+CS198.2x+1T2019)
* [The Blockchain & Bitcoin - Computerphile](https://www.youtube.com/watch?v=qcuc3rgwZAE)
* [Bitcoin how miner adds transactions to the blockchain](https://blog.goodaudience.com/how-a-miner-adds-transactions-to-the-blockchain-in-seven-steps-856053271476)
