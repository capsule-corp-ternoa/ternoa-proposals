# Ternoa Improvement Proposal - SGX Pallet

| Author(s)      |  |
| ----------- | ----------- |
| Created   | 15 Nov 2022       |
| TIP Number   | TIP-520       |
| Version   | v0.1       |
| Status | In Progress       |
| Category   | Blockchain, SGX       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions

## Simple Summary

Improvements for SGX Pallet to support on chain operations for enclave

## Abstract



## Motivation

Gives the ability to create clusters to hold enclaves

## Specification

* Proposal for creating a cluster, removing a cluster and registering a service provider should come through the technical committee.

* When adding an enclave to a cluster, it will be added to the primary cluster or the next available cluster
* Checks the enclave provider is registered and valid
* Checks the digital signatre of `ra_report` with the provided public key of enclave provider
* A cluster can be forgotten or removed when there are no enclaves
* Can unregister enclave providerIds and this will invalidates all the stored enclaved and this happens via a technical committee proposal (future implementation)



### External Interfaces


### Existing Interfaces changed

```rust
interface {

  /// Registers enclavce providers on chain :- Intel, AMD
  /// Different manufacturers can provide different enclaves
  /// Register's an enclave provider should come through the mandate call
  register_enclave_provider(origin: OriginFor<T>, enclave_provider: Vec<u8>)

  /// Given provider may have different processor architectures (enclave_class)
  /// and for a given enclave class there can be different public keys
  register_provider_keys(origin: OriginFor<T>, enclave_provider: Vec<u8>, enclave_class: Vec<u8>, provider_public_key: Vec<u8>)

  /// Registers an account which responsible for creating / submitting an enclave report
  register_enclave_operator(origin: OriginFor<T>, operator: <T::Lookup as StaticLookup>::Source,)

  /// Register an enclave, parse ra_report and performs validations
  register_enclave(origin: OriginFor<T>, ra_report: Vec<u8>, api_uri: TextFormat)
		
  /// Assigns a clusterId for an enclave
  /// Mandate call and make an update 
  assign_enclave(origin: OriginFor<T>, cluster_id: ClusterId, enclave_id: EnclaveId)

  ///  Updates metadata for an enclave
  update_enclave(origin: OriginFor<T>, api_uri: TextFormat, enclave_id: EnclaveId, cluster_id: ClusterId)

  /// Change the ownership of an enclave
  /// 
  change_enclave_operator(origin: OriginFor<T>, prev_owner: <T::Lookup as StaticLookup>::Source, new_operator: <T::Lookup as StaticLookup>::Source, enclave_id: EnclaveId)

  /// Creates a cluster
  /// A cluster can hold up to 5 enclaves
  /// Mandate call
  register_cluster(origin: OriginFor<T>)

  /// Removes a cluster
  /// Mandate call
  /// Cluster must be empty
  unregister_cluster(origin: OriginFor<T> cluster_id: ClusterId)

  /// Removes an enclave from the system
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
