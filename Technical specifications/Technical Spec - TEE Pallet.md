# TEE Pallet
## Enums

```rust
pub enum ClusterType {
	Disabled,
	Admin,
	Public,
	Private,
}
```
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
	pub enclaves: BoundedVec<(AccountId, SlotId), ClusterSize>,
	pub cluster_type: ClusterType,
}

/// The ledger of a (bonded) operator.
pub struct TeeStakingLedger<AccountId, BlockNumber>
where
	AccountId: Clone + PartialEq + Debug,
	BlockNumber: Clone + PartialEq + Debug + sp_std::cmp::PartialOrd + AtLeast32BitUnsigned + Copy,
{
	/// The operator account whose balance is actually locked and at stake.
	pub operator: AccountId,
	/// State variable to know whether the staked amount is unbonded
	pub is_unlocking: bool,
	/// Block Number of when unbonded happened
	pub unbonded_at: BlockNumber
}

/// Report Parameters that metrics servers submits
pub struct MetricsServerReport<AccountId>
where
	AccountId: Clone + PartialEq + Debug,
{
	pub param_1: u8,
	pub param_2: u8,
	pub param_3: u8,
	pub param_4: u8,
	pub param_5: u8,
	pub submitted_by: AccountId,
}

/// Report Parameters weightage
pub struct ReportParamsWeightage
{
	pub param_1_weightage: u8,
	pub param_2_weightage: u8,
	pub param_3_weightage: u8,
	pub param_4_weightage: u8,
	pub param_5_weightage: u8,
}

/// Metrics Server details
pub struct MetricsServer<AccountId>
{
	pub metrics_server_address: AccountId,
	pub supported_cluster_type: ClusterType,
}
```

## Types
```rust
type ClusterSize: Get<u32>;
type MaxUriLen: Get<u32>;
type ListSizeLimit: Get<u32>;
type TeeBondingDuration: Get<u32>;
type InitialStakingAmount: Get<BalanceOf<Self>>;
type MaxMetricsReports: Get<u32>;
type MaxRewards: Get<u32>;
type HistoryDepth: Get<u32>;
type PalletId: Get<PalletId>;
type InitalDailyRewardPool: Get<BalanceOf<Self>>;
```

## Storages
```rust
	/// Tee Staking details mapped to operator address
	#[pallet::storage]
	#[pallet::getter(fn tee_staking_ledger)]
	pub type StakingLedger<T: Config> = StorageMap<
		_,
		Blake2_128Concat,
		T::AccountId,
		TeeStakingLedger<T::AccountId, T::BlockNumber>,
		OptionQuery,
	>;

	#[pallet::storage]
	#[pallet::getter(fn metrics_reports)]
	pub type MetricsReports<T: Config> = StorageDoubleMap<
		_,
		Blake2_128Concat,
		EraIndex,
		Blake2_128Concat,
		T::AccountId,
		BoundedVec<MetricsServerReport<T::AccountId>, T::ListSizeLimit>,
		OptionQuery,
	>;

	/// Report params weightage
	#[pallet::storage]
	#[pallet::getter(fn report_params_weightages)]
	pub type ReportParamsWeightages<T: Config> = StorageValue<_, ReportParamsWeightage, ValueQuery>;

	/// Daily reward amount for TEE operator.
	#[pallet::storage]
	#[pallet::getter(fn daily_reward_pool)]
	pub(super) type DailyRewardPool<T: Config> =
		StorageValue<_, BalanceOf<T>, ValueQuery, T::InitalDailyRewardPool>;

	#[pallet::storage]
	#[pallet::getter(fn rewards)]
	pub type ClaimedRewards<T: Config> = StorageDoubleMap<
		_,
		Blake2_128Concat,
		EraIndex,
		Blake2_128Concat,
		T::AccountId,
		BalanceOf<T>,
		OptionQuery,
	>;
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
  register_enclave_and_bond(origin: OriginFor<T>, enclave_address: Vec<u8>, api_uri: Vec<u8>)

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
  assign_enclave(origin: OriginFor<T>, operator: Account, cluster_id: ClusterId, slot_id: SlotId)
  
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
  create_cluster(origin: OriginFor<T>, cluster_type: ClusterType)

  /// Updates a cluster type
  /// Origin : Root
  /// Cluster must not be empty
  update_cluster(origin: OriginFor<T> cluster_id: ClusterId, cluster_type: ClusterType)

  /// Set Staking Amount
  /// Origin : Root
  set_staking_amount(origin: OriginFor<T>, staking_amount: BalanceOf<T>)

  /// Set report parameters weightage
  /// Origin : Root
  set_report_params_weightage(origin: OriginFor<T>, report_params_weightage: ReportParamsWeightage)

  /// Removes a cluster
  /// Origin : Root
  /// Cluster must be empty
  remove_cluster(origin: OriginFor<T> cluster_id: ClusterId)

  /// Withdraw the unbonded amount
  /// Origin: Operator address
  withdraw_unbonded(origin: OriginFor<T>)

  /// Register metrics server
  /// Origin : Root
  register_metrics_server(origin: OriginFor<T>, metrics_server: MetricsServer<T::AccountId>)

  /// Register submit report of metrics server
  /// Origin : Metric Server address
  submit_metrics_server_report(origin: OriginFor<T>, era_index: Option<EraIndex>, operator_address: T::AccountId, metrics_server_report: MetricsServerReport<T::AccountId>)

  /// Claim rewards by Era
  /// Origin: Operator Address
  claim_rewards(origin: OriginFor<T>, era: EraIndex)
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
		MetricsServerAdded { metrics_server_address: T::AccountId },
		/// Metrics server report submitted
		MetricsServerReportSubmitted {
			era: EraIndex,
			operator_address: T::AccountId,
			metrics_server_report: MetricsServerReport<T::AccountId>,
		},
		/// Report parameters weightage modified
		ReportParamsWeightageModified {
			param_1_weightage: u8,
			param_2_weightage: u8,
			param_3_weightage: u8,
			param_4_weightage: u8,
			param_5_weightage: u8,
		},
		/// Rewards claimed by operator
		RewardsClaimed { era: EraIndex, operator_address: T::AccountId, amount: BalanceOf<T> },
		

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
	StakingNotFound,
	UnbondingNotStarted,
	WithdrawProhibited,
	MetricsServerAlreadyExists,
	MetricsServerLimitReached,
	MetricsServerAddressNotFound,
        EnclaveNotFoundForTheOperator,
        FailedToGetActiveEra,
        ReportAlreadySubmittedForEra,
        MetricsReportsLimitReached,
        InvalidEraToClaimRewards,
}
```
## Side effects
```
NFT Pallet
- Checks on is_enclave
```
