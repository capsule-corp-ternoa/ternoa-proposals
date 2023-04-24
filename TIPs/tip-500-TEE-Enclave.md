# Ternoa Improvement Proposal - TEE Enclaves

| Author(s)      | Amin Razavi, Prabhu Eshwarla, Mohsin Riaz |
| ----------- | ----------- |
| Created   | 8 Sep 2022       |
| TIP Number   | TIP-500       |
| Version   | v0.1       |
| Requires   | <Link to Basic NFT TIP here>       |
| Status | In Progress       |
| Category   | NFT       |
| Discussions-To   | https://github.com/capsule-corp-ternoa/ternoa-hub/discussions     |

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
TEE is a descendant of HSM and TPM, with an affordable cost and flexibility of executing general purpose applications rather than just cryptographic functions. 

In spite of decades of scientific and technological efforts for securely outsourcing the computation tasks and storage to untrusted parties, we don't have any real-world feasible cryptographic protocol yet, and the available candidates need more decade of research and development. Hardware based security however has been the ultimate solution for critical areas of financial and military applications, in modern Android mobile devices, the TEE is already unknowingly used every day, by millions of people as an HSM equivalent, through the use of a Trusted Application (TA) providing the Android KeyMaster functionality. Recently there has been increasing attention to TEE as a source of trust for executing smart-contracts in blockchains.

While Basic NFTs merely represent ownership, Secure NFTs can further contain sensitive information or allow access to services which can only be accessed by the respective owner. One of the primary challenges to be solved in implementing secret NFTs is the secure storage and retrieval of the encryption keys associated with each Secret NFT.

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
In Ternoa architecture, enclaves are organized in clusters to support secret sharing. Each cluster is made of 5 different encalves. Enclaves of a cluster can either belong to a single private owner (for permissioned use) or be publicly distributed in different geographical locations or cloud providers. 

Proposal to register or remove a cluster requires approval from technical committee of the Ternoa network, which is accessible from [polkadotjs](https://polkadot.js.org/apps/?rpc=wss://mainnet.ternoa.network/) app for everyone.

Each TEE can contain multiple enclaves, and each enclave has an operator. The operator can register the enclave to cluster or remove it. 

Enclaves of a cluster essentially do not share any data between each other, however separate public clusters can p2p synchronize their corresponding slots to help network resilience.  
  
&nbsp;
## Enclave Specifications

- Intel SGX
- Gramine Library
- Rust programming language

&nbsp;

---
&nbsp;

## [Enclave API](https://github.com/capsule-corp-ternoa/sgx_server/tree/master/client)

&nbsp;
### General API
&nbsp;
* Healthchek
```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/health
METHOD:     GET
BODY:       -
PARAMETER:  -
RESPONSE:   
{
    "status": 200,
    "date": "2023-04-24 14:16:24",
    "description": "SGX server is running!",
    "enclave_address": "5C4xwspW1txnjCM71ZM2hpqqoqYa5x7iSkjAAYQqSNcSDnwG"
}

```

&nbsp;
* Attestation Quote

```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/quote
METHOD:     GET
BODY:       -
PARAMETER:  -
RESPONSE:   
{
    "status": "Success",
    "data": "020001002b0c00000e000e000000000015ad86b4cfa46b327a8bfb79aa0d67b70000000000000000000000000000000006080216ffff0400000000000000000000000000000000000000000000000000000000000000000000000000000000000700000000000000e7000000000000004747dc15efe9e8897dc0f2bfdc74a2c7b58bb09984c77f4c9fa01b2e6209ccfc0000000000000000000000000000000000000000000000000000000000000000ae790e644a1476cf87d2b64e547ed454879d351825507104e055a13048e71f6d00..."
}

```
&nbsp;
### Secret-NFT API
&nbsp;
* Availability of secret-nft on an enclave
```json
ENDPOINT:   https://mainnet-tee-c1n1v1.ternoa.network:8100//api/secret-nft/is-keyshare-available/123654
METHOD:     GET
BODY:       -
PARAMETER:  nft-id (uint32)
RESPONSE:   
{
    "enclave_id": "MAINNET-C1N1E1",
    "nft_id": 123654,
    "exists": false
}

```


&nbsp;
* View logs of secret-nft

```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/secret-nft/get-views-log/73331
METHOD:     GET
BODY:       -
PARAMETER:  nft-id (uint32)
RESPONSE:   
{
    "enclave_id": "ALPHANET-C1N1V2EI",
    "nft_id": 73390,

    "log": {
        "secret_nft": {
            "0": {
                "date": "2023-03-10 18:47:06",
                "account": {
                    "address": "5EyFqdBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7DvoeMdJ",
                    "role": "OWNER"
                },
                "event": "STORE"
            },
            "1": {
                "date": "2023-03-10 18:48:02",
                "account": {
                    "address": "5C4xwspW1txnjCM71ZM2hpqqoqYa5x7iSkjAAYQqSNcSDnwG",
                    "role": "DELEGATEE"
                },
                "event": "VIEW"
            },
            "2": {
                "date": "2023-03-10 18:48:08",
                "account": {
                    "address": "5EyFqdBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7DvoeMdJ",
                    "role": "OWNER"
                },
                "event": "VIEW"
            }
        },
        "capsule": {
            "0": {
                "date": "2023-03-10 18:48:23",
                "account": {
                    "address": "5EyFqdBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7DvoeMdJ",
                    "role": "OWNER"
                },
                "event": "VIEW"
            }
        }
    },

    "description": "Successful"
}

```

&nbsp;
* <span style="color:white">Store secret to enclave</span>.
```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/secret-nft/store-keyshare
METHOD:     POST
PARAMETER:  -
BODY:       
            {
                "owner_address": "5Ccq....v7tC",
                "signer_address": "5Fsr...UPQe_<Current Block Number>_<Validity Interval>",
                "signersig": " Above 'signer_address' field signed by 'owner' ",
                "data": "<NFT ID>_<Secret String>_<Current Block Number>_<Validity Interval>",
                "signature": " Above 'data' field signed by 'signer' "
            }
RESPONSE:   
{
    "status": "EXPIREDSIGNER",
    "enclave_id": "ALPHANET-C1N1E1",
    "nft_id": 3,
    "description": "TEE Key-share NFTSTORE: The signer account has been expired or is not in valid range."
}

```

&nbsp;
* <span style="color:white">Retrieve secret from enclave</span>.
```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/secret-nft/retrieve-keyshare
METHOD:     POST
PARAMETER:  -
BODY:       
            {
                "requester_address": "5Ccq....v7tC",
                "requester_type": "OWNER",
                "data": "<NFT ID>_<Current Block Number>_<Validity Interval>",
                "signature": "0x8ebd....c08f82"
            }
            
RESPONSE:   
{
    "status": "RETRIEVESUCCESS",
    "nft_id": 110,
    "enclave_id": "ALPHANET-C1N1E1",
    "keyshare_data": "dBXjyUXHEk3Thuxy5tVD617nyD8xnUYSAUY7Dvoe",
    "description": "TEE Key-share NFTRETRIEVE: Success retrieving nft_id key-share.",
}

```

&nbsp;
* <span style="color:white">Remove secret from enclave</span>.
```json
ENDPOINT:   https://alphanet-c1n1v2.ternoa.dev:8100/api/secret-nft/remove-keyshare
METHOD:     POST
PARAMETER:  -
BODY:       
            {
                "requester_address": "5G1AGc....DrzFs",
                "nft_id": 1336
            }
            
RESPONSE:   
{
    "status": "REMOVESUCCESS",
    "nft_id": 1336,
    "enclave_id": "ALPHANET-C1N1E1",
    "description": "Keyshare is successfully removed from enclave.",
}

```


&nbsp;
### Capsule API
Capsule endpoints are exactly the same as Secret-NFT API, by replacing *secret-nft* to *capsule-nft* someone can communicate capsules secrets.



&nbsp;

---

&nbsp;
### APP/SDK Interface
&nbsp;

```javascript
interface {
    /// Store a secret share
    StoreSecretShare(   uint32 nft_id, 
                        bytes encrypted_secret_share, 
                        bytes signature);

    /// Store a secret share
    ChangeSecretShare(  uint32 nft_id, 
                        bytes encrypted_secret_share, 
                        bytes signature);


    /// Retrieve a secret share
    RetrieveSecretShare(uint32 nft_id,
                        bytes optional_data,
                        bytes signature);
}

```
&nbsp;
### Blockchain Interfaces (SGX-Pallet)[TIP-800]
&nbsp;

``` javascript

// Add an enclave on blockchain
RegisterEnclave(bytes enclave_ias_report, bytes enclave_url);

// Remove an enclave from blockchain
UnregisterEnclave(bytes enclave_url)

```

&nbsp;
### Extrinsics
&nbsp;
``` javascript

// sgx-pallet : Scheduled keep-alive report to blockchain
// unsigned extrinsic
xt_heartbeat(bytes activity_proof);

// secret_nft-pallet, capsule-pallet : secret-share received,stored and sealed on enclave
// signed extrinsic
xt_secret(bytes xt_)

```
&nbsp;

---

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
