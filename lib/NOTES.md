# DID BTC1 Notes

## Abstract

* off-chain DID creation
* aggregated on-chain DID Document updates
* off-chain DID / DID Document creation
* private DID Documents (communication and resolution)
* non-repudiation of the DID and DID Document

## Introduction and Motivation

* Pairwise DIDs: DIDs created for every different special purpose
  * Methodology implemented by Web5 Agent design
* Pairwise DIDs leak every time the DID is used making them no longer private
* Sidecar delivery is a method of sharing an offline, private DID and DID Doc with another party willing to cooperate with the privacy reqiurements of the sharing party
* The receiving party must be willing to act as their own resolver
* The Sidecar data includes the DID, DID Doc, and a partial proof to allow the receiver to trace the document update history
* Uses Bitcoin's Blochain as timechain of DID Document updates to establish a chain-of-custody
  * Enables non-repudiation of the DID and DID Document
* Claims that did btc1 "has it all" but the glaring thing missing is scalability and discoverability (?)
* Other did method comparisons
  * btcr: stores doc on chain in OP RETURN; non-private; expensive; repudiable
  * ion: repudiable / late publishing vulnerable; non-private on IPFS
  * btco: stores doc on-chain as inscription; non-private; expensive
  * btc: uses inscriptions optionally; adds batching; data non-private, on-chain; expensive
* btc1 features
  * uses bitcoin blockchain as timechain
  * allows for offline DID creation (no on-chain tx required)
  * introduces aggregators to mitigate update costs
  * enables non-repudiation and avoids late-publishing by validating full update history
  * enables private dids & did docs:
    * via off-chain Sidecar did history data
    * via DID history inclusion as SMT within on-chain txs
  * reduces resolution work by filtering txs for updates about specific DIDs of interest
  * flexible inclusion of any key type in DID Doc
  * seed words can deterministically recover DID
  * using descriptors instead of address in did doc enables non-reuse of addresses
* btc1 limitations
  * resolvers require read-only access to all bitcoin blocks
  * controllers are responsible for storing and providing data required to verify against the "beacons" (did, did doc, cid, smt)
  * invalid references by beacons require scripts allowing for controllers to veto and UTXO sharing
  * ZCAPs needed to update part of did doc
  * Aggregators won't scale well until covenants enabled (CTV, CAT)

## Terminology

* Beacon: DID doc update mechanism; form of a bitcoin address as service object in DID Doc; spending from beacon adderss => DID update announcements; Two types of beacons
  * Singleton: single entity to independently post update; every did doc will have 1 singleton as fallback
  * Aggregate: multiple entities collectively announce did updates
* Beacon Signal: bitcoin transactions spending from a beacon address including OP_RETURN <32_bytes>; announce 1+ did updates and a means for verification; Two types of aggregator beacons:
  * SMTAggregator
  * CIDAggregator

## Syntax

1. `did:btc` (required)
2. `version` (optional, implied to be 1)
3. `network` (optional, implied to be mainnet)
4. `method-id` (required; `k1` + bech32 encoded secp256k1 pubkey || `x1` + hash of initial did doc)

* Examples
  * pubkey - version - network: `did:btc1:k1t5rm7vud58tyspensq9weyxc49cyxyvyh72w0n5hc7g5t859aq7sz45d5a`
  * pubkey + version + network: `did:btc1:1:mainnet:k1t5rm7vud58tyspensq9weyxc49cyxyvyh72w0n5hc7g5t859aq7sz45d5a`
  * doc - version - network: `did:btc1:x1<hash>`
  * doc + version + network: `did:btc1:1:mainnet:x1<hash>`

## CRUD Ops - CREATE

* created deterministically from a cryptographic seed
* created from an arbitrary initiating DID document
* can be done offline (no on-chain tx required)
* Deterministic
  * encodes a Secp256k1 public key
  * uses public key to deterministically generate initial DID document
  