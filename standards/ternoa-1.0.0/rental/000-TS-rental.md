
# Rent Technical Specifications

## Title
TSP Number: TSP-000
Authors: @Leouarz
Status: Call for Feedback
Created: 2022-09-01
Reference Implementation [Link to a first reference implementation]

## Summary
NFT Renting allows user to put their NFTs for rent. Rent can be used for any digital asset. A rent contract has multiple parameters defining the terms of the lease. (Duration, subscription, cancellation fee, ...).

## Motivation
This pallet is implement to give more flexbility to Ternoa's NFTs. It can replace all types of subscription and simple renting.

## Specification
### Blockchain Type Interfaces
```json
[
	{
		"name": "Duration",
		"description": "Represent the duration of the contract, it can be a fixed one, a subscription or an infinite amount of time.",
		"type": "enum",
		"format": "Duration<BlockNumber>": {
			"Fixed(BlockNumber)": "The contract will last a fixed amount of blocks.",
			"Subscription(BlockNumber, Option<BlockNumber>)": "The contract will take payment periodically until an optional maximum block number.",
			"Infinite": "The contract will take the fees then last forever or until the rentee revokes."
		}
	},
	{
		"name": "Acceptance Type",
		"description": "Represent how the contract works. It can be automatic on rent or manual, meaning that the contract creator will approve the rentee. Both method can specify a list of preapproved accounts.",
		"type": "enum",
		"format": "AcceptanceType<AccountList>": {
			"AutoAcceptance(Option<AccountList>)": "Contract will be accepted automatically.",
			"ManualAcceptance(Option<AccountList>)": "Contract rentee must be approved by contract creator."
		}
	},
	{
		"name": "Revocation Type",
		"description": "Represent how the contract can be revoked. This applies only to the contract creator since the rentee can revoke at any time (paying cancellation fee if it exists).",
		"type": "enum",
		"format": "RevocationType": {
			"NoRevocation": "The contract creator cannot revoke the contract.",
			"OnSubscriptionChange": "The contract will be revoked if the contract creator changes the terms and if the new terms are not accepted in time. (can only be used with Subscription duration).",
			"Anytime": "The contract creator can revoke at will."
		}
	},
	{
		"name": "Rent Fee",
		"description": "Represent the payment from the rentee to the contract creator.",
		"type": "enum",
		"format": "RentFee<Balance>": {
			"Tokens(Balance)": "An amount of tokens",
			"NFT(NFTId)": "An NFT (can only be used for fixed or infinite duration)",
		}
	},
	{
		"name": "Cancellation Fee",
		"description": "Represent the fee that will be transfered to the other side in case of revocation.",
		"type": "enum",
		"format": "CancellationFee<Balance>": {
			"FixedTokens(Balance)": "A fixed amount of tokens.",
			"FlexibleTokens(Balance)": "A flexible amount of tokens. The actual amount will be calculated depending on remaining blocks until the end of the contract (Can only be used with Fixed contract duration).",
			"NFT(NFTId)",
		}
	}
]
```
### Blockchain Extrinsic Interfaces
```json
[
  {
    "module": "rent",
    "call": "create_contract",
    "description": "Creates an on-chain rent contract for an NFT.",
    "privilege_level": "Signed",
    "constaints": [
	    "To chose OnSubscriptionChange, duration must be subscription.",
	    "To chose NFT as RentFee, duration must not be subscription.",
	    "To chose a renter cancellation fee, revocation type must not be NoRevocation.",
	    "To chose FlexibleFee for cancellation fee, duration must be fixed."
    ]
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "Id of the NFT to put for rent.",
        "constraints": [
          "The NFT must be owned by the caller, must not be listed, capsule, delegated, soulbound, rented."
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      },
      {
        "name": "duration",
        "type": "Duration<BlockNumber>",
        "description": "The duration of the contract.",
        "constraints": [
        ]
        "examples": [
			"Duration": {
				"Subscription": "(100, 1000)"
			},
			"Duration": {
				"Subscription": "(100, None)"
			},
			"Duration": {
				"Fixed": "(100)"
			},
			"Duration": "Infinite"
		]
      },
      {
        "name": "revocation_type",
        "type": "RevocationType",
        "description": "The revocation type of the contract applied to the contract creator.",
        "constraints": [
        ]
        "examples": [
			"RevocationType": "Anytime",
			"RevocationType": "NoRevocation",
			"RevocationType": "OnSubscriptionChange",
		]
      },
      {
        "name": "rent_fee",
        "type": "RentFee<Balance>",
        "description": "The payment from the rentee to the contract creator.",
        "constraints": [
        ]
        "examples": [
			"RentFee": "Tokens(200000000000000000000)",
			"RentFee": "NFT(1)",
		]
      },
      {
        "name": "renter_cancellation_fee",
        "type": "CancellationFee<Balance>",
        "description": "The payment from the contract creator to the rentee. Paid only in case of revocation.",
        "constraints": [
	        "Must have enough balance in case of fixed or flexible tokens.",
	        "Must be owned in case of NFT.",
	        "Must not be listed, capsule, delegated, soulbound, rented in case of NFT."
        ]
        "examples": [
			"CancellationFee": "FixedTokens(200000000000000000000)",
			"CancellationFee": "FlexibleTokens(200000000000000000000)",
			"CancellationFee": "NFT(1)",
		]
      },
      {
        "name": "rentee_cancellation_fee",
        "type": "CancellationFee<Balance>",
        "description": "The payment from the rentee to the contract creator. Paid only in case of revocation.",
        "constraints": [
	      	"Must not be listed, capsule, delegated, soulbound, rented in case of NFT."
        ]
        "examples": [
			"CancellationFee": "FixedTokens(200000000000000000000)",
			"CancellationFee": "FlexibleTokens(200000000000000000000)",
			"CancellationFee": "NFT(1)",
		]
      },
    ],
    "events": [
      {
        "module": "nft",
        "name": "NFTCreated",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put for rent."
          },
          {
            "name": "renter",
            "type": "AccountId",
            "description": "The contract creator."
          },
          {
            "name": "duration",
            "type": "Duration<BlockNumber>",
            "description": "The duration of the contract."
          },
          {
            "name": "acceptance_type",
            "type": "AcceptanceType<AccountList>",
            "description": "The method of contract acceptation."
          },
          {
            "name": "revocation_type",
            "type": "RevocationType",
            "description": "The method of contract revocation."
          },
          {
            "name": "rent_fee",
            "type": "RentFee<Balance>",
            "description": "The payment from the rentee to the renter."
          },
          {
            "name": "renter_cancellation_fee",
            "type": "CancellationFee<Balance>",
            "description": "The fee paid by the renter to the rentee in case of revocation."
          },
          {
            "name": "rentee_cancellation_fee",
            "type": "CancellationFee<Balance>",
            "description": "The fee paid by the rentee to the renter in case of revocation."
          }
        ]
      }
    ],
    "errors": [
      "rent::MaxSimultaneousContractReached",
      "rent::NFTNotFound",
      "rent::NotTheNFTOwner",
      "rent::CannotUseListedNFTs",
      "rent::CannotUseCapsuleNFTs",
      "rent::CannotUseDelegatedNFTs",
      "rent::CannotUseSoulboundNFTs",
      "rent::CannotUseRentedNFTs",
      "rent::SubscriptionChangeForSubscriptionOnly",
      "rent::NoNFTRentFeeWithSubscription",
      "rent::NoRenterCancellationFeeWithNoRevocation",
      "rent::FlexibleFeeOnlyForFixedDuration",
      "rent::InvalidFeeNFT",
      "rent::NFTNotFoundForRentFee",
      "rent::NFTNotFoundForCancellationFee",
      "rent::NotTheNFTOwnerForCancellationFee",
      "rent::NotEnoughBalanceForCancellationFee",
      "balance::InsufficientBalance",
      "balance::KeepAlive"
    ]
  },
  {
    "module": "rent",
    "call": "revoke_contract",
    "description": "Cancel the contract if not started yet, else revoke it.",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
          "The caller must renter or rentee if contract has started."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT which is put for rent.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractRevoked",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was put for rent."
          },
          {
            "name": "revoked_by",
            "type": "AccountId",
            "description": "The caller who revoked the contract."
          }
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::NFTNotFound",
      "rent::NotTheRenterOrRentee",
      "rent::CannotRevoke"
    ]
  },
  {
    "module": "rent",
    "call": "rent",
    "description": "Start a rent contract in case of automatic acceptance, makes an offer for rent in case of manual acceptance.",
    "privilege_level": "Signed",
    "constraints": [
      "Caller must have enough balance to cover tokens rentee cancellation fee if it exists.",
      "Caller must own NFT to cover NFT rentee cancellation fee if it exists.",
      "NFT must not be listed, capsule, delegated, soulbound or rented in case of NFT rentee cancellation fee.",
    ],
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
          "The caller needs to be authorized in case an account list was provided."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT that is put for rent.",
        "constraints": [
          "The caller needs to not be the owner of the NFT."
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractOfferCreated",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was put for rent."
          },
          {
            "name": "rentee",
            "type": "AccountId",
            "description": "The person who made an offer to rent the NFT."
          },
        ]
      },
      {
        "module": "rent",
        "name": "ContractStarted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was put for rent."
          },
          {
            "name": "rentee",
            "type": "AccountId",
            "description": "The person who rented the NFT."
          },
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::CannotRentOwnContract",
      "rent::NotEnoughBalanceForCancellationFee",
      "rent::NotTheNFTOwnerForCancellationFee",
      "rent::NFTNotFoundForRentFee",
      "rent::NotTheNFTOwnerForRentFee",
      "rent::NotEnoughBalanceForRentFee",
      "rent::CannotUseListedNFTs",
      "rent::CannotUseCapsuleNFTs",
      "rent::CannotUseDelegatedNFTs",
      "rent::CannotUseSoulboundNFTs",
      "rent::CannotUseRentedNFTs",
      "rent::NotEnoughBalance",
      "rent::NotAuthorizedForRent"
    ]
  },
  {
    "module": "rent",
    "call": "accept_rent_offer",
    "description": "Accept a rentee for manual acceptance contract.",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT that is put for rent.",
        "constraints": [
          "The caller needs to not be the owner of the NFT."
        ]
      },
      {
        "name": "rentee",
        "type": "AccountId",
        "description": "The person who will rent the contract.",
        "constraints": [
          "The rentee must have made an offer."
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractStarted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was put for rent."
          },
          {
            "name": "rentee",
            "type": "AccountId",
            "description": "The person who rented the NFT."
          },
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound"
      "rent::NotTheRenter"
      "rent::CannotAcceptOfferForAutoAcceptance"
      "rent::NoOffersForThisContract"
      "rent::NoOfferFromRentee"
      "rent::NotAuthorizedForRent"
      "rent::NotEnoughBalanceForCancellationFee",
      "rent::NotTheNFTOwnerForCancellationFee",
      "rent::NFTNotFoundForRentFee",
      "rent::NotTheNFTOwnerForRentFee",
      "rent::NotEnoughBalanceForRentFee",
      "rent::CannotUseListedNFTs",
      "rent::CannotUseCapsuleNFTs",
      "rent::CannotUseDelegatedNFTs",
      "rent::CannotUseSoulboundNFTs",
      "rent::CannotUseRentedNFTs",
      "rent::NotEnoughBalance"
    ]
  },
  {
    "module": "rent",
    "call": "retract_rent_offer",
    "description": "Retract a rent offer before contract has started.",
    "privilege_level": "Signed",
    "constraints": [
		"The contract must have not started.",
	]
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
          "The caller needs to have made an offer.",
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT that is put for rent for which the caller has made an offer.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractOfferRetracted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was put for rent."
          },
          {
            "name": "rentee",
            "type": "AccountId",
            "description": "The person who retracted the offer."
          },
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::CannotRetractOfferForAutoAcceptance",
      "rent::ContractHasStarted"
    ]
  },
  {
    "module": "rent",
    "call": "change_subscription_terms",
    "description": "Change the duration and the payment amount of a subscription contract.",
    "privilege_level": "Signed",
    "constraints": [
		"The contract must have started.",
	]
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
          "The caller must be owner of the contract.",
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT that is put for rent for which the caller wants to change terms.",
        "constraints": [
	        "Contract must be Subscription Duration."
        ]
      },
      {
        "name": "duration",
        "type": "Duration<BlockNumber>",
        "description": "The new duration of the contract.",
        "constraints": [
	        "Duration must be subscription with new values."
        ]
      },
      {
        "name": "amount",
        "type": "BalanceOf<T>",
        "description": "The new payment fee of the contract.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractSubscriptionTermsChanged",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "ID of the NFT that is put for rent for which the caller changed terms."
          },
          {
            "name": "duration",
            "type": "Duration<BlockNumber>",
            "description": "The new duration of the contract."
          },
          {
            "name": "rent_fee",
            "type": "RentFee<Balance>",
            "description": "The new payment fee of the contract."
          },
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::NotTheRenter",
      "rent::ContractHasNotStarted",
      "rent::CanChangeTermForSubscriptionOnly",
      "rent::CanSetTermsForSubscriptionOnly"
    ]
  },
  {
    "module": "rent",
    "call": "accept_subscription_terms",
    "description": "Accept the new contract terms set by the renter.",
    "privilege_level": "Signed",
    "constraints": [
		"The contract must have started.",
	]
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
          "The caller must be rentee of the contract.",
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT that is put for rent for which the caller wants to accept terms.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractSubscriptionTermsAccepted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "ID of the NFT that is put for rent for which the caller accepted new terms."
          }
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::NotTheRentee",
      "rent::ContractTermsAlreadyAccepted"
    ]
  },
  {
    "module": "rent",
    "call": "end_contract",
    "description": "End a contract when it reaches completion.",
    "privilege_level": "Root",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of root origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT Rented to end contract.",
        "constraints": [
        ]
      },
      {
        "name": "revoker",
        "type": "option<AccountId>",
        "description": "The person who triggers the ending of the contract in case there is one.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractEnded",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "ID of the NFT corresponding to the ended contract."
          },
          {
            "name": "revoker",
            "type": "option<AccountId>",
            "description": "THe person who triggered the end of the contract if it exists."
          }
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::NFTNotFound",
      "rent::ContractHasNotStarted",
      "Runtime::BadOrigin"
    ]
  },
  {
    "module": "rent",
    "call": "renew_contract",
    "description": "Renew a subscription based contract for a new period.",
    "privilege_level": "Root",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of root origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT corresponding to contract to renew. Will end the contract if rentee cannot pay the fee, subscription has reached maximum or subscription terms were not accepted.",
        "constraints": [
        ]
      },
      {
        "name": "now",
        "type": "Blocknumber",
        "description": "The current blocknumber.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractSubscriptionPeriodStarted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "ID of the NFT corresponding to the renewed contract."
          }
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::ContractHasNotStarted",
      "rent::RenewalOnlyForSubscription",
      "Runtime::BadOrigin"
    ]
  },
  {
    "module": "rent",
    "call": "remove_expired_contract",
    "description": "Removes a contract from storage in case it has not been rented for the expiration duration.",
    "privilege_level": "Root",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of root origin."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "ID of the NFT corresponding to contract remove from storage.",
        "constraints": [
        ]
      }
    ],
    "events": [
      {
        "module": "rent",
        "name": "ContractAvailableExpired",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "ID of the NFT corresponding to the expired contract."
          }
        ]
      }
    ],
    "errors": [
      "rent::ContractNotFound",
      "rent::NFTNotFound",
      "Runtime::BadOrigin"
    ]
  }
]
```

### SDK
