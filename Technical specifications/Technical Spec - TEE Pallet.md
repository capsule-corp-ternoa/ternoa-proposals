# Transmission Protocols
## Structures
```rust
pub struct Enclave<AccountId, MaxUriLen>
where
	AccountId: Clone + PartialEq + Debug,
	MaxUriLen: Get<u32>,
{
	pub enclave_address: AccountId,
	pub api_uri: BoundedVec<u8, MaxUriLen>,
	pub is_staked: bool,
}

pub struct Cluster<AccountId, ClusterSize>
where
	AccountId: Clone + PartialEq + Debug,
	ClusterSize: Get<u32>,
{
	pub enclaves: BoundedVec<AccountId, ClusterSize>,
}


```

## Types
```rust
type ClusterSize: Get<u32>;
type MaxUriLen: Get<u32>;
type ListSizeLimit: Get<u32>;
```

## Enums
```rust
/// Reward Destination enum
pub enum RewardDestination<AccountId> {
	/// Pay into the operator account, not increasing the amount at stake.
	Operator,
	/// Pay into a specified account.
	Account(AccountId),
	/// Receive no reward.
	None,
}
```

## Extrinsic
```rust
interface {
  /// Ask for registration of an enclave
  /// Origin : operator Account Address
  /// enclave_address :- Generated within each enclave (PK)
  /// api_uri :- API URI of the enclave
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
  								payee: RewardDestination<T::AccountId>)

  /// Removes an enclave from the system OR ask for removal if the enclave is assigned
  /// Origin : operator account address
  /// If the enclave is assigned, he will be placed in queue for tech committee approval and can 
  /// withdraw the unbonded stake after the unbonding period.
  /// If the enclave is not already assigned, he can exit without permission and can unbond and withdraw immediately.
  unregister_enclave_and_unbond(origin: OriginFor<T>)
  
  /// Withdraws the unbonded amount after the unlocking period
  withdraw_unbonded(origin: OriginFor<T>)

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
## Events
```rust
pub enum Event {
		EnclaveAddedForRegistration {
			operator_address: T::AccountId,
			enclave_address: T::AccountId,
			api_uri: BoundedVec<u8, T::MaxUriLen>,
		},
		RegistrationRemoved { operator_address: T::AccountId },
		UpdateRequestCancelled { operator_address: T::AccountId },
		UpdateRequestRemoved { operator_address: T::AccountId },
		MovedForUnregistration { operator_address: T::AccountId },
		EnclaveAssigned { operator_address: T::AccountId, cluster_id: ClusterId },
		EnclaveRemoved { operator_address: T::AccountId },
		MovedForUpdate {
			operator_address: T::AccountId,
			new_enclave_address: T::AccountId,
			new_api_uri: BoundedVec<u8, T::MaxUriLen>,
		},
		EnclaveUpdated {
			operator_address: T::AccountId,
			new_enclave_address: T::AccountId,
			new_api_uri: BoundedVec<u8, T::MaxUriLen>,
		},
		ClusterAdded { cluster_id: ClusterId },
		ClusterRemoved { cluster_id: ClusterId },
		Bonded { operator: T::AccountId, amount: BalanceOf<T> },
		Unbonded { operator: T::AccountId, amount: BalanceOf<T> },
		Withdrawn { operator: T::AccountId, amount: BalanceOf<T> },

}
```
## Errors
```rust
pub enum Error {
  	// TEE Pallet Errors 
	EnclaveNotFound,
	RegistrationAlreadyExists,
	OperatorAlreadyExists,
	EnclaveAddressAlreadyExists,
	UnregistrationAlreadyExists,
	UnregistrationLimitReached,
	RegistrationNotFound,
	EnclaveAddressNotFound,
	ClusterNotFound,
	ClusterIdNotFound,
	ClusterIsNotEmpty,
	ClusterIsFull,
	OperatorAndEnclaveAreSame,
	UpdateRequestAlreadyExists,
	UpdateRequestNotFound,
	UpdateProhibitedForUnassignedEnclave,
	AlreadyBonded,
}
```
## Side effects
```
NFT Pallet
- Checks on is_enclave
```
