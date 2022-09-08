# Ternoa Improvement Proposal - Secret NFTs

| Author(s)      | Mohsin Riaz, Amin Razavi, Prabhu Eshwarla |
| ----------- | ----------- |
| Created   | 7 Sep 2022       |
| TIP Number   | TIP500       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. the media associated with the NFTs are public in nature and can be viewed by anyone. Secret NFTs are a special kind of NFTs in Ternoa that support encrypted data. Only the current owner of the secret NFT can decrypt the secret NFT at any point.

## Abstract

Secret NFTs require the generation , exchange and storage of cryptographic keys stored in an offchain enclave running within a Trusted execution environment. The keys to encrypt and decrypt the data associated with the Secret NFTs are generated within the wallet or dApp, and transferred to the TEE enclave in a secure manner. The owner of the NFT can request the TEE enclave to retrieve the decryption key associated with the NFT, and use it to view the unencrypted data. Secret NFTs can be transferred peer-to-peer or traded in marketplaces. As an additional layer of security, TEE enclaves are configured in clusters that support Shamir secret sharing scheme.

## Motivation

While the blockchain as a public ledger of transactions provides irrefutable proof of ownership of NFTs, it does not meet the privacy of data needed for many NFT use cases. Secret NFTs have been designed by Ternoa with this in mind. Examples of private data that can be stored in NFTs include private images or videos, limited edition audio releases by music artists, private documents with long-term storage such as legal deed containing inheritance details and confidential company details.

## Specification - Onchain

### Secret NFT onchain lifecycle states
Secret NFTs will have the following lifecycle associated with them:
Pending Mint -> Minted -> Pending Burn -> Burned.

Secret NFT should support the following onchain interfaces:
```
interface {

  /// Interface Id: TIP500-01
  /// Description: User can convert a Basic NFT into a Secret NFT
  /// Constraint(s): 
  ///       - The Basic NFT should not be listed in a marketplace or delegated at the time of conversion to Secret NFT
  ///       - The Secret NFT when minted should initially be set to 'Pending Mint' State. Only when all the secret shares associated with the NFT have been stored in the enclaves, should the Secret NFT move to 'Minted' state.
  
  ConvertBasicToSecretNFT(cid ipfs_hash, uint256 nft_id );
  
  /// Interface Id: TIP500-02
  /// Description: User can directly mint a Secret NFT
  /// Constraint(s): 
  ///       - The Secret NFT when minted should initially be set to 'Pending Mint' State. Only when all the secret shares associated with the NFT have been stored in the enclaves, should the Secret NFT move to 'Minted' state.

  MintSecretNFT(cid offchain_uri, uint256 royalty, u32 collection_id?, bool is_soulbound);

  /// Interface Id: TIP500-04
  /// Description: This interface would be used to burn a secret NFT.
  /// Constraint(s):
  ///     - When Secret NFT is burned, its status should be set to 'Pending Burn'. Burning of secret NFT would be done only when all the secret shares associated with the NFT have been deleted from the enclaves.

  BurnSecretNFT(uint256 nft_id)

  /// Interface Id: TIP500-03
  /// Description: This interface is called by each of the TEE enclaves to confirm receipt of secret share for a given NFT. When all enclaves from a cluster confirm receipt of threshold shares, the secret NFT status goes to 'Minted', after which it can be transferred or listed on marketplace. This is a private interface available only for the enclaves to use
  /// Constraint(s): 
  ///       - Only enclaves can use this interface. Not to be used by dApps or users.
  ///       - When all the secret shares associated with a secret NFT have been confirmed to be received, then the NFT state should be changed from 'Pending Mint' to 'Minted'
  
  SecretShareReceivedForNFT(uint256 nft_id, uint32 enclave_id)

  /// Interface Id: TIP500-04
  /// Description: This interface is called by each of the TEE enclaves when the secret share associated with a secret NFT has been deleted by the wallet/dApp. This is a private interface available only for the enclaves to use
  /// Constraint(s): 
  ///       - Only enclaves can use this interface. Not to be used by dApps or users.
  ///       - When all the secret shares for an NFT have been confirmed to be deleted, the Secret NFT can be burned. Verify that the NFT is in 'Pending Burn' State before the burn is performed.  
  
  SecretShareDeletedForNFT(uint256 nft_id, uint32 enclave_id)

}

Additionally Secret NFTs should support the following interfaces which are already implemented in Basic NFT (so, no additional work is expected to support these interfaces):
1. Transfer a secret NFT from one wallet to another
2. List/Buy/Sell a secret NFT on a marketplace
```

## Additional Info

While the primary use case of Secret NFTs is to provide the ability to securely store encryption keys in TEE enclave, there can be many uses of this feature beyond encrypting NFT media. The key management and TEE enclaves system can be used to securely store any kind of data.

## Metadata

The secret NFT is an extension of the Basic NFT. The Basic NFT has its own metadata (link here) that is stored in json format in IPFS. 
The format for the offchain metadata of Secret NFT is suggested here:
```
{
2   "title":"(Optional) This the title of the Secret NFT",
3   "description":"(Optional) Description of the secret",
5   "properties":{
6      "encrypted_media":{
7         "hash":"media hash",
8         "type":"Type of media (file format)",
9         "size":"size of the encrypted media"
10      },
      "public_key_of_nft": "(Optional) public key associated with the Secret NFT",
11   }
12}
```
This metadata of secret NFT will be stored on IPFs, and it’s content Id (CID) will be stored onchain.

## Specification - Offchain

One of the primary challenges to be solved in implementing secret NFTs is the secure storage and retrieval of the encryption keys associated with each Secret NFT. The architecture proposed involves storage and retrieval of keys in a trusted execution environment (TEE) which is an offchain component associated with the secret NFT solution. TEE programs running on processors such as SGX provide strong trust guarantees in terms of data privacy and verification of the programs running within them. This can be achieved through techniques such as remote attestation that gives assurance that the program running inside the enclave is running on genuine TEE hardware (such as SGX), and the programs have not been modified by the TEE node operators. Data storage on TEEs are also secured by sealing them with the secure keys associated with the TEE hardware and/or author of the TEE programs.

The following is the workflow proposed for minting secret NFTs:
1. User selects a media or custom data for storing privately in a secret NFT.
2. The Wallet or dApp use the Ternoa SDK to generate a key pair for each NFT.
3. The private key of the generated keypair is used to encrypt the secret data.
4. The encrypted secret data is stored on IPFS and its content id (CID) is recorded.
5. The CID of the encrypted secret data is used to construct the offchain metadata json file
6. The offchain metadata file is stored on IPFS, and its content id (CID) is used to trigger an extrinsic on the blockchain to mint a secret NFT.
7. The id of the new NFT is obtained from the blockchain by the wallet/Dapp using  Ternoa SDK. 
8. The secret key used to encrypt the secret data is split into shares using a threshold secret scheme (such as Shamir Secret Shares). 
9. Each threshold share is then stored on a different enclave along with the associated NFT id. The number of threshold shares of the encryption key should be equal to the number of TEE enclaves running as offchain components.
10. Each TEE enclaves then posts a transaction on the blockchain confirming receipt of the secret share for a given NFT. 

The following is the workflow proposed for viewing a secret NFT by the owner:
1. User requests the wallet or dApp to decrypt the data associated with the NFT owned by them.
2. The wallet/dApp sends a request to each enclave with a signature generated from the user's account key, asking for the secret share associated with a given NFT.
3. The TEE enclave verifies if the requestor of the secret share is the owner of the NFT. Invalid requests are rejected. If the ownership is successfully verified from the blockchain, the secret share is sent to the requesting wallet/dApp.
4. The wallet/dApp receives the minimum threshold of shares from the set of available enclaves, and  reconstructs the encryption key for the secret. 
5. The encryption key is used to retrieve the original secret from IPFs which is then displayed to the user.

The enclave program should run within a TEE environment with the following characteristics:
1. The enclave program is deployed on a set of TEE-enabled hardware. It is recommended to have a minimum set of 5 TEE machines running in a cluster.
2. The enclave program should support interfaces to store and retrive secrets. The details of the interfaces provided by the TEE enclave program is described later in this section.
3. The enclave program should support remote attestation, which is a servie offered by the TEE processor vendor.
4. Each of the TEE machines should store one of the secret shares associated with an NFT. If the TEE cluster comprises of 5 machines as recommended, there would be a set of five secret shares generated each of which is stored on one of the TEE machines through a request to the TEE enclave program.
5. There should be a stand-by TEE cluster of 5 machines that can be manually activated if the primary TEE cluster malfunctions. There should be ability for the secret shares stored in the primary TEE cluster to be securely transferred to the backup TEE cluster, so that the secondary TEE cluster can be activated in case of contingencies.

The interfaces supported by the TEE enclave are describe below:
```
interface {
/// Store a secret share
StoreSecretShare(uint256 nft_id, bytes secret_share);

/// Retrieve a secret share
RetrieveSecretShare(uint256 nft_id);
}

```

## Constraints
* The secret NFT on the blockchain should be kept in 'Pending' status until all the secret shares for that NFT are stored on the TEE enclaves.

## Test cases

* User can mint a secret NFT directly
* User can convert a Basic NFT to secret NFT
* Owner can decrypt and view the secret associated the the secret NFT
* User can list secret NFT in the marketplace
* User can trade secret NFT in any marketplace
 
## References
TBD

## Copyright
TBD