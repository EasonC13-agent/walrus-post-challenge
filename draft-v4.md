# Why Verifiable Data Matters: A Researcher's Perspective

*By Eason Chen — PhD Student at Carnegie Mellon University*

---

## The Trust Problem

In the age of deepfakes, data poisoning, and AI-generated media, one question has become foundational: **Can we trust the data we consume?**

For decades, the answer relied on institutional trust. We trust AWS not to tamper with our files, hospitals not to alter medical records, publishers to preserve scholarly integrity. This trust model works — until it doesn't. A compromised administrator, a state censorship order, or a quiet data breach can silently alter information that millions depend on, and no one would know.

Verifiable data offers a different answer. Instead of trusting institutions to *tell you* the data is intact, cryptographic proofs let you *check for yourself*. And this isn't just a blockchain concern — it touches healthcare, journalism, supply chains, and scientific research. Anywhere data matters and trust is assumed, verifiable data has something to say.

---

## The Storage-Verification Gap

Here's an irony of the blockchain era: blockchains are great at verifying *that* something happened — a transaction, a state change — but terrible at verifying *what* was stored off-chain.

Think about it. An NFT's metadata points to an IPFS hash. That hash might resolve to the artwork you paid for, or it might resolve to nothing at all. An AI company claims their model was trained on a curated, bias-free dataset — but there's no way for regulators or researchers to verify that claim after the fact.

I call this the **storage-verification gap**. Blockchains gave us trustless computation, but left trustless storage as an unsolved problem. Decentralized storage networks are the missing piece.

---

## How We Got Here: A Tour of Existing Solutions

The decentralized storage landscape has evolved through several generations, each solving part of the puzzle while leaving gaps for the next.

**IPFS** (2015) introduced a beautifully simple idea: identify files by their content hash, not their location. If you have the hash, you can verify the file is correct no matter where you got it. But IPFS makes no promise that anyone will keep hosting your file. It answers "Is this the right file?" but not "Will it still exist tomorrow?" In practice, most IPFS data lives on centralized pinning services like Pinata — which quietly reintroduces the trust problem IPFS was supposed to eliminate.

**Filecoin** (2017) tackled the incentive gap by paying storage providers to keep data. Its clever proof system — Proof-of-Replication (PoRep) ensures a provider actually stored a unique copy; Proof-of-Spacetime (PoSt) ensures they keep storing it (Fisch, 2019). The trade-off? Full replication means ~25x storage overhead for strong security. The sealing process is hardware-intensive and slow. And retrieval can take hours because data must be "unsealed" before it can be read. Great for cold archival; less great for anything interactive.

**Arweave** (2018) took the boldest bet: pay once, store forever. An economic endowment model estimates declining storage costs over 200+ years. It's a compelling narrative, and ideal for permanent archival. But "permanent" here is an economic prediction, not a cryptographic guarantee — it assumes storage costs will keep falling faster than the endowment depletes. And like Filecoin, it uses full replication with the same overhead constraints.

**Storj** (2018) went in a different direction entirely. Instead of replicating whole files, it uses Reed-Solomon erasure coding — splitting files into 80 pieces where any 29 can reconstruct the original. This slashes overhead to ~2.7x, and retrieval is fast (competitive with S3). The catch? Verification depends on centralized "satellite" nodes that coordinate audits. If you need *trustless, on-chain* verification, Storj can't provide it.

**Sia** (2014) was one of the earliest to put storage contracts on-chain with erasure coding. Conceptually sound, but a small host network limits practical redundancy, and recovering a lost shard still requires downloading the equivalent of the entire file — the fundamental bottleneck of one-dimensional erasure coding.

Here's how they compare:

| Property | IPFS | Filecoin | Arweave | Storj | Sia | **Walrus** |
|---|---|---|---|---|---|---|
| Encoding | Replication | Full replication | Full replication | RS erasure coding | RS erasure coding | **2D erasure (Red Stuff)** |
| Replication overhead | N/A | ~25x | ~25x | ~2.7x | ~3x | **4.5x** |
| Shard recovery cost | N/A | O(\|blob\|) | O(\|blob\|) | O(\|blob\|) | O(\|blob\|) | **O(\|blob\|/n)** |
| Async challenges | ✗ | ✗ | ✗ | ✗ | ✗ | **✓** |
| On-chain verification | ✗ | ✓ | ✓ | ✗ | ✓ | **✓ (Sui)** |
| Retrieval speed | Variable | Slow | Gateway-dep. | Fast | Moderate | **~800ms** |

*Synthesized from Benisi et al. (2020) and Danezis et al. (2025).*

A pattern emerges: every existing system makes a painful trade-off. You get efficiency *or* trustless verification *or* fast retrieval — pick two.

---

## Walrus: What's Different

Walrus tries to break this three-way trade-off with **Red Stuff**, a two-dimensional erasure coding protocol (Danezis et al., 2025). The "two-dimensional" part is key — it's not just an incremental improvement over Reed-Solomon, but a fundamentally different encoding geometry.

**Self-healing recovery.** In traditional erasure coding, when a storage node goes offline, replacing its shard requires downloading data from *all* remaining nodes — transferring O(|blob|) across the network. Red Stuff's 2D structure means a replacement node only needs data proportional to its own shard: O(|blob|/n). This makes the network self-healing — nodes recover lost data from their neighbors without centralized coordination, and without the bandwidth explosion that makes other systems fragile under churn.

**Asynchronous storage challenges.** Every other system assumes a synchronous network for storage proofs: the adversary can't read data from honest nodes fast enough to fake a challenge response. In the real world, permissionless networks have variable, unpredictable latency. Red Stuff's two encoding dimensions use different thresholds — one for recovery (low threshold, easy), one for challenge verification (high threshold, hard to fake). This makes Walrus the first erasure-coded system to support storage challenges without synchrony assumptions.

**Defense against malicious clients.** An underappreciated attack: a malicious uploader sends *inconsistent* slivers to different nodes, making the data impossible to reconstruct. Walrus uses Merkle-tree authentication so each node can independently verify its slivers are consistent with the whole — a defense most decentralized storage systems don't even consider.

And because Walrus runs on Sui, storage becomes **programmable**. Access control, payment logic, data lifecycle — all expressible as Move smart contracts. This isn't storage bolted onto a blockchain; it's storage woven into one.

---

## Beyond Blockchain: Why This Matters to Everyone

It's easy to dismiss verifiable data as a crypto-native concern. But the need extends far beyond NFTs and DeFi — into systems that affect billions of people who have never touched a wallet.

**Healthcare.** Patients today cannot independently verify that their medical records are complete and unaltered. When a disputed diagnosis ends up in court, both sides rely on records controlled by the hospital. Verifiable storage doesn't put medical records "on the blockchain" — it adds a tamper-evident cryptographic layer over existing health IT, giving patients an independent check.

**Journalism.** In 2024, AI-generated audio of a political figure was used in robocalls to suppress voter turnout. The deepfake crisis has moved from hypothetical to operational. Storing raw media on verifiable storage the moment it's captured creates a timestamped, tamper-evident record — not proof that content is *true*, but proof that *this specific file existed at this specific time and hasn't been altered since*. The C2PA standard, backed by Adobe, Microsoft, and the BBC, needs exactly this kind of backend infrastructure.

**Supply chains.** The WHO estimates 1 in 10 medical products in developing countries is substandard or falsified. Walmart demonstrated the power of blockchain-based provenance by reducing food tracing from 7 days to 2.2 seconds. Verifiable decentralized storage can bring similar capabilities without requiring every participant to trust a single corporate coordinator.

**Academic integrity.** This one is personal. As a researcher studying retrieval-augmented generation for education, I see the reproducibility crisis firsthand — 70% of scientists have tried and failed to reproduce another's results (Baker, 2016). Storing datasets and code on verifiable infrastructure at publication time creates an immutable record. Future researchers don't just download a dataset — they can *prove* it's the same one the original study used.

**Government records.** Property deeds, court decisions, legislative histories — these should be public goods with tamper-evident trails. Verifiable storage makes it so that no single administration can quietly rewrite the record.

The pattern across all these domains is the same: **we currently trust the same institution that controls the data to tell us the data is trustworthy.** That's a circular dependency. Verifiable data infrastructure breaks the loop.

---

## A Builder's Reflection

I don't just study this — I build on it. At 3MateLabs, we developed the **Walrus Sponsor SDK**, an open-source toolkit that lets dApp developers offer gasless, sponsored storage. Think "storage account abstraction": users upload files without needing tokens or understanding the protocol.

What building on Walrus taught me — and what papers alone can't — is that **developer experience is a make-or-break property of verifiable infrastructure.** Walrus's ~800ms retrieval, HTTP-compatible APIs, and Sui-native integration make it practical for interactive applications. Compare that to Filecoin's hours-long sealing or IPFS's unpredictable pinning — the difference between "theoretically secure" and "actually usable" is where most decentralized systems quietly fail.

---

## Conclusion

Verifiable data will become default infrastructure, just as HTTPS became default for the web. Not because users demanded encryption, but because it was made easy enough that there was no reason *not* to use it.

Walrus represents meaningful progress toward that future — breaking the traditional trade-off between storage efficiency, verification rigor, and practical performance. But the bigger insight is this: verifiable data is not a feature. It is infrastructure. And infrastructure wins by becoming invisible.

The question isn't *whether* we need verifiable data. It's how fast we can make it so seamless that trusting unverifiable data feels as reckless as connecting to an HTTP site does today.

---

### References

- Baker, M. (2016). 1,500 scientists lift the lid on reproducibility. *Nature*, 533(7604), 452–454.
- Benisi, N. Z. et al. (2020). Blockchain-based decentralized storage networks: A survey. *JNCA*, 162, 102656.
- Danezis, G. et al. (2025). Walrus: An efficient decentralized storage network. *arXiv:2505.05370v2*.
- Dimakis, A. G. et al. (2010). Network coding for distributed storage systems. *IEEE Trans. Info. Theory*, 56(9).
- Fisch, B. (2019). Proof of replication. *Eurocrypt 2019*.
- Lamport, L. et al. (1982). The Byzantine generals problem. *ACM TOPLAS*, 4(3), 382–401.
- Reed, I. S. & Solomon, G. (1960). Polynomial codes over certain finite fields. *JSIAM*, 8(2), 300–304.

---

*Submission for the Walrus Protocol Blog Post Challenge. Follow @WalrusProtocol · @EasonC13*
