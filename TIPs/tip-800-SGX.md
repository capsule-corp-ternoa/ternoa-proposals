# Ternoa Improvement Proposal - SGX Pallet

| Author(s)      |  |
| ----------- | ----------- |
| Created   | 15 Nov 2022       |
| TIP Number   | TIP-800       |
| Version   | v0.1       |
| Status | In Progress       |
| Category   | Blockchain, SGX       |
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


  register_enclave(origin: OriginFor<T>, ra_report: Vec<u8>, api_uri: TextFormat)
		
  assign_enclave(origin: OriginFor<T>, cluster_id: ClusterId, enclave_id: EnclaveId)

  unassign_enclave(origin: OriginFor<T>, cluster_id: ClusterId, enclave_id: EnclaveId)

  update_enclave(origin: OriginFor<T>, api_uri: TextFormat, enclave_id: EnclaveId, cluster_id: ClusterId)

  change_enclave_owner(origin: OriginFor<T>, new_owner: <T::Lookup as StaticLookup>::Source, enclave_id: EnclaveId)

  create_cluster(origin: OriginFor<T>)

  remove_cluster(origin: OriginFor<T> cluster_id: ClusterId)

  remove_enclave(origin: OriginFor<T>, cluster_id: ClusterId, enclave_id: EnclaveId)
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
