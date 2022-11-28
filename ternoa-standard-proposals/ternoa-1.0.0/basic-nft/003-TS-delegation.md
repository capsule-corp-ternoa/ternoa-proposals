# Basic NFT Delegation Technical Specifications

## Title
TSP Number: TSP-003
Authors: @markopoloparadox, @peshwar9
Status: Call for Feedback
Created: 2022-08-25
Reference Implementation [Link to a first reference implementation]

## Summary
A summary of the standard and the addressed issue. TODO

## Motivation
The motivation should describe what motivated the development of the standard as well as why particular decisions were made. TODO

## Specification
### Blockcahin Extrinsic Interfaces:
```json
[
  {
    "module": "nft",
    "call": "delegate_nft",
    "description": "Delegates an NFT to someone or cancels a existing delegation.",
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
        "description": "The ID of the NFT that needs to be delegated or undelegated.",
        "constraints": [
          "The NFT needs to be owned by the caller.",
          "The NFT needs to be not listed for sale.",
          "The NFT needs to be not a capsule."
        ]
      },
      {
        "name": "recipient",
        "type": "Option<<T::Lookup as StaticLookup>::Source>",
        "description": "The account which is going to get the delegated NFT. If set to None it will undelegated the NFT."
      }
    ],
    "events": [
      {
        "module": "nft",
        "name": "NFTDelegated",
        "fields": [
          {
            "name": "nft_id",
            "type": "NFTId",
            "description": "The ID of the NFT that was delegated or undelegated."
          },
          {
            "name": "recipient",
            "type": "Option<AccountId>",
            "description": "If available, it represents the account that got the delegated NFT."
          }
        ]
      }
    ],
    "errors": [
      "nft::NotTheNFTOwner",
      "nft::CannotDelegateListedNFTs",
      "nft::CannotDelegateCapsuleNFTs"
    ]
  }
]
```

### SDK

> #### [Events](#events)
> #### [Extrinsics helpers](#extrinsics-helpers)

#### Events

```typescript
import { Event } from "@polkadot/types/interfaces/system";
export declare enum EventType {
    NFTDelegated = "nft.NFTDelegated",
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
 * This class represents the on-chain NFTDelegatedEvent event.
 */
export declare class NFTDelegatedEvent extends BlockchainEvent {
    nftId: number;
    recipient: string | null;
    /**
     * Construct the data object from the NFTDelegatedEvent event
     * @param event The NFTDelegatedEvent event
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
  BlockFinalization = 1,
}

/**
 * @name delegateNftTx
 * @summary           Creates an unsigned unsubmitted Delegate-NFT Transaction Hash.
 * @param id          The ID of the NFT.
 * @param recipient   Destination account. If set to undefined this functions acts as a way to undelegate a delegated NFT.
 * @returns           Unsigned unsubmitted Delegate-NFT Transaction Hash. The Hash is only valid for 5 minutes.
 */
export declare const delegateNftTx: (id: number, recipient?: string | undefined) => Promise<TransactionHashType>;
/**
 * @name delegateNft
 * @summary           Delegates an NFT to someone.
 * @param id          The ID of the NFT.
 * @param recipient   Destination account. If set to undefined this functions acts as a way to undelegate a delegated NFT.
 * @param keyring     Account that will sign the transaction.
 * @param waitUntil   Execution trigger that can be set either to BlockInclusion or BlockFinalization.
 * @returns           NFTDelegatedEvent Blockchain event.
 */
export declare const delegateNft: (id: number, recipient: string | undefined, keyring: IKeyringPair, waitUntil: WaitUntil) => Promise<NFTDelegatedEvent>;
```
