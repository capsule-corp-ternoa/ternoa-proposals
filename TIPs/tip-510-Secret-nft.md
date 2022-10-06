
# Ternoa Improvement Proposal - Secret NFTs

| Author(s)      | Mohsin Riaz, Amin Razavi, Prabhu Eshwarla, Ghali El Ouarzazi |
| ----------- | ----------- |
| Created   | 7 Sep 2022       |
| TIP Number   | TIP-510       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | Draft       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |


## Simple Summary

NFTs represent proof of ownership on the blockchain. the media associated with the NFTs are public in nature and can be viewed by anyone. Secret NFTs are a special kind of NFTs in Ternoa that support encrypted data. Only the current owner of the secret NFT can decrypt the secret NFT at any point.

## Abstract

Secret NFTs require the generation , exchange and storage of cryptographic keys stored in an offchain enclave running within a Trusted execution environment. The keys to encrypt and decrypt the data associated with the Secret NFTs are generated within the wallet or dApp, and transferred to the TEE enclave in a secure manner. The owner of the NFT can request the TEE enclave to retrieve the decryption key associated with the NFT, and use it to view the unencrypted data. Secret NFTs can be transferred peer-to-peer or traded in marketplaces. As an additional layer of security, TEE enclaves are configured in clusters that support Shamir secret sharing scheme.

## Motivation

While the blockchain as a public ledger of transactions provides irrefutable proof of ownership of NFTs, it does not meet the privacy of data needed for many NFT use cases. Secret NFTs have been designed by Ternoa with this in mind. Examples of private data that can be stored in NFTs include private images or videos, limited edition audio releases by music artists, private documents with long-term storage such as legal deed containing inheritance details and confidential company details.

## Specification

### Lifecycle states

Secret NFTs will have the following lifecycle associated with them:
Pending Mint -> Minted -> Burned.

### External interfaces

Secret NFT should support the following onchain interfaces:
```rust
interface {

  /// Interface Id: TIP501-01
  /// Description: User can convert an existing Basic NFT into a Secret NFT
  /// Constraint(s): Refer to section 'Rules'
  add_secret(nft_id: NFTId, secret_offchain_data: offchain_data: BoundedVec<u8, NFTOffchainDataLimit>);
  
  /// Interface Id: TIP501-02
  /// Description: User can directly create an on-chain Secret NFT
  /// Constraint(s): Refer to section 'Rules'
  create_secret_nft(offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, secret_offchain_data: offchain_data: BoundedVec<u8, NFTOffchainDataLimit>, royalty: Permill, collection_id: Option<CollectionId>, is_soulbound: bool);


  /// Interface Id: TIP501-03
  /// Description: This interface is called by each of the TEE enclaves to confirm receipt of secret share for a given NFT. When all enclaves from a cluster confirm receipt of threshold shares, the secret NFT status goes to 'Minted', after which it can be transferred or listed on marketplace. This is a private interface available only for the enclaves to use
  /// Constraint(s): Refer to section 'Rules'
  add_secret_share(NFTId nft_id, uint32 enclave_id)
}

```
### Rules and constraints

#### convert_basic_to_secret_nft
- The Basic NFT should not be listed in a marketplace or delegated at the time of conversion to Secret NFT
- The Secret NFT when minted should initially be set to 'Pending Mint' State. Only when all the secret shares associated with the NFT have been stored in the enclaves, should the Secret NFT move to 'Minted' state.

#### create_secret_nft
- The Secret NFT when minted should initially be set to 'Pending Mint' State. Only when all the secret shares associated with the NFT have been stored in the enclaves, should the Secret NFT move to 'Minted' state.

#### secret_share_received_for_nft
- Only enclaves can use this interface. Not to be used by dApps or users.
- When all the secret shares associated with a secret NFT have been confirmed to be received, then the NFT state should be changed from 'Pending Mint' to 'Minted'

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

## End-to-end workflow (Ternoa-specific)

The following is the workflow proposed for minting secret NFTs:
1. User selects a media or custom data for storing privately in a secret NFT.
2. The Wallet or dApp use the Ternoa SDK to generate a key pair for each NFT.
3. The private key of the generated keypair is used to encrypt the secret data.
4. The encrypted secret data is stored on IPFS and its content id (CID) is recorded.
5. The CID of the encrypted secret data is used to construct the offchain metadata json file
6. The offchain metadata file is stored on IPFS, and its content id (CID) is used to trigger an extrinsic on the blockchain to mint a secret NFT.
7. The id of the new NFT is obtained from the blockchain by the wallet/Dapp using  Ternoa SDK. 
8. The secret key used to encrypt the secret data is split into shares using a threshold secret scheme (such as Shamir Secret Shares). 
9. Discovery of enclave locations (TBD)
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
