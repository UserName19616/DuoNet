# DuoNet Economic Model

## Detailed Description of Tokenomics and Network Incentives

**Version 1.0 | March 2026**

**Author:** Alexei Nikolaev
**Contact:** [@Root_NM](https://t.me/Root_NM) | leha.nikolaev@gmail.com

---

## 1. Introduction

The DuoNet economic model is built on the principles of fairness, decentralization, and long-term sustainability. Unlike traditional cryptocurrencies with pre-mining or ICOs, DuoNet has no "team wallets" or "development funds." All coins are created exclusively through launching new servers and are distributed among participants who support the network.

The key innovation is a **two-level economy** where servers behind NAT (home PCs, office machines) receive 70% of fees for initiated transactions, creating an unprecedented incentive for mass adoption.

---

## 2. DNET Coin

### 2.1 Basic Parameters

| Parameter                                  | Value                                                   |
|--------------------------------------------|---------------------------------------------------------|
| **Name**                                   | DuoNet Ecosystem Token                                  |
| **Ticker**                                 | DNET                                                    |
| **Decimals**                               | 8                                                       |
| **Minimum unit**                           | 1 satoshi = 0.00000001 DNET                             |
| **Emission type**                          | Only through creating new servers                       |
| **Initial emission (up to 10,000 servers)**| ~1,010,000 DNET (emitted)                               |
| **Actually in circulation (with burning)** | ~505,000 DNET                                           |
| **Maximum emission**                       | Unlimited, but burning creates deflationary pressure    |

### 2.2 Emission Principles

1. **No pre-mining** — not a single coin created before network launch
2. **No ICOs** — coins cannot be bought from the team
3. **No "team wallets"** — developers have no special privileges
4. **Only honest mining** — coins go to those who run servers
5. **Burning at creation** — 50% of emission is immediately destroyed, creating a deflationary effect

---

## 3. Progressive Emission

### 3.1 Rationale

Early network participants take on greater risk and should receive proportionally higher rewards. The progressive emission scale solves this problem while simultaneously protecting the network from hyperinflation at the initial stage.

### 3.2 Emission Scale

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

### 3.3 Mathematical Model

```python
# Emission function based on server count
def emission(server_count):
    if server_count < 100:
        return 1000
    elif server_count < 200:
        return 800
    elif server_count < 500:
        return 500
    elif server_count < 1000:
        return 300
    elif server_count < 2000:
        return 150
    elif server_count < 5000:
        return 75
    elif server_count < 7500:
        return 40
    elif server_count < 8000:
        return 30
    elif server_count < 10000:
        return 20
    else:
        return 10
        
3.4 Cumulative Effect
Stage         Servers       Total Emitted     Total Burned      In Circulation
1                 100             100,000           50,000              50,000
2                 200             180,000           90,000              90,000
3                 500             330,000          165,000             165,000
4                1000             480,000          240,000             240,000
5                2000             630,000          315,000             315,000
6                5000             855,000          427,500             427,500
7                7500             955,000          477,500             477,500
8                8000             970,000          485,000             485,000
9               10000           1,010,000          505,000             505,000
Emission Graph (in circulation):
500k │                                    ●
     │                                 ●
400k │                            ●
     │                        ●
300k │                   ●
     │              ●
200k │         ●
     │    ●
100k │●
     └───────────────────────────────
      0   2k  4k  6k  8k  10k servers
      
4. Fee Distribution
4.1 Basic Transaction Fee

Each transaction in the DuoNet network is subject to a 0.02% fee of the transfer amount. The minimum fee is 0.0001 DNET (10,000 satoshi).

4.2 Fee Distribution

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
┌─────────────────┐       ┌─────────────────┐
│ 70% to initiator│       │ 10% to validator│
│ 0.14 DNET       │       │ 0.02 DNET       │
└─────────────────┘       └─────────────────┘
        │                       │
        └───────────┬───────────┘
                    │
                    ▼
            ┌───────────────┐
            │ 20% burned    │
            │ 0.04 DNET     │
            └───────────────┘
            
Recipient	Share	Rationale
Initiator (NAT server)	70%	Stimulates mass server deployment by ordinary users. They make the network truly distributed and resistant to blocking.
Validator (public IP server)	10%	Earns through scale (serves thousands of transactions). Even with 10%, their income exceeds any initiator's.
Burning	20%	Deflationary mechanism, offsetting emission and creating long-term coin value.

4.3 Block Reward (Emission)

Each new block additionally creates 0.01 DNET (1,000,000 satoshi), which is distributed as follows:

Recipient	Share	Purpose
Block validator	70%	Reward for block creation
Burning	30%	Deflationary mechanism

5. Participant Income Comparison
5.1 Modeling for a Network of 1000 Servers

Model parameters:

Total servers: 1000

Validators (with public IP): 100

Initiators (behind NAT): 900

Transactions per day: 10,000

Average fee: 0.1 DNET

Metric                    Initiator (NAT)        Validator (public IP)
Transactions through one    ~11 (average)                 10,000 (all)
Income per transaction         70% of fee                   10% of fee
Daily income from fees          ~7.7 DNET                    ~200 DNET
Monthly income from fees        ~230 DNET                  ~6,000 DNET
Income from blocks (emission)      0         + ~21 DNET (at 10 blocks)
Total monthly income            ~230 DNET                  ~6,021 DNET

5.2 Modeling for a Network of 10,000 Servers
Metric                    Initiator (NAT)         Validator (public IP)
Transactions through one    ~10 (average)                 100,000 (all)
Monthly income from fees        ~210 DNET                  ~15,000 DNET

5.3 Income in Fiat Equivalent (Example)

At DNET price = $0.10:

Server Type	Income in DNET	Income in USD
Initiator (NAT, 10 clients)	230 DNET/month	$23/month
Initiator (NAT, 50 clients)	1150 DNET/month	$115/month
Validator (public IP, 1000 servers)	6000 DNET/month	$600/month
Validator (public IP, 10,000 servers)	15,000 DNET/month	$1,500/month

6. Coin Burning
6.1 Burning Mechanisms

DuoNet implements three coin burning mechanisms:

Mechanism                 Burn Share                   Frequency
Server creation           50% of emission              With each new server
Transaction fees          20% of each fee              Every transaction
Block reward              30% of block emission        Every block
Unfulfilled contracts    100% of amount                Upon deadline expiration

6.2 Burn Address

All burned coins are sent to a special burn address:
0000000000000000000000000000000000000000

It is impossible to spend coins from this address. Anyone can check the burn address balance and verify the reality of burning.

6.3 Deflationary Effect

Despite constant emission of new coins (from new servers and blocks), burning creates deflationary pressure:
# Balance of emission and burning
net_emission = new_emission - burned_amount

# With sufficient transactions
# burning may exceed emission
if net_emission < 0:
    print("Deflation: total coin supply is decreasing")
    
7. Server Economics
7.1 Public IP Server (Validator)

Income:

10% of fees from all network transactions

70% of block reward (emission)

Fees for relay services (optional)

Expenses:

Hosting/VPS ($5 to $50/month)

Electricity (if home server)

Equipment depreciation

Break-even point:

With 1000 servers in network: ~6,000 DNET/month (~$600 at $0.1)

With 10,000 servers: ~15,000 DNET/month (~$1,500 at $0.1)

7.2 NAT Server (Initiator)

Income:

70% of fees from transactions initiated by its clients

Expenses:

Electricity (insignificant, PC is often already on)

Depreciation (minimal)

Income with different client counts:

Clients	        Transactions/day         Income/month (DNET)            Income/month (USD at $0.1)
5                             50                  ~105 DNET                     $10.5
10                           100                  ~210 DNET                     $21
20                           200                  ~420 DNET                     $42
50                           500                ~1,050 DNET                     $105
100                         1000                ~2,100 DNET                     $210

7.3 Profitability Comparison

Server Type                    Investment             Monthly Expense      Potential Income         ROI
Validator (VPS)	             $10-50/month                      $10-50            $600-1,500         1200-3000%
Validator (home)                $500 (PC)            $5 (electricity)            $600-1,500         ∞ (after payoff)
Initiator (NAT, 10 clients)       $0 (existing PC)               $2-5             $21               420-1050%
Initiator (NAT, 50 clients)       $0                             $2-5            $105               2100-5250%

8. Client Economics
8.1 Free Basic Level

Feature                                 Cost
Account registration                    Free
Text messages (E2EE)                    Free
Double key (additional phrase)          Free
Offline message storage (up to 30 days) Free
Balance viewing                         Free

8.2 Paid Features (Future Versions)

Feature                         Cost               Rationale
File/photo sending              0.01 DNET/MB	   Traffic compensation
Group chats (>10 participants)	1 DNET/month       Network load
Video calls (P2P)            0.1 DNET/minute       Relay usage
Priority service               10 DNET/month       Premium servers

8.3 Where Clients Get Coins

Buy from miners (via built-in P2P exchange)

Receive as gift from a friend with a server

Earn (e.g., for inviting friends)

Exchange on external markets (after listing)

9. Escrow Smart Contracts
9.1 Escrow Economics

Non-refundable smart contracts create unique economic incentives:

Participant	Incentive
Buyer	Give the phrase, otherwise lose money
Seller	Perform work well, otherwise won't receive the phrase
Both parties	Reach agreement at any cost, otherwise both lose (seeking real win-win)

IMPORTANT: Why do coins burn?

This ensures the key principle of a free market - parties must be equal in contractual relations.

9.2 Contract Fees

Action                          Fee
Contract creation               0.05% of amount (min. 0.1 DNET)
Contract execution              0.01% of amount
Burning upon non-fulfillment	100% of amount

9.3 Calculation Example

Contract amount: 1000 DNET
Creation fee (0.05%): 0.5 DNET
- 70% to initiator (0.35 DNET)
- 10% to validator (0.05 DNET)
- 20% burned (0.1 DNET)

Upon non-fulfillment:
- 1000 DNET burned (deflation)
- Nobody gets anything

10. Inflation and Deflation
10.1 Emission and Burning Balance

# Annual model (at 10,000 servers)
new_servers_per_year = 1000
emission_from_servers = 1000 * 10 * 0.5 = 5000 DNET  # after burning
emission_from_blocks = 365 * 24 * 60 * 6 * 0.01 * 0.7 = ~22,000 DNET  # after burning
total_emission = ~27,000 DNET

transactions_per_day = 100,000
fees_per_day = 100,000 * 0.1 * 0.0002 = 2 DNET
fee_burning_per_year = 2 * 365 * 0.2 = 146 DNET

net_growth = 27,000 - 146 = ~26,854 DNET (inflation ~5% with 500k in circulation)

10.2 Long-term Trend

Period            Circulation          Inflation         Comment
Year 1                 50,000               High         Network growth
Year 2                200,000             Medium         Network growth
Year 3                400,000           Moderate         Stabilization
Year 4+               500,000+     Low/deflation         Mature network

11. Comparison with Other Models
11.1 DuoNet vs Bitcoin

Parameter                   Bitcoin             DuoNet
Max emission             21 million             Unlimited
Inflation      Decreasing (halving)             Controlled by burning
Mining       Validators only (ASIC)             Any PC (NAT included)
Fees                  100% to miner             70% initiator, 10% validator
Burning                          No             20% of all fees

11.2 DuoNet vs Ethereum

Parameter                     Ethereum              DuoNet
Consensus             PoS (since 2022)              Proof-of-Trust
Emission                ~0.5% per year              Depends on network growth
Burning            EIP-1559 (base fee)              20% fees + 50% at creation
Validator income            % of stake              10% fees + 70% block rewards

11.3 DuoNet vs Classical Messengers

Parameter                    Telegram              Signal            DuoNet
Model                      Free, ads?           Donations            Free + economics
Incentives                       None                None            70% fees to initiators
Who pays                  Advertisers              Donors            Premium feature users
Decentralization                   No                  No            Yes

12. Economic FAQ
12.1 Why no "development fund" or "team wallets"?

Answer: DuoNet is built on the principle of honesty. If developers want to earn, they can run servers like everyone else. This creates healthy motivation: the better the network, the more valuable the coin, the more everyone earns, including developers.

12.2 Won't high initial emission lead to inflation?
Answer: No, because:

50% of emission is immediately burned

Early servers take on risk and receive compensation

Upon reaching 10,000 servers, emission drops to 10 DNET (5 after burning)

12.3 Why is burning needed?
Answer: Three reasons:

Compensating emission (inflation control)

Creating long-term coin value

Punishment for unfulfilled contracts (escrow)

12.4 How much can be earned on a home PC?
Answer: With 10 active clients (family, friends) — about 200-300 DNET per month. At $0.1 price, that's $20-30; at $1 price, $200-300.

12.5 What gives value to the coin?
Answer:

Necessity for premium features

Staking (increasing server weight)

Contract fees

Speculative demand (exchange)

Deflationary mechanisms

13. Mathematical Formulas
13.1 Emission

E(n) = ⎧ 1000, n < 100
       ⎪ 800,  100 ≤ n < 200
       ⎪ 500,  200 ≤ n < 500
       ⎪ 300,  500 ≤ n < 1000
       ⎪ 150,  1000 ≤ n < 2000
       ⎪ 75,   2000 ≤ n < 5000
       ⎪ 40,   5000 ≤ n < 7500
       ⎪ 30,   7500 ≤ n < 8000
       ⎪ 20,   8000 ≤ n < 10000
       ⎩ 10,   n ≥ 10000
       
13.2 Real Coins After Burning

R(n) = E(n) × 0.5

13.3 Initiator Income

I = (0.0002 × A) × 0.7
where A — transaction amount

13.4 Validator Income from Transactions

V_tx = Σ(0.0002 × A_i) × 0.1

13.5 Validator Income from Blocks

V_block = 0.01 × 0.7 × blocks_per_day

13.6 Server Weight (Proof-of-Trust)

W = 0.3·log₁₀(B) + 0.2·U + 0.1/(1+V) + 0.2·Tₐ + 0.15·(1-R/5000) + 0.05·min(D/365,1)
where:
B — balance
U — uptime (days/30)
V — violations
Tₐ — activity (tx/10000)
R — response time (ms)
D — age (days)

14. Conclusion
The DuoNet economic model creates a fair and sustainable ecosystem where:

Regular users communicate for free

Home PCs earn real money

Validators receive income for infrastructure

Businesses use unique escrow contracts

Deflation protects against devaluation

Three key principles distinguish DuoNet from all existing projects:

70% fees to initiators (NAT servers) — incentive for mass adoption

50% burning at creation — inflation control

Non-refundable escrow — ideal transaction economics

Document Version: 1.0
Last Updated: March 2026
Author: Alexei Nikolaev
