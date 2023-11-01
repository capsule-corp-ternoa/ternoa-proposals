# Ternoa Improvement Proposal - TEE Enclaves

| Author(s)      | Amin Razavi, Prabhu Eshwarla, Mohsin Riaz |
| ----------- | ----------- |
| Created   | 8 Sep 2022       |
| TIP Number   | TIP-500       |
| Version   | v0.2       |
| Requires   | <[Basic NFT TIP](https://github.com/capsule-corp-ternoa/ternoa-proposals/blob/main/TIPs/tip-100-Basic-NFT.md)>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | <[Discussion](https://github.com/capsule-corp-ternoa/ternoa-hub/discussions)>     |

&nbsp;
## Simple Summary

Hardware trusted execution environment (TEE) is the most practical solution for confidentiality and privacy problem in blockchains, comparing to cryptographic algorithms like ZKP, FHE and SMPC. This document provides a standard for interfacing offchain TEE with essential components of blockchain with a focus on secret content for NFTs.

&nbsp;

## Abstract

The proposed architecture involves storage and retrieval of keys in a trusted execution environment (TEE) which is an offchain component associated with the secret NFT solution. TEE programs running on processors such as SGX provide strong trust guarantees in terms of data privacy and verification of the programs running within them. This can be achieved through techniques such as remote attestation that gives assurance that the program running inside the enclave is running on genuine TEE hardware (such as SGX), and the programs have not been modified by the TEE node operators. Data storage on TEEs are also secured by sealing them with the secure keys associated with the TEE hardware and/or author of the TEE programs.

As an offchain extension of Key Management, Secure Computation and Confidential Storage for blockchains, there are at least five responsibilities of TEE :

- Using Remote Attestation mechanism to prove the genuinity of hardware and the codes running on it to be approved by blockchain validators and registered on the blockchain

- Validation of the offchain requests from the application, comparing to onchain data (i.e NFT ownership)

- Processing the application request in a secure environment (i.e sealing the secrets)

- Providing the blockchain with verified offchain data gathered from application  (i.e Availability of encryption key for a secure NFT, Oracle)

- Secure distributed backup and secure migration of secrets to other TEE machines

&nbsp;

## Motivation

TEE is a descendant of hardware-based security solutions like HSM and TPM, with an affordable cost and flexibility of executing general purpose applications rather than pure cryptographic functions.

In spite of decades of scientific and technological efforts for securely outsourcing the computation tasks and storage to untrusted parties, we don't have any real-world feasible cryptographic protocol yet, and the available candidates need more decade of research and development. Hardware based security however has been the ultimate solution for critical areas of financial and military applications, in modern Android mobile devices, the TEE is already unknowingly used every day, by millions of people as an HSM equivalent, through the use of a Trusted Application (TA) providing the Android KeyMaster functionality. Recently there has been increasing attention to TEE as a source of trust for executing smart-contracts in blockchains.

While Basic NFTs merely represent ownership, Secret NFTs can further contain sensitive information or allow access to services which can only be accessed by the respective owner. One of the primary challenges to be solved in implementing secret NFTs is the secure storage and retrieval of the encryption keys associated with each Secret NFT.

Leveraging TEEs to bring a confidentiality aspect to NFTs, enables a variety of different use cases across different segments to provide their services more efficiently and securely. With the help of Secure NFTs, blockchains can enable real-world services such as Digital Rights Management, Secret Markets and Access control.

&nbsp;
## Threat Model

Major goal is to ensure the confidentiality of user data hosted by an untrusted server node. We assume a strong adversary with privileged access to OS and storage, who can not only monitor the content of all server’s memory, disk and communication, but also actively tamper with it. However, the adversary cannot access enclaves provided by TEE. In particular, data and computation inside an enclave are protected with respect to the confidentiality and integrity.

We exclude TEE side-channel attacks from our scope, since these vulnerabilities are implementation specific and we can adopt more secure TEEs when needed.
We do not consider the confidentiality of metadata and coarse statistical properties, such as the name of files and urls, and the length of values. In addition, we do not pursue strict indistinguishability of encrypted data and storage operations, since it usually leads to impractical performance penalty. Instead, we aim to provide operational data confidentiality where the information that the adversary learns is a function of data operations that have been performed.
Since the adversary do not have the Data Encryption Key (DEK), it cannot perform arbitrary data operations of its choice.

Note that other security guarantees, such as data integrity and protection from denial of service or TEE/Adversaries collusion, should be considered by designers. In fact, the complexity of additionally protecting data integrity also depends on the design choices.

&nbsp;

## Threshold Scheme

Despite being offchain entities, Enclaves are part of a blockchain ecosystem, so they should obey the rule of fault tolerability. No matter how secure are the Enclaves, there should not be a single point of failure in the whole system.

Threshold cryptosystems protect information by encrypting and distributing secrets amongst a cluster of independent computers,so the system will be able to continue operating despite failures or malfunctions.

Since the general idea is to assume that the participating entities are susceptible to compromise, decrypting the shared secret or signing a message requires the cooperation of some, not all, participants, usually above a minimum number. Threshold encryption also ensures that keyholders are able to collaborate without seeing other participants’ parts of the key.

With a minimum threshold, it can be ensured that even if some of the participants collude, the secret remains secure. In essence, this technique makes sure that a single individual doesn’t have the full authority, which can lead to vulnerability in the system. It also ensures that even if one or more individuals are unavailable, the secret can be unlocked and doesn’t cause bottlenecks.

&nbsp;

## Cluster Network

In Ternoa architecture, enclaves are organized in clusters to support secret-sharing threshold scheme. Each cluster is made of 5 different enclaves. Enclaves of a cluster can either belong to a single private owner (for permissioned use) or be publicly distributed in different geographical locations or cloud providers.

Proposal to register or remove a cluster requires approval from technical committee of the Ternoa network, which is accessible from [polkadotjs](https://polkadot.js.org/apps/?rpc=wss://mainnet.ternoa.network/) app for everyone.

Each TEE can contain multiple enclaves, while each enclave has an independent operator account. The operator can request to register the enclave to cluster or remove it from the network regarding the staking rules.

Enclaves which are members of a cluster essentially do not share any data between each other, while separate public clusters can peer-to-peer synchronize their corresponding "slots" to help the network reliability.

There are two types of general clusters :

- Public cluster
- Enterprise cluster

 Public clusters serve to Ternoa secret network unconditionally while enterprise clusters have constraints. The key difference is the Enclave Operator. Whenever the operators are controlled or assigned by a company or being limited by legal terms, their enclaves and clusters are considered as Enterprise. The famous example is the medical documnets that can not be stored on servers outside of a country. This case is a geographical limitation on enclave server location.

&nbsp;

## Inter-Enclave Synchronization

In order to create a decentralized fault-tolerant network of TEE enclaves, they should asynchronously communicate and replicate their confidential state.
Whenever an enclave receives, stores and seals a secret-share, it will send a confirmation transaction to the blockchain. When the blockchain receives 5 different confirmations for the same NFT-ID from a cluster, the blockchain will generate a 'NFT-synced' event in the current block. Since all encalves in the network are listening to the blockchain events, as soon as they notice the 'NFT-synced' event, they send a request to the original cluster and corresponding slot number for the new secret-share(s).

### Mutual Remote Attestation

Communication channel of enclaves is secured by expirable tokens, confidential signatures and TLS standard. However the enclaves have to prove their genuineness in each secret exchange. That means they should prove that : they are running in a verifiable updated (cpu microcodes) hardware, they are running under a secured mechanism of TEE in terms of memory, processing and storage and finally they are running an updated official code of open-source Ternoa encalve.

These are only possible through a hardware generated quote in TEE and then verifying it with the Intel as the manufacturer. To make the process bullet proof, we inject a signature of some confidential data to the quote before being signed by hardware key. These injected data will provide critical identity information to the other party that helps establishing the trust. Above all at the final stage, the data will be encrypted another time with a temporary public key which is generated securely inside the enclave.

&nbsp;

### New Enclave Synchronization

When a new enclaves gets registered on the network, it starts discovering the existing clusters and specially the corresponding slots in each cluster to its own slot number.

Then it chooses a proper enclave to fetch all of its current stored secret-shares. If the process fails, it goes to another cluster until all the secret-shares of available NFT-IDs be stored on the new enclave.

Then enclave will be ready to receive new data or synchronize its data to other enclaves.

&nbsp;

### Resuming Enclave Synchronization

Encalve should persistently keep the track of their lastest sychronization state, to be able to recover after downtimes, either because of network connection problem or hardware maintenance, etc.

Resuming after downtime, enclaves get the current block-number from the blockchain, then start crawling the chain looking for minted secret-nft or capsules. Result of the crawling is a list of NFT-IDs which their secre-shares should be fetched using synchronization process we've described above.

&nbsp;

## Enterprise Synchroniation

TBD

&nbsp;

## Enclave Specifications

- Hardware: Intel SGX
- SGX Development Library: Gramine Library
- Programming Language: Rust

&nbsp;

---
&nbsp;

## [Enclave API](https://github.com/capsule-corp-ternoa/sgx_server/tree/master/client)

&nbsp;

### General API

&nbsp;

- Health check

&nbsp;
Expresses the current status of enclave, as well as information about the identity of enclave, including an account address generated inside the enclave.
If this API does not return anything or status is not 200, client should wait or use another enclave.
&nbsp;

- sync_state: shows the last blocknumber that enclave has synchronized its data with other enclaves in the same slot. If the field is empty then the enclave is not registered on blockchain yet.

```json
// ENDPOINT:   https://[enclave-domain]:[port]/api/health
// METHOD:     GET
// BODY:       -
// PARAMETER:  -

//SAMPLE RESPONSE:  
{
    "chain": "mainnet",
    "block_number": 7847947,
    "sync_state": "[empty | setup | blocknumber]",
    "enclave_address": "SS58 AccountId securly generated inside enclave",
    "version": "0.4.4",
    "description": "SGX server is running!"
}
    
```

&nbsp;

- Attestation Quote
  
&nbsp;
To do a Remote Attestation against an enclave, a valid quote is essential. Then quote will be sent to Intel API with additional parameters like SPID and Key to receive the Attestation Report.

&nbsp;
The quote has timestamp and is signed with enclave secret hardware key that can only be validated with Intel IAS endpoint. In Ternoa protocol an additional vital part of the quote is 64-bytes user_data field which enclave fills it with a signature of critical identity information(MRENCLAVE, BLOCKNUMBER, ACCOUNTID, ENCRYPTION_PUBLICKEY). This generated quote will guarantee the genuineness of both enclave and the Ternoa's source code inside it. More information about quote structure and report protocol is on [Intel attestation API documentation](https://api.trustedservices.intel.com/documents/sgx-attestation-api-spec.pdf).

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/quote
//METHOD:     GET
//BODY:       -
//PARAMETER:  -

//SAMPLE RESPONSE:   
{
    "block_number": 7402861,
    "data": "Hex encoded string containing new quote and signature payload"
}

```

&nbsp;

### Secret-NFT API

&nbsp;

- Availability of a secret-nft on an enclave

&nbsp;
This endpoint is a short-cut to check if a specific nft-id is available onchain and its encryption key shares are stored on this enclave or not. Since this request does not need signed payload, it is more efficient to check before requesting to retrieve a specific nft secrets.

&nbsp;
Response contains block_number field, If the keyshare of the requested nft-id is not available (e.g exists: false) then the block_number is the current chain blocknumber. If the result is positive (e.g exists: true) then block_number is the lastest blocknumber on which the secret keyshares of the specified nft-id are updated and synced. If the keyshare on this enclave of the nft-id is available and updated but the sync event has not been detected (one of the remaining four keyshares on other enclaves has not been updated), the block_number will be 0.

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/secret-nft/is-keyshare-available/[nft-id]
//METHOD:     GET
//BODY:       -
//PARAMETER:  nft-id (uint32)

//SAMPLE RESPONSE:  
{
    "enclave_account": "5DDEgaWLgdrwCfJjy4B71bf54q5Gt14ffhumAkhmW4sK6Uox",
    "block_number": 7865927,
    "nft_id": 123456,
    "exists": false
}

```

&nbsp;

- View logs of secret-nft

&nbsp;
It is necessary in secrets marketplace for buyer to know by which owners and how many times a secret-nft is opened. Using this API, the full history of the secret-nft can be traced back up to the creation moment. It also shows capsule history if secret-nft is converted to capsule.

&nbsp;
Because of multi-cluster distribution of enclaves, one may store a keyshare to a cluster then update it on another cluster, thus following the update history can be a complex task if changes to keyshares are not done on the same cluster.

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/secret-nft/get-views-log/[nft-id]
//METHOD:     GET
//BODY:       -
//PARAMETER:  nft-id (uint32)

//SAMPLE RESPONSE:   
{
    "enclave_account": "5DDEgaWLgdrwCfJjy4B71bf54q5Gt14ffhumAkhmW4sK6Uox",
    "nft_id": 343451,
    "log": {
        "secret_nft": {
            "0": {
                "date": "2023-05-26 19:15:01",
                "block": 5883115,
                "account": {
                    "address": "5F6QubMDRvmogN2baV7MVXSRucdKgFD63MJu9kGw5rK1RzF5",
                    "role": "OWNER"
                },
                "event": "STORE"
            },
            "1": {
                "date": "2023-05-26 19:22:06",
                "block": 5883186,
                "account": {
                    "address": "5F6QubMDRvmogN2baV7MVXSRucdKgFD63MJu9kGw5rK1RzF5",
                    "role": "OWNER"
                },
                "event": "VIEW"
            },
            .
            .
            .
            // Removed Intentionally
            .
            .
            .
            "11": {
                "date": "2023-06-05 17:25:10",
                "block": 6025903,
                "account": {
                    "address": "5F6QubMDRvmogN2baV7MVXSRucdKgFD63MJu9kGw5rK1RzF5",
                    "role": "OWNER"
                },
                "event": "VIEW"
            },
            "12": {
                "date": "2023-06-05 17:25:21",
                "block": 6025904,
                "account": {
                    "address": "5F6QubMDRvmogN2baV7MVXSRucdKgFD63MJu9kGw5rK1RzF5",
                    "role": "OWNER"
                },
                "event": "VIEW"
            }
        },
        "capsule": {}
    },
    "description": "Successful"
}

```

&nbsp;

- <span style="color:white">Store secret to enclave</span>.

&nbsp;
It is possible for someone to mint multiple secret-nft at the same time, however every minting request requires approval from owner that is not pleasant from user experience point of view. Thus dApp can implicitly create a secure temporary account named "signer" for current owner specially for current multiple-minting request. The Signer account should be approved by real owner to be able to sign the requests to enclave. Regarding this protocol, the store request will need five fields :

- Owner address : Account ID of real Wallet/dApp owner who is minting the secret-nft. Enclave verifies the ownership of NFT-ID against this owner address.
  
- Authentication token : This two part token is a suffix to important data fields before signing. Its presence is significant to prevent replay attack to Enclaves.
  - Current Block Number : This proves that the request is not too old or for future
  - Validity Interval : produce a margin around current block number. Maximum value of this field is 20 blocks.
  
- Signer address: Temporary account generated for signing the secrets on behalf of the owner. This field must be concatenated with a valid authentication token through an underscore.

- Signer Signature : The owner should sign the signer address and authenticated token to approve the validity of it. Enclaves verify the signature then the token validity before processing the data.
  
- Secret data : Contains the NFT-ID, Secret Share of encryption key for secret-NFT and an Authentication token. This is the data that Enclaves seal and store.
  
- Signature : The signer account signs the Secret-Data to prove that this secret is owned by the owner of claimed NFT-ID.

Enclaves only support SR25519 signatures.

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/secret-nft/store-keyshare
//METHOD:     POST
//PARAMETER:  -
//POST BODY:       
            {
                "owner_address": "Owner's account id in SS58 format",
                "signer_address": "<Signer account id>_<Current Block Number>_<Validity Interval>",
                "signersig": " SR25519 Signature of above 'signer_address' field signed by 'owner' ",
                "data": "<NFT ID>_<Secret String>_<Current Block Number>_<Validity Interval>",
                "signature": " SR25519 Signature of above 'data' field signed by 'signer'  private key"
            }

//SAMPLE RESPONSE:
{
    "status": "EXPIREDSIGNER",
    "enclave_account": "5DDEgaWLgdrwCfJjy4B71bf54q5Gt14ffhumAkhmW4sK6Uox",
    "nft_id": 3,
    "description": "TEE Key-share NFTSTORE: The signer account has been expired or is not in valid range."
}

```

&nbsp;

- <span style="color:white">Retrieve secret from enclave</span>.

&nbsp;
Retrieve means request a secret-share to reconstruct the decryption key of secret-NFT media. So Enclave has to make sure that requester has enough privilege to view the NFT content.

There are three persons who can view a secret :

- Current nft owner
- Who has Rented the nft from current owner
- Who has been delegated by current owner

Enclaves needs to validate the requester with onchain data before letting her to view the secret. So requester must specify her privilege type specially if she is a Renter or Delegatee.
The requested NFT-ID has to be concatenated with an Authentication Token which contains current block number and validity interval. Then requester has to sign the data string to prove the ownership of the account id and authenticity of the request.

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/secret-nft/retrieve-keyshare
//METHOD:     POST
//PARAMETER:  -
//POST BODY:       
            {
                "requester_address": "Requester's account id in SS58 format",
                "requester_type": "[OWNER | RENTEE | DELEGATEE]",
                "data": "<NFT ID>_<Current Block Number>_<Validity Interval>",
                "signature": "SR25519 Signature of above 'data' field signed by 'requester' private key"
            }

//SAMPLE RESPONSE:   
{
    "status": "RETRIEVESUCCESS",
    "nft_id": 110,
    "enclave_account": "5DDEgaWLgdrwCfJjy4B71bf54q5Gt14ffhumAkhmW4sK6Uox",
    "keyshare_data": "dBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7Dvoe",
    "description": "TEE Key-share NFTRETRIEVE: Success retrieving nft_id key-share.",
}

```

&nbsp;

- <span style="color:white">Remove secret from enclave</span>.

&nbsp;
Only METRICS server can request to remove the data for a NFT-ID. However Enclaves checks the blockchain if the NFT associated to the requested NFT-ID has been burnt or converted already.

&nbsp;

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/secret-nft/remove-keyshare
//METHOD:     POST
//PARAMETER:  -
//POST BODY:       
            {
                "requester_address": "5G1AGc....DrzFs",
                "nft_id": 1336
            }

//SAMPLE RESPONSE:   
{
    "status": "REMOVESUCCESS",
    "nft_id": 1336,
    "enclave_account": "5DDEgaWLgdrwCfJjy4B71bf54q5Gt14ffhumAkhmW4sK6Uox",
    "description": "Keyshare is successfully removed from enclave.",
}

```

&nbsp;

### Capsule API

Capsule endpoints are exactly the same as Secret-NFT API, by replacing *secret-nft* to *capsule-nft* someone can communicate capsules secrets :

```json
Capsule Endpoints {
    //METHOD:     GET
    //BODY:       -
    //PARAMETER:  nft-id (uint32)
    "Keyshare Availability" : "/api/capsule-nft/is-keyshare-available/<nftid>",

    //METHOD:     GET
    //BODY:       -
    //PARAMETER:  nft-id (uint32)
    "Views Log" : "/api/capsule-nft/get-views-log/<nftid>",

    //METHOD:     POST
    //PARAMETER:  -
    "Store Secret": "/api/capsule-nft/store-keyshare",
    "body":
            {
                "owner_address": "Owner's account id in SS58 format",
                "signer_address": "<Signer account id>_<Current Block Number>_<Validity Interval>",
                "signersig": " SR25519 Signature of above 'signer_address' field signed by 'owner' ",
                "data": "<NFT ID>_<Secret String>_<Current Block Number>_<Validity Interval>",
                "signature": " SR25519 Signature of above 'data' field signed by 'signer' "
            }

    //METHOD:     POST
    //PARAMETER:  -
    "Retrieve Secret": "/api/capsule-nft/retrieve-keyshare",
    "body":       
            {
                "requester_address": "Requester's account id in SS58 format",
                "requester_type": "[OWNER | RENTEE | DELEGATEE]",
                "data": "<NFT ID>_<Current Block Number>_<Validity Interval>",
                "signature": "SR25519 Signature of above 'data' field signed by 'requester'"
            }

    //METHOD:     POST
    //PARAMETER:  -
    "Remove Secret" : "/api/capsule-nft/remove-keyshare"
    //POST BODY:       
            {
                "requester_address": "5G1AGc....DrzFs",
                "nft_id": 1336
            }
}
````

&nbsp;

### Cluster API

&nbsp;

- Request to synchronize with a slot

```json
//ENDPOINT:   https://[enclave-domain]:[port]/api/slot-sync/
//METHOD:     POST
//PARAMETER:  -
//POST BODY:       
            {
                "requester_address": "SS58 format AccountId of the Requester",
                "data": "<NFT ID>_<[List of NFTID]>_<Validity Interval>",
                "signature": "SR25519 Signature of above 'data' field signed by the Requester private key"
            }

//SAMPLE RESPONSE:   
{
    "status": "RETRIEVESUCCESS",
    "nft_id": 110,
    "enclave_id": "ALPHANET-C1N1E1",
    "keyshare_data": "dBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7Dvoe",
    "description": "TEE Key-share NFTRETRIEVE: Success retrieving nft_id key-share.",
}

```

&nbsp;

---

&nbsp;

### External Interfaces

&nbsp;

- [JS SDK Store Keyshare](https://github.com/capsule-corp-ternoa/ternoa-js/blob/3ed613d9c60c54d097f971975ba8cac66cee24ae/src/helpers/tee.ts#L234)
- [JS SDK Retrieve Keyshare](https://github.com/capsule-corp-ternoa/ternoa-js/blob/3ed613d9c60c54d097f971975ba8cac66cee24ae/src/helpers/tee.ts#L317)
&nbsp;
- [Blockchain Interfaces (TEE-Pallet)](https://github.com/capsule-corp-ternoa/ternoa-proposals/blob/eb26b8c0aea2350b567b006a411051e53bedf320/TIPs/tip-510-TEE-Pallet.md#existing-interfaces-changed)
&nbsp;

&nbsp;

## End-to-end workflow (Ternoa-specific)

### Store Secret

To mint a secret-NFT or Change the key of a Capsule:

- Client DApp encrypts NFT media with a Key.
- The Encryption Key  will be split into 5 shares using Shamir's algorithm.
- Client selects a cluster.
- Each share will be sent to a distinct enclave of the cluster.
- Enclave verifies the request, then
- Secrets will be encrypted and stored on enclave machine.
- A log file will be generated on each enclave that logs any access or changes to that nft-id

### Retrieve Secret

View a secret-NFT or Change Capsule media:

- Client selects the cluster that contains corresponding key-shares.
- Client asks every enclave to retrieve the secret of the specified nft-id.
- Enclave verifies the request, then decrypts the secret and returns it to client.
- When 3 of 5 enclaves return correct data, the original encryption key will be reconstructed in client wallet.
- Client uses the reconstructed key to decrypt the NFT/Capsule media.

### Remove Secret

- Client asks enclave to remove a burnt nft
- Enclave verifies the request
- The remaining secrets and logs will be purged from enclave

### P2P Synchronization

- Client DApp encrypts NFT media with a Key.
- The Encryption Key  will be split into 5 shares using Shamir's algorithm.
- Client selects a cluster.
- Each share will be sent to a distinct enclave of the cluster.

&nbsp;

- Client DApp encrypts NFT media with a Key.
- The Encryption Key  will be split into 5 shares using Shamir's algorithm.
- Client selects a cluster.
- Each share will be sent to a distinct enclave of the cluster.
- Enclave verifies the request, then
- Secrets will be encrypted and stored on enclave machine.
- A log file will be generated on each enclave that logs any access or changes to that nft-id

### Retrieve Secret
View a secret-NFT or Change Capsule media:

- Client selects the cluster that contains corresponding key-shares.
- Client asks every enclave to retrieve the secret of the specified nft-id.
- Enclave verifies the request, then decrypts the secret and returns it to client.
- When 3 of 5 enclaves return correct data, the original encryption key will be reconstructed in client wallet.
- Client uses the reconstructed key to decrypt the NFT/Capsule media.

### Remove Secret
- Client asks enclave to remove a burnt nft
- Enclave verifies the request
- The remaining secrets and logs will be purged from enclave

### P2P Synchronization

- Client DApp encrypts NFT media with a Key.
- The Encryption Key  will be split into 5 shares using Shamir's algorithm.
- Client selects a cluster.
- Each share will be sent to a distinct enclave of the cluster.

&nbsp;
## Constraints

TBD

&nbsp;

## Test cases

- Wallet/SDK stores secret-share to an enclave
- Wallet/SDK changes secret-share on enclave
- Wallet/SDK retrieves stored secret-share from enclave

- Wallet/SDK stores secret-share to cluster with success reports
- Wallet/SDK changes secret-share on cluster with success reports
- Wallet/SDK retrieves stored secret-share from cluster

&nbsp;

## References

TBD

&nbsp;
## Copyright

TBD
