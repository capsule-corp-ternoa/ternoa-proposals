# Basic NFT Collections Technical Specifications

## Title
TSP Number: TSP-001
Authors: @markopoloparadox, @peshwar9, @ipapandinas
Status: Call for Feedback
Created: 2022-08-25
Reference Implementation [Link to a first reference implementation]

## Summary
A summary of the standard and the addressed issue. TODO

## Motivation
The motivation should describe what motivated the development of the standard as well as why particular decisions were made. TODO

## Specification
### Blockchain Extrinsic Interfaces:
```json
[
  {
    "module": "nft",
    "call": "create_collection",
    "description": "Creates an on-chain collection.",
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
        "name": "offchain_data",
        "type": "U8BoundedVec<CollectionOffchainDataLimit>",
        "description": "Content or Path to collection metadata. This can be an IPFS reference, link to a custom web location or just plain text.",
        "constraints": [
          "The length cannot be longer than CollectionOffchainDataLimit limit."
        ],
        "examples": [
          "QmUcXT3zVGtvpoeFXun32BGGUtWTyuQi1C3obBpbMVjncU",
          "https://cdn.pixabay.com/photo/2018/12/17/07/43/labrador-3879893_960_720.jpg",
          "Max the Dog #4"
        ]
      },
      {
        "name": "limit",
        "type": "Option<u32>",
        "description": "If provided, sets the maximum amount of NFTs that can be associated with that collection.",
        "constraints": "The value cannot be more than the CollectionSizeLimit limit."
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "CollectionCreated",
        "fields": [
          {
            "name": "collection_id",
            "type": "CollectionId",
            "description": "The autogenerated ID of the collection."
          },
          {
            "name": "owner",
            "type": "AccountId",
            "description": "The account address of the collection creator."
          },
          {
            "name": "offchain_data",
            "type": "U8BoundedVec<CollectionOffchainDataLimit>",
            "description": "Content or Path to collection metadata"
          },
          {
            "name": "limit",
            "type": "Option<u32>",
            "description": "If available, it represents the maximum of NFTs that can be associated with that collection."
          }
        ]
      }
    ],
    "errors": [
      "nft::CollectionLimitExceededMaximumAllowed"
    ]
  },
  {
    "module": "nft",
    "call": "burn_collection",
    "description": "Removes a collection from storage. This operation is irreversible.",
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
        "name": "collection_id",
        "type": "CollectionId",
        "description": "ID of the collection that needs to be removed from storage.",
        "constraints": [
          "The caller needs to be the owner of the collection."
        ]
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "CollectionBurned",
        "fields": [
          {
            "name": "collection_id",
            "type": "CollectionId",
            "description": "The ID of the collection that was removed from storage."
          }
        ]
      }
    ],
    "errors": [
      "nft::CollectionNotFound",
      "nft::NotTheCollectionOwner",
      "nft::CollectionIsNotEmpty"
    ]
  },
  {
    "module": "nft",
    "call": "close_collection",
    "description": "Closes a collection so that no new NFTs cannot be associated with this collection.",
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
        "name": "collection_id",
        "type": "CollectionId",
        "description": "The ID of the collection that needs to be closed.",
        "constraints": [
          "The caller needs to be the owner of the collection."
        ]
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "CollectionClosed",
        "fields": [
          {
            "name": "collection_id",
            "type": "CollectionId",
            "description": "The ID of the collection that was closed."
          }
        ]
      }
    ],
    "errors": [
      "nft::CollectionNotFound",
      "nft::NotTheCollectionOwner"
    ]
  },
  {
    "module": "nft",
    "call": "limit_collection",
    "description": "Limits a collection so that no more than the limit amount of NFTs can be associated with this collection.",
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
        "name": "collection_id",
        "type": "CollectionId",
        "description": "The ID of the collection that needs to be closed.",
        "constraints": [
          "The caller needs to be the owner of the collection."
        ]
      },
      {
        "name": "limit",
        "type": "u32",
        "description": "The maximum amount of NFTs that can be associated with a collection.",
        "constraints": [
          "The limit value needs to be bigger or equal to the existing mount of NFTs in the collection. The value cannot be more than the CollectionSizeLimit limit."
        ]
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "CollectionLimited",
        "fields": [
          {
            "name": "collection_id",
            "type": "CollectionId",
            "description": "The ID of the collection that was limited."
          },
          {
            "name": "limit",
            "type": "u32",
            "description": "The new maximum amount of NFTs that can be associated with a collection."
          }
        ]
      }
    ],
    "errors": [
      "nft::CollectionNotFound",
      "nft::NotTheCollectionOwner",
      "nft::CollectionLimitAlreadySet",
      "nft::CollectionIsClosed",
      "nft::CollectionHasTooManyNFTs",
      "nft::CollectionLimitExceededMaximumAllowed"
    ]
  },
  {
    "module": "nft",
    "call": "add_nft_to_collection",
    "description": "Associates an NFT with a collection.",
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
        "description": "The ID of the NFT that needs to associated with a collection.",
        "constraints": [
          "The caller needs to be the owner of the NFT.",
          "The NFT needs to be not associated with any collection."
        ]
      },
      {
        "name": "collection_id",
        "type": "CollectionID",
        "description": "The ID of the collection that needs to associated with an NFT.",
        "constraints": [
          "The caller needs to be the owner of the collection.",
          "The collection cannot be closed",
          "The collection needs to have less NFT associated with it than the maximum limit."
        ]
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "NFTAddedToCollection",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was associated with a collection."
          },
          {
            "name": "collection_id",
            "type": "CollectionId",
            "description": "The ID of the collection that was associated with a NFT."
          }
        ]
      }
    ],
    "errors": [
      "nft::CollectionNotFound",
      "nft::NotTheCollectionOwner",
      "nft::CollectionIsClosed",
      "nft::CollectionHasReachedLimit",
      "nft::NFTNotFound",
      "nft::NotTheNFTOwner",
      "nft::NFTBelongToACollection",
      "nft::CannotAddMoreNFTsToCollection"
    ]
  }
]
```

### SDK

> #### [Events](#events)
> #### [Extrinsics helpers](#extrinsics-helpers)
> #### [Storage getters](#storage-getters)
> #### [Constants getters](#constants-getters)

#### Events

```typescript
import { Event } from "@polkadot/types/interfaces/system";

export declare enum EventType {
    CollectionCreated = "nft.CollectionCreated",
    CollectionLimited = "nft.CollectionLimited",
    CollectionClosed = "nft.CollectionClosed",
    CollectionBurned = "nft.CollectionBurned",
}
export declare class BlockchainEvent {
    type: EventType;
    raw: Event;
    section: string;
    method: string;
    constructor(raw: Event, type: EventType);
    static fromEvent(event: Event): BlockchainEvent;
}

/**
 * This class represents the on-chain CollectionCreatedEvent event.
 */
export declare class CollectionCreatedEvent extends BlockchainEvent {
    collectionId: number;
    owner: string;
    offchainData: string;
    limit: number | null;
    /**
     * Construct the data object from the CollectionCreatedEvent event
     * @param event The CollectionCreatedEvent event
     */
    constructor(event: Event);
}
/**
 * This class represents the on-chain blockchain CollectionLimitedEvent event.
 */
export declare class CollectionLimitedEvent extends BlockchainEvent {
    collectionId: number;
    limit: number;
    /**
     * Construct the data object from the CollectionLimitedEvent event
     * @param event The CollectionLimitedEvent event
     */
    constructor(event: Event);
}
/**
 * This class represents the on-chain CollectionClosedEvent event.
 */
export declare class CollectionClosedEvent extends BlockchainEvent {
    collectionId: number;
    /**
     * Construct the data object from theCollectionClosedEvent event
     * @param event The CollectionClosedEvent event
     */
    constructor(event: Event);
}
/**
 * This class represents the on-chain CollectionBurnedEvent event.
 */
export declare class CollectionBurnedEvent extends BlockchainEvent {
    collectionId: number;
    /**
     * Construct the data object from the CollectionBurnedEvent event
     * @param event The CollectionBurnedEvent event
     */
    constructor(event: Event);
}
```

#### Extrinsics helpers

```typescript
import { IKeyringPair } from "@polkadot/types/types";

export declare type TransactionHashType = `0x${string}`;
export declare enum WaitUntil {
  BlockInclusion = 0,
  BlockFinalization = 1
}

/**
 * @name createCollectionTx
 * @summary               Creates an unsigned unsubmitted Create-Collection Transaction Hash.
 * @param offchainData    Off-chain related Collection metadata. Can be an IPFS Hash, an URL or plain text.
 * @param limit           The maximum amount that NFTs that the collection can hold. This is optional
 * @returns               Unsigned unsubmitted Create-Collection Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const createCollectionTx: (offchainData: string, limit?: number | undefined) => Promise<TransactionHashType>;

/**
 * @name createCollection
 * @summary               Creates a collection.
 * @param offchainData    Off-chain related Collection metadata. Can be an IPFS Hash, an URL or plain text.
 * @param limit           Amount of NFTs that can be associated with this collection. This is optional
 * @param keyring         Account that will sign the transaction.
 * @param waitUntil       Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns               CollectionCreatedEvent Blockchain event.
 */
export declare const createCollection: (offchainData: string, limit: number | undefined, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<CollectionCreatedEvent>;

/**
 * @name limitCollectionTx
 * @summary       Creates an unsigned unsubmitted Limit-Collection Transaction Hash.
 * @param id      The ID of the Collection.
 * @param limit   Amount of NFTs that can be associated with this collection.
 * @returns       Unsigned unsubmitted Limit-Collection Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const limitCollectionTx: (id: number, limit: number) => Promise<TransactionHashType>;

/**
 * @name limitCollection
 * @summary           Limits how many NFTs can be associated with this collection.
 * @param id          The ID of the Collection.
 * @param limit       Amount of NFTs that can be associated with this collection.
 * @param keyring     Account that will sign the transaction.
 * @param waitUntil   Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns           CollectionLimitedEvent Blockchain event.
 */
export declare const limitCollection: (id: number, limit: number, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<CollectionLimitedEvent>;

/**
 * @name closeCollectionTx
 * @summary   Creates an unsigned unsubmitted Close-Collection Transaction Hash.
 * @param id  The ID of the Collection.
 * @returns   Unsigned unsubmitted Close-Collection Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const closeCollectionTx: (id: number) => Promise<TransactionHashType>;

/**
 * @name closeCollection
 * @summary           Closes the collection so that no new NFTs can be added.
 * @param id          The ID of the Collection.
 * @param keyring     Account that will sign the transaction.
 * @param waitUntil   Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns           CollectionClosedEvent Blockchain event.
 */
export declare const closeCollection: (id: number, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<CollectionClosedEvent>;

/**
 * @name burnCollectionTx
 * @summary   Creates an unsigned unsubmitted Burn-Collection Transaction Hash.
 * @param id  The ID of the Collection.
 * @returns   Unsigned unsubmitted Burn-Collection Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const burnCollectionTx: (id: number) => Promise<TransactionHashType>;

/**
 * @name burnCollection
 * @summary           Burns an existing collection. The collections needs to be empty before it can be burned.
 * @param id          The ID of the Collection.
 * @param keyring     Account that will sign the transaction.
 * @param waitUntil   Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns           CollectionBurnedEvent Blockchain event.
 */
export declare const burnCollection: (id: number, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<CollectionBurnedEvent>;

```

#### Storage getters

```typescript
export interface ICollectionData {
  owner: string;
  offchainData: string;
  nfts: number[];
  limit: number;
  isClosed: boolean;
}

/**
 * @name getNextCollectionId
 * @summary Get the next collection Id available.
 * @returns Number.
 */
export declare const getNextCollectionId: () => Promise<number>;

/**
 * @name getCollectionData
 * @summary             Provides the data related to one NFT collection. ex:{owner, creator, offchainData, limit, isClosed(...)}
 * @param collectionId  The collection id.
 * @returns             A JSON object with data of a single NFT collection.
 */
export declare const getCollectionData: (collectionId: number) => Promise<ICollectionData | null>;
```

#### Constants getters

```typescript
/**
 * @name getCollectionSizeLimit
 * @summary Maximum collection length.
 * @returns Number.
 */
export declare const getCollectionSizeLimit: () => Promise<number>;
/**
 * @name getCollectionOffchainDataLimit
 * @summary Provides the maximum offchain data length.
 * @returns Number.
 */
export declare const getCollectionOffchainDataLimit: () => Promise<number>;
```
