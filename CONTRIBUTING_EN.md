# How to Contribute to DuoNet

**Version 1.0 | March 2026**

Thank you for your interest in the DuoNet project! This is an open project, and we welcome any help — from fixing typos to developing the blockchain core.

---

## 1. About the Project

DuoNet is a decentralized P2P communication network with economic incentives for participants. The project is at the stage of seeking a team for implementation. All documentation (White Paper, Technical Spec, Economic Model) is already prepared and available in the repository.

---

## 2. Ways to Contribute

### 2.1 📢 Spread the Word

The simplest way to help is to tell others about the project:

- Star the repository on GitHub (this increases visibility)
- Share the link on social media (Twitter, LinkedIn, Telegram channels)
- Write a post or article about DuoNet on Medium, Reddit, or other platforms
- Mention the project in relevant communities (crypto enthusiasts, P2P networks, messengers)

### 2.2 💬 Feedback and Discussion

Even if you're not a developer, your opinion matters:

- **Read the documentation** — start with the README and White Paper
- **Ask questions** — if something is unclear, chances are others have the same question
- **Suggest ideas** — how to improve the economics, architecture, UX
- **Report errors** — typos, inaccuracies, logical inconsistencies

### 2.3 🔧 Development

We are looking for developers to implement the project. If you want to participate in coding:

#### Technologies We Need:

| Area | Technologies | Difficulty |
|------|--------------|------------|
| **P2P Network** | libp2p (Go/Rust), NAT Traversal, DHT | High |
| **Blockchain Core** | Python (prototype), Go/Rust (prod) | High |
| **Cryptography** | Double Ratchet, libsodium, Ed25519 | High |
| **Client (TUI)** | Python + Textual | Medium |
| **Client (GUI)** | PySide6 / Qt | Medium |
| **Rendezvous Server** | Python + FastAPI/gRPC | Low |
| **Documentation** | Markdown, English/Russian | Low |

#### What Can Be Done Right Now:

1. **Rendezvous server** — a separate microservice in Python/FastAPI. Doesn't require blockchain, can start with this.
2. **TUI client** — basic interface for sending messages via a mock server.
3. **Blockchain prototype** — transactions, blocks, hashing (without P2P).
4. **Testing tools** — key generation, transaction signing.

### 2.4 🎨 Design and UX

If you're a designer, you can help with:

- **Logo** and visual style of the project
- **Architecture diagrams** (for documentation)
- **Interface mockups** (for the future GUI client)
- **Presentation materials** (for pitches and conferences)

### 2.5 🌐 Translation

The project targets an international audience. Help with documentation translation:

- English ✅ (basic ready)
- Russian ✅ (basic ready)
- Chinese, Spanish, Arabic, Hindi — help needed

### 2.6 📊 Marketing and Community

We are looking for people to help:

- Create and moderate a Telegram chat / Discord server
- Write posts and articles
- Find partners and potential investors
- Organize online meetups

---

## 3. How to Get Started

### Step 1: Familiarize Yourself with the Project

Start by reading:

- [README_EN.md](README_EN.md) — overview
- [WHITEPAPER_EN.md](WHITEPAPER_EN.md) — concept and philosophy
- [TECHNICAL_SPEC_EN.md](TECHNICAL_SPEC_EN.md) — technical details
- [ECONOMICS_EN.md](ECONOMICS_EN.md) — economic model

### Step 2: Contact the Author

- **Telegram:** [@Root_NM](https://t.me/Root_NM)
- **Email:** leha.nikolaev@gmail.com

Write about what you'd like to help with and what experience you have.

### Step 3: Join the Discussion

We will create a chat when the first interested group appears. For now, all discussions happen in direct messages with the author.

### Step 4: Start Working

If you're a developer, choose a task from the list below and propose your approach.

---

## 4. Tasks for Early Contributors

### 🔹 Level: Beginner (no blockchain experience needed)

| Task | Technologies | Estimated Effort |
|------|--------------|------------------|
| Write a script for Ed25519 key generation | Python + PyNaCl | 1-2 hours |
| Create a simple HTTP rendezvous server | Python + FastAPI | 2-3 hours |
| Write tests for SeedPhrase (BIP39) | Python + pytest | 1-2 hours |
| Translate documentation to another language | - | Depends on language |

### 🔸 Level: Intermediate

| Task | Technologies | Estimated Effort |
|------|--------------|------------------|
| TUI client with basic functionality | Python + Textual | 1-2 weeks |
| libp2p integration (prototype) | py-libp2p | 2-3 weeks |
| Double Ratchet implementation (without additional phrase) | Python + cryptography | 1-2 weeks |
| Proof-of-Trust consensus prototype | Python + asyncio | 2-3 weeks |

### 🔹 Level: Advanced

| Task | Technologies | Estimated Effort |
|------|--------------|------------------|
| Full blockchain node in Go/Rust | Go/Rust + libp2p | 2-3 months |
| Escrow contract implementation | VM engine | 1-2 months |
| Protocol security audit | - | 1-2 weeks |

---

## 5. Work Process

### If You Found a Bug or Want to Suggest an Improvement:

1. Check if there's already such an issue in the repository
2. Create a new issue with a detailed description
3. If it's a bug — provide steps to reproduce
4. If it's a suggestion — explain why it would improve the project

### If You Want to Submit Code Changes:

1. Fork the repository
2. Create a branch for your work (`feature/your-feature-name`)
3. Write code following the style guidelines (PEP 8 for Python)
4. Add tests if possible
5. Submit a pull request to the main repository
6. Describe what you changed and why

---

## 6. Code Style and Conventions

### Python

- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Use docstrings for classes and functions
- Type hints are welcome

### Other

- Code comments — in English (for international team)
- Documentation — in Russian and English
- Variable names — meaningful, no abbreviations

---

## 7. License and Copyright

All project code will be distributed under the **MIT** license. Documentation — under the **CC BY-NC 4.0** license.

**Important:** Commercial use of documentation without the author's consent is prohibited. This protects the concept from direct copying while allowing the community to develop the project.

By submitting a pull request, you agree that your code will be distributed under the MIT license.

**Important:** The author places themselves on equal terms with all project participants — no premine, no team wallets, etc. If the author contributes only the idea and does not participate in the network's development, they will receive only moral satisfaction. This is fair.

---

## 8. Contacts and Community

- **Author:** Alexei Nikolaev
- **Telegram:** [@Root_NM](https://t.me/Root_NM)
- **Email:** leha.nikolaev@gmail.com
- **GitHub:** *to be published*

---

## 9. Acknowledgments

Thank you for reading this far! Even just showing interest in the project is a contribution. If you're not ready to actively participate yet, follow updates (star the repository on GitHub) — maybe later you'll find a task you like.

**Together we can build a truly free communication network!**
