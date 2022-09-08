# Auction Technical Specifications

## Title
TSP Number: TSP-000
Authors: @Leouarz
Status: Call for Feedback
Created: 2022-09-02
Reference Implementation [Link to a first reference implementation]

## Summary
NFT Auction allows user to put their NFTs for sale in an auction format.

## Motivation
This pallet is implemented to give more flexbility to selling Ternoa's NFTs.

## Specification
### Blockchain Extrinsic Interfaces
```json
[
  {
    "module": "auction",
    "call": "create_auction",
    "description": "Creates an on-chain auction for an NFT.",
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
        "name": "marketplace_id",
        "type": "MarketplaceId",
        "description": "Marketplace on which the auction will take place.",
        "constraints": [
        ]
        "examples": [
			"0",
			"10",
		]
      },
      {
        "name": "start_block",
        "type": "BlockNumber",
        "description": "The start block of the auction.",
        "constraints": [
	        "The start block must not be after current block + auction delay."
        ]
        "examples": [
			"2_000_000",
		]
      },
      {
        "name": "end_block",
        "type": "BlockNumber",
        "description": "The end block of the auction.",
        "constraints": [
	        "The end block must not be after start block.",
	        "The end block minus the start block cannot be lower than minimum duration.",
	        "The end block minus the start block cannot be greater than maximum duration."
        ]
        "examples": [
			"2_000_000",
		]
      },
      {
        "name": "start_price",
        "type": "Balance",
        "description": "The price at which the auction will start.",
        "constraints": [
        ]
        "examples": [
			"200000000000000000000",
		]
      },
      {
        "name": "buy_it_price",
        "type": "option<Balance>",
        "description": "The optional price at which someone can buy the NFT directly.",
        "constraints": [
        ]
        "examples": [
			"Some(2000000000000000000000)",
			"None",
		]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "AuctionCreated",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          },
          {
            "name": "marketplace_id",
            "type": "MarketplaceId",
            "description": "The marketplace on which the NFT was put on auction."
          },
          {
            "name": "creator",
            "type": "AccountId",
            "description": "The creator of the auction."
          },
          {
            "name": "start_price",
            "type": "Balance",
            "description": "The start price of the auction."
          },
          {
            "name": "buy_it_price",
            "type": "Balance",
            "description": "The price at which someone can directly buy the NFT."
          },
          {
            "name": "start_block",
            "type": "BlockNumber",
            "description": "The block when the auction will start."
          },
          {
            "name": "end_block",
            "type": "BlockNumber",
            "description": "The block when the auction will end."
          }
        ]
      }
    ],
    "errors": [
      "auction::AuctionCannotStartInThePast",
      "auction::AuctionCannotEndBeforeItHasStarted",
      "auction::AuctionDurationIsTooLong",
      "auction::AuctionDurationIsTooShort",
      "auction::AuctionStartIsTooFarAway",
      "auction::BuyItPriceCannotBeLessOrEqualThanStartPrice",
      "auction::CannotListNotOwnedNFTs",
      "auction::CannotListListedNFTs",
      "auction::CannotListCapsulesNFTs",
      "auction::CannotListDelegatedNFTs",
      "auction::CannotListNotCreatedSoulboundNFTs",
      "auction::CannotListRentedNFTs",
      "auction::MarketplaceNotFound",
      "auction::AccountNotAllowedToList",
      "auction::PriceCannotCoverMarketplaceFee",
      "auction::MaximumAuctionsLimitReached",
    ]
  },
  {
    "module": "auction",
    "call": "cancel_auction",
    "description": "Cancels an on-chain auction for an NFT before it has started.",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
          "The caller needs to be the creator of the auction."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "Id of NFT to cancel auction for.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "AuctionCancelled",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          }
        ]
      }
    ],
    "errors": [
      "auction::NFTNotFound",
      "auction::AuctionDoesNotExist",
      "auction::NotTheAuctionCreator",
      "auction::CannotCancelAuctionInProgress",
    ]
  },
  {
    "module": "auction",
    "call": "end_auction",
    "description": "End an auction if it is in the extended period. The extended period means that the auction duration was increased because someone made a bid during the few last minutes of the auction (to avoid auction sniping)",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
          "The caller needs to be the creator of the auction."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "Id of NFT to end auction for.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "AuctionCompleted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          },
          {
            "name": "new_owner",
            "type": "AccountId",
            "description": "The address of the new NFT owner."
          },
          {
            "name": "paid_amount",
            "type": "Balance",
            "description": "The amount paid by the new owner."
          },
          {
            "name": "marketplace_cut",
            "type": "Balance",
            "description": "The amount given to the marketplace owner."
          },
          {
            "name": "royalty_cut",
            "type": "Balance",
            "description": "The amount given to the NFT creator."
          },
          {
            "name": "auctioneer_cut",
            "type": "Balance",
            "description": "The amount given to the auctioneer."
          }
        ]
      }
    ],
    "errors": [
      "auction::AuctionDoesNotExist",
      "auction::NotTheAuctionCreator",
      "auction::CannotEndAuctionThatWasNotExtended",
      "auction::NFTNotFound",
      "auction::AuctionDoesNotExist",
      "auction::NFTNotFound",
    ]
  },
  {
    "module": "auction",
    "call": "add_bid",
    "description": "Add a bid for an auction, increase the amount in case of existing bid.",
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
        "description": "Id of NFT to bid for.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      },
      {
        "name": "amount",
        "type": "Balance",
        "description": "Amount of the bid.",
        "constraints": [
        ],
        "examples": [
          "10000000000000000000",
          "20000000000000000000000",
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "BidAdded",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT to bid for."
          },
          {
            "name": "bidder",
            "type": "AccountId",
            "description": "Account of the bidder."
          }
        ]
      },
      {
        "module": "auction",
        "name": "BidDropped",
        "description": "Event triggered when an old bid has been removed from list. Example: We store the last 25 bids. When the 26th will come, the first one will be dropped."
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT to bid for."
          },
          {
            "name": "bidder",
            "type": "AccountId",
            "description": "Account of the bidder that was dropped."
          },
          {
            "name": "amount",
            "type": "Balance",
            "description": "The amount of the bid that was dropped."
          }
        ]
      },
    ],
    "errors": [
      "auction::AuctionDoesNotExist",
      "auction::CannotAddBidToYourOwnAuctions",
      "auction::AuctionNotStarted",
      "auction::CannotBidLessThanTheHighestBid",
      "auction::CannotBidLessThanTheStartingPrice",
      "balance::KeepAlive",
    ]
  },
  {
    "module": "auction",
    "call": "remove_bid",
    "description": "Remove a bid. Can be called only if auction is not in the ending period.",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin."
          "The caller needs to have made a bid."
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "Id of NFT to remove bid for.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "BidRemoved",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          },
          {
            "name": "bidder",
            "type": "AccountId",
            "description": "Account that removed bid."
          },
          {
            "name": "amount",
            "type": "Balance",
            "description": "Amount of the bid that was removed."
          }
        ]
      }
    ],
    "errors": [
      "auction::AuctionDoesNotExist",
      "auction::CannotRemoveBidAtTheEndOfAuction",
      "auction::BidDoesNotExist"
    ]
  },
  {
    "module": "auction",
    "call": "buy_it_now",
    "description": "Buy and auctioned NFTs at the bu_it_price if it was specified by auction creator. Will not work if there is a bid greater than buy_it_price.",
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
        "description": "Id of NFT to buy now.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "AuctionCompleted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          },
          {
            "name": "new_owner",
            "type": "AccountId",
            "description": "The address of the new NFT owner."
          },
          {
            "name": "paid_amount",
            "type": "Balance",
            "description": "The amount paid by the new owner."
          },
          {
            "name": "marketplace_cut",
            "type": "Balance",
            "description": "The amount given to the marketplace owner."
          },
          {
            "name": "royalty_cut",
            "type": "Balance",
            "description": "The amount given to the NFT creator."
          },
          {
            "name": "auctioneer_cut",
            "type": "Balance",
            "description": "The amount given to the auctioneer."
          }
        ]
      }
    ],
    "errors": [
      "auction::NFTNotFound",
      "auction::AuctionDoesNotExist",
      "auction::AuctionDoesNotSupportBuyItNow",
      "auction::CannotBuyItNowToYourOwnAuctions",
      "auction::AuctionNotStarted",
      "auction::CannotBuyItWhenABidIsHigherThanBuyItPrice"
    ]
  },
  {
    "module": "auction",
    "call": "complete_auction",
    "description": "End an auction when time is due.",
    "privilege_level": "Root",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of root origin.",
        ]
      },
      {
        "name": "nft_id",
        "type": "NFTId",
        "description": "Id of NFT to complete auction for.",
        "constraints": [
        ],
        "examples": [
          "0",
          "256",
          "1000"
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "AuctionCompleted",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT put on auction."
          },
          {
            "name": "new_owner",
            "type": "AccountId",
            "description": "The address of the new NFT owner."
          },
          {
            "name": "paid_amount",
            "type": "Balance",
            "description": "The amount paid by the new owner."
          },
          {
            "name": "marketplace_cut",
            "type": "Balance",
            "description": "The amount given to the marketplace owner."
          },
          {
            "name": "royalty_cut",
            "type": "Balance",
            "description": "The amount given to the NFT creator."
          },
          {
            "name": "auctioneer_cut",
            "type": "Balance",
            "description": "The amount given to the auctioneer."
          }
        ]
      }
    ],
    "errors": [
      "auction::NFTNotFound",
      "auction::AuctionDoesNotExist"
    ]
  },
  {
    "module": "auction",
    "call": "claim",
    "description": "Claim an amount that was bid but did not won the auction.",
    "privilege_level": "Signed",
    "parameters": [
      {
        "name": "origin",
        "type": "OriginFor<T>",
        "description": "",
        "constraints": [
          "The caller needs to be of signed origin.",
        ]
      }
    ],
    "events": [
      {
        "module": "auction",
        "name": "BalanceClaimed",
        "fields": [
          {
            "name": "account",
            "type": "AccountId",
            "description": "The account that claimed an amount."
          },
          {
            "name": "amount",
            "type": "Balance",
            "description": "The amount returned to the called."
          }
        ]
      }
    ],
    "errors": [
      "auction::ClaimDoesNotExist"
    ]
  },
]
```

### SDK
