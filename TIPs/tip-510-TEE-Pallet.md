# Ternoa Improvement Proposal - TEE Pallet

| Author(s)      |  |
| ----------- | ----------- |
| Created   | 15 Nov 2022       |
| TIP Number   | TIP-510       |
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
  /// Ask for registration of an enclave
  /// Origin : operator Account Address
  /// enclave_address :- Generated within each enclave (PK)
  /// api_uri :- API URI of the enclave
  /// value :- Token value that is being staked with the registration
  /// payee :- Reward destination account ID
  /// Signed by operator (the origin)
  /// Notes:
  /// Enclave should have public endpoints to return:
  ///     1. Simple health check - 200
  ///     2. Binary hash
  ///     3. Quote
  ///     4. Enclave address & operator address
  /// One operator can have one enclave.
  register_enclave_and_bond(origin: OriginFor<T>, enclave_address: Vec<u8>, api_uri: Vec<u8>, 
  				#[pallet::compact] value: BalanceOf<T>, payee: RewardDestination<T::AccountId>)

  /// Removes an enclave from the system OR ask for removal if the enclave is assigned
  /// Origin : operator account address
  /// If the enclave is assigned, he will be placed in queue for tech committee approval and can 
  /// withdraw the unbonded stake after the unbonding period.
  /// If the enclave is not already assigned, he can exit without permission and can unbond immediately.
  unregister_enclave_and_unbond(origin: OriginFor<T>)
  
  /// Ask for update of the enclave fields
  /// Origin : operator account address
  /// If the enclave is not assigned, should trigger an error
  /// If the enclave is assigned, he will be placed in queue for tech committee approval
  /// Notes:
  /// Enclave should have public endpoints to return:
  ///     1. Simple health check - 200
  ///     2. Binary hash
  ///     3. Quote
  ///     4. Enclave address & operator address
  ///     5. Enclave should have backup of all the secret shares (in case the operator changes machine)
  update_enclave(origin: OriginFor<T>, enclave_address: Vec<u8>, api_uri: Vec<u8>)
  
  /// Cancels an update request
  /// Origin : operator account address
  cancel_update(origin: OriginFor<T>)
  
  /// Assigns an EnclaveId to a cluster
  /// Origin : Root
  assign_enclave(origin: OriginFor<T>, operator: Account, cluster_id: ClusterId)
  
  /// Remove a registration from the registration list (not assigned)
  /// Origin : Root
  remove_registration(origin: OriginFor<T>, operator: Account)
  
  /// Remove an update request from the update list (assigned)
  /// Origin : Root
  remove_update(origin: OriginFor<T>, operator: Account)
  
  /// Origin : Root
  /// Performs all cleanup
  remove_enclave(Origin, enclave_id)

  /// Force update enclave
  /// Origin : Root
  force_update_enclave(origin: OriginFor<T>, operator: Account, enclave_address: Account, api_url: Vec<u8>)

  /// Creates a cluster
  /// Origin : Root
  /// A cluster can hold up to 5 enclaves
  create_cluster(origin: OriginFor<T>)

  /// Removes a cluster
  /// Origin : Root
  /// Cluster must be empty
  remove_cluster(origin: OriginFor<T> cluster_id: ClusterId)
}
```

## Constraints

## Additional Info

### How to Sync:

`isSyncing` is set to `true` by default. Here are the steps to be sure you can sync your secret NFTs to enclaves:

1. Submit a new proposal to tech committee to create a cluster using “tee” pallet with mandate: 
   * PROPOSAL: mandate > mandate(call)
   * CALL: tee > `createCluster()`
2. Once cluster create. You can now register enclaves (minimum 5 enclaves required):
   * EXTRINSIC: tee > `registerEnclave()`
3. Assign Enclaves to the cluster
4. Each Enclave address should trigger `addSecretShard()` from “nft” pallet
5. Now it’s synced

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
