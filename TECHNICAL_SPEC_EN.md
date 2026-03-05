# DuoNet Technical Specification

## Server and Client Architecture for Decentralized P2P Network

**Version 1.0 | March 2026**

**Author:** Alexei Nikolaev
**Contact:** [@Root_NM](https://t.me/Root_NM) | leha.nikolaev@gmail.com

---

## 1. General Provisions

### 1.1 Purpose of the Document

This document contains a detailed technical description of the DuoNet architecture, interaction protocols, data structures, and algorithms necessary for implementing server and client software.

### 1.2 Key System Features

- **Two types of participants:** servers (miners/validators) and clients (users)
- **Two-level economics:** 70% fee to initiator (NAT server), 10% to validator, 20% burning
- **Progressive emission:** tied to server count + 50% burning at creation
- **Proof-of-Trust consensus:** multidimensional trust metric
- **Unique features:** non-refundable escrow, double key, NAT server incentives

### 1.3 Recommended Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-------------|
| **P2P Transport** | libp2p (Go/Rust) | Ready-made solution for NAT Traversal, DHT, GossipSub |
| **Blockchain Core** | Python (prototype), Go/Rust (prod) | Development speed vs performance |
| **Data Storage** | RocksDB / LevelDB | High performance for blockchain |
| **Cryptography** | libsodium (NaCl) | Audited primitives |
| **Client (TUI)** | Python + Textual | Fast prototyping |
| **Client (GUI)** | PySide6 / Qt | Native KDE integration |
| **Serialization** | Protocol Buffers | Compactness, compatibility |
| **API** | gRPC | Bidirectional streams |

---

## 2. Identification and Cryptography

### 2.1 Key Pair

Each network participant (server or client) has a key pair based on **Ed25519**.

```python
import nacl.signing
import nacl.hash

class Identity:
    def __init__(self, seed=None):
        if seed:
            self.signing_key = nacl.signing.SigningKey(seed)
        else:
            self.signing_key = nacl.signing.SigningKey.generate()
        
        self.verify_key = self.signing_key.verify_key
        self.public_key_bytes = bytes(self.verify_key)
        
        # Participant ID = first 20 bytes of public key hash
        self.id = nacl.hash.sha256(self.public_key_bytes)[:20]
    
    def sign(self, message):
        return self.signing_key.sign(message)
    
    @staticmethod
    def verify(signature, message, public_key):
        verify_key = nacl.signing.VerifyKey(public_key)
        try:
            verify_key.verify(message, signature)
            return True
        except:
            return False
            
2.2 Seed Phrase for Clients (BIP39)
For user convenience, a standard 12-word seed phrase (BIP39) is used.

import hashlib
import hmac
from mnemonic import Mnemonic

class SeedPhrase:
    @staticmethod
    def generate():
        mnemo = Mnemonic("english")
        return mnemo.generate(strength=128)  # 12 words
    
    @staticmethod
    def to_seed(phrase, passphrase=""):
        mnemo = Mnemonic("english")
        return mnemo.to_seed(phrase, passphrase)
    
    @staticmethod
    def to_keypair(seed):
        # HMAC-SHA512 for master key generation
        h = hmac.new(b"DuoNet seed", seed, hashlib.sha512)
        master_key = h.digest()
        return Identity(seed=master_key[:32])
        
2.3 Server Hardware Binding
Servers use hardware fingerprinting to limit multiple server creation by one entity (Sybil attack protection).

import hashlib
import subprocess
import platform

class HardwareFingerprint:
    @staticmethod
    def get_machine_id():
        """Get unique system ID"""
        if platform.system() == "Linux":
            with open("/etc/machine-id", "r") as f:
                return f.read().strip()
        elif platform.system() == "Windows":
            # wmic csproduct get uuid
            result = subprocess.run(
                ["wmic", "csproduct", "get", "uuid"], 
                capture_output=True, text=True
            )
            return result.stdout.split("\n")[2].strip()
        elif platform.system() == "Darwin":  # macOS
            result = subprocess.run(
                ["ioreg", "-rd1", "-c", "IOPlatformExpertDevice"], 
                capture_output=True, text=True
            )
            # parse IOPlatformUUID
            return "mac_uuid_placeholder"
    
    @staticmethod
    def get_disk_serial():
        """System disk serial number"""
        if platform.system() == "Linux":
            result = subprocess.run(
                ["lsblk", "--noheadings", "-o", "SERIAL", "/dev/sda"], 
                capture_output=True, text=True
            )
            return result.stdout.strip()
        # similar for other OS
    
    @staticmethod
    def generate_fingerprint():
        machine_id = HardwareFingerprint.get_machine_id()
        disk_serial = HardwareFingerprint.get_disk_serial()
        
        # Combine with salt
        salt = b"DuoNet Hardware Salt v1"
        data = f"{machine_id}:{disk_serial}".encode() + salt
        
        return hashlib.sha256(data).hexdigest()
        
2.4 Participant Types and Honesty Modes

class ParticipantType(Enum):
    SERVER_PUBLIC_IP = 1   # Server with public IP (validator)
    SERVER_NAT = 2         # Server behind NAT (initiator)
    CLIENT_HONEST = 3      # Client knowing device type
    CLIENT_UNSURE = 4      # Client unsure of type ("honest ignorance")
    
3. Transactions and Blocks
3.1 Transaction Format (Protocol Buffers)

syntax = "proto3";

package duonet;

enum TransactionType {
  TRANSFER = 0;           // Regular transfer
  SERVER_REGISTER = 1;    // New server registration
  SERVER_RENEW = 2;       // Renewal/return after penalty
  CONTRACT_CREATE = 3;    // Smart contract creation (escrow)
  CONTRACT_RELEASE = 4;   // Contract execution (with phrase)
  CONTRACT_EXPIRE = 5;    // Burning after deadline
  STAKE_UPDATE = 6;       // Stake change
  MIGRATION = 7;          // Wallet migration to new hardware
}

message Transaction {
  uint32 version = 1;
  TransactionType type = 2;
  bytes from = 3;              // Sender address (20 bytes)
  bytes to = 4;                // Recipient address (20 bytes)
  uint64 amount = 5;           // Amount in satoshi (1 DNET = 10^8)
  uint64 fee = 6;              // Fee
  uint64 nonce = 7;            // Sender counter (replay protection)
  uint64 timestamp = 8;        // Creation time (Unix timestamp)
  bytes initiator = 9;         // Server ID that initiated the transaction
  bytes data = 10;             // Additional data (depends on type)
  bytes signature = 11;        // Sender signature
}

// Server registration data
message ServerRegisterData {
  string hardware_fingerprint = 1;
  ParticipantType participant_type = 2;
  repeated string multiaddrs = 3;  // libp2p addresses
}

// Escrow contract data
message EscrowData {
  bytes seller = 1;              // Seller address
  bytes secret_hash = 2;         // SHA256 of phrase
  uint64 deadline = 3;           // Expiration deadline
  bytes contract_id = 4;         // Contract ID (hash of parameters)
}

// Contract execution data
message EscrowReleaseData {
  bytes contract_id = 1;
  bytes secret = 2;              // Phrase (not hash!)
}

3.2 Block Format

message Block {
  uint32 version = 1;
  uint64 height = 2;              // Block number
  bytes previous_hash = 3;        // Previous block hash
  bytes merkle_root = 4;          // Merkle tree root
  repeated Transaction transactions = 5;
  uint64 timestamp = 6;           // Creation time
  bytes validator = 7;             // Validator address
  bytes signature = 8;            // Validator signature
  
  message Approval {
    bytes server = 1;
    bytes signature = 2;
    uint64 weight = 3;            // Server weight at signature time
  }
  repeated Approval approvals = 9;  // Confirmations from other servers
  
  uint64 total_weight = 10;        // Total weight of all signers
  bytes state_root = 11;           // State hash after block
}

3.3 Nonce and Replay Protection
Each account maintains a counter of outgoing transactions (nonce). A transaction with nonce N is accepted only if the last accepted transaction from this account had nonce N-1.

class NonceManager:
    def __init__(self):
        self.nonce_cache = {}  # address -> last_nonce
    
    def validate_nonce(self, address, nonce):
        last_nonce = self.nonce_cache.get(address, -1)
        if nonce != last_nonce + 1:
            return False, f"Expected nonce {last_nonce + 1}, got {nonce}"
        return True, None
    
    def commit_nonce(self, address, nonce):
        self.nonce_cache[address] = nonce
        
4. Economic Model (Implementation Details)
4.1 Emission Parameters

class EmissionCalculator:
    # Progressive scale: (lower_bound, upper_bound, emission)
    EMISSION_SCHEDULE = [
        (1, 100, 1000),
        (101, 200, 800),
        (201, 500, 500),
        (501, 1000, 300),
        (1001, 2000, 150),
        (2001, 5000, 75),
        (5001, 7500, 40),
        (7501, 8000, 30),
        (8001, 10000, 20),
        (10001, float('inf'), 10)
    ]
    
    BURN_RATE = 0.5  # 50% burned at creation
    
    @classmethod
    def get_emission(cls, server_count):
        """Get emission for new server"""
        for lower, upper, amount in cls.EMISSION_SCHEDULE:
            if lower <= server_count + 1 <= upper:
                return amount
        return 10  # Fallback
    
    @classmethod
    def create_server_transaction(cls, server_address, server_count, fingerprint):
        """Create server registration transaction"""
        emission = cls.get_emission(server_count)
        burn_amount = int(emission * cls.BURN_RATE * 10**8)  # in satoshi
        deposit_amount = int(emission * (1 - cls.BURN_RATE) * 10**8)
        
        # Create special transaction
        tx = Transaction(
            type=TransactionType.SERVER_REGISTER,
            from=b"00000000000000000000000000000000",  # zero address (emission)
            to=server_address,
            amount=deposit_amount,
            fee=0,  # no fee
            nonce=0,
            timestamp=current_time(),
            data=ServerRegisterData(
                hardware_fingerprint=fingerprint
            ).SerializeToString()
        )
        return tx, burn_amount
        
4.2 Fee Distribution

class FeeDistributor:
    INITIATOR_SHARE = 0.70    # 70% to initiator
    VALIDATOR_SHARE = 0.10    # 10% to validator
    BURN_SHARE = 0.20         # 20% burning
    
    @classmethod
    def distribute(cls, transaction, block_validator):
        """Distribute transaction fee"""
        fee = transaction.fee
        
        initiator_reward = int(fee * cls.INITIATOR_SHARE)
        validator_reward = int(fee * cls.VALIDATOR_SHARE)
        burn_amount = fee - initiator_reward - validator_reward
        
        rewards = {
            'initiator': (transaction.initiator, initiator_reward),
            'validator': (block_validator, validator_reward),
            'burn': (BURN_ADDRESS, burn_amount)
        }
        
        return rewards
    
    @classmethod
    def distribute_block_reward(cls, validator, block_height):
        """Distribute block reward (emission)"""
        BLOCK_REWARD = 1_000_000  # 0.01 DNET in satoshi
        
        validator_reward = int(BLOCK_REWARD * 0.7)  # 70% to validator
        burn_amount = BLOCK_REWARD - validator_reward  # 30% burning
        
        return {
            'validator': (validator, validator_reward),
            'burn': (BURN_ADDRESS, burn_amount)
        }
        
4.3 Server Weight Calculation (Proof-of-Trust)

class TrustScoreCalculator:
    # Factor weights
    WEIGHTS = {
        'balance': 0.30,
        'uptime': 0.20,
        'purity': 0.10,
        'activity': 0.20,
        'speed': 0.15,
        'tenure': 0.05
    }
    
    @classmethod
    def calculate(cls, server):
        """Calculate server weight"""
        score = 0
        
        # 1. Balance (logarithmic scale)
        max_balance = 1_000_000 * 10**8  # 1 million DNET in satoshi
        balance_score = min(
            math.log10(server.balance + 1) / math.log10(max_balance),
            1.0
        )
        score += balance_score * cls.WEIGHTS['balance']
        
        # 2. Uptime (percentage online over 30 days)
        uptime_score = server.uptime_30d / 30.0
        score += min(uptime_score, 1.0) * cls.WEIGHTS['uptime']
        
        # 3. Purity (absence of penalties)
        purity_score = 1.0 / (1.0 + server.violations)
        score += purity_score * cls.WEIGHTS['purity']
        
        # 4. Activity (transaction count)
        max_tx = 10000  # maximum per period
        activity_score = min(server.transactions_count / max_tx, 1.0)
        score += activity_score * cls.WEIGHTS['activity']
        
        # 5. Response speed
        # response_time in milliseconds, 5000ms = 0
        speed_score = max(0, 1.0 - server.avg_response_time / 5000)
        score += speed_score * cls.WEIGHTS['speed']
        
        # 6. Tenure (age in days)
        tenure_score = min(server.age_days / 365, 1.0)
        score += tenure_score * cls.WEIGHTS['tenure']
        
        return score
        
5. Network and Transport (libp2p)
5.1 Basic Node Setup

from libp2p import new_node
from libp2p.crypto.keys import KeyPair
from libp2p.peer.id import ID

class DuoNetNode:
    def __init__(self, identity, is_public_ip=False):
        self.identity = identity
        self.is_public_ip = is_public_ip
        self.peer_id = ID.from_pubkey(identity.verify_key)
        
        # Create libp2p node
        self.node = new_node(
            key_pair=KeyPair(identity.signing_key, identity.verify_key),
            listen_addrs=self._get_listen_addresses()
        )
        
        # Setup protocols
        self._setup_protocols()
        
        # Clients connected to this server
        self.clients = {}  # client_id -> connection
        
        # Neighboring servers
        self.peers = {}    # peer_id -> connection
    
    def _get_listen_addresses(self):
        if self.is_public_ip:
            return ["/ip4/0.0.0.0/tcp/9876"]
        else:
            # NAT server doesn't listen for incoming
            return []
            
5.2 libp2p Protocols

def _setup_protocols(self):
    # 1. GossipSub for transaction and block propagation
    self.gossip = self.node.get_pubsub()
    self.gossip.subscribe("duonet-transactions", self._handle_transaction)
    self.gossip.subscribe("duonet-blocks", self._handle_block)
    
    # 2. DHT for peer discovery
    self.dht = self.node.get_kademlia()
    
    # 3. RPC protocol for direct messages
    self.node.set_stream_handler("/duonet/1.0.0", self._handle_stream)
    
5.3 NAT Traversal and Hole Punching

class NATTraversal:
    def __init__(self, node, rendezvous_servers):
        self.node = node
        self.rendezvous_servers = rendezvous_servers
        
    async def hole_punch(self, target_peer_id):
        """Attempt direct connection via Hole Punching"""
        # 1. Request target addresses from rendezvous server
        target_addrs = await self._discover_peer(target_peer_id)
        
        # 2. Send signal to target via rendezvous
        await self._signal_peer(target_peer_id, self.node.get_addrs())
        
        # 3. Start simultaneous connections
        for addr in target_addrs:
            try:
                conn = await self.node.connect(addr, timeout=2)
                return conn
            except:
                continue
        
        return None
    
    async def _discover_peer(self, peer_id):
        """Find peer addresses via DHT and rendezvous"""
        addrs = []
        
        # Search in DHT
        dht_addrs = await self.dht.find_peer(peer_id)
        addrs.extend(dht_addrs)
        
        # Search via rendezvous servers
        for rv in self.rendezvous_servers:
            rv_addrs = await rv.lookup(peer_id)
            addrs.extend(rv_addrs)
        
        return list(set(addrs))
        
5.4 Relay Transmission
For NAT servers that cannot establish direct connections:

class RelayService:
    def __init__(self, node):
        self.node = node
        self.relay_connections = {}  # peer_id -> connection
        
    async def relay_message(self, from_peer, to_peer, message):
        """Forward message via relay"""
        if to_peer in self.relay_connections:
            # Direct forwarding
            await self.relay_connections[to_peer].write(message)
        else:
            # Store for later delivery
            self._store_offline(to_peer, message)
    
    def _store_offline(self, peer_id, message):
        """Store offline message (encrypted)"""
        # In real implementation - store in database
        pass
        
6. Rendezvous Server
6.1 Rendezvous Server API (gRPC)

service Rendezvous {
  // Server registration
  rpc RegisterServer(ServerInfo) returns (RegistrationResponse);
  
  // Heartbeat (activity confirmation)
  rpc Heartbeat(HeartbeatRequest) returns (HeartbeatResponse);
  
  // Find server by ID
  rpc LookupPeer(LookupRequest) returns (LookupResponse);
  
  // Get list of active servers
  rpc GetServers(GetServersRequest) returns (stream ServerInfo);
  
  // Signaling for Hole Punching
  rpc SignalHolePunch(SignalRequest) returns (SignalResponse);
}

message ServerInfo {
  bytes server_id = 1;
  repeated string multiaddrs = 2;
  string region = 3;
  uint32 capacity = 4;
  uint32 current_load = 5;
  bool accepting_clients = 6;
  uint64 timestamp = 7;
  bytes signature = 8;  // Signature of server_id + timestamp
}

message LookupRequest {
  bytes peer_id = 1;
}

message LookupResponse {
  bool online = 1;
  repeated string multiaddrs = 2;
  uint64 last_seen = 3;
}

message SignalRequest {
  bytes from_peer = 1;
  bytes to_peer = 2;
  repeated string from_addrs = 3;
}

6.2 Rendezvous Server Implementation

import asyncio
import time
from collections import OrderedDict

class RendezvousServer:
    def __init__(self, cleanup_interval=300):
        self.servers = {}  # server_id -> ServerInfo
        self.last_seen = {}  # server_id -> timestamp
        self.cleanup_interval = cleanup_interval
        self.pending_signals = {}  # signal_key -> signal_data
        
        # Start background cleanup
        asyncio.create_task(self._cleanup_loop())
    
    async def register_server(self, request):
        """Register server"""
        server_id = request.server_id
        
        # Verify signature
        if not self._verify_signature(request):
            return RegistrationResponse(success=False, error="Invalid signature")
        
        # Save information
        self.servers[server_id] = request
        self.last_seen[server_id] = time.time()
        
        # Return list of other servers (up to 50)
        other_servers = [
            s for sid, s in self.servers.items() 
            if sid != server_id and time.time() - self.last_seen[sid] < 300
        ][:50]
        
        return RegistrationResponse(
            success=True,
            servers=other_servers
        )
    
    async def heartbeat(self, request):
        """Update server status"""
        server_id = request.server_id
        if server_id in self.servers:
            self.last_seen[server_id] = time.time()
            return HeartbeatResponse(success=True)
        return HeartbeatResponse(success=False, error="Unknown server")
    
    async def lookup_peer(self, request):
        """Find peer"""
        peer_id = request.peer_id
        if peer_id in self.servers:
            last = self.last_seen.get(peer_id, 0)
            online = (time.time() - last) < 180  # 3 minutes
            
            return LookupResponse(
                online=online,
                multiaddrs=self.servers[peer_id].multiaddrs if online else [],
                last_seen=last
            )
        
        return LookupResponse(online=False)
    
    async def signal_hole_punch(self, request):
        """Signaling for Hole Punching"""
        signal_key = f"{request.from_peer}:{request.to_peer}"
        self.pending_signals[signal_key] = {
            'from_addrs': request.from_addrs,
            'timestamp': time.time()
        }
        
        return SignalResponse(success=True)
    
    async def _cleanup_loop(self):
        """Background cleanup of inactive servers"""
        while True:
            await asyncio.sleep(self.cleanup_interval)
            now = time.time()
            to_remove = [
                sid for sid, last in self.last_seen.items()
                if now - last > 600  # 10 minutes inactive
            ]
            for sid in to_remove:
                del self.servers[sid]
                del self.last_seen[sid]
                
7. Client-Server Interaction
7.1 Client-to-Server API (gRPC)

service ClientService {
  // Authentication
  rpc Auth(ClientAuth) returns (SessionToken);
  
  // Send message
  rpc SendMessage(OutboundMessage) returns (MessageReceipt);
  
  // Receive messages (streaming)
  rpc PollMessages(PollRequest) returns (stream InboundMessage);
  
  // Offline messages
  rpc StoreOffline(OfflineMessage) returns (Status);
  rpc FetchOffline(Empty) returns (stream OfflineMessage);
  
  // Wallet
  rpc GetBalance(Empty) returns (Balance);
  rpc BroadcastTx(SignedTransaction) returns (TxHash);
  rpc GetTransactionHistory(HistoryRequest) returns (stream Transaction);
  
  // Contacts
  rpc ResolveContact(ContactID) returns (ContactInfo);
}

message ClientAuth {
  bytes client_id = 1;
  uint64 timestamp = 2;
  bytes signature = 3;  // Signature of client_id + timestamp
}

message OutboundMessage {
  bytes to = 1;
  bytes encrypted_payload = 2;
  uint64 timestamp = 3;
  bool store_offline = 4;
}

message InboundMessage {
  bytes from = 1;
  bytes encrypted_payload = 2;
  uint64 timestamp = 3;
  uint64 message_id = 4;
}

7.2 Client Connection Implementation

class DuoNetClient:
    def __init__(self, identity, server_selector):
        self.identity = identity
        self.server_selector = server_selector
        self.current_server = None
        self.session_token = None
        self.message_queue = asyncio.Queue()
        self.last_message_id = 0
    
    async def connect(self):
        """Connect to selected server"""
        server = await self.server_selector.select()
        self.current_server = server
        
        # Authentication
        timestamp = int(time.time())
        auth = ClientAuth(
            client_id=self.identity.id,
            timestamp=timestamp,
            signature=self.identity.sign(self.identity.id + str(timestamp))
        )
        
        response = await server.stub.Auth(auth)
        self.session_token = response.token
        
        # Start message polling
        asyncio.create_task(self._poll_messages())
        
        return True
    
    async def send_message(self, to_id, encrypted_payload):
        """Send message"""
        msg = OutboundMessage(
            to=to_id,
            encrypted_payload=encrypted_payload,
            timestamp=int(time.time()),
            store_offline=True
        )
        
        try:
            receipt = await self.current_server.stub.SendMessage(
                msg, metadata=[('token', self.session_token)]
            )
            return receipt
        except Exception as e:
            # If server unavailable - reconnect
            await self._reconnect()
            return await self.send_message(to_id, encrypted_payload)
    
    async def _poll_messages(self):
        """Long poll for receiving messages"""
        while True:
            try:
                request = PollRequest(last_id=self.last_message_id)
                async for msg in self.current_server.stub.PollMessages(
                    request, metadata=[('token', self.session_token)]
                ):
                    await self.message_queue.put(msg)
                    self.last_message_id = msg.message_id
            except Exception as e:
                # Connection lost, wait and reconnect
                await asyncio.sleep(5)
                await self._reconnect()
                
7.3 Server Selection by Client (4 Modes)

class ServerSelector:
    def __init__(self, rendezvous_client):
        self.rendezvous = rendezvous_client
        self.mode = 'auto'  # auto, geo, premium, manual
        self.preferred_servers = []
    
    async def select(self):
        if self.mode == 'auto':
            return await self._auto_select()
        elif self.mode == 'geo':
            return await self._geo_select()
        elif self.mode == 'premium':
            return await self._premium_select()
        elif self.mode == 'manual':
            return self.preferred_servers[0] if self.preferred_servers else None
    
    async def _auto_select(self):
        # 1. Check local network (multicast DNS)
        local_server = await self._discover_local()
        if local_server:
            return local_server
        
        # 2. Ask rendezvous
        servers = await self.rendezvous.get_servers(limit=20)
        
        # 3. Ping and choose fastest
        for server in servers:
            latency = await self._ping(server)
            if latency < 100:  # less than 100 ms
                return server
        
        return servers[0] if servers else None
    
    async def _geo_select(self):
        # Determine region (by IP or GPS)
        region = await self._get_region()
        
        # Request servers in this region
        servers = await self.rendezvous.get_servers(region=region)
        
        # Sort by load
        servers.sort(key=lambda s: s.current_load)
        return servers[0] if servers else None
    
    async def _premium_select(self):
        # For future versions with subscriptions
        pass
    
    async def _discover_local(self):
        """Find server on local network via mDNS"""
        # Use libp2p mDNS discovery
        pass
    
    async def _ping(self, server):
        """Measure latency to server"""
        # Simple ping implementation
        return 50  # mock value
    
    async def _get_region(self):
        """Get client region"""
        return "eu"  # mock value
        
8. Smart Contracts (Escrow)
8.1 Contract Structure

import hashlib
import time

class EscrowContract:
    def __init__(self, contract_id, buyer, seller, amount, secret_hash, deadline):
        self.contract_id = contract_id
        self.buyer = buyer
        self.seller = seller
        self.amount = amount
        self.secret_hash = secret_hash  # SHA256
        self.deadline = deadline
        self.released = False
        self.expired = False
    
    @classmethod
    def create(cls, buyer, seller, amount, secret, days=30):
        """Create new contract"""
        secret_hash = hashlib.sha256(secret).digest()
        deadline = int(time.time()) + days * 86400
        
        # Contract ID = hash of parameters
        contract_data = f"{buyer}{seller}{amount}{secret_hash}{deadline}"
        contract_id = hashlib.sha256(contract_data.encode()).digest()
        
        return cls(contract_id, buyer, seller, amount, secret_hash, deadline)
    
    def release(self, secret, seller_key):
        """Execute contract (seller presents phrase)"""
        if self.released or self.expired:
            return False, "Contract already finalized"
        
        if time.time() > self.deadline:
            self.expired = True
            return False, "Contract expired"
        
        # Verify phrase
        if hashlib.sha256(secret).digest() != self.secret_hash:
            return False, "Invalid secret"
        
        # Verify caller is seller
        if seller_key != self.seller:
            return False, "Only seller can release"
        
        self.released = True
        return True, "Contract released"
    
    def expire(self):
        """Burn funds after deadline"""
        if not self.released and time.time() > self.deadline:
            self.expired = True
            return True
        return False
        
8.2 VM for Contract Execution (Simplified)

class ContractVM:
    def __init__(self):
        self.contracts = {}  # contract_id -> EscrowContract
    
    def execute_transaction(self, tx):
        """Execute contract-related transaction"""
        if tx.type == TransactionType.CONTRACT_CREATE:
            return self._create_contract(tx)
        elif tx.type == TransactionType.CONTRACT_RELEASE:
            return self._release_contract(tx)
        elif tx.type == TransactionType.CONTRACT_EXPIRE:
            return self._expire_contract(tx)
        return False, "Unknown transaction type"
    
    def _create_contract(self, tx):
        data = EscrowData()
        data.ParseFromString(tx.data)
        
        contract = EscrowContract(
            contract_id=tx.hash,  # ID = transaction hash
            buyer=tx.from,
            seller=data.seller,
            amount=tx.amount,
            secret_hash=data.secret_hash,
            deadline=data.deadline
        )
        
        self.contracts[tx.hash] = contract
        
        # Lock funds at contract address
        # In blockchain state: move tx.amount to special address
        
        return True, "Contract created"
    
    def _release_contract(self, tx):
        data = EscrowReleaseData()
        data.ParseFromString(tx.data)
        
        contract = self.contracts.get(data.contract_id)
        if not contract:
            return False, "Contract not found"
        
        success, message = contract.release(data.secret, tx.from)
        if success:
            # Transfer funds to seller
            # In blockchain state: move from contract address to tx.from
            pass
        
        return success, message
    
    def _expire_contract(self, tx):
        contract = self.contracts.get(tx.data)
        if not contract:
            return False, "Contract not found"
        
        if contract.expire():
            # Burn funds
            # In blockchain state: send to burn address
            return True, "Contract expired, funds burned"
        
        return False, "Contract cannot be expired"
        
9. Message Encryption (Double Key)
9.1 Base Double Ratchet Protocol

import os
from cryptography.hazmat.primitives.ciphers.aead import ChaCha20Poly1305

class DoubleRatchet:
    def __init__(self, shared_secret):
        self.root_key = shared_secret
        self.send_chain_key = self._derive_chain(self.root_key, b"send")
        self.recv_chain_key = self._derive_chain(self.root_key, b"recv")
        self.send_counter = 0
        self.recv_counter = 0
    
    def _derive_chain(self, key, label):
        """Derive new chain key"""
        # Simplified KDF
        return hashlib.sha256(key + label).digest()
    
    def _get_message_key(self, counter):
        """Get message key for specific counter"""
        # Simplified - in real implementation, store all keys
        return self.send_chain_key
    
    def ratchet_step(self):
        """Update keys (after each message)"""
        self.send_chain_key = self._derive_chain(self.send_chain_key, b"next")
        self.send_counter += 1
    
    def encrypt(self, plaintext, additional_key=None):
        """Encrypt message"""
        # Mix with additional key if present
        message_key = self.send_chain_key
        if additional_key:
            message_key = self._xor_keys(message_key, additional_key)
        
        # Encrypt
        nonce = os.urandom(12)
        cipher = ChaCha20Poly1305(message_key)
        ciphertext = cipher.encrypt(nonce, plaintext, None)
        
        self.ratchet_step()
        
        return {
            'nonce': nonce,
            'ciphertext': ciphertext,
            'counter': self.send_counter - 1
        }
    
    def decrypt(self, packet, additional_key=None):
        """Decrypt message"""
        # Restore key for this message
        message_key = self._get_message_key(packet['counter'])
        if additional_key:
            message_key = self._xor_keys(message_key, additional_key)
        
        cipher = ChaCha20Poly1305(message_key)
        
        try:
            plaintext = cipher.decrypt(
                packet['nonce'],
                packet['ciphertext'],
                None
            )
            return plaintext
        except:
            return None  # Wrong key
    
    def _xor_keys(self, key1, key2):
        """XOR two keys"""
        return bytes(a ^ b for a, b in zip(key1, key2))
        
9.2 Additional Phrase (Double Key)

import hashlib
import hmac
from argon2 import PasswordHasher

class DoubleKeyEncryption:
    @staticmethod
    def derive_phrase_key(phrase, salt):
        """Derive key from phrase (Argon2)"""
        ph = PasswordHasher(
            time_cost=3,
            memory_cost=64*1024,  # 64 MB
            hash_len=32
        )
        # Argon2 returns encoded string with parameters, extract hash
        hash_str = ph.hash(phrase, salt=salt)
        # In real implementation - proper extraction
        return hashlib.sha256(phrase.encode() + salt).digest()
    
    @staticmethod
    def encrypt_message(plaintext, session_key, phrase):
        """Encrypt with additional phrase"""
        salt = os.urandom(16)
        phrase_key = DoubleKeyEncryption.derive_phrase_key(phrase, salt)
        
        # Mix keys
        combined_key = DoubleRatchet._xor_keys(session_key, phrase_key)
        
        nonce = os.urandom(12)
        cipher = ChaCha20Poly1305(combined_key)
        ciphertext = cipher.encrypt(nonce, plaintext, None)
        
        # Format: salt (16) + nonce (12) + ciphertext
        return salt + nonce + ciphertext
    
    @staticmethod
    def decrypt_message(packet, session_key, phrase):
        """Decrypt with phrase verification"""
        salt = packet[:16]
        nonce = packet[16:28]
        ciphertext = packet[28:]
        
        phrase_key = DoubleKeyEncryption.derive_phrase_key(phrase, salt)
        combined_key = DoubleRatchet._xor_keys(session_key, phrase_key)
        
        cipher = ChaCha20Poly1305(combined_key)
        
        try:
            plaintext = cipher.decrypt(nonce, ciphertext, None)
            return plaintext
        except:
            return None  # Wrong phrase
            
9.3 Chat Session Management

class ChatSession:
    def __init__(self, peer_id, peer_public_key, phrase=None):
        self.peer_id = peer_id
        self.peer_key = peer_public_key
        self.phrase_hash = hashlib.sha256(phrase).digest() if phrase else None
        
        # X3DH for initial key establishment
        self.ratchet = self._establish_session(peer_public_key)
        
        self.message_counter = 0
    
    def _establish_session(self, peer_public_key):
        """Establish initial session keys (simplified X3DH)"""
        # In real implementation - proper X3DH
        shared_secret = os.urandom(32)
        return DoubleRatchet(shared_secret)
    
    def send_message(self, plaintext):
        """Send message"""
        if self.phrase_hash:
            # Additional protection: include phrase hash in packet
            # (not the phrase itself!)
            check = hmac.new(
                self.phrase_hash, 
                str(self.message_counter).encode(),
                hashlib.sha256
            ).digest()
        else:
            check = None
        
        encrypted = self.ratchet.encrypt(plaintext)
        
        packet = {
            'encrypted': encrypted,
            'phrase_check': check,
            'counter': self.message_counter
        }
        
        self.message_counter += 1
        return packet
    
    def receive_message(self, packet):
        """Receive message"""
        if self.phrase_hash:
            # Verify phrase (without revealing it)
            expected = hmac.new(
                self.phrase_hash,
                str(packet['counter']).encode(),
                hashlib.sha256
            ).digest()
            
            if not hmac.compare_digest(expected, packet['phrase_check']):
                return None  # Wrong phrase
        
        # Decrypt
        return self.ratchet.decrypt(packet['encrypted'])
        
10. Data Storage
10.1 Server Database Structure (RocksDB)

// Keys in RocksDB
// "block:" + height -> Block
// "tx:" + tx_hash -> Transaction
// "balance:" + address -> uint64
// "nonce:" + address -> uint64
// "server:" + server_id -> ServerInfo
// "contract:" + contract_id -> EscrowContract
// "offline:" + client_id + timestamp -> OfflineMessage

10.2 Client Database Structure (SQLCipher)

-- Master key for DB = hash of user password
PRAGMA key = 'user_password_hash';

CREATE TABLE contacts (
    id INTEGER PRIMARY KEY,
    peer_id TEXT UNIQUE NOT NULL,
    nickname TEXT,
    public_key BLOB NOT NULL,
    phrase_hash BLOB,  -- hash of additional phrase (optional)
    last_seen INTEGER,
    created_at INTEGER
);

CREATE TABLE conversations (
    id INTEGER PRIMARY KEY,
    contact_id INTEGER NOT NULL,
    FOREIGN KEY(contact_id) REFERENCES contacts(id)
);

CREATE TABLE messages (
    id INTEGER PRIMARY KEY,
    conversation_id INTEGER NOT NULL,
    direction TEXT CHECK(direction IN ('in', 'out')),
    encrypted_payload BLOB NOT NULL,
    timestamp INTEGER NOT NULL,
    is_delivered BOOLEAN DEFAULT 0,
    is_read BOOLEAN DEFAULT 0,
    is_deleted BOOLEAN DEFAULT 0,
    FOREIGN KEY(conversation_id) REFERENCES conversations(id)
);

CREATE TABLE transactions (
    tx_hash TEXT PRIMARY KEY,
    type TEXT NOT NULL,
    from_address TEXT NOT NULL,
    to_address TEXT NOT NULL,
    amount INTEGER NOT NULL,
    fee INTEGER NOT NULL,
    status TEXT CHECK(status IN ('pending', 'confirmed', 'failed')),
    timestamp INTEGER NOT NULL,
    memo BLOB  -- encrypted memo
);

CREATE TABLE contracts (
    contract_id TEXT PRIMARY KEY,
    buyer TEXT NOT NULL,
    seller TEXT NOT NULL,
    amount INTEGER NOT NULL,
    secret_hash BLOB NOT NULL,
    deadline INTEGER NOT NULL,
    status TEXT CHECK(status IN ('active', 'released', 'expired')),
    created_at INTEGER NOT NULL
);

11. Penalties and Security
11.1 Violation Types

from enum import Enum

class ViolationType(Enum):
    INACTIVITY = 1      # Inactivity >3 days
    DOUBLE_SIGN = 2     # Double block signing
    INVALID_BLOCK = 3   # Proposing invalid block
    SPAM = 4            # Excessive transactions
    FALSE_HARDWARE = 5  # Lying about hardware type
    
11.2 Penalty System

class PenaltySystem:
    PENALTIES = {
        ViolationType.INACTIVITY: {
            1: (0.1 * 10**8, False),  # fine, confiscation?
            2: (1.0 * 10**8, False),
            3: (0, True)  # confiscation
        },
        ViolationType.DOUBLE_SIGN: {
            1: (0.1 * 10**8, False),
            2: (1.0 * 10**8, False),
            3: (0, True)
        },
        ViolationType.INVALID_BLOCK: {
            1: (0.1 * 10**8, False),
            2: (1.0 * 10**8, False),
            3: (0, True)
        },
        ViolationType.SPAM: {
            1: (0.1 * 10**8, False),
            2: (1.0 * 10**8, False),
            3: (0, True)
        },
        ViolationType.FALSE_HARDWARE: {
            1: (0, True)  # immediate confiscation
        }
    }
    
    def __init__(self, state):
        self.state = state
        self.violation_counts = {}  # server_id -> {type: count}
    
    def report_violation(self, server_id, violation_type):
        """Report violation"""
        counts = self.violation_counts.get(server_id, {})
        count = counts.get(violation_type, 0) + 1
        counts[violation_type] = count
        self.violation_counts[server_id] = counts
        
        # Get penalty
        penalty, confiscate = self.PENALTIES[violation_type][count]
        
        if confiscate:
            # Full deposit confiscation
            balance = self.state.get_balance(server_id)
            self.state.burn(server_id, balance)
            self.state.deactivate_server(server_id)
        else:
            # Fine
            self.state.burn(server_id, penalty)
        
        return penalty, confiscate
        
12. Installation and Configuration
12.1 Minimum Requirements
Server:

CPU: 2 cores, 2 GHz

RAM: 4 GB

Disk: 100 GB SSD

OS: Linux (Ubuntu 22.04+ recommended)

Client:

CPU: 1 core, 1 GHz

RAM: 512 MB

Disk: 1 GB

OS: Linux, Windows 10+, macOS 11+

12.2 Server Configuration Example (YAML)

# config.yaml
node:
  identity_path: "/var/lib/duonet/identity.key"
  participant_type: "server_public_ip"  # or server_nat
  listen_addrs:
    - "/ip4/0.0.0.0/tcp/9876"
    - "/ip4/0.0.0.0/udp/9876/quic"
  
rendezvous:
  enabled: true
  bootstrap_servers:
    - "/ip4/95.217.0.1/tcp/9876/p2p/Qm..."
    - "/ip4/95.217.0.2/tcp/9876/p2p/Qm..."
  
blockchain:
  data_dir: "/var/lib/duonet/data"
  max_peers: 50
  min_peers: 5
  
logging:
  level: "info"
  file: "/var/log/duonet/node.log"
  
12.3 Client Configuration Example (YAML)

# client_config.yaml
identity:
  seed_phrase: "word1 word2 word3 ..."  # or path to file
  password: "user_password"  # for local DB decryption

server_selection:
  mode: "auto"  # auto, geo, premium, manual
  preferred_servers: []  # for manual mode

storage:
  db_path: "~/.duonet/client.db"
  message_retention_days: 30

ui:
  theme: "dark"
  language: "en"
  
13. Testing and Debugging
13.1 Test Network (Testnet)

# testnet_config.py
TESTNET_PARAMS = {
    'block_time': 2,  # 2 seconds for fast testing
    'emission_schedule': [
        (1, 10, 10000),  # accelerated emission for tests
    ],
    'min_peers': 2,
    'consensus_threshold': 0.6,  # 60% for tests
    'rendezvous_servers': [
        "/ip4/127.0.0.1/tcp/9876/p2p/QmTest1"
    ]
}

13.2 Debugging Tools

# Check connection to server
duonet-cli ping /ip4/95.217.0.1/tcp/9876

# View balance
duonet-cli balance

# Send test transaction
duonet-cli send --to 7f3a...b8e2 --amount 10 --memo "test"

# View real-time logs
duonet-node --log-level debug

14. Conclusion
This technical specification contains all necessary details for implementing DuoNet:

Cryptography: Ed25519, Double Ratchet, Argon2, ChaCha20-Poly1305

Network: libp2p, DHT, GossipSub, NAT Traversal

Blockchain: transactions, blocks, Proof-of-Trust consensus

Economics: progressive emission, 70/10/20 distribution

Smart Contracts: non-refundable escrow

Client: 4 server selection modes, double key

Storage: RocksDB (server), SQLCipher (client)

The project is ready for implementation by a development team.

Document Version: 1.0
Last Updated: March 2026
Author: Alexei Nikolaev
