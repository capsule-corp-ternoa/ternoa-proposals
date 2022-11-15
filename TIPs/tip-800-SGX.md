---
tip: tip_800
title: 
version: 
category: NFT/Token/Blockchain
authors: Name <email@email.com>
created: creation date (ex: 2022-09-09)
discussions-To: discussion link (not required)
---

# Ternoa Improvement Proposal - SGX Pallet

| Author(s)      | Tapioca Boy |
| ----------- | ----------- |
| Created   | 15 Nov 2022       |
| TIP Number   | TIP-800       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | Blockchain       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions

## Simple Summary

Improvements for SGX Pallet to support on chain operations for enclave

## Abstract



## Motivation

Why it's necessary for Ternoa?

## Specification

### External Interfaces


### Existing Interfaces changed

```rust
interface {

  register_enclave(origin: OriginFor<T>, api_uri: TextFormat)

  assign_enclave(origin: OriginFor<T>, cluster_id: ClusterId)

  unassign_enclave(origin: OriginFor<T>)

  update_enclave(origin: OriginFor<T>, api_uri: TextFormat)

  change_enclave_owner(origin: OriginFor<T>, new_owner: <T::Lookup as StaticLookup>::Source)

  create_cluster(origin: OriginFor<T>)

  remove_cluster(origin: OriginFor<T> cluster_id: ClusterId)
}
```

## Constraints

## Additional Info

## Metadata

metadata example

```json
{
	
}
```

## End-to-end workflows (Ternoa-specific)

## Test cases

* An enclave creator can register an enclave
* An enclave creator should be able to verify RA report upon creation
* Should store enclave metadata on chain after verification
* Should be able to get enclave metadata
* Should be able to unassign enclave
* Should be able to update enclave by the creator
 
## References
* TBD

## Copyright
* TBD
