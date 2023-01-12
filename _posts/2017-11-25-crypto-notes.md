---
layout: basic
title: Crypto-notes
---

## Bitcoin, Ethereum, Cardano, NEO, etc.

- A system for securely transmitting and storing value ([whitepaper](https://bitcoin.org/bitcoin.pdf))
- [Reference implementation (first version)](https://github.com/bitcoin/bitcoin/commit/4405b78d6059e536c36974088a8ed4d9f0f29898)
- People often think of Bitcoin as only virtual money or a transaction system. But if you look closer, you'll see that the monetary aspect is just the tip of the iceberg. Money is merely one of the possible applications.
- [PGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) was the **first** major iteration of applied cryptography and Bitcoin the **second**. The interaction and integration of crypto into the very fabric of a decentralized web will be the refined **third** implementation.
- Intend to support numerous advanced features, including
  - User-issued currencies
  - [Smart Contracts](https://en.wikipedia.org/wiki/Smart_contract)
  - and even what we think is the first proper implementation of [decentralized autonomous organizations](https://en.wikipedia.org/wiki/Decentralized_autonomous_organization) (DAOs) or companies (DACs)
- Smart Contracts: decentralized logic, or distributed stored-procedures
  - **Executable programs that run on top of an immutable distributed database whose inputs and outputs are maintained globally consistent by a distributed [consensus protocol](https://www.adjoint.io/posts/different-concensus.html).**
  - A contract is like an autonomous agent that runs a certain piece of code every time a transaction is sent to it, and this code can modify the contract's internal data storage or send transactions. Advanced contracts can even modify their own code.
  - Contracts that run on a blockchain is a specific minimal implementation of a distributed database which has certain properties that are amenable to creation of so-called cryptocurrencies.
  - [Stephen Diehl](http://www.stephendiehl.com/) [wrote](http://www.stephendiehl.com/posts/smart_contracts.html):
    - *Basically a Contract is a Smalltalk object that allows message passing through transactions on a blockchain. The code is deployed by it's owner and then anyone on the network can interact it with it by message passing which may result in data or state changes on the global network. The execution occurs on the 'miners' on the network who execute the contract code with the inputs and outputs specified in the transaction and compute a solution to a RAM-hard SHA inversion problem called Ethash which adds the new block to the chain and updates the global state of contract network. Whichever miner solves the problem first gets to append the newly hashed block and the process continues ad-infinitum with the global network converging on consensus.<br>
    From a programming language perspective this introduces the non-trivial constraint that programs must necessarily terminate. The current implementation accomplishes this by attaching a cost semantics to each opcode in the virtual machine that expends a finite resource called 'gas' that is a function of the current block.*
   - [Smart Contracts Topology](https://www.adjoint.io/posts/database_blockchain.html) (Adjoint Blog)
   - In 1997, [Nick Szabo](https://en.wikipedia.org/wiki/Nick_Szabo) came out with the idea of Smart Contracts. He gave a definition of Smart Contracts based on a simple idea:
     - *Smart Contract: A set of promises, including protocols within which the parties perform on the other promises. The protocols are usually implemented with programs on a computer network, or in other forms of digital electronics, thus these contracts are "smarter" than their paper-based ancestors.*
   - In simple terms, a Smart Contract is a program that captures all the obligations of legal contracts and executes them automatically if predetermined events occur. Smart Contracts have then no vocation to replace existing legal contracts, but simply make their executions much more efficient.
 - [Swarm](https://ethereum.stackexchange.com/a/388): decentralized storage
   - When imagining [Ethereum](https://www.ethereum.org/) as a metaphor for a shared computer, it should be noted that computation alone is not enough. For a computer to be fully useful, it also needs storage to "remember" things and bandwidth to "communicate" them.
   - More specifically, Swarm is being designed as an accounting protocol that benefits from the automatic execution of so-called "Smart Contracts" running on the Ethereum Virtual Machine ([EVM](https://ethereum.stackexchange.com/questions/268/ethereum-block-architecture/6413#6413)). This accounting protocol is independent of the physical storage mechanism. That is, it is not intrinsically tied to a specific storage system. It could be [IPFS](https://ipfs.io/), [BitTorrent](http://www.bittorrent.com/) , or some future technology not yet invented.
 - [Whisper](https://ethereum.stackexchange.com/a/388): decentralized messaging
 - [DApp](https://ethereum.stackexchange.com/a/384) is an abbreviated form for **decentralized application**.
   - A DApp has its backend code running on a decentralized peer-to-peer network. Contrast this with an app where the backend code is running on centralized servers.
   - A DApp can have frontend code and user interfaces written in any language (just like an app) that can make calls to its backend. Furthermore, its frontend can be hosted on decentralized storage such as Swarm or IPFS.
   - If an App = frontend + server, since Ethereum contracts are code that runs on the global Ethereum decentralized peer-to-peer network, then:
      - *DApp = frontend + contracts*
 - This effort will also allow replacement of typical content delivery networks ([CDN](https://en.wikipedia.org/wiki/Content_delivery_network)) with a distributed hash table ([DHT](https://en.wikipedia.org/wiki/Distributed_hash_table)) pointing to file blobs, much how BitTorrent works.

## Hash function

- [Definition](https://en.wikipedia.org/wiki/Hash_function)
- Properties
  - [Pre-image resistance](https://crypto.stackexchange.com/a/1174)
  - [Second pre-image resistance](https://crypto.stackexchange.com/a/20998)
  - [Collision resistance](https://crypto.stackexchange.com/a/1174)
- [SHA-3 (Secure Hash Algorithm 3)](https://en.wikipedia.org/wiki/SHA-3)
  - *Variant: SHA3-256*
- [Message digest](https://en.wikipedia.org/wiki/Cryptographic_hash_function): The output of a one-way function when applied to a stream of data.

## Merkle tree

- [Definition](https://en.wikipedia.org/wiki/Merkle_tree)
- [Reference implementation](https://github.com/adjoint-io/merkle-tree) (Adjoint)
- Binary trees of hashed data
- Constructed from the leaves up
- Leaves are usually [network transactions](https://en.bitcoin.it/wiki/Transaction)
  - **If one leave change the root will too**
- A way of hashing a large number of "chunks" of data together which relies on splitting the chunks into buckets, where each bucket contains only a few chunks, then taking the hash of each bucket and repeating the same process, continuing to do so until the total number of hashes remaining becomes only one: **the root** hash.
- The benefit of this strange kind of hashing algorithm is that it allows for a neat mechanism known as Merkle proofs ([definition](https://www.certificate-transparency.org/log-proofs-work))
  - They tell us whether a piece of data belongs to a merkle tree or not
  - [Why is it (Merkle proofs) an ideal Proof of Work algorithm?](https://github.com/zcoinofficial/zcoin/wiki/What-is-MTP-(Merkle-Tree-Proof)-and-why-is-it-an-ideal-Proof-of-Work-algorithm%3F)
  - Consists of a chunk, the root hash of the tree, and the "branch" consisting of all of the hashes going up along the path from the chunk to the root. Someone reading the proof can verify that the hashing, at least for that branch, is consistent going all the way up the tree, and therefore that the given chunk actually is at that position in the tree.
    - The Bitcoin blockchain uses Merkle proofs in order to store the transactions in every block.

## Finite field

- [Definition](https://en.wikipedia.org/wiki/Finite_field)
- A finite set of positive integers of which multiplication, division, and substraction, is defined and done with a modulus.
- F<sub>π</sub> = { 0, 1, 2, 3, 4, 5, 6 } *(7 elements)*
- g<sub>n</sub> = 3 *(a generator)*
  - 1 + 5 = 6
  - 2 + 6 = 1
  - 4 * 1 = 4
  - 3 * 4 = 12 % 7 = 5 *(7, because we have 7 elements)*
- Watch [this](https://youtube.com/watch?v=wjyiOXRuUdo) talk by Thomas Dietert

## Elliptic-curve cryptography

- [Definition](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography)
- y<sup>2</sup> = χ<sup>3</sup> + αχ + β
- (χ, y) ∋ F<sub>π</sub>
- [Secp256k1](https://en.bitcoin.it/wiki/Secp256k1)
- Elliptic Curve Digital Signature Algorithm (ECDSA)
  - [Definition](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
  - a pair of algorithms used to sign and verify a piece of data using ECC
  - Sign
    - is to combine and ECC private key and the hashed data to produce a pair of integers (from which the private key cannot be derived)
  - Verify
    - is to mathematically prove, using the public key of the signer, that the signature was produced using the data and private key
- [Cryptonite](https://hackage.haskell.org/package/cryptonite) (Haskell library)

## Blockchain

- A distributed public ledger which securely records and facilitates irreversible peer-to-peer transactions which cannot be easily amended.
- A replicated [finite-state machine](https://en.wikipedia.org/wiki/Finite-state_machine) maintaining a cryptographically verifiable immutable history of
- A monitically increasing chain of blocks in which subsequent blocks are verifies based on the data of the preciding blocks in the chain.
- Constructed by forcing every [block](https://en.bitcoin.it/wiki/Block) to contain a [hash](#hash-function) of the previous block's [header](https://en.bitcoin.it/wiki/Block) in its header
a transaction database shared by all nodes participating in a system based on the Bitcoin protocol. A full copy of a currency's block chain contains every transaction ever executed in the currency. With this information, one can find out how much value belonged to each [address](https://en.bitcoin.it/wiki/Address) at any point in history.
- Every block contains a hash of the previous block. This has the effect of creating a chain of blocks from the [genesis block](https://en.bitcoin.it/wiki/Genesis_block) to the current block. Each block is guaranteed to come after the previous block chronologically because the previous block's hash would otherwise not be known. Each block is also computationally impractical to modify once it has been in the chain for a while because every block after it would also have to be regenerated. These properties are what make bitcoins transactions irreversible. The blockchain is the main innovation of Bitcoin.
- A blockchain is a very simplistic database that is a append-only immutable store that can be written to by a collection of agents who sign their transactions and engage in confirming other agents transactions through a distributed consensus protocol based on cryptographic hashing.
  - The blockchain is a specialized use case rather than a complete divergence from traditional database technology.
  - [Do you need a blockchain?](https://www.adjoint.io/posts/database_blockchain.html) (Adjoint Blog)
- A blockchain is an immutable distributed ledger of verified transactions. In basic terms, it is a distributed database of verified transactions that is replicated along a network of participants, called nodes, making it secure, efficient and transparent.
  - Transactions, in order to be added to a blockchain, will be filled into a block. The block will contain all the transactions' details along with the unique signatures of the parties involved and a link to the transactions already in the blockchain (linking all the previous and current blocks together - making the database immutable).
  - Before being added to the chain, the block needs to be validated through a cryptographic signing of transactions and a consensus protocol often built on hashing of these blocks. Once validated, this block will be added to the other blocks and replicated along the nodes of the distributed database.
- A blockchain is only one among different types of distributed ledgers.
- A blockchain is a magic computer that anyone can upload programs to and leave the programs to self-execute, where the current and all previous states of every program are always publicly visible, and which carries a very strong cryptoeconomically secured guarantee that programs running on the chain will continue to execute in exactly the way that the blockchain protocol specifies.
  - the fact that it's "cryptoeconomic", a technical term roughly meaning: it's decentralized, it uses public key cryptography for authentication, and it uses economic incentives to ensure that it keeps going and doesn't go back in time or incur any other glitch
- **Consensus process** - the process for determining what blocks get added to the chain and what the current state is. Or how to create trust between parties with different incentives. Or how to reach an agreement in a distributed system
  - rules that dictate how the different agents in a distributed ledger authenticate and validate the transactions added to it in order to prevent different versions of the ledger from being created or previous transactions from being edited
  - Bitcoin uses [Proof-of-Work](https://en.bitcoin.it/wiki/Proof_of_work). But the landscape of consensus is evolving fast. Some of the currently used consensus are [Proof-of-Stake](https://en.bitcoin.it/wiki/Proof_of_Stake), [Proof-of-Authority](https://en.wikipedia.org/wiki/Proof-of-authority), [Raft Consensus](https://raft.github.io/) and [Federated Consensus](https://www.adjoint.io/docs/consensus.html#id1).
  - the consensus mechanism used by [Stellar](https://www.stellar.org/) is a decentralized version of the [Practical Byzantine Fault Tolerance (PBFT)](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance#cite_note-15) called [Stellar Consensus Protocol](https://www.stellar.org/developers/guides/concepts/scp.html), whereas [Ripple](https://ripple.com/) uses [probabilistic voting](https://www.quora.com/What-is-the-main-difference-between-Stellar-and-Ripple-protocols).
  - **Proof-of-Work** - a difficult solution to find but easy to verify.
    - In order to validate a block of transactions, several nodes of the distributed ledger (called [Miners](https://en.bitcoin.it/wiki/Mining)) compete to solve a complicated mathematical problem. The principle is that the solution of the problem is hard to find, but easy to verify by the rest of the network. Once the first Miner has found the solution, it provides the other nodes with the solution. The solution is then verified and the consensus is reached.
    - Time consuming
    - Energy consuming
    - The [51% attack](https://www.investopedia.com/terms/1/51-attack.asp): since it is time and energy consuming to validate the transactions, some Miners have decided to create Mining pools (pools where Miners share their resources) to get more efficient and generate cost savings.
  - **Proof-of-Stake** - the higher the stake of a validating node in the network, the more chances and legitimacy it has to validate transactions.
    - Validators of transactions are called *Minters*. Each Minter will commit some of the stake it owns in the network (to simplify, they "deposit" some cryptocurrency as a sort of collateral). Then, an algorithm will select one of the Minters based on the stake they own. In basic terms, if a node has a 5% stake, it will validate 5% of the transactions. It is then easy for the other nodes to verify that transactions have been validated by Minters that effectively have corresponding stakes in the distributed ledger.
  - **Raft** - validation through randomly won elections
    - based on a randomly won election that will designate a *Leader* in charge of the validation of the transactions.
  - **Federated** - blocks are validated if signed by a specified quorum of signers
    - define a "M-of-N [multisignature](https://en.bitcoin.it/wiki/Multisignature)" rule, with M representing the number of signatures needed for a transaction to be valid and N the number of validators (signers). Then, as long as less than 2M - N - 1 block, signers don't violate the protocol, the consensus is considered as secure
  - Most of these consensus are built on hashing the transactions
- See also <https://anders.com/blockchain/>
- **Transactions**
  - A transaction is a transfer of Bitcoin value that is broadcast to the network and collected into blocks. A transaction typically references previous transaction outputs as new transaction inputs and dedicates all input Bitcoin values to new outputs. Transactions *may not* be encrypted, so it is possible to browse and view every transaction ever collected into a block. Once transactions are buried under enough confirmations they can be considered irreversible.
  - Standard transaction outputs nominate addresses, and the redemption of any future inputs requires a relevant signature.
  - All transactions are visible in the blockchain, and can be viewed with a hex editor
  - **Address**
    - A unique [Base58](https://en.wikipedia.org/wiki/Base58) encoded value representing a public key of a node in the network
    - Maps a hash of binary encoded [ECDSA](#elliptic-curve-cryptography) public key to a unique, irreversible identity that uniquely defines a participant in the network
  - A data structure repesentinfg a single atom modification to the ledger with some extra data to prove validity
  - An atomic ledger state modification
  - In order to be signed by the issuer they must first be converted to a string of bytes ([serialization](https://en.wikipedia.org/wiki/Serialization))
  - **Header**
    - Denotes the type of transaction
    - Sender key *(public key of the issuer)*
    - Recipient *(the address to transfer)*
    - Amount *(the amount to transfer)*
    - Signature *(the issuers serialized [ECC](#elliptic-curve-cryptography) signature of the header)*
- **Blocks**
  - A list of ordered transactions with some extra data to prove validity
  - Simply ordered lists of transactions, plus a few other things to help us validate their integrity
  - Index *(block height)*
  - Header
    - Origin *(public key of block miner)*
    - Previous hash *(previous block hash)*
    - Markle root *(markle tree root representing all bock transactions)*
    - [none](https://en.bitcoin.it/wiki/Nonce) *(nonce for proof of work)*
    - Transactions *(list of TXs)*
    - Signature *(block signature)*
  - [Hashing](#hash-function)
  - **Validation**
    - *current (ledger) state + previous block + current block*
- **Distributed Ledger**
  - Stateful in-memory representation of all Transactions happened
  - A mapping of Addresses to [Balances](https://en.bitcoin.it/wiki/Address#Address_balances)
  - The datatype that reflects the culmination of Blocks
  - Ordered list of Blocks
    - Blocks encapsulate an order lists of transactions
    - Transactions are atomic stateful ledger updates
  - **The state resulting from the ordered, sequential application of all Transactions in all Blocks**
- **Common operations**
  - Lookup Balance of an Address
  - Add Balance
  - Add Address
  - Transfer
- **Distributed system**
  - Network of computers that run instances of the same program
  - Each instance interacts with one another through a *messaging protocol*
  - Networking
    - Bitcoin uses a simple broadcast network to propagate Transactions and Blocks. All communications are done over TCP
    - See **Bitcoin network** <https://en.wikipedia.org/wiki/Bitcoin_network>

## Credits, References

- [What is blockchain and why do you need it?](https://adjoint.io/posts/database_blockchain.html) -- Adjoint Blog
- [Distributed ledger technology - A disrupting technology of trust](https://adjoint.io/posts/disrupting-technology-of-trust.html) -- Adjoint Blog
- [Ethereum: Now Going Public](https://blog.ethereum.org/2014/01/23/ethereum-now-going-public/) -- Vitalik Buterin
- [What is Ethereum? Project, Platform, Fuel, Stack](https://blog.ethereum.org/2014/05/14/what-is-ethereum-project-platform-fuel-stack/) -- Joseph Lubin
- [How ethereum could shard the web](https://blog.ethereum.org/2014/08/18/building-decentralized-web/) -- Taylor Gerring
- [Merkling in Ethereum](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/) -- Vitalik Buterin
- [What is a DApp?](https://ethereum.stackexchange.com/a/384) -- Ethereum Stack Exchange
- [What is Swarm and what is it used for?](https://ethereum.stackexchange.com/a/388) -- Ethereum Stack Exchange
- [Why Haskellers should be interested in Smart Contracts](http://stephendiehl.com/posts/smart_contracts.html) -- Stephen Diehl
- [Introduction to Cryptocurrencies in Haskell](https://youtube.com/watch?v=wjyiOXRuUdo) -- Thomas Dietert
