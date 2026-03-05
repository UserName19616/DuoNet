# DuoNet White Paper

## Decentralized P2P Communication Network with Two-Level Economics

**Version 1.0 | March 2026**

**Author:** Alexei Nikolaev
**Contact:** [@Root_NM](https://t.me/Root_NM) | leha.nikolaev@gmail.com

---

## Abstract

DuoNet is a decentralized peer-to-peer network that combines secure messaging with economic incentives for participants. Unlike traditional messengers that rely on centralized servers, and classical blockchains focused exclusively on finance, DuoNet creates a **free communication infrastructure** where every participant can contribute and receive fair compensation.

The project's key innovation is a **two-level economic model** where servers behind NAT (home PCs, office machines) receive 70% of transaction fees for initiated transactions, creating an unprecedented incentive for mass adoption and making the network truly distributed.

---

## 1. Introduction: The Centralization Problem

### 1.1 Modern Communications

Today, most people use centralized services for communication: Telegram, WhatsApp, Signal, Viber. They all share common drawbacks:

- **Single point of failure.** Servers belong to one company and can be blocked by government decision.
- **Metadata control.** Companies see who communicates with whom, when, and how often.
- **No economic motivation.** Users provide their data for free, while the company pays for infrastructure.
- **Censorship.** Companies can delete content and block users at their discretion.

### 1.2 Pure P2P Solutions

Decentralized messengers do exist: Tox, RetroShare, Briar. However, they haven't achieved mass adoption because:

- **They require resources without compensation.** Why keep a computer running 24/7 if it brings no profit?
- **They're complex for ordinary users.** NAT configuration, port forwarding — these are technical barriers.
- **No economics.** Networks don't grow because there's no incentive to support them.

### 1.3 Blockchain Solutions

Blockchains (Bitcoin, Ethereum) solved the economic incentive problem, but:

- **Not designed for communication.** They're financial systems, not communication platforms.
- **Slow and expensive.** Sending every message as a transaction is impossible.
- **Not private.** All transactions are public.

### 1.4 Conclusion

We need a system that combines the best of all worlds:
- Free and private communication (like messengers)
- Economic incentives for participants (like blockchains)
- Accessibility for ordinary users (requiring no technical knowledge)
- Resistance to blocking (decentralized architecture)

**DuoNet was created to solve this problem.**

---

## 2. Philosophy and Principles

### 2.1 Core Principles

1. **Communication is a basic human right.** It should not depend on corporations or governments.
2. **Every contribution matters.** Network support should be rewarded, not an act of altruism.
3. **Privacy is not a luxury, but a norm.** End-to-end encryption by default, enhanced with user control.
4. **Economics must be fair.** Basic functionality is free, and income is distributed among those who actually support the network.
5. **Decentralization is achieved gradually.** The network starts small and evolves, but mechanisms stimulating growth are built in from the beginning.

### 2.2 The Name DuoNet

The name reflects the duality that permeates the entire project:

| Aspect | First Part | Second Part |
|--------------------|-------------------------|--------------------------------------|
| **Participants**   | Servers (miners)        | Clients (users)                      |
| **Protection**     | E2EE encryption         | Additional passphrase (double key)   |
| **Server Income**  | Mining (block creation) | Transaction initiation (70% of fees) |
| **Server Types**   | Public IP (validators)  | Behind NAT (initiators)              |
| **Network Layers** | Blockchain (value)      | P2P messenger (communication)        |

---

## 3. Network Architecture

### 3.1 Two Types of Participants

#### 3.1.1 Servers (Nodes / Miners)

Servers are the "workhorses" of the network. They run on enthusiasts' equipment and perform three functions:

**1. Blockchain Validation (Mining)**
- Participation in Proof-of-Trust consensus
- Block creation and signing
- Receiving block rewards (emission) and fees (10% from transactions)

**2. Rendezvous Server**
- Helping clients discover each other
- Storing temporary addresses of active nodes
- Assisting in establishing direct connections (NAT Hole Punching)

**3. Client Service**
- Receiving and storing offline messages (encrypted)
- Message routing
- Providing API for clients

**Server Requirements:**
- Minimum: 2 CPU cores, 4 GB RAM, 100 GB disk
- Recommended: public IP (for validator status)
- Even behind NAT, a server can participate and earn through transaction initiation

#### 3.1.2 Clients (Users)

Clients are ordinary users who want to communicate. They run a lightweight application on their device.

**Client Functions:**
- Secure message exchange (E2EE)
- Wallet management (optional)
- Creating and executing smart contracts
- Unique "Double Key" feature for ultra-private chats

**Registration:** email + password + seed phrase (12 words) for recovery.

### 3.2 Overall Architecture
┌─────────────────────────────────────────────────────────────────┐
│                           INTERNET                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐                │
│  │  Server A       │◄───────►│  Server B       │                │
│  │  (validator,    │   P2P   │  (initiator,    │                │
│  │   public IP)    │         │   behind NAT)   │                │
│  └────────┬────────┘         └────────┬────────┘                │
│           │                           │                         │
│           │ (protocol)                │ (protocol)              │
│           ▼                           ▼                         │
│  ┌─────────────────┐         ┌─────────────────┐                │
│  │  Client Peter   │         │  Client Mary    │                │
│  │  (chat + wallet)│         │  (chat only)    │                │
│  └─────────────────┘         └─────────────────┘                │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐        │
│  │          Rendezvous Server                          │        │
│  │          (any server with public IP can be this)    │        │
│  └─────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘

---

## 4. Economic Model

### 4.1 DNET Coin

- **Ticker:** DNET
- **Decimals:** 8 (1 DNET = 100,000,000 satoshi)
- **Emission:** only through creating new servers (no ICO, no premine, no "team wallets")

### 4.2 Progressive Emission

Emission is tied to the number of existing servers. The earlier a participant joins the network, the higher the reward. This fairly compensates early adopters for their risk.

| Server Range | Emission per Server | Burned (50%) | To Deposit |
|--------------|---------------------|--------------|------------|
| 1-100        | 1000 DNET           | 500 DNET     | 500 DNET   |
| 101-200      | 800 DNET            | 400 DNET     | 400 DNET   |
| 201-500      | 500 DNET            | 250 DNET     | 250 DNET   |
| 501-1000     | 300 DNET            | 150 DNET     | 150 DNET   |
| 1001-2000    | 150 DNET            | 75 DNET      | 75 DNET    |
| 2001-5000    | 75 DNET             | 37.5 DNET    | 37.5 DNET  |
| 5001-7500    | 40 DNET             | 20 DNET      | 20 DNET    |
| 7501-8000    | 30 DNET             | 15 DNET      | 15 DNET    |
| 8001-10000   | 20 DNET             | 10 DNET      | 10 DNET    |
| 10001+       | 10 DNET             | 5 DNET       | 5 DNET     |

**Mathematical justification:** total coins in circulation after 10,000 servers will be ~505,000 DNET, providing sufficient liquidity without hyperinflation.

### 4.3 Transaction Fee Distribution

Each transaction is subject to a **0.02%** fee. The fee is distributed as follows:
┌─────────────────────────────────────┐
│         TRANSACTION                 │
│  Amount: 1000 DNET                  │
│  Fee (0.02%): 0.2 DNET              │
└─────────────────────────────────────┘
                    │
                    ▼
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌───────────────┐       ┌───────────────┐
│ 70% to initiator      │ 10% to validator│
│ 0.14 DNET             │ 0.02 DNET       │
└───────────────┘       └───────────────┘
        │                       │
        └───────────┬───────────┘
                    │
                    ▼
            ┌───────────────┐
            │ 20% burned    │
            │ 0.04 DNET     │
            └───────────────┘


**Distribution Rationale:**

| Recipient | Share | Why |
|-----------|-------|-----|
| **Initiator** (NAT server) | 70% | Stimulates mass server deployment by ordinary users. They are the ones who make the network truly distributed and resistant to blocking. |
| **Validator** (public IP server) | 10% | Earns through scale (serves thousands of transactions). Even with 10%, their income exceeds any initiator's. |
| **Burning** | 20% | Deflationary mechanism, offsetting emission and creating long-term coin value. |

### 4.4 Income Comparison (Modeling)

With a network of 1000 servers (100 validators, 900 initiators) and 10,000 transactions per day:

| Metric                   | Initiator (NAT) | Validator (public IP) |
|--------------------------|-----------------|-----------------------|
| Transactions through one | ~11 (average)   | 10,000 (all)          |
| Income per transaction   | 70% of fee      | 10% of fee            |
| **Daily Income**         | ~7.7 DNET       | ~200 DNET             |
| **Monthly Income**       | ~230 DNET       | ~6000 DNET            |

The initiator receives fair compensation for serving their clients, while the validator is rewarded for ensuring the entire network's operation.

---

## 5. Consensus: Proof-of-Trust

### 5.1 Problems with Existing Mechanisms

| Mechanism                         | Problem                                                 |
|-----------------------------------|---------------------------------------------------------|
| **Proof-of-Work** (Bitcoin)       | Enormous electricity consumption, mining centralization |
| **Proof-of-Stake** (Ethereum 2.0) | "Rich get richer," requires large initial capital       |
| **Delegated PoS** (EOS)           | De facto centralization with a few large delegates      |

### 5.2 Solution: Multidimensional Trust

DuoNet proposes a third path — **Proof-of-Trust**. A server's consensus weight is calculated using a multidimensional metric:
Weight = 30% (Balance) + 20% (Uptime) + 10 (Purity) + 20% (Activity) + 15% (Speed) + 5% (Tenure)

### 5.3 Weight Components

| Factor | Weight | Description | Formula |
|---------------|--------|-----------------------------------|---------------------------------------|
| **Balance**   | 30%    | Current server deposit            | `log10(balance) / log10(max_balance)` |
| **Uptime**    | 20%    | Percentage of time online         | `online_time / 30_days`               |
| **Purity**    | 10%    | Absence of penalties              | `1 / (1 + violations_count)`          |
| **Activity**  | 20%    | Number of transactions (not self) | `transactions / max_transactions`     |
| **Speed**     | 15%    | Average block signing time        | `1 - (response_time / 5000)`          |
| **Tenure**    | 5%     | Server age in days                | `min(age / 365, 1)`                   |

### 5.4 Advantages of Proof-of-Trust

- **Not just the rich.** A server with modest balance but excellent uptime and activity can have significant weight.
- **Encourages good behavior.** Penalties reduce weight, making violations unprofitable.
- **Adaptability.** Metrics can be adjusted through community voting.
- **Attack resistance.** To attack the network, one needs to control not 51% of coins, but 51% of "trust" — significantly more difficult.

---

## 6. Key Innovations

### 6.1 Non-Refundable Escrow (Unique Feature #1)

#### 6.1.1 The Problem with Classic Escrow Contracts

In all existing blockchains (Ethereum, Solana, BSC), escrow contracts provide for **funds return** to the buyer if conditions aren't met. This creates a problem:

- Seller may perform the work, but buyer may not confirm receipt
- Buyer may be dishonest and revoke the payment
- No perfect incentive for either party

#### 6.1.2 DuoNet's Solution

Funds are locked at a special contract address and can be obtained **only** by presenting the correct passphrase. If the phrase is not presented by the deadline, the funds are **burned forever**.
┌──────────────────────────────────────────────────────────────┐
│                    ESCROW SCHEME                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Buyer creates a contract:                                │
│     - Amount: 1000 DNET                                      │
│     - Seller: @seller                                        │
│     - Phrase: "secret123" (hash is stored)                   │
│     - Deadline: 30 days                                      │
│                                                              │
│  2. Funds are locked at the contract address                 │
│                                                              │
│  3. Seller performs the work and receives the phrase         │
│                                                              │
│  4. Seller presents the phrase to the contract               │
│                                                              │
│  5. If phrase is correct → funds go to seller                │
│                                                              │
│  6. If phrase not presented within 30 days → funds are burned│
│                                                              │
└──────────────────────────────────────────────────────────────┘

#### 6.1.3 Economic Effect

| Participant       | Incentive                                             |
|-------------------|-------------------------------------------------------|
| **Buyer**         | Give the phrase, otherwise lose money                 |
| **Seller**        | Perform work well, otherwise won't receive the phrase |
| **Both parties**  | Reach agreement at any cost, otherwise both lose      |

This creates an **ideal market mechanism** where honesty becomes the only rational strategy.

### 6.2 Double Key (Unique Feature #2)

#### 6.2.1 The Problem

Standard E2EE encryption (Double Ratchet) protects the communication channel, but:

- If a device is compromised, attackers can read all messages
- If authorities gain access to a device (lawfully or not) — correspondence is exposed
- Users cannot control access to individual chats

#### 6.2.2 DuoNet's Solution

Each chat can be protected with an additional passphrase that conversation partners exchange in person, by phone, or through another channel.

**Technical Implementation:**

```python
def encrypt_with_phrase(message, session_key, phrase):
    # 1. Mix session key with the phrase
    salt = os.urandom(16)
    phrase_key = argon2(phrase, salt, memory=64*1024, iterations=3)
    combined_key = xor(session_key, phrase_key)
    
    # 2. Encrypt the message
    nonce = os.urandom(12)
    cipher = ChaCha20_Poly1305(combined_key, nonce)
    ciphertext = cipher.encrypt(message)
    
    # 3. Send salt + nonce + ciphertext
    return salt + nonce + ciphertext

def decrypt_with_phrase(packet, session_key, phrase):
    salt = packet[:16]
    nonce = packet[16:28]
    ciphertext = packet[28:]
    
    phrase_key = argon2(phrase, salt, memory=64*1024, iterations=3)
    combined_key = xor(session_key, phrase_key)
    
    cipher = ChaCha20_Poly1305(combined_key, nonce)
    try:
        return cipher.decrypt(ciphertext)
    except:
        return None  # Wrong phrase -> "garbage"
6.2.3 Advantages
Protection against device compromise. Even knowing the password, an attacker cannot read the chat without the phrase.

Protection against authorities. Physical access to the device does not grant access to correspondence.

User control. The phrase can be changed at any time, creating "floating" keys.

Plausible deniability. Without the phrase, messages appear as random noise.

6.3 NAT Server Incentives (Unique Feature #3)
6.3.1 The Problem
In existing P2P networks, servers behind NAT are "second-class citizens." They cannot accept incoming connections and typically do not participate in mining.

6.3.2 DuoNet's Solution
Servers behind NAT receive 70% of transaction fees for initiated transactions. This flips the economics:

A home PC with 10 clients (family, friends) can earn ~230 DNET per month

An office server with 50 employees earns even more

The network gains thousands of distributed nodes resistant to blocking

Comparison with Traditional Approach:

Aspect	Traditional Blockchain	DuoNet
NAT servers' role	Passive observers	Active initiators
NAT servers' income	0% (or very little)	70% of fees
Incentive to run	None	High
Resistance to blocking	Low	Very high
7. Security and Privacy
7.1 Transport Layer
Protocol: libp2p with Noise Protocol

Encryption: TLS 1.3 over each connection

Masking: traffic can be wrapped in WebSocket (looks like regular HTTPS)

7.2 Application Layer (Messages)
Base encryption: Double Ratchet (Signal protocol)

Additional protection: user-defined passphrase (double key)

Authentication: Ed25519 signatures

7.3 Application Layer (Transactions)
Signatures: Ed25519

Replay protection: nonce (transaction counter)

Confidentiality: addresses are public key hashes (pseudonymity)

7.4 Metadata Protection
What is visible	Server	Network Observer
Message content	No (encrypted)	No (encrypted)
Who communicates with whom	Yes (for routing)	Partially (traffic analysis)
IP addresses	Yes	Yes
Message timing	Yes	Partially
For maximum privacy, Tor or I2P is recommended.

7.5 Penalty System
Violation	1st Offense	2nd Offense	3rd Offense
Inactivity (>3 days)	Warning	Remove from trusted	-
Double block signing	0.1 DNET fine	1 DNET fine	Confiscation
Invalid block	0.1 DNET fine	1 DNET fine	Confiscation
Spam	0.1 DNET fine	1 DNET fine	Confiscation
8. Server Selection by Client
Clients can connect to servers in four ways:

8.1 Auto Mode (Default)
Client sends a broadcast request to the local network

If a server exists in the local network — connect to it

If not — contact a public rendezvous server

Rendezvous server returns a list of nearby active servers

Client connects to the first responder

8.2 Geo Mode (Geographic)
Client determines their region (by IP or GPS)

Requests servers in that region from the rendezvous server

Sorts by ping and connects to the optimal one

8.3 Premium Mode (Future Versions)
Client has a subscription (paid in DNET)

Connects to servers marked "premium"

Receives priority service, more features

8.4 Manual Mode
User manually enters a server address, e.g.: server.duonet.net:9876

9. Resistance to Blocking
9.1 Comparison with Existing Services
Service	Architecture	Resistance to Blocking
Telegram	Centralized	Low (few data centers)
WhatsApp	Centralized	Low (Meta servers)
Signal	Centralized	Low (AWS/Google Cloud)
Tor	Decentralized	Medium (bridge blocking)
DuoNet	Decentralized	High
9.2 Why DuoNet is Difficult to Block
Thousands of independent servers. Blocking one IP solves nothing.

Servers behind NAT. Their IPs are dynamic and constantly changing.

Traffic masking. The protocol can look like regular HTTPS.

Economic incentive. Miners are motivated to bypass blocks.

Autonomy. Clients store server lists and can function without centralized entry points.

9.3 What to Do During Blocking
Use Tor/I2P (built-in support)

Exchange addresses through friends (social relay)

Run rendezvous servers on hidden services

10. Roadmap
The project is seeking a development team. The timelines below are from the moment the team is formed.

Phase 1: Prototyping (1-2 quarters)
Rendezvous server (Python + FastAPI)

Basic blockchain (transactions, blocks)

Consensus mechanism (Proof-of-Trust)

TUI client for testing

Open-source code publication

Phase 2: Alpha Version (1-2 quarters)
Migration to libp2p (P2P network)

Full consensus implementation

Smart contracts (escrow)

Client with wallet and double key

Phase 3: Beta and Mainnet (1-2 quarters)
Security audit

Mainnet launch

First 100 servers (invited miners)

Initial coin distribution

Phase 4: Ecosystem Development
Graphical client (Qt)

Mobile applications

Developer tools

Group chats, files, media

11. DNET Tokenomics
11.1 Parameters
Parameter	Value
Name	DuoNet Ecosystem Token
Ticker	DNET
Decimals	8
Initial emission (up to 10,000 servers)	~1,010,000 DNET
Actually in circulation (with burning)	~505,000 DNET
Maximum emission	Unlimited, but burning creates deflation
11.2 Block Reward Distribution
Recipient	Share	Purpose
Block validator	70%	Reward for block creation
Burning	30%	Deflationary mechanism
11.3 Coin Utility
Premium features payment (future)

Smart contract creation (fees)

Staking (increasing server weight)

Exchange trading (optional)

12. Community Governance (DAO)
In the future, network governance will be transferred to coin holders:

Protocol parameter changes (fees, emission)

Grant distribution (development fund)

Dispute resolution

Trusted server selection

Mechanism: voting proportional to server weight (not number of coins!).

13. Conclusion
DuoNet is not just another messenger and not just another blockchain. It is a free communication ecosystem where every participant can find their role:

Regular users get a free, secure, and private messenger with unique "Double Key" protection.

Enthusiasts run servers (even on home PCs), earn DNET coins, and help the network grow.

Businesses use revolutionary non-refundable escrow smart contracts for secure transactions.

Developers build new services on an open platform.

Three unique innovations set DuoNet apart from everything that exists today:

🔥 Non-Refundable Escrow — the world's first mechanism where non-cooperating parties lose everything.

🔐 Double Key — the only messenger where even device compromise won't expose your conversations.

🏠 NAT Server Incentives — for the first time, home PCs receive 70% of fees, becoming the network's foundation.

We invite everyone who values free communication to join the project. The source code will be open (MIT license), governance will become decentralized, and the future will be determined by the community.

Join the free communication revolution!

14. Contact
Author: Alexei Nikolaev

Telegram: @Root_NM

Email: leha.nikolaev@gmail.com

GitHub: to be published

DuoNet is an open project. No token presales, no ICOs, no "locked team tokens." Only code, community, and freedom.
