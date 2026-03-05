# DuoNet Roadmap

**Version 1.0 | March 2026**

**Author:** Alexei Nikolaev
**Contact:** [@Root_NM](https://t.me/Root_NM) | leha.nikolaev@gmail.com

---

## 1. Important Note

The DuoNet project is at the **seeking implementation team** stage. All documentation (White Paper, Technical Spec, Economic Model) is fully ready, but the code has not been written yet.

The timelines below are counted **from the moment the team is formed**. Until a team exists, the project exists only as a concept and documentation.

---

## 2. Current Status (March 2026)

| Component                       |    Status      |
|---------------------------------|----------------|
| Concept and Philosophy          | ✅ Ready       |
| White Paper                     | ✅ Ready       |
| Technical Specification         | ✅ Ready       |
| Economic Model                  | ✅ Ready       |
| Contributor Documentation       | ✅ Ready       |
| Team Search                     | 🟡 In Progress |
| Code                            | ⬜ Not Started |
| Test Network                    | ⬜ Not Started |
| Mainnet                         | ⬜ Not Started |

---

## 3. Development Stages

### Stage 0: Team Formation

*Duration: indefinite (depends on community interest)*

**Goal:** Find like-minded people to implement the project.

**Tasks:**
- [x] Prepare complete documentation
- [x] Publish repository on GitHub
- [ ] Attract first developers (P2P, blockchain, cryptography)
- [ ] Attract designers (logo, diagrams, UI/UX)
- [ ] Create a discussion chat (Telegram / Discord)
- [ ] Gather feedback from the community

**Result:** A team of 3-5 key participants formed, ready to start development.

---

### Stage 1: Prototyping (Testnet Alpha)

*Duration: 2-3 months after team formation*

**Goal:** Create a working prototype for testing basic functionality.

#### Technical Tasks:

| Task                        | Technologies            | Difficulty | Expected Result                                    |
|-----------------------------|-------------------------|------------|----------------------------------------------------|
| Rendezvous server           | Python + FastAPI/gRPC   | Low        | Working server for peer registration and discovery |
| Basic identification        | Python + PyNaCl         | Low        | Ed25519 key generation, BIP39 seed phrases         |
| Blockchain prototype        | Python                  | Medium     | Transactions, blocks, hashing (without P2P)        |
| Minimal TUI client          | Python + Textual        | Medium     | Message sending via mock server                    |
| NAT server emulation        | Python + Docker         | Medium     | Testing mechanics for servers behind NAT           |

**Stage Result:**
- Working rendezvous server prototype
- Ability to create keys and sign transactions
- TUI client for manual testing
- Code published in open repository

---

### Stage 2: Alpha Version (Testnet Beta)

*Duration: 3-4 months after Stage 1*

**Goal:** Implement blockchain core and P2P network.

#### Technical Tasks:

| Task                    | Technologies            | Difficulty | Expected Result                               |
|-------------------------|-------------------------|------------|-----------------------------------------------|
| libp2p integration      | Go/Rust or py-libp2p    | High       | P2P network with DHT and GossipSub            |
| Full consensus          | Go/Rust                 | High       | Proof-of-Trust with server weight calculation |
| Emission mechanism      | Go/Rust                 | Medium     | Progressive emission with 50% burning         |
| Fee distribution        | Go/Rust                 | Medium     | 70/10/20 (initiator/validator/burn)           |
| Improved TUI client     | Python + Textual        | Medium     | Wallet support, history                       |
| Basic GUI client        | PySide6 / Qt            | Medium     | First version for testing                     |

**Stage Result:**
- Working P2P network with 10-20 test servers
- Proof-of-Trust consensus in action
- Ability to send transactions and see them in blocks
- Closed testing with first participants

---

### Stage 3: Beta Version (Mainnet Candidate)

*Duration: 2-3 months after Stage 2*

**Goal:** Prepare the network for mainnet launch.

#### Technical Tasks:

| Task                         | Technologies       | Difficulty | Expected Result                  |
|------------------------------|--------------------|------------|----------------------------------|
| Smart contracts (escrow)     | VM engine          | High       | Working non-refundable contracts |
| Double Key                   | Python/C++         | Medium     | Additional phrase integration    |
| Security audit               | External experts   | High       | Vulnerability fixes              |
| Load testing                 | -                  | Medium     | Performance optimization         |
| Full GUI client              | PySide6 / Qt       | Medium     | Ready client for users           |
| User documentation           | Markdown           | Low        | Instructions, FAQ                |

#### Organizational Tasks:

| Task                          | Responsible | Expected Result                       |
|-------------------------------|-------------|---------------------------------------|
| Mainnet launch                | Team        | Mainnet with real coins               |
| Attract first 100 servers     | Marketing   | Starter network with sufficient power |
| Initial coin distribution     | Team        | Progressive emission by scale         |
| Exchange listings (optional)  | Marketing   | DNET trading on small exchanges       |

**Stage Result:**
- Working mainnet
- First 100 servers from invited miners
- Beginning of DNET coin circulation
- Community of early users

---

### Stage 4: Ecosystem Development

*Duration: ongoing after mainnet launch*

**Goal:** Expand functionality and achieve mass adoption.

#### Planned Directions:

| Direction                          | Priority | Expected Result                 |
|------------------------------------|----------|---------------------------------|
| Mobile applications                | High     | iOS/Android clients             |
| Group chats                        | High     | E2EE encryption for groups      |
| File and media sharing             | Medium   | IPFS integration                |
| Enhanced privacy                   | Medium   | Tor/I2P support                 |
| Decentralized exchanges            | Low      | P2P DNET exchange               |
| Developer tools                    | Medium   | SDK, API, documentation         |
| Integration with other blockchains | Low      | Bridges, cross-chain operations |
| DAO and community governance       | Medium   | Voting on network parameters    |

**Stage Result:**
- Full ecosystem with thousands of users
- Regular updates and improvements
- Community participating in governance

---

## 4. Realistic Timelines

Timelines depend on many factors: team size, participant experience, funding availability, etc.

| Stage                  | Optimistic    | Realistic         | Conservative    |
|------------------------|---------------|-------------------|-----------------|
| **Team Search**        | 1-3 months    | 3-6 months        | 6-12 months     |
| **Stage 1 (prototype)**| 2 months      | 3-4 months        | 6 months        |
| **Stage 2 (alpha)**    | 3 months      | 4-6 months        | 9 months        |
| **Stage 3 (mainnet)**  | 2 months      | 3-4 months        | 6 months        |
| **Total to mainnet**   | **~8 months** | **~13-18 months** | **~24+ months** |

*All timelines — from the moment the team is formed.*

---

## 5. What Can Be Done BEFORE a Team Appears

Even without a team, you can help the project right now:

### 🔹 For Everyone
- ⭐ Star the repository on GitHub
- 🔁 Share the link on social media and relevant chats
- 💬 Write a post or article about DuoNet
- 📝 Suggest ideas and improvements

### 🔸 For Developers
- Study [TECHNICAL_SPEC_EN.md](TECHNICAL_SPEC_EN.md)
- Try writing small prototypes (e.g., rendezvous server)
- Contact the author to discuss possible participation

### 🔹 For Designers
- Create a logo or architecture diagrams
- Design interface mockups

### 🔸 For Translators
- Help translate documentation into other languages

---

## 6. How to Influence Priorities

The roadmap is not a dogma but a plan. Priorities may change depending on:

- **Community opinion** — what matters most to users
- **Team capabilities** — what technologies are available
- **Market situation** — what is relevant now
- **Funding availability** — grants, investments

If you want to influence priorities:
1. Join the discussion (chat / issues)
2. Argue your position
3. Offer help with implementation

---

## 7. Risks and Mitigation

| Risk                        | Probability | Impact | Mitigation                                       |
|-----------------------------|-------------|--------|--------------------------------------------------|
| Unable to find a team       | Medium      | High   | Active promotion, encouraging early participants |
| Critical vulnerabilities    | Low         | High   | Security audit, bug bounties                     |
| Low miner activity          | Medium      | Medium | Economic incentives (70% fees to NAT servers)    |
| Regulatory risks            | Medium      | Medium | Decentralization, legal neutrality               |
| Competition                 | High        | Low    | Unique features (escrow, double key)             |

---

## 8. Conclusion

DuoNet is an ambitious project, but its implementation is quite realistic with a team of enthusiasts. The roadmap is realistic, taking into account that the code has not yet been written.

**The main thing now is to find people willing to invest their time and talent in creating a truly free communication network.**

If you are ready to participate — contact the author. If not yet — help spread the information. Every contribution matters.

---

## 9. Contacts

- **Author:** Alexei Nikolaev
- **Telegram:** [@Root_NM](https://t.me/Root_NM)
- **Email:** leha.nikolaev@gmail.com
- **GitHub:** *to be published*

---

**Together we can build a truly free communication network!**
