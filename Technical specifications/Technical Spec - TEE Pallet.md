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
}

pub struct Cluster<AccountId, ClusterSize>
where
	AccountId: Clone + PartialEq + Debug,
	ClusterSize: Get<u32>,
{
	pub enclaves: BoundedVec<AccountId, ClusterSize>,
}

pub struct EnclaveStakingLedger<T: Config> {
	/// The operator account whose balance is actually locked and at stake.
	pub operator: T::AccountId,
	/// The total amount of the stash's balance that we are currently accounting for.
	/// It's just `active` plus all the `unlocking` balances.
	#[codec(compact)]
	pub total: BalanceOf<T>,
	/// The total amount of the stash's balance that will be at stake in any forthcoming
	/// rounds.
	#[codec(compact)]
	pub active: BalanceOf<T>,
	/// Any balance that is becoming free, which may eventually be transferred out of the stash
	/// (assuming it doesn't get slashed first). It is assumed that this will be treated as a first
	/// in, first out queue where the new (higher value) eras get pushed on the back.
	pub unlocking: BoundedVec<UnlockChunk<BalanceOf<T>>, T::MaxUnlockingChunks>,
	/// List of eras for which the stakers have claimed rewards. 
	pub claimed_rewards: BoundedVec<EraIndex, T::HistoryDepth>,
}

pub struct UnlockChunk<Balance: HasCompact + MaxEncodedLen> {
	/// Amount of funds to be unlocked.
	#[codec(compact)]
	value: Balance,
	/// Era number at which point it'll be unlocked.
	#[codec(compact)]
	era: EraIndex,
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
	/// Pay into the operator account, increasing the amount at stake accordingly.
	Staked,
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
