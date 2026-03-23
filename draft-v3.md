# Why Verifiable Data Matters: A Researcher's Perspective

*By Eason Chen — PhD Student at Carnegie Mellon University*

---

## The Trust Problem

In the age of deepfakes, data poisoning, and AI-generated media, one question has become foundational: **Can we trust the data we consume?**

For decades, the answer relied on institutional trust — we trust AWS not to tamper with our files, hospitals not to alter medical records, and publishers to preserve scholarly integrity. But this trust model is fragile. A compromised administrator, a state censorship order, or a data breach can silently alter information that millions depend on.

Verifiable data — data whose integrity and availability can be independently confirmed through cryptographic proofs — offers a way out. This isn't just a blockchain concern. It touches healthcare, journalism, supply chains, and scientific research.

---

## The Storage-Verification Gap

Blockchains can verify *that* something happened (a transaction, a state change), but they cannot efficiently verify *what* was stored off-chain. NFT metadata points to IPFS hashes that may resolve to nothing. AI models claim specific training data with no way to verify the claim. I call this the **storage-verification gap** — and it's the core problem decentralized storage networks try to solve.

---

## Comparing Existing Solutions

The decentralized storage landscape has matured significantly. Here's how the major systems stack up:

**IPFS** provides content addressing — files identified by hash — but no availability guarantee. If nobody pins your file, it vanishes. It answers "Is this the right file?" but not "Will it be here tomorrow?"

**Filecoin** adds an incentive layer with Proof-of-Replication (PoRep) and Proof-of-Spacetime (PoSt), proving that storage providers maintain dedicated copies over time (Fisch, 2019). But full replication requires ~25x overhead for high security, sealing is hardware-intensive, and retrieval can be slow.

**Arweave** offers "pay once, store forever" via a storage endowment model. Compelling for archival, but the permanence guarantee is economic (betting on declining storage costs for 200+ years), not cryptographic. Also 25x replication overhead.

**Storj** uses Reed-Solomon erasure coding (29-of-80 shards, ~2.7x overhead) for efficiency, but its satellite-based audit model reintroduces centralization — verification depends on trusted coordinators rather than on-chain proofs.

**Sia** pioneered on-chain storage contracts with erasure coding, but its small host network limits redundancy, and recovering a lost shard still requires O(|blob|) bandwidth.

| Property | IPFS | Filecoin | Arweave | Storj | Sia | **Walrus** |
|---|---|---|---|---|---|---|
| Encoding | Replication | Full replication | Full replication | RS erasure coding | RS erasure coding | **2D erasure (Red Stuff)** |
| Replication overhead | N/A | ~25x | ~25x | ~2.7x | ~3x | **4.5x** |
| Shard recovery cost | N/A | O(\|blob\|) | O(\|blob\|) | O(\|blob\|) | O(\|blob\|) | **O(\|blob\|/n)** |
| Async challenges | ✗ | ✗ | ✗ | ✗ | ✗ | **✓** |
| On-chain verification | ✗ | ✓ | ✓ | ✗ | ✓ | **✓ (Sui)** |
| Retrieval speed | Variable | Slow | Gateway-dep. | Fast | Moderate | **~800ms** |

*Synthesized from Benisi et al. (2020) and Danezis et al. (2025).*

---

## Walrus: What's Different

Walrus introduces **Red Stuff**, a two-dimensional erasure coding protocol (Danezis et al., 2025) that solves three problems simultaneously:

1. **Efficient recovery**: Lost slivers are recovered with bandwidth proportional only to the lost data — O(|blob|/n) instead of O(|blob|). This makes the system **self-healing** without centralized coordination.

2. **Asynchronous challenges**: Prior systems assume synchronous networks for storage proofs — an adversary can't read data from honest nodes fast enough to fake proofs. Red Stuff's two-dimensional encoding is the **first protocol to support storage challenges in asynchronous networks**, preventing adversaries from exploiting real-world network delays.

3. **Malicious client defense**: Merkle-tree authentication ensures storage nodes can independently verify that the slivers they received are consistent — preventing a malicious uploader from creating unrecoverable data.

Combined with Sui's Move-based smart contracts, Walrus makes storage **programmable**: access control, payment logic, and data lifecycle management live on-chain.

---

## Beyond Blockchain: Why Everyone Should Care

Verifiable data isn't a crypto feature — it's infrastructure for a trustworthy digital society:

- **Healthcare**: Patients can't independently verify that their medical records are complete and unaltered. Verifiable storage creates tamper-evident audit trails over existing health IT.
- **Journalism**: With deepfakes operational in political contexts, content provenance is a crisis. Storing raw media on verifiable storage creates timestamped, tamper-evident records — a "digital notary" for authenticity. The C2PA standard (backed by Adobe, Microsoft, BBC) needs exactly this kind of backend.
- **Supply chains**: The WHO estimates 1 in 10 medical products in developing countries is falsified. Cryptographically verified data trails from manufacturing to delivery could save lives. Walmart already reduced food tracing from 7 days to 2.2 seconds using blockchain-based provenance.
- **Academic integrity**: As a researcher studying RAG systems for education, I see the reproducibility crisis firsthand — 70% of scientists have failed to reproduce others' results (Baker, 2016). Datasets stored on verifiable infrastructure at publication time create an immutable record that future researchers can independently verify.
- **Government records**: Property deeds, court decisions, and legislative histories stored on verifiable infrastructure create audit trails no single administration can alter.

The pattern is the same across all domains: **existing systems require you to trust the same institution that controls the data.** Verifiable data breaks this circular dependency.

---

## A Builder's Reflection

I don't just study this — I build on it. At 3MateLabs, we developed the **Walrus Sponsor SDK**, enabling gasless, sponsored storage for dApp users. Building on Walrus taught me that developer experience matters as much as cryptographic properties. Walrus's ~800ms retrieval, HTTP APIs, and Sui-native integration make it practical for interactive applications — unlike Filecoin's hours-long sealing or IPFS's unpredictable availability.

The gap between "theoretically secure" and "actually usable" is where most decentralized systems fail. Walrus bridges it.

---

## Conclusion

Verifiable data will become default infrastructure — just as HTTPS became default for web communication. Not because users demanded it, but because it was made easy enough that there was no reason not to use it.

With systems like Walrus making verifiable storage practical, performant, and programmable, we are closer than ever. The question isn't *whether* we need verifiable data — it's how fast we can make it invisible, so deeply embedded that trusting unverifiable data feels as reckless as HTTP does today.

---

### References

- Baker, M. (2016). 1,500 scientists lift the lid on reproducibility. *Nature*, 533(7604), 452–454.
- Benisi, N. Z. et al. (2020). Blockchain-based decentralized storage networks: A survey. *JNCA*, 162, 102656.
- Danezis, G. et al. (2025). Walrus: An efficient decentralized storage network. *arXiv:2505.05370v2*.
- Dimakis, A. G. et al. (2010). Network coding for distributed storage systems. *IEEE Trans. Info. Theory*, 56(9), 4539–4551.
- Fisch, B. (2019). Proof of replication. *Eurocrypt 2019*.
- Lamport, L. et al. (1982). The Byzantine generals problem. *ACM TOPLAS*, 4(3), 382–401.
- Reed, I. S. & Solomon, G. (1960). Polynomial codes over certain finite fields. *JSIAM*, 8(2), 300–304.

---

*Submission for the Walrus Protocol Blog Post Challenge. Follow @WalrusProtocol · @EasonC13*
