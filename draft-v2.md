# Why Verifiable Data Matters: A Researcher's Perspective on Decentralized Storage

*By Eason Chen — PhD Student & Blockchain Researcher at Carnegie Mellon University*

---

## 1. Introduction

In the age of large language models, deepfakes, and synthetic media, a deceptively simple question has become one of the most important in computer science: **Can we trust the data we consume?**

For decades, the answer has relied on institutional trust — we trust AWS not to tamper with our S3 objects, we trust Google not to alter our Drive files, and we trust academic publishers to preserve the integrity of scholarly records. But this trust model is fragile. A single compromised administrator, a rogue insider, or a state-level censorship order can silently alter or erase data that millions depend on.

Verifiable data — data whose integrity, provenance, and availability can be independently confirmed through cryptographic proofs rather than institutional reputation — represents a paradigm shift. This is not merely a blockchain concern; it touches every domain where data authenticity matters, from hospital records to journalism to the food on your table.

This post surveys the academic landscape of verifiable decentralized storage, compares the major existing solutions, examines how Walrus Protocol addresses long-standing open problems, and argues that verifiable data is not just a Web3 feature — it is foundational infrastructure for a trustworthy digital society.

---

## 2. The Problem Space: Data Integrity in Distributed Systems

### 2.1 Classical Approaches and Their Limitations

The problem of ensuring data integrity in distributed systems is not new. Byzantine Fault Tolerance (BFT), formalized by Lamport et al. (1982), established that distributed systems can tolerate up to *f* faulty nodes out of *3f + 1* total nodes. However, classical BFT protocols like PBFT (Castro & Liskov, 1999) were designed for state machine replication — they ensure consensus on computation, not efficient storage of large binary objects (blobs).

When blockchains emerged as practical BFT systems, they inherited this limitation. Ethereum, for example, requires every validator to replicate every piece of state data, resulting in replication factors of 100–1000x depending on the validator set size (Danezis et al., 2025). This is acceptable for transaction records measured in bytes, but entirely impractical for storing images, videos, AI training datasets, or scientific data measured in gigabytes.

### 2.2 The Storage-Verification Gap

This creates what I call the **storage-verification gap**: blockchains can verify *that* something happened (a transaction, a state transition), but they cannot efficiently verify *what* was stored off-chain. NFT metadata points to IPFS hashes that may resolve to nothing. DeFi protocols reference price feeds from centralized oracles. AI models claim to be trained on specific datasets with no way to verify the claim.

Bridging this gap — enabling verifiable storage at scale — is the core challenge that decentralized storage networks attempt to solve.

### 2.3 Beyond Blockchain: The Universal Need for Verifiable Data

Critically, the storage-verification gap is not unique to crypto-native applications. Consider:

- **Healthcare**: Electronic health records (EHRs) are routinely modified — sometimes legitimately (corrections), sometimes not (billing fraud, liability cover-ups). The U.S. Department of Health and Human Services reported over 700 major healthcare data breaches in 2023 alone. Patients currently have no way to independently verify that their records are complete and unaltered.
- **Journalism**: In an era of deepfakes, news organizations struggle to prove the authenticity of documentary evidence. The New York Times's "Content Authenticity Initiative" and similar efforts demonstrate the industry-wide recognition that content provenance is a crisis, not a feature request.
- **Legal discovery**: Court proceedings depend on establishing that digital evidence has not been tampered with. Current chain-of-custody procedures are manual, error-prone, and expensive.
- **Supply chains**: The World Health Organization estimates that 1 in 10 medical products in low- and middle-income countries is substandard or falsified. Verifiable data trails from manufacturing to delivery could directly save lives.

These are not hypothetical use cases — they are active, urgent problems that existing centralized systems fail to solve precisely because they rely on the same institutions whose trustworthiness is in question.

---

## 3. A Comparative Survey of Decentralized Storage Solutions

The landscape of decentralized storage has matured significantly since IPFS's debut in 2015. This section provides a detailed comparison of the major systems, organized by their architectural approach to data encoding, verification, and incentive design.

### 3.1 IPFS: Content-Addressable Networking (2015)

**Architecture**: IPFS (InterPlanetary File System) is a peer-to-peer content-addressed storage network. Files are identified by their cryptographic hash (CID), meaning any node hosting the same file will produce the same identifier. IPFS uses a Distributed Hash Table (DHT) based on Kademlia for content routing.

**Verification model**: IPFS provides **content integrity** — you can verify that the file you received matches its hash. However, it provides **no availability guarantees**. If no node is actively hosting (pinning) a file, the CID resolves to nothing. There is no economic incentive for nodes to persist data.

**Strengths**: Elegant content-addressing model; widely adopted; strong censorship resistance for popular content.

**Limitations**: No native incentive layer; no guarantee of data persistence; garbage collection deletes unpinned content; relies on altruistic pinning or third-party services (Pinata, Infura) that reintroduce centralization.

**Key insight**: IPFS solves *addressing* and *integrity*, but not *availability* or *verifiability over time*. It answers "Is this the right file?" but not "Will this file still be here tomorrow?"

### 3.2 Filecoin: Incentivized Full Replication (2017)

**Architecture**: Built by Protocol Labs as an incentive layer for IPFS, Filecoin uses full data replication with a dedicated proof system. Storage providers seal sectors (32 GiB or 64 GiB blocks) and submit proofs to the Filecoin blockchain.

**Verification model**: Two-layer proof system:
- **Proof-of-Replication (PoRep)**: A computationally expensive sealing process proves that the storage provider has created a unique, dedicated copy of the data. This prevents Sybil attacks — a provider cannot pretend to store multiple copies by pointing to the same physical data (Fisch, 2019).
- **Proof-of-Spacetime (PoSt)**: Periodic challenges prove that the provider *continues* to store the data over time, not just at the moment of the initial deal.

**Strengths**: Largest decentralized storage network by capacity (>20 EiB committed); robust economic incentive model; strong theoretical security guarantees.

**Limitations**: 
- High onboarding friction: sealing a 32 GiB sector requires significant hardware (128 GiB RAM, high-end GPU), creating barriers to storage provider entry.
- Retrieval is not incentivized at the protocol level — data can be stored but slow to retrieve.
- Full replication means 25x overhead for high security guarantees.
- Deal-making process is complex for end users.

### 3.3 Arweave: Permanent Storage via Endowment (2018)

**Architecture**: Arweave takes a radically different approach — **pay once, store forever**. Users pay an upfront fee that includes a "storage endowment" calculated to cover the declining cost of storage over 200+ years. Data is stored in a blockweave structure (a blockchain where each block links not only to its predecessor but also to a random previous block).

**Verification model**: 
- **Succinct Proofs of Random Access (SPoRA)**: Miners must prove they have access to random chunks from the weave's history, incentivizing full data replication across the network.
- Content addressing via transaction IDs on the permaweb.

**Strengths**: Compelling "permanent storage" narrative; simple user experience (pay once); strong for archival use cases; active ecosystem (Arweave Name System, ArDrive).

**Limitations**:
- Permanent storage guarantee is economic, not cryptographic — it depends on the storage endowment assumptions holding for centuries.
- No formal mechanism to handle the scenario where declining storage costs don't outpace endowment depletion.
- High per-byte cost compared to alternatives (reflecting the "infinity duration" premium).
- Full replication model with the same 25x overhead constraints.
- Read performance depends on gateway infrastructure that can be centralized.

### 3.4 Storj: Erasure-Coded Distributed Storage (2018)

**Architecture**: Storj distributes data across a global network of storage node operators using Reed-Solomon erasure coding. A typical configuration splits files into 80 pieces, of which any 29 can reconstruct the original — a 2.7x expansion factor.

**Verification model**: Satellite nodes (trusted coordinators) perform periodic audits by requesting specific segments from storage nodes and verifying against pre-computed hashes.

**Strengths**: Competitive pricing with centralized cloud (often cheaper than S3); good retrieval performance; S3-compatible API; low hardware requirements for node operators.

**Limitations**:
- Satellite architecture introduces centralization — Storj Labs runs the primary satellites, and they mediate all storage deals and audits.
- Audit model does not support asynchronous challenges; assumes synchronous network for verification.
- No on-chain proof mechanism; verification is satellite-dependent.
- Not suitable for applications requiring fully trustless verification.

### 3.5 Sia: Smart Contract-Based Storage (2014)

**Architecture**: One of the earliest decentralized storage projects, Sia uses file contracts (smart contracts on its own blockchain) to establish agreements between renters and hosts. Data is erasure-coded using Reed-Solomon coding.

**Verification model**: Storage proofs are submitted on-chain at regular intervals. Hosts that fail to submit proofs lose their collateral. 

**Strengths**: Fully on-chain contract enforcement; transparent pricing; open-source ecosystem (Skynet/Lume for web hosting).

**Limitations**:
- Small network of hosts limits geographic distribution and redundancy.
- Relatively slow proof submission and verification cycle.
- User experience requires managing contracts, collateral, and renewals.
- Reed-Solomon recovery still requires O(|blob|) bandwidth per lost shard.

### 3.6 Comparative Analysis

| Property | IPFS | Filecoin | Arweave | Storj | Sia | **Walrus** |
|---|---|---|---|---|---|---|
| **Encoding** | Replication (manual pinning) | Full replication (sealed sectors) | Full replication (blockweave) | RS erasure coding (29-of-80) | RS erasure coding | **Red Stuff 2D erasure coding** |
| **Replication Factor** (10⁻¹² security) | N/A (no guarantee) | ~25x | ~25x | ~2.7x | ~3x | **4.5x** |
| **Availability Guarantee** | None (best-effort) | Contract-based (time-limited deals) | Permanent (endowment model) | Satellite-enforced | Contract-based | **On-chain proof + storage challenges** |
| **Recovery Cost** (single shard) | N/A | O(\|blob\|) copy | O(\|blob\|) copy | O(\|blob\|) | O(\|blob\|) | **O(\|blob\|/n)** |
| **Async Challenge Support** | ✗ | ✗ | ✗ | ✗ | ✗ | **✓** |
| **Self-Healing** | ✗ | ✗ | ✗ | ✗ | ✗ | **✓** |
| **On-chain Verification** | ✗ | ✓ (Filecoin chain) | ✓ (Arweave chain) | ✗ (satellite) | ✓ (Sia chain) | **✓ (Sui)** |
| **Retrieval Speed** | Variable | Slow (unsealing) | Gateway-dependent | Fast (~S3-level) | Moderate | **~800ms (small files)** |
| **Smart Contract Integration** | ✗ | Limited | SmartWeave (lazy eval) | ✗ | File contracts | **Native Sui Move** |

*Table synthesized from Benisi et al. (2020), Danezis et al. (2025), and system documentation.*

### 3.7 Key Observations

Several patterns emerge from this comparison:

1. **The replication-coding spectrum**: Systems face a fundamental trade-off. Full replication (Filecoin, Arweave) is simple to reason about but expensive. Erasure coding (Storj, Sia) is efficient but introduces recovery complexity. Walrus's Red Stuff occupies a novel position — slightly higher overhead than minimal erasure coding (4.5x vs. 3x) but dramatically better recovery properties.

2. **The verification centralization trap**: Many systems that claim decentralization rely on centralized components for verification. Storj's satellites, IPFS's pinning services, and Arweave's gateways all introduce trust assumptions that undermine the verifiability promise. Systems with on-chain verification (Filecoin, Sia, Walrus) avoid this, though at the cost of blockchain dependency.

3. **Synchrony assumptions matter**: Every system prior to Walrus assumes a synchronous network for storage challenges — meaning an adversary cannot exploit network delays to pass verification without actually storing data. In real-world permissionless networks with variable latency, this is a significant assumption. Walrus's asynchronous challenge support (via Red Stuff's two-dimensional encoding) is a genuine first.

4. **Smart contract integration unlocks programmability**: Walrus's native integration with Sui's Move-based smart contracts enables "programmable storage" — access control, payment logic, and data lifecycle management can be expressed as on-chain programs. This is qualitatively different from systems where storage and computation are loosely coupled.

---

## 4. Walrus Protocol: Technical Deep Dive

### 4.1 Red Stuff: Two-Dimensional Erasure Coding

The central contribution of Walrus is **Red Stuff**, a novel two-dimensional erasure coding protocol (Danezis et al., 2025). Unlike traditional one-dimensional RS coding, Red Stuff encodes data along two dimensions with different thresholds:

- **Low-threshold dimension**: Enables nodes that missed data during the write phase to recover their assigned slivers from a small subset of peers — achieving **self-healing** with bandwidth proportional only to the lost data, i.e., O(|blob|/n) rather than O(|blob|).
- **High-threshold dimension**: Used during the read/challenge phase, preventing adversaries in asynchronous networks from passively collecting enough information from honest nodes to fake storage proofs.

This is a significant theoretical advancement. Prior to Red Stuff, **no erasure coding protocol supported storage challenges in asynchronous networks** — all existing solutions assumed synchronous communication, where an adversary cannot read data from honest nodes fast enough to respond to challenges. In real-world permissionless networks, this assumption rarely holds.

### 4.2 Epoch Change Protocol

A practical challenge for any erasure-coded storage system is handling committee transitions — when the set of storage nodes changes between epochs. Walrus introduces a multi-stage epoch change protocol that maintains uninterrupted read and write availability during transitions, avoiding the "race condition" where departing nodes must simultaneously accept new writes and transfer existing data to replacement nodes.

### 4.3 Authenticated Data Structures

Walrus incorporates Merkle-tree-based authenticated data structures to defend against **malicious clients** — a threat model often overlooked in decentralized storage literature. A malicious client might upload inconsistent slivers to different storage nodes, creating data that can never be reconstructed. Walrus's authentication mechanism ensures that storage nodes can independently verify the consistency of the slivers they receive.

---

## 5. Beyond Blockchain: Verifiable Data in Everyday Life

The discussion of verifiable data often remains confined to the crypto-native bubble — NFTs, DeFi, DAOs. This is a mistake. The need for verifiable data extends far beyond blockchain applications into systems that affect billions of people daily. Here I examine five domains where verifiable data infrastructure could transform existing practice.

### 5.1 Healthcare: Patient-Controlled Medical Records

Today's electronic health record (EHR) systems — Epic, Cerner, Allscripts — store patient data in proprietary, centralized databases. Patients have limited ability to verify their own records' completeness or detect unauthorized modifications. The 21st Century Cures Act (2016) mandated interoperability, but in practice, records are still fragmented across providers, and audit logs are controlled by the same institutions being audited.

**How verifiable storage changes this**: Medical records stored on a verifiable data layer would allow patients to independently confirm:
- That their records have not been altered after a disputed diagnosis
- That all records from all providers are accounted for (no selective deletion)
- That access logs are tamper-proof (who viewed their data and when)

This is not about putting medical records "on the blockchain" — it is about using cryptographic proofs as an independent verification layer over existing health IT infrastructure. Walrus's combination of efficient blob storage (for large imaging files like MRIs and CT scans), on-chain proofs, and programmable access control (via Sui Move) makes this architecturally feasible.

### 5.2 Journalism and Content Provenance

The deepfake crisis has moved from hypothetical to operational. In 2024, AI-generated audio of a political figure was used in robocalls attempting to suppress voter turnout. News organizations face a credibility crisis: how do you prove that a photograph is authentic when AI can generate indistinguishable fakes?

**How verifiable storage changes this**: A journalist capturing a photo or video could immediately store the raw file on verifiable storage, creating a timestamped, tamper-evident record. The cryptographic proof serves as a "digital notary" — it cannot prove the content is *true*, but it can prove that *this specific file existed at this specific time and has not been altered since*.

The Coalition for Content Provenance and Authenticity (C2PA), backed by Adobe, Microsoft, BBC, and others, is building standards for exactly this kind of content provenance metadata. Verifiable decentralized storage is the natural backend for these standards — it removes the need to trust any single organization to maintain the provenance records.

### 5.3 Supply Chain Verification: From Farm to Table

Global supply chains are complex, opaque, and rife with fraud. The European Commission estimates that food fraud costs the EU food industry €30 billion annually. "Organic" labels, "fair trade" certifications, and "country of origin" claims are difficult to verify and easy to falsify.

**How verifiable storage changes this**: At each stage of a supply chain — harvesting, processing, shipping, retail — participants record data (timestamps, locations, certifications, sensor readings) to verifiable storage. Because the data is immutable and independently verifiable, no single participant can retroactively alter the trail. A consumer scanning a QR code on a product could trace its provenance through cryptographically verified records, not just self-reported claims.

This is already being piloted: Walmart's food traceability system (built on Hyperledger, a permissioned blockchain) reduced the time to trace a bag of mangoes from farm to store from 7 days to 2.2 seconds. Verifiable decentralized storage could bring similar capabilities without requiring participants to trust a single corporate coordinator.

### 5.4 Education: Verifiable Credentials and Academic Integrity

As a PhD student, I have a personal stake in this one. Academic credential fraud is a growing problem — the FBI has investigated diploma mills selling fake degrees, and the pandemic-era shift to online learning made cheating easier to commit and harder to detect.

**How verifiable storage changes this**: Universities could issue credentials (diplomas, transcripts, certifications) as verifiable documents stored on decentralized infrastructure. An employer verifying a candidate's degree would not need to contact the university — they would verify the cryptographic proof directly. The credential is valid even if the issuing institution ceases to exist.

More broadly, academic papers could store their associated datasets, code, and supplementary materials on verifiable storage at the time of publication, creating an immutable record of the research artifacts. This directly addresses the reproducibility crisis — a topic I discussed in my own work on comparing RAG and GraphRAG for educational retrieval systems.

### 5.5 Government Records and Democratic Accountability

Public records — property deeds, business registrations, legislative histories, court decisions — are supposed to be public goods. In practice, they are often difficult to access, inconsistently maintained, and vulnerable to tampering by those in power. Historical examples of retroactive modification of government records are disturbingly common.

**How verifiable storage changes this**: Government records stored on verifiable infrastructure create a tamper-evident audit trail that no single administration can alter. This is not about distrust of any particular government — it is about building systems where trust is *verifiable rather than assumed*. The transparency benefit extends to any regime: democratic governments can point to verifiable records as evidence of accountability; citizens can audit records independently.

### 5.6 The Common Thread

Across all these domains, the pattern is the same: **existing systems require you to trust the same institution that controls the data.** Hospitals audit their own records. News organizations vouch for their own authenticity. Governments certify their own documents. Verifiable data infrastructure breaks this circular dependency by introducing an independent, cryptographic layer of verification.

This is not a radical proposition. It is the logical extension of principles we already accept — financial auditors are independent from the companies they audit; election observers are independent from the governments they monitor. Verifiable data simply extends this principle to the digital realm.

---

## 6. From Theory to Practice: A Builder's Reflection

I study this space academically, but I also build on it. My team at 3MateLabs developed the **Walrus Sponsor SDK** — an open-source toolkit that lets dApp developers offer gasless, sponsored storage to their users. Think of it as "storage account abstraction": end users upload files without needing WAL tokens or understanding the underlying protocol.

Building on Walrus taught me something that papers alone cannot: **the developer experience of verifiable infrastructure matters as much as its cryptographic properties.** Walrus's ~800ms retrieval times, HTTP-compatible APIs, and Sui-native integration make it practical to build real applications. The gap between "theoretically secure" and "actually usable" is where many decentralized systems fail — Walrus bridges it.

What struck me most during development was the contrast with building on Filecoin and IPFS. With IPFS, you get content addressing but no availability guarantee — you are constantly worrying about pinning services and gateway reliability. With Filecoin, the sealing process introduces hours of latency between storing data and it being retrievable. With Walrus, the store-and-verify cycle is fast enough to build interactive applications, not just archival systems.

---

## 7. Open Questions and Future Directions

Despite these advances, several open research questions remain:

1. **Scalability to petabyte-scale**: Current evaluations demonstrate practical performance, but the behavior of 2D erasure coding under extreme scale warrants further study.
2. **Cross-chain data availability**: How can Walrus's verifiable storage guarantees be extended to ecosystems beyond Sui? Bridges and light client verification of Sui proofs on other chains could unlock multi-chain verifiable storage.
3. **Privacy-preserving verifiability**: Can we achieve verifiable storage while maintaining data confidentiality? The integration with encryption and key management systems (noted in the Walrus whitepaper) is a promising direction — storing encrypted blobs while maintaining verifiable availability.
4. **Economic sustainability**: The long-term dynamics of WAL token incentives for storage nodes under varying network conditions merit formal game-theoretic analysis.
5. **Formal verification**: Applying formal methods to verify the correctness of Red Stuff's security properties under the asynchronous model would strengthen confidence in the protocol.
6. **Non-blockchain integration**: How can existing Web2 institutions (hospitals, news organizations, governments) integrate verifiable storage into their existing data pipelines without requiring end-users to understand blockchain? The answer likely involves middleware abstractions similar to our Sponsor SDK — hiding cryptographic complexity behind familiar APIs.

---

## 8. Conclusion

Verifiable data is not a blockchain buzzword — it is a fundamental requirement for trustworthy digital infrastructure. From reproducible research to AI safety, from hospital records to the food on your table, the ability to independently verify that data is authentic, unmodified, and available underpins the integrity of systems that billions of people depend on daily.

The existing landscape of decentralized storage solutions — IPFS, Filecoin, Arweave, Storj, Sia — each made important contributions, but each carries fundamental trade-offs that limit their applicability. IPFS provides addressing without availability. Filecoin provides security without speed. Arweave provides permanence without scalability. Storj provides performance without trustless verification.

Walrus Protocol represents a meaningful advance because it addresses these trade-offs simultaneously. Its Red Stuff encoding achieves a novel balance between storage efficiency (4.5x replication), recovery efficiency (O(|blob|/n) per shard), and security guarantees (asynchronous challenge support) — while delivering practical performance (~800ms retrieval) and native smart contract integration.

But perhaps the most important insight is not about any single protocol. It is that **verifiable data is infrastructure, not application.** Just as HTTPS became the default for web communication — not because users demanded it, but because it was made easy enough that there was no reason not to use it — verifiable data storage will become default infrastructure when the cost and complexity drop below the threshold of indifference.

We are not there yet. But with systems like Walrus making verifiable storage practical, performant, and programmable, we are closer than we have ever been.

The question is no longer *whether* we need verifiable data. It is *how fast* we can make it invisible — so deeply embedded in our infrastructure that trusting unverifiable data feels as reckless as connecting to an HTTP site feels today.

---

### References

- Baker, M. (2016). 1,500 scientists lift the lid on reproducibility. *Nature*, 533(7604), 452–454.
- Benet, J. (2014). IPFS - Content addressed, versioned, P2P file system. *arXiv:1407.3561*.
- Benisi, N. Z., Aminian, M., & Javadi, B. (2020). Blockchain-based decentralized storage networks: A survey. *Journal of Network and Computer Applications*, 162, 102656.
- Biggio, B., Nelson, B., & Laskov, P. (2012). Poisoning attacks against support vector machines. *ICML 2012*.
- Castro, M., & Liskov, B. (1999). Practical Byzantine fault tolerance. *OSDI '99*.
- Danezis, G., Giuliari, G., Kokoris-Kogias, L., Legner, M., Smith, J.-P., Sonnino, A., & Wüst, K. (2025). Walrus: An efficient decentralized storage network. *arXiv:2505.05370v2*.
- Dimakis, A. G., Godfrey, P. B., Wu, Y., Wainwright, M. J., & Ramchandran, K. (2010). Network coding for distributed storage systems. *IEEE Transactions on Information Theory*, 56(9), 4539–4551.
- Fisch, B. (2019). Proof of replication. *Eurocrypt 2019*.
- Lamport, L., Shostak, R., & Pease, M. (1982). The Byzantine generals problem. *ACM TOPLAS*, 4(3), 382–401.
- Protocol Labs. (2017). Filecoin: A decentralized storage network. *Whitepaper*.
- Reed, I. S., & Solomon, G. (1960). Polynomial codes over certain finite fields. *Journal of the Society for Industrial and Applied Mathematics*, 8(2), 300–304.
- Williams, S. et al. (2019). Arweave: A protocol for economically sustainable information permanence. *Yellow Paper*.

---

*This post is my submission for the Walrus Protocol Blog Post Challenge. I am a PhD student at Carnegie Mellon University researching AI for education, and a contributor to the Walrus ecosystem through 3MateLabs. The views expressed are my own.*

*Follow @WalrusProtocol for updates on decentralized storage. Follow me @EasonC13 for more at the intersection of research and Web3.*
