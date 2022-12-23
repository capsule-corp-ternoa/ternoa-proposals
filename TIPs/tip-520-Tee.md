# Ternoa Improvement Proposal - TEE Pallet

| Author(s)      |  |
| ----------- | ----------- |
| Created   | 15 Nov 2022       |
| TIP Number   | TIP-520       |
| Version   | v0.1       |
| Status | In Progress       |
| Category   | Blockchain, TEE       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions

## Simple Summary

Improvements for TEE Pallet to support on chain operations for enclave

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
  /// Register an enclave, parse ra_report and performs validations
  /// Origin :- Operator Account Address
  /// enclave_address :- Generated within each enclave (PK)
  /// Signed by operator (the origin)
  ///   Notes:
  /// Enclave should have public endpoints to return:
  ///     1. Simple health check - 200
  ///     2. Binary hash
  ///     3. Quote
  ///     4. Enclave address & operator address
  /// This transaction will be signed by operator.
  /// Verify if the origin does not already have an enclave address associated with it.
  register_enclave(origin: OriginFor<T>, enclave_address: Vec<u8>, api_uri: Vec<u8>)

  /// Removes an enclave from the system
  /// Origin- operator account address
  /// If the enclave is assigned, he will be placed in queue for tech committee approval
  /// If enclave is not already assigned, he can exit without permission.
  unregister_enclave(origin: OriginFor<T>)

  /// Removes an enclave from the system
  /// Origin- operator account address
  /// If the enclave is assigned, he will be placed in queue for tech committee approval
  /// If enclave is not already assigned, he can exit without permission.
  update_registration(origin: OriginFor<T>, enclave_id: EnclaveId)

  /// Assigns a clusterId for an enclave
  /// Tech committee proposal:
  assign_enclave(cluster_id: ClusterId, enclave_address: Vec<u8>, enclave_id: EnclaveId)

  /// Origin :- Operator Account
  /// Performs all cleanup
  unassign_enclave(Origin, enclave id)

  /// Force update enclave
  /// Technical committee call
  force_update_enclave(enclave_id: EnclaveId, enclave_address: Vec<u8>, api_url: Vec<u8>)


  /// Creates a cluster
  /// A cluster can hold up to 5 enclaves
  /// Mandate call
  register_cluster(origin: OriginFor<T>)

  /// Removes a cluster
  /// Mandate call
  /// Cluster must be empty
  unregister_cluster(origin: OriginFor<T> cluster_id: ClusterId)

  /// Secret NFT pallet needs to provide and interface to confirm 
  ///if an account belongs to SGX machine.
  /// return None if there is no accoiunt or else returns cluster_id, enclave_id
  ensure_enclave(sgx_account: T::Account)

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
