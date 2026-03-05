# DuoNet

**Decentralized P2P Communication Network with Two-Level Economics**

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

**Idea Author & Architect:** Alexei Nikolaev ([@Root_NM](https://t.me/Root_NM)) | leha.nikolaev@gmail.com

**Project Status:** Seeking implementation team (concept fully developed, looking for developers, cryptographers, enthusiasts)

---

## About the Project

DuoNet is not just another messenger and not just another blockchain. It's an ecosystem of free communication where everyone can find their role:

- **Regular users** get a free, secure, and private messenger with unique "Double Key" protection
- **Enthusiasts** run servers (even on home PCs behind NAT), earn DNET coins, and help the network grow
- **Businesses** use revolutionary non-refundable escrow smart contracts for secure transactions
- **Developers** build new services on an open platform

The name DuoNet reflects the duality that permeates the entire project: two types of participants (servers and clients), two levels of protection (encryption + additional passphrase), two sources of income for servers (mining and transaction initiation).

---

## Key Innovations

### 🔥 Non-Refundable Escrow (Unique!)
The world's first smart contract mechanism where non-cooperating parties lose everything. Funds are locked at a special address and can only be obtained by the seller presenting the correct passphrase. If the phrase is not presented by the deadline, the funds are burned. This creates the perfect incentive to fulfill obligations: either the deal is completed, or the money disappears forever.

### 🔐 Double Key (Unique!)
Standard E2EE encryption (Double Ratchet) is enhanced with a user-defined passphrase that conversation partners exchange in person or by phone. The phrase is mixed with the session key — without it, decryption yields only "digital garbage." Even if a device is compromised or authorities gain access to data, without the second phrase the correspondence cannot be read.

### ⚖️ Proof-of-Trust
A multidimensional consensus mechanism that considers not only server balance (30%), but also uptime (20%), penalty-free record (10%), activity (20%), response speed (15%), and tenure (5%). This prevents the "rich" from dominating — any stable and honest server has weight in the network.

### 🏠 NAT Server Incentives
For the first time in blockchain networks: servers behind NAT (home PCs, office machines) receive **70% of transaction fees** for initiated transactions — significantly more than validators with public IPs! This creates incentives for mass adoption and makes the network truly distributed.

---

## Economic Model

### DNET Coin
- **Ticker:** DNET
- **Decimals:** 8 (1 DNET = 100,000,000 satoshi)
- **Emission:** only through creating new servers (no ICO, no premine, no "team wallets")

### Progressive Emission with Burning

| Server Range | Emission per Server | Burned (50%) | To Deposit | Real Coins in Circulation (cumulative) |
|--------------|---------------------|--------------|------------|----------------------------------------|
| 1-100 | 1000 DNET | 500 DNET | 500 DNET | 50,000 DNET |
| 101-200 | 800 DNET | 400 DNET | 400 DNET | 90,000 DNET |
| 201-500 | 500 DNET | 250 DNET | 250 DNET | 165,000 DNET |
| 501-1000 | 300 DNET | 150 DNET | 150 DNET | 240,000 DNET |
| 1001-2000 | 150 DNET | 75 DNET | 75 DNET | 315,000 DNET |
| 2001-5000 | 75 DNET | 37.5 DNET | 37.5 DNET | 427,500 DNET |
| 5001-7500 | 40 DNET | 20 DNET | 20 DNET | 477,500 DNET |
| 7501-8000 | 30 DNET | 15 DNET | 15 DNET | 485,000 DNET |
| 8001-10000 | 20 DNET | 10 DNET | 10 DNET | 505,000 DNET |
| 10001+ | 10 DNET | 5 DNET | 5 DNET | - |

**The first 100 servers** receive a deposit of 500 real DNET each — fair compensation for the risk and contribution to the network's establishment.

### Transaction Fee Distribution (0.02%)
- **70% — to initiator** (server that first received the transaction from a client)
- **10% — to validator** (server that included the transaction in a block)
- **20% — burned** (deflationary mechanism)

No "development fund" or team wallets — only fair distribution among network participants. This allocation creates real motivation for mass deployment of NAT servers, making the network truly decentralized.

---

## Two Types of Participants

### Servers (Node / Miner)
A single application that performs three functions:
- **Mining** — participating in Proof-of-Trust consensus, creating blocks
- **Rendezvous Server** — helping clients discover each other
- **Client Service** — receiving, storing, and forwarding messages

**Requirements:**
- Minimum: 2 cores, 4 GB RAM, 100 GB disk
- Recommended: public IP (for full functionality)
- Even behind NAT, you can participate and earn through transaction initiation

### Clients (Messenger)
Lightweight application for end users:
- Free text messages with E2EE encryption
- Optional wallet for sending/receiving DNET
- Creating and executing smart contracts
- Unique "Double Key" feature for ultra-private chats

**Registration:** email + password + seed phrase (12 words) for recovery.

---

## Client Server Selection (4 Modes)

| Mode | Description | For Whom |
|------|-------------|----------|
| **Auto** | First responding server | Beginners |
| **Geo** | Geographically closest | Cost-conscious |
| **Premium** | Paid servers with extra services (future) | Demanding users |
| **Manual** | Manual address entry | Experts |

---

## Roadmap

*The project is seeking a development team. The timelines below are from the moment the team is formed.*

### Phase 1: Prototyping (1-2 quarters after development start)

| Task | Technologies | Expected Result |
|------|--------------|-----------------|
| Rendezvous server (prototype) | Python + FastAPI | Working rendezvous server |
| Basic blockchain | Python + Cryptography | Transactions, blocks, hashing |
| Consensus mechanism (Proof-of-Trust) | Python + asyncio | Test network with 10-20 servers |
| TUI client | Textual | Basic interface for testing |
| Open-source code publication | GitHub | Repository with documentation |

**Phase result:** Working testnet for early users.

### Phase 2: Alpha Version (1-2 quarters)

| Task | Technologies | Expected Result |
|------|--------------|-----------------|
| Migration to libp2p | Go/Rust or py-libp2p | Stable P2P network |
| Full consensus | - | Proof-of-Trust with multidimensional metrics |
| Smart contracts (escrow) | VM engine | Working non-refundable contracts |
| Client with wallet | - | DNET integration |
| Private chat with Double Key | - | Unique feature ready |

**Phase result:** Closed alpha version for the first 100 servers.

### Phase 3: Beta and Mainnet (1-2 quarters)

| Task | Technologies | Expected Result |
|------|--------------|-----------------|
| Security audit | External experts | Vulnerability fixes |
| Mainnet launch | - | Production network with real coins |
| First 100 servers | - | Invited miners |
| Initial distribution | - | Progressive emission with burning |

**Phase result:** Working mainnet with community.

### Phase 4: Ecosystem Development (after mainnet launch)

| Task | Priority | Expected Result |
|------|----------|-----------------|
| Graphical client (Qt) | High | Convenient interface for mass adoption |
| Mobile apps | High | iOS/Android clients |
| Developer tools | Medium | SDK, API, documentation |
| Exchange listings | Low | DNET trading (optional) |
| Group chats | Medium | Functionality expansion |
| Files and media | Low | IPFS integration |

**Phase result:** Full ecosystem with thousands of users.

---

## Who We're Looking For

### 👨‍💻 Developers
- **Backend/P2P:** experience with libp2p, Python/Go/Rust, distributed systems
- **Blockchain:** understanding of consensus algorithms, cryptography, smart contracts
- **Frontend:** building clients (TUI, Qt, mobile)

### 🔐 Cryptographers
- Security protocol auditing
- Double Ratchet implementation and additional crypto primitives

### 📢 Marketers & Community Managers
- Community building
- Project promotion
- Partner search

### 💰 Investors (later stage)
- Infrastructure development support
- Grants and funds

---

## How to Join

1. **Read the documentation** (White Paper, Technical Spec — will be published in the repository)
2. **Contact the author:**
   - Telegram: [@Root_NM](https://t.me/Root_NM)
   - Email: leha.nikolaev@gmail.com
3. **Suggest your ideas** — the project is open to improvements
4. **Help with translation** of documentation into other languages

---

## License

The documentation is licensed under [Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).

**You are free to:**
- Share — copy and redistribute the material
- Adapt — remix, transform, and build upon the material

**Under the following terms:**
- **Attribution** — you must give appropriate credit (Alexei Nikolaev)
- **NonCommercial** — you may not use the material for commercial purposes without author's consent

This protects the idea from direct copying into commercial projects while allowing the community to develop the concept.

---

## Contact

- **Author:** Alexei Nikolaev
- **Telegram:** [@Root_NM](https://t.me/Root_NM)
- **Email:** leha.nikolaev@gmail.com
- **GitHub:** *link will appear after repository publication*

---

**Stars welcome ⭐ — more stars mean higher chance of finding a team!**

**Join the free communication revolution!**
