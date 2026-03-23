# Why Verifiable Data Matters: A Researcher's Perspective on Decentralized Storage

*By Eason Chen — PhD Student & Blockchain Researcher at Carnegie Mellon University*

---

## 1. Introduction

In the age of large language models, deepfakes, and synthetic media, a deceptively simple question has become one of the most important in computer science: **Can we trust the data we consume?**

For decades, the answer has relied on institutional trust — we trust AWS not to tamper with our S3 objects, we trust Google not to alter our Drive files, and we trust academic publishers to preserve the integrity of scholarly records. But this trust model is fragile. A single compromised administrator, a rogue insider, or a state-level censorship order can silently alter or erase data that millions depend on.

Verifiable data — data whose integrity, provenance, and availability can be independently confirmed through cryptographic proofs rather than institutional reputation — represents a paradigm shift. This post surveys the academic landscape of verifiable decentralized storage, examines how systems like Walrus Protocol address long-standing open problems, and reflects on why this matters from both a research and practitioner perspective.

---

## 2. The Problem Space: Data Integrity in Distributed Systems

### 2.1 Classical Approaches and Their Limitations

The problem of ensuring data integrity in distributed systems is not new. Byzantine Fault Tolerance (BFT), formalized by Lamport et al. (1982), established that distributed systems can tolerate up to *f* faulty nodes out of *3f + 1* total nodes. However, classical BFT protocols like PBFT (Castro & Liskov, 1999) were designed for state machine replication — they ensure consensus on computation, not efficient storage of large binary objects (blobs).

When blockchains emerged as practical BFT systems, they inherited this limitation. Ethereum, for example, requires every validator to replicate every piece of state data, resulting in replication factors of 100–1000x depending on the validator set size (Danezis et al., 2025). This is acceptable for transaction records measured in bytes, but entirely impractical for storing images, videos, AI training datasets, or scientific data measured in gigabytes.

### 2.2 The Storage-Verification Gap

This creates what I call the **storage-verification gap**: blockchains can verify *that* something happened (a transaction, a state transition), but they cannot efficiently verify *what* was stored off-chain. NFT metadata points to IPFS hashes that may resolve to nothing. DeFi protocols reference price feeds from centralized oracles. AI models claim to be trained on specific datasets with no way to verify the claim.

Bridging this gap — enabling verifiable storage at scale — is the core challenge that decentralized storage networks attempt to solve.

---

## 3. A Survey of Decentralized Storage Approaches

The academic literature identifies two broad categories of decentralized storage systems, each with distinct trade-offs (Benisi et al., 2020):

### 3.1 Full Replication Systems

**Filecoin** (Protocol Labs, 2017) and **Arweave** (Williams et al., 2019) rely on full data replication across storage nodes. Their key innovation is cryptographic proof mechanisms:

- **Proof-of-Replication (PoRep)**: Proves that a storage provider has created a unique physical copy of the data, preventing Sybil attacks where a single node pretends to store multiple copies (Fisch, 2019).
- **Proof-of-Spacetime (PoSt)**: Proves continuous storage over time, ensuring nodes don't discard data after the initial proof.

The advantage is simplicity: any single honest node holds the complete file, so recovery from node failure is trivial. The disadvantage is cost. Achieving "twelve nines" of security (probability < 10⁻¹² of data loss) under a 1/3 adversary model requires at least **25 full copies** stored across the network — a 25x storage overhead (Danezis et al., 2025).

### 3.2 Erasure Coding Systems

**Storj** and **Sia** pioneered the use of Reed-Solomon (RS) erasure coding (Reed & Solomon, 1960) in decentralized storage. RS coding splits data into *k* fragments and generates *n - k* parity fragments such that any *k* of *n* fragments can reconstruct the original data. This dramatically reduces the replication factor — from 25x to approximately **3x** for equivalent security guarantees.

However, a critical weakness emerges during **node churn** — when storage nodes go offline and must be replaced. In classical RS-coded systems, recovering a single lost fragment requires downloading the entire original file's worth of data from all remaining nodes, transmitting O(|blob|) data across the network. Under frequent churn (common in permissionless networks), these recovery costs erode the storage savings (Dimakis et al., 2010).

### 3.3 Comparative Analysis

| Property | Full Replication | Classic Erasure Coding | Walrus (Red Stuff) |
|---|---|---|---|
| Replication Factor (10⁻¹² security) | 25x | 3x | **4.5x** |
| Single Shard Recovery Cost | O(\|blob\|) | O(\|blob\|) | **O(\|blob\|/n)** |
| Asynchronous Challenges | ✗ | ✗ | **✓** |
| Self-Healing | ✗ | ✗ | **✓** |

*Table adapted from Danezis et al. (2025), "Walrus: An Efficient Decentralized Storage Network."*

---

## 4. Walrus Protocol: Technical Innovations

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

## 5. Why This Matters: Use Cases Through an Academic Lens

### 5.1 Reproducible Research and Data Provenance

As a researcher working on AI and education technology, I see the crisis of reproducibility firsthand. A 2016 Nature survey found that over 70% of researchers have tried and failed to reproduce another scientist's experiments (Baker, 2016). A key contributor is **data provenance** — training datasets get modified, preprocessing pipelines change, and there is no verifiable record of the exact data used in a published result.

Verifiable decentralized storage offers a solution: researchers can store datasets on Walrus with on-chain proofs attesting to the exact content at the time of publication. Future researchers can verify that the dataset they downloaded is bit-for-bit identical to what was used in the original study. This goes beyond simple hash verification (which IPFS provides) — Walrus's storage challenges ensure the data *remains available*, not just that it *once existed*.

### 5.2 AI Training Data Integrity

The rise of data poisoning attacks (Biggio et al., 2012) and model supply chain attacks makes verifiable data storage increasingly critical for AI safety. If a model claims to be trained on a specific dataset, verifiable storage can provide an immutable audit trail:

- The training data is stored with cryptographic commitments
- Any modification to the dataset after storage is detectable
- The data remains retrievable for independent verification

This is especially relevant as regulatory frameworks like the EU AI Act begin requiring transparency about training data.

### 5.3 Decentralized Application Integrity

The irony of "Web3" is that most decentralized applications are served from centralized web hosts. A DNS hijack or a compromised CDN can serve malicious frontend code to users, who then interact with smart contracts under false pretenses. Walrus enables serving dApp frontends directly from decentralized storage, ensuring that the code users execute matches what developers published — with cryptographic, not institutional, guarantees.

### 5.4 Data Availability for Rollups

Ethereum's scaling roadmap relies heavily on rollups, which post compressed transaction data to a data availability layer. The security of a rollup depends entirely on the guarantee that this data can be retrieved when needed for fraud proofs or state reconstruction. Walrus's properties — verifiable storage, efficient retrieval, and guaranteed availability — make it a natural fit for this critical infrastructure role.

---

## 6. From Theory to Practice: A Builder's Reflection

I study this space academically, but I also build on it. My team at 3MateLabs developed the **Walrus Sponsor SDK** — an open-source toolkit that lets dApp developers offer gasless, sponsored storage to their users. Think of it as "storage account abstraction": end users upload files without needing WAL tokens or understanding the underlying protocol.

Building on Walrus taught me something that papers alone cannot: **the developer experience of verifiable infrastructure matters as much as its cryptographic properties.** Walrus's ~800ms retrieval times, HTTP-compatible APIs, and Sui-native integration make it practical to build real applications. The gap between "theoretically secure" and "actually usable" is where many decentralized systems fail — Walrus bridges it.

---

## 7. Open Questions and Future Directions

Despite these advances, several open research questions remain:

1. **Scalability to petabyte-scale**: Current evaluations demonstrate practical performance, but the behavior of 2D erasure coding under extreme scale warrants further study.
2. **Cross-chain data availability**: How can Walrus's verifiable storage guarantees be extended to ecosystems beyond Sui?
3. **Privacy-preserving verifiability**: Can we achieve verifiable storage while maintaining data confidentiality? The integration with encryption and key management systems (noted in the Walrus whitepaper) is a promising direction.
4. **Economic sustainability**: The long-term dynamics of WAL token incentives for storage nodes under varying network conditions merit formal game-theoretic analysis.
5. **Formal verification**: Applying formal methods to verify the correctness of Red Stuff's security properties under the asynchronous model would strengthen confidence in the protocol.

---

## 8. Conclusion

Verifiable data is not a blockchain buzzword — it is a fundamental requirement for trustworthy digital infrastructure. From reproducible research to AI safety to decentralized finance, the ability to independently verify that data is authentic, unmodified, and available underpins the integrity of increasingly critical systems.

Walrus Protocol represents a meaningful advance in this space. Its Red Stuff encoding achieves a novel balance between storage efficiency (4.5x replication), recovery efficiency (O(|blob|/n) per shard), and security guarantees (asynchronous challenge support) — addressing limitations that have constrained decentralized storage systems since their inception.

As both a researcher and a builder in this space, I believe the convergence of rigorous cryptographic protocols and practical, developer-friendly infrastructure is what will ultimately make verifiable data the default rather than the exception.

The question is no longer *whether* we need verifiable data. It is *how fast* we can make it the standard.

---

### References

- Baker, M. (2016). 1,500 scientists lift the lid on reproducibility. *Nature*, 533(7604), 452–454.
- Benisi, N. Z., Aminian, M., & Javadi, B. (2020). Blockchain-based decentralized storage networks: A survey. *Journal of Network and Computer Applications*, 162, 102656.
- Biggio, B., Nelson, B., & Laskov, P. (2012). Poisoning attacks against support vector machines. *ICML 2012*.
- Castro, M., & Liskov, B. (1999). Practical Byzantine fault tolerance. *OSDI '99*.
- Danezis, G., Giuliari, G., Kokoris-Kogias, L., Legner, M., Smith, J.-P., Sonnino, A., & Wüst, K. (2025). Walrus: An efficient decentralized storage network. *arXiv:2505.05370v2*.
- Dimakis, A. G., Godfrey, P. B., Wu, Y., Wainwright, M. J., & Ramchandran, K. (2010). Network coding for distributed storage systems. *IEEE Transactions on Information Theory*, 56(9), 4539–4551.
- Fisch, B. (2019). Proof of replication. *Eurocrypt 2019*.
- Lamport, L., Shostak, R., & Pease, M. (1982). The Byzantine generals problem. *ACM TOPLAS*, 4(3), 382–401.
- Reed, I. S., & Solomon, G. (1960). Polynomial codes over certain finite fields. *Journal of the Society for Industrial and Applied Mathematics*, 8(2), 300–304.

---

*This post is my submission for the Walrus Protocol Blog Post Challenge. I am a PhD student at Carnegie Mellon University researching AI for education, and a contributor to the Walrus ecosystem through 3MateLabs. The views expressed are my own.*

*Follow @WalrusProtocol for updates on decentralized storage. Follow me @EasonC13 for more at the intersection of research and Web3.*
