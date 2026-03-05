# DuoNet Техническая спецификация

## Серверная и клиентская архитектура децентрализованной P2P-сети

**Версия 1.0 | Март 2026**

**Автор:** Алексей Николаев
**Контакт:** [@Root_NM](https://t.me/Root_NM) | leha.nikolaev@gmail.com

---

## 1. Общие положения

### 1.1 Назначение документа

Настоящий документ содержит детальное техническое описание архитектуры DuoNet, протоколов взаимодействия, структур данных и алгоритмов, необходимых для реализации серверного и клиентского программного обеспечения.

### 1.2 Ключевые особенности системы

- **Два типа участников:** серверы (майнеры/валидаторы) и клиенты (пользователи)
- **Двухуровневая экономика:** 70% комиссии инициатору (NAT-сервер), 10% валидатору, 20% сжигание
- **Прогрессивная эмиссия:** привязка к количеству серверов + 50% сжигания при создании
- **Консенсус Proof-of-Trust:** многомерная метрика доверия
- **Уникальные фишки:** escrow с невозвратом, двойной ключ, мотивация NAT-серверов

### 1.3 Технологический стек (рекомендуемый)

| Компонент | Технология | Обоснование |
|-----------|------------|-------------|
| **P2P-транспорт** | libp2p (Go/Rust) | Готовое решение для NAT Traversal, DHT, GossipSub |
| **Блокчейн-ядро** | Python (прототип), Go/Rust (prod) | Скорость разработки vs производительность |
| **Хранение данных** | RocksDB / LevelDB | Высокая производительность для блокчейна |
| **Криптография** | libsodium (NaCl) | Аудированные примитивы |
| **Клиент (TUI)** | Python + Textual | Быстрый прототип |
| **Клиент (GUI)** | PySide6 / Qt | Нативная интеграция с KDE |
| **Сериализация** | Protocol Buffers | Компактность, совместимость |
| **API** | gRPC | Двунаправленные потоки |

---

## 2. Идентификация и криптография

### 2.1 Ключевая пара

Каждый участник сети (сервер или клиент) обладает ключевой парой на базе **Ed25519**.

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
        
        # ID участника = первые 20 байт от хеша публичного ключа
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

2.2 Сид-фраза для клиентов (BIP39)
Для удобства пользователей используется стандартная сид-фраза из 12 слов (BIP39).

import hashlib
import hmac
from mnemonic import Mnemonic

class SeedPhrase:
    @staticmethod
    def generate():
        mnemo = Mnemonic("english")
        return mnemo.generate(strength=128)  # 12 слов
    
    @staticmethod
    def to_seed(phrase, passphrase=""):
        mnemo = Mnemonic("english")
        return mnemo.to_seed(phrase, passphrase)
    
    @staticmethod
    def to_keypair(seed):
        # HMAC-SHA512 для генерации мастер-ключа
        h = hmac.new(b"DuoNet seed", seed, hashlib.sha512)
        master_key = h.digest()
        return Identity(seed=master_key[:32])

2.3 Привязка сервера к железу
Для серверов используется аппаратный fingerprint, чтобы ограничить создание множества серверов одним лицом (защита от Sybil-атак).

import hashlib
import subprocess
import platform

class HardwareFingerprint:
    @staticmethod
    def get_machine_id():
        """Получение уникального ID системы"""
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
            # парсинг IOPlatformUUID
            return "mac_uuid_placeholder"
    
    @staticmethod
    def get_disk_serial():
        """Серийный номер системного диска"""
        if platform.system() == "Linux":
            result = subprocess.run(
                ["lsblk", "--noheadings", "-o", "SERIAL", "/dev/sda"], 
                capture_output=True, text=True
            )
            return result.stdout.strip()
        # для других ОС - аналогично
    
    @staticmethod
    def generate_fingerprint():
        machine_id = HardwareFingerprint.get_machine_id()
        disk_serial = HardwareFingerprint.get_disk_serial()
        
        # Комбинируем с солью
        salt = b"DuoNet Hardware Salt v1"
        data = f"{machine_id}:{disk_serial}".encode() + salt
        
        return hashlib.sha256(data).hexdigest()
        
2.4 Типы участников и режимы честности

class ParticipantType(Enum):
    SERVER_PUBLIC_IP = 1   # Сервер с белым IP (валидатор)
    SERVER_NAT = 2         # Сервер за NAT (инициатор)
    CLIENT_HONEST = 3      # Клиент, знающий тип устройства
    CLIENT_UNSURE = 4      # Клиент, не знающий тип ("честное незнание")
    
3. Транзакции и блоки
3.1 Формат транзакции (Protocol Buffers)

syntax = "proto3";

package duonet;

enum TransactionType {
  TRANSFER = 0;           // Обычный перевод
  SERVER_REGISTER = 1;    // Регистрация нового сервера
  SERVER_RENEW = 2;       // Продление/возврат после штрафа
  CONTRACT_CREATE = 3;    // Создание смарт-контракта (escrow)
  CONTRACT_RELEASE = 4;   // Исполнение контракта (с фразой)
  CONTRACT_EXPIRE = 5;    // Сжигание по истечении срока
  STAKE_UPDATE = 6;       // Изменение стейка
  MIGRATION = 7;          // Миграция кошелька на новое железо
}

message Transaction {
  uint32 version = 1;
  TransactionType type = 2;
  bytes from = 3;              // Адрес отправителя (20 байт)
  bytes to = 4;                // Адрес получателя (20 байт)
  uint64 amount = 5;           // Сумма в сатоши (1 DNET = 10^8)
  uint64 fee = 6;              // Комиссия
  uint64 nonce = 7;            // Счётчик отправителя (защита от повторов)
  uint64 timestamp = 8;        // Время создания (Unix timestamp)
  bytes initiator = 9;         // ID сервера, инициировавшего транзакцию
  bytes data = 10;             // Дополнительные данные (зависит от типа)
  bytes signature = 11;        // Подпись отправителя
}

// Данные для регистрации сервера
message ServerRegisterData {
  string hardware_fingerprint = 1;
  ParticipantType participant_type = 2;
  repeated string multiaddrs = 3;  // Адреса для libp2p
}

// Данные для escrow-контракта
message EscrowData {
  bytes seller = 1;              // Адрес продавца
  bytes secret_hash = 2;         // SHA256 от фразы
  uint64 deadline = 3;           // Срок истечения
  bytes contract_id = 4;         // ID контракта (хеш от параметров)
}

// Данные для исполнения контракта
message EscrowReleaseData {
  bytes contract_id = 1;
  bytes secret = 2;              // Фраза (не хеш!)
}

3.2 Формат блока

message Block {
  uint32 version = 1;
  uint64 height = 2;              // Номер блока
  bytes previous_hash = 3;        // Хеш предыдущего блока
  bytes merkle_root = 4;          // Корень дерева Меркла
  repeated Transaction transactions = 5;
  uint64 timestamp = 6;           // Время создания
  bytes validator = 7;             // Адрес валидатора
  bytes signature = 8;            // Подпись валидатора
  
  message Approval {
    bytes server = 1;
    bytes signature = 2;
    uint64 weight = 3;            // Вес сервера на момент подписи
  }
  repeated Approval approvals = 9;  // Подтверждения от других серверов
  
  uint64 total_weight = 10;        // Суммарный вес всех подписавших
  bytes state_root = 11;           // Хеш состояния после блока
}

3.3 Nonce и защита от повторов
Каждый аккаунт ведёт счётчик исходящих транзакций (nonce). Транзакция с nonce N принимается, только если последняя принятая транзакция от этого аккаунта имела nonce N-1.

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
        
4. Экономическая модель (детали реализации)
4.1 Параметры эмиссии

class EmissionCalculator:
    # Прогрессивная шкала: (нижняя_граница, верхняя_граница, эмиссия)
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
    
    BURN_RATE = 0.5  # 50% сжигается при создании
    
    @classmethod
    def get_emission(cls, server_count):
        """Получить эмиссию для нового сервера"""
        for lower, upper, amount in cls.EMISSION_SCHEDULE:
            if lower <= server_count + 1 <= upper:
                return amount
        return 10  # На всякий случай
    
    @classmethod
    def create_server_transaction(cls, server_address, server_count, fingerprint):
        """Создать транзакцию регистрации сервера"""
        emission = cls.get_emission(server_count)
        burn_amount = int(emission * cls.BURN_RATE * 10**8)  # в сатоши
        deposit_amount = int(emission * (1 - cls.BURN_RATE) * 10**8)
        
        # Создаём специальную транзакцию
        tx = Transaction(
            type=TransactionType.SERVER_REGISTER,
            from=b"00000000000000000000000000000000",  # zero address (эмиссия)
            to=server_address,
            amount=deposit_amount,
            fee=0,  # без комиссии
            nonce=0,
            timestamp=current_time(),
            data=ServerRegisterData(
                hardware_fingerprint=fingerprint
            ).SerializeToString()
        )
        return tx, burn_amount
        
4.2 Распределение комиссии

class FeeDistributor:
    INITIATOR_SHARE = 0.70    # 70% инициатору
    VALIDATOR_SHARE = 0.10    # 10% валидатору
    BURN_SHARE = 0.20         # 20% сжигание
    
    @classmethod
    def distribute(cls, transaction, block_validator):
        """Распределить комиссию транзакции"""
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
        """Распределить награду за блок (эмиссия)"""
        BLOCK_REWARD = 1_000_000  # 0.01 DNET в сатоши
        
        validator_reward = int(BLOCK_REWARD * 0.7)  # 70% валидатору
        burn_amount = BLOCK_REWARD - validator_reward  # 30% сжигание
        
        return {
            'validator': (validator, validator_reward),
            'burn': (BURN_ADDRESS, burn_amount)
        }
        
4.3 Расчёт веса сервера (Proof-of-Trust)

class TrustScoreCalculator:
    # Веса факторов
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
        """Расчёт веса сервера"""
        score = 0
        
        # 1. Баланс (логарифмическая шкала)
        max_balance = 1_000_000 * 10**8  # 1 млн DNET в сатоши
        balance_score = min(
            math.log10(server.balance + 1) / math.log10(max_balance),
            1.0
        )
        score += balance_score * cls.WEIGHTS['balance']
        
        # 2. Uptime (процент времени онлайн за 30 дней)
        uptime_score = server.uptime_30d / 30.0
        score += min(uptime_score, 1.0) * cls.WEIGHTS['uptime']
        
        # 3. Чистота (отсутствие штрафов)
        purity_score = 1.0 / (1.0 + server.violations)
        score += purity_score * cls.WEIGHTS['purity']
        
        # 4. Активность (кол-во транзакций)
        max_tx = 10000  # максимум за период
        activity_score = min(server.transactions_count / max_tx, 1.0)
        score += activity_score * cls.WEIGHTS['activity']
        
        # 5. Скорость реакции
        # response_time в миллисекундах, 5000ms = 0
        speed_score = max(0, 1.0 - server.avg_response_time / 5000)
        score += speed_score * cls.WEIGHTS['speed']
        
        # 6. Стаж (возраст в днях)
        tenure_score = min(server.age_days / 365, 1.0)
        score += tenure_score * cls.WEIGHTS['tenure']
        
        return score
        
5. Сеть и транспорт (libp2p)
5.1 Базовая настройка узла

from libp2p import new_node
from libp2p.crypto.keys import KeyPair
from libp2p.peer.id import ID

class DuoNetNode:
    def __init__(self, identity, is_public_ip=False):
        self.identity = identity
        self.is_public_ip = is_public_ip
        self.peer_id = ID.from_pubkey(identity.verify_key)
        
        # Создаём libp2p узел
        self.node = new_node(
            key_pair=KeyPair(identity.signing_key, identity.verify_key),
            listen_addrs=self._get_listen_addresses()
        )
        
        # Настраиваем протоколы
        self._setup_protocols()
        
        # Клиенты, подключённые к этому серверу
        self.clients = {}  # client_id -> connection
        
        # Соседние серверы
        self.peers = {}    # peer_id -> connection
    
    def _get_listen_addresses(self):
        if self.is_public_ip:
            return ["/ip4/0.0.0.0/tcp/9876"]
        else:
            # NAT-сервер не слушает входящие
            return []
            
5.2 Протоколы libp2p

def _setup_protocols(self):
    # 1. GossipSub для распространения транзакций и блоков
    self.gossip = self.node.get_pubsub()
    self.gossip.subscribe("duonet-transactions", self._handle_transaction)
    self.gossip.subscribe("duonet-blocks", self._handle_block)
    
    # 2. DHT для поиска пиров
    self.dht = self.node.get_kademlia()
    
    # 3. RPC протокол для прямых сообщений
    self.node.set_stream_handler("/duonet/1.0.0", self._handle_stream)
    
5.3 NAT Traversal и Hole Punching

class NATTraversal:
    def __init__(self, node, rendezvous_servers):
        self.node = node
        self.rendezvous_servers = rendezvous_servers
        
    async def hole_punch(self, target_peer_id):
        """Попытка прямого соединения через Hole Punching"""
        # 1. Запрашиваем у rendezvous-сервера адреса цели
        target_addrs = await self._discover_peer(target_peer_id)
        
        # 2. Отправляем сигнал цели через rendezvous
        await self._signal_peer(target_peer_id, self.node.get_addrs())
        
        # 3. Начинаем одновременно стучаться
        for addr in target_addrs:
            try:
                conn = await self.node.connect(addr, timeout=2)
                return conn
            except:
                continue
        
        return None
    
    async def _discover_peer(self, peer_id):
        """Поиск адресов пира через DHT и rendezvous"""
        addrs = []
        
        # Поиск в DHT
        dht_addrs = await self.dht.find_peer(peer_id)
        addrs.extend(dht_addrs)
        
        # Поиск через rendezvous-серверы
        for rv in self.rendezvous_servers:
            rv_addrs = await rv.lookup(peer_id)
            addrs.extend(rv_addrs)
        
        return list(set(addrs))
        
5.4 Релейная передача (Relay)
Для серверов за NAT, которые не могут установить прямое соединение:

class RelayService:
    def __init__(self, node):
        self.node = node
        self.relay_connections = {}  # peer_id -> connection
        
    async def relay_message(self, from_peer, to_peer, message):
        """Переслать сообщение через релей"""
        if to_peer in self.relay_connections:
            # Прямая пересылка
            await self.relay_connections[to_peer].write(message)
        else:
            # Сохраняем для последующей доставки
            self._store_offline(to_peer, message)
    
    def _store_offline(self, peer_id, message):
        """Сохранить офлайн-сообщение (в зашифрованном виде)"""
        # В реальной реализации - сохранять в БД
        pass
        
6. Сервер знакомств (Rendezvous)
6.1 API сервера знакомств (gRPC)

service Rendezvous {
  // Регистрация сервера
  rpc RegisterServer(ServerInfo) returns (RegistrationResponse);
  
  // Heartbeat (подтверждение активности)
  rpc Heartbeat(HeartbeatRequest) returns (HeartbeatResponse);
  
  // Поиск сервера по ID
  rpc LookupPeer(LookupRequest) returns (LookupResponse);
  
  // Получение списка активных серверов
  rpc GetServers(GetServersRequest) returns (stream ServerInfo);
  
  // Сигнализация для Hole Punching
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
  bytes signature = 8;  // Подпись server_id + timestamp
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

6.2 Реализация сервера знакомств

import asyncio
import time
from collections import OrderedDict

class RendezvousServer:
    def __init__(self, cleanup_interval=300):
        self.servers = {}  # server_id -> ServerInfo
        self.last_seen = {}  # server_id -> timestamp
        self.cleanup_interval = cleanup_interval
        
        # Запускаем фоновую очистку
        asyncio.create_task(self._cleanup_loop())
    
    async def register_server(self, request):
        """Регистрация сервера"""
        server_id = request.server_id
        
        # Проверяем подпись
        if not self._verify_signature(request):
            return RegistrationResponse(success=False, error="Invalid signature")
        
        # Сохраняем информацию
        self.servers[server_id] = request
        self.last_seen[server_id] = time.time()
        
        # Возвращаем список других серверов (до 50)
        other_servers = [
            s for sid, s in self.servers.items() 
            if sid != server_id and time.time() - self.last_seen[sid] < 300
        ][:50]
        
        return RegistrationResponse(
            success=True,
            servers=other_servers
        )
    
    async def heartbeat(self, request):
        """Обновление статуса сервера"""
        server_id = request.server_id
        if server_id in self.servers:
            self.last_seen[server_id] = time.time()
            return HeartbeatResponse(success=True)
        return HeartbeatResponse(success=False, error="Unknown server")
    
    async def lookup_peer(self, request):
        """Поиск пира"""
        peer_id = request.peer_id
        if peer_id in self.servers:
            last = self.last_seen.get(peer_id, 0)
            online = (time.time() - last) < 180  # 3 минуты
            
            return LookupResponse(
                online=online,
                multiaddrs=self.servers[peer_id].multiaddrs if online else [],
                last_seen=last
            )
        
        return LookupResponse(online=False)
    
    async def signal_hole_punch(self, request):
        """Сигнализация для Hole Punching"""
        # В реальной реализации - отправка сигнала цели
        # Здесь просто сохраняем в очередь
        signal_key = f"{request.from_peer}:{request.to_peer}"
        self.pending_signals[signal_key] = {
            'from_addrs': request.from_addrs,
            'timestamp': time.time()
        }
        
        return SignalResponse(success=True)
    
    async def _cleanup_loop(self):
        """Фоновая очистка неактивных серверов"""
        while True:
            await asyncio.sleep(self.cleanup_interval)
            now = time.time()
            to_remove = [
                sid for sid, last in self.last_seen.items()
                if now - last > 600  # 10 минут неактивности
            ]
            for sid in to_remove:
                del self.servers[sid]
                del self.last_seen[sid]
                
7. Клиент-серверное взаимодействие
7.1 API клиента к серверу (gRPC)

service ClientService {
  // Аутентификация
  rpc Auth(ClientAuth) returns (SessionToken);
  
  // Отправка сообщения
  rpc SendMessage(OutboundMessage) returns (MessageReceipt);
  
  // Получение сообщений (streaming)
  rpc PollMessages(PollRequest) returns (stream InboundMessage);
  
  // Офлайн-сообщения
  rpc StoreOffline(OfflineMessage) returns (Status);
  rpc FetchOffline(Empty) returns (stream OfflineMessage);
  
  // Кошелёк
  rpc GetBalance(Empty) returns (Balance);
  rpc BroadcastTx(SignedTransaction) returns (TxHash);
  rpc GetTransactionHistory(HistoryRequest) returns (stream Transaction);
  
  // Контакты
  rpc ResolveContact(ContactID) returns (ContactInfo);
}

message ClientAuth {
  bytes client_id = 1;
  uint64 timestamp = 2;
  bytes signature = 3;  // Подпись client_id + timestamp
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

7.2 Реализация клиентского подключения

class DuoNetClient:
    def __init__(self, identity, server_selector):
        self.identity = identity
        self.server_selector = server_selector
        self.current_server = None
        self.session_token = None
        self.message_queue = asyncio.Queue()
    
    async def connect(self):
        """Подключение к выбранному серверу"""
        server = await self.server_selector.select()
        self.current_server = server
        
        # Аутентификация
        auth = ClientAuth(
            client_id=self.identity.id,
            timestamp=int(time.time()),
            signature=self.identity.sign(self.identity.id + str(timestamp))
        )
        
        response = await server.stub.Auth(auth)
        self.session_token = response.token
        
        # Запускаем polling сообщений
        asyncio.create_task(self._poll_messages())
        
        return True
    
    async def send_message(self, to_id, encrypted_payload):
        """Отправка сообщения"""
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
            # Если сервер недоступен - переподключаемся
            await self._reconnect()
            return await self.send_message(to_id, encrypted_payload)
    
    async def _poll_messages(self):
        """Долгий poll для получения сообщений"""
        while True:
            try:
                request = PollRequest(last_id=self.last_message_id)
                async for msg in self.current_server.stub.PollMessages(
                    request, metadata=[('token', self.session_token)]
                ):
                    await self.message_queue.put(msg)
                    self.last_message_id = msg.message_id
            except Exception as e:
                # Потеря соединения, ждём и переподключаемся
                await asyncio.sleep(5)
                await self._reconnect()
                
7.3 Выбор сервера клиентом (4 режима)

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
        # 1. Проверяем локальную сеть (multicast DNS)
        local_server = await self._discover_local()
        if local_server:
            return local_server
        
        # 2. Спрашиваем у rendezvous
        servers = await self.rendezvous.get_servers(limit=20)
        
        # 3. Пингуем и выбираем быстрый
        for server in servers:
            latency = await self._ping(server)
            if latency < 100:  # меньше 100 мс
                return server
        
        return servers[0] if servers else None
    
    async def _geo_select(self):
        # Определяем регион (по IP или GPS)
        region = await self._get_region()
        
        # Запрашиваем сервера в этом регионе
        servers = await self.rendezvous.get_servers(region=region)
        
        # Сортируем по нагрузке
        servers.sort(key=lambda s: s.current_load)
        return servers[0] if servers else None
    
    async def _discover_local(self):
        """Поиск сервера в локальной сети через mDNS"""
        # Используем libp2p mDNS discovery
        pass
        
8. Смарт-контракты (Escrow)
8.1 Структура контракта

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
        """Создание нового контракта"""
        secret_hash = hashlib.sha256(secret).digest()
        deadline = int(time.time()) + days * 86400
        
        # ID контракта = хеш от параметров
        contract_data = f"{buyer}{seller}{amount}{secret_hash}{deadline}"
        contract_id = hashlib.sha256(contract_data.encode()).digest()
        
        return cls(contract_id, buyer, seller, amount, secret_hash, deadline)
    
    def release(self, secret, seller_key):
        """Исполнение контракта (продавец предъявляет фразу)"""
        if self.released or self.expired:
            return False, "Contract already finalized"
        
        if time.time() > self.deadline:
            self.expired = True
            return False, "Contract expired"
        
        # Проверяем фразу
        if hashlib.sha256(secret).digest() != self.secret_hash:
            return False, "Invalid secret"
        
        # Проверяем, что вызывает seller
        if seller_key != self.seller:
            return False, "Only seller can release"
        
        self.released = True
        return True, "Contract released"
    
    def expire(self):
        """Сжигание средств по истечении срока"""
        if not self.released and time.time() > self.deadline:
            self.expired = True
            return True
        return False
        
8.2 VM для выполнения контрактов (упрощённо)

class ContractVM:
    def __init__(self):
        self.contracts = {}  # contract_id -> EscrowContract
    
    def execute_transaction(self, tx):
        """Выполнить транзакцию, связанную с контрактом"""
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
            contract_id=tx.hash,  # ID = хеш транзакции
            buyer=tx.from,
            seller=data.seller,
            amount=tx.amount,
            secret_hash=data.secret_hash,
            deadline=data.deadline
        )
        
        self.contracts[tx.hash] = contract
        
        # Блокируем средства на контракт-адресе
        # В состоянии блокчейна: перемещаем tx.amount на специальный адрес
        
        return True, "Contract created"
    
    def _release_contract(self, tx):
        data = EscrowReleaseData()
        data.ParseFromString(tx.data)
        
        contract = self.contracts.get(data.contract_id)
        if not contract:
            return False, "Contract not found"
        
        success, message = contract.release(data.secret, tx.from)
        if success:
            # Переводим средства продавцу
            # В состоянии блокчейна: перемещаем с контракт-адреса на tx.from
            pass
        
        return success, message
        
9. Шифрование сообщений (Double Key)
9.1 Базовый протокол Double Ratchet

class DoubleRatchet:
    def __init__(self, shared_secret):
        self.root_key = shared_secret
        self.send_chain_key = self._derive_chain(self.root_key, b"send")
        self.recv_chain_key = self._derive_chain(self.root_key, b"recv")
        self.send_counter = 0
        self.recv_counter = 0
    
    def ratchet_step(self):
        """Обновление ключей (после каждого сообщения)"""
        self.send_chain_key = self._derive_chain(self.send_chain_key, b"next")
        self.send_counter += 1
    
    def encrypt(self, plaintext, additional_key=None):
        """Шифрование сообщения"""
        # Смешиваем с дополнительным ключом, если есть
        message_key = self.send_chain_key
        if additional_key:
            message_key = self._xor_keys(message_key, additional_key)
        
        # Шифрование
        nonce = os.urandom(12)
        cipher = ChaCha20_Poly1305.new(key=message_key, nonce=nonce)
        ciphertext, tag = cipher.encrypt_and_digest(plaintext)
        
        self.ratchet_step()
        
        return {
            'nonce': nonce,
            'ciphertext': ciphertext,
            'tag': tag,
            'counter': self.send_counter - 1
        }
    
    def decrypt(self, packet, additional_key=None):
        """Расшифровка сообщения"""
        # Восстанавливаем ключ для этого сообщения
        message_key = self._get_message_key(packet['counter'])
        if additional_key:
            message_key = self._xor_keys(message_key, additional_key)
        
        cipher = ChaCha20_Poly1305.new(
            key=message_key, 
            nonce=packet['nonce']
        )
        
        try:
            plaintext = cipher.decrypt_and_verify(
                packet['ciphertext'], 
                packet['tag']
            )
            return plaintext
        except:
            return None  # Неверный ключ
    
    def _xor_keys(self, key1, key2):
        """XOR двух ключей"""
        return bytes(a ^ b for a, b in zip(key1, key2))
        
9.2 Дополнительная фраза (Double Key)

class DoubleKeyEncryption:
    @staticmethod
    def derive_phrase_key(phrase, salt):
        """Получение ключа из фразы (Argon2)"""
        return argon2(
            password=phrase.encode(),
            salt=salt,
            memory=64*1024,  # 64 MB
            iterations=3,
            hash_len=32
        )
    
    @staticmethod
    def encrypt_message(plaintext, session_key, phrase):
        """Шифрование с дополнительной фразой"""
        salt = os.urandom(16)
        phrase_key = DoubleKeyEncryption.derive_phrase_key(phrase, salt)
        
        # Смешиваем ключи
        combined_key = DoubleRatchet._xor_keys(session_key, phrase_key)
        
        nonce = os.urandom(12)
        cipher = ChaCha20_Poly1305.new(key=combined_key, nonce=nonce)
        ciphertext, tag = cipher.encrypt_and_digest(plaintext)
        
        # Формат: salt (16) + nonce (12) + tag (16) + ciphertext
        return salt + nonce + tag + ciphertext
    
    @staticmethod
    def decrypt_message(packet, session_key, phrase):
        """Расшифровка с проверкой фразы"""
        salt = packet[:16]
        nonce = packet[16:28]
        tag = packet[28:44]
        ciphertext = packet[44:]
        
        phrase_key = DoubleKeyEncryption.derive_phrase_key(phrase, salt)
        combined_key = DoubleRatchet._xor_keys(session_key, phrase_key)
        
        cipher = ChaCha20_Poly1305.new(key=combined_key, nonce=nonce)
        
        try:
            plaintext = cipher.decrypt_and_verify(ciphertext, tag)
            return plaintext
        except:
            return None  # Неверная фраза
            
9.3 Управление сессиями чата

class ChatSession:
    def __init__(self, peer_id, peer_public_key, phrase=None):
        self.peer_id = peer_id
        self.peer_key = peer_public_key
        self.phrase_hash = hashlib.sha256(phrase).digest() if phrase else None
        
        # X3DH для установки начального ключа
        self.ratchet = self._establish_session(peer_public_key)
        
        self.message_counter = 0
    
    def send_message(self, plaintext):
        """Отправка сообщения"""
        if self.phrase_hash:
            # Дополнительная защита: включаем хеш фразы в пакет
            # (не саму фразу!)
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
        """Получение сообщения"""
        if self.phrase_hash:
            # Проверяем фразу (не раскрывая её)
            expected = hmac.new(
                self.phrase_hash,
                str(packet['counter']).encode(),
                hashlib.sha256
            ).digest()
            
            if not hmac.compare_digest(expected, packet['phrase_check']):
                return None  # Неверная фраза
        
        # Расшифровываем
        return self.ratchet.decrypt(packet['encrypted'])
        
10. Хранение данных
10.1 Структура БД сервера (RocksDB)

// Ключи в RocksDB
// "block:" + height -> Block
// "tx:" + tx_hash -> Transaction
// "balance:" + address -> uint64
// "nonce:" + address -> uint64
// "server:" + server_id -> ServerInfo
// "contract:" + contract_id -> EscrowContract
// "offline:" + client_id + timestamp -> OfflineMessage

10.2 Структура БД клиента (SQLCipher)

-- Мастер-ключ для БД = хеш от пароля пользователя
PRAGMA key = 'user_password_hash';

CREATE TABLE contacts (
    id INTEGER PRIMARY KEY,
    peer_id TEXT UNIQUE NOT NULL,
    nickname TEXT,
    public_key BLOB NOT NULL,
    phrase_hash BLOB,  -- хеш от дополнительной фразы (опционально)
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
    memo BLOB  -- зашифрованное примечание
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

11. Штрафы и безопасность
11.1 Типы нарушений

class ViolationType(Enum):
    INACTIVITY = 1      # Неактивность >3 дней
    DOUBLE_SIGN = 2     # Двойная подпись блока
    INVALID_BLOCK = 3   # Предложение невалидного блока
    SPAM = 4            # Чрезмерное количество транзакций
    FALSE_HARDWARE = 5  # Обман о типе железа
    
11.2 Система штрафов

class PenaltySystem:
    PENALTIES = {
        ViolationType.INACTIVITY: {
            1: (0.1 * 10**8, False),  # штраф, конфискация?
            2: (1.0 * 10**8, False),
            3: (0, True)  # конфискация
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
            1: (0, True)  # сразу конфискация
        }
    }
    
    def __init__(self, state):
        self.state = state
        self.violation_counts = {}  # server_id -> {type: count}
    
    def report_violation(self, server_id, violation_type):
        """Зафиксировать нарушение"""
        counts = self.violation_counts.get(server_id, {})
        count = counts.get(violation_type, 0) + 1
        counts[violation_type] = count
        self.violation_counts[server_id] = counts
        
        # Получаем наказание
        penalty, confiscate = self.PENALTIES[violation_type][count]
        
        if confiscate:
            # Полная конфискация депозита
            balance = self.state.get_balance(server_id)
            self.state.burn(server_id, balance)
            self.state.deactivate_server(server_id)
        else:
            # Штраф
            self.state.burn(server_id, penalty)
        
        return penalty, confiscate
        
12. Установка и конфигурация
12.1 Минимальные требования
Сервер:

CPU: 2 ядра, 2 ГГц

RAM: 4 ГБ

Диск: 100 ГБ SSD

ОС: Linux (рекомендуется Ubuntu 22.04+)

Клиент:

CPU: 1 ядро, 1 ГГц

RAM: 512 МБ

Диск: 1 ГБ

ОС: Linux, Windows 10+, macOS 11+

12.2 Пример конфигурации сервера (YAML)

# config.yaml
node:
  identity_path: "/var/lib/duonet/identity.key"
  participant_type: "server_public_ip"  # или server_nat
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
  
12.3 Пример конфигурации клиента (YAML)

# client_config.yaml
identity:
  seed_phrase: "word1 word2 word3 ..."  # или путь к файлу
  password: "user_password"  # для расшифровки локальной БД

server_selection:
  mode: "auto"  # auto, geo, premium, manual
  preferred_servers: []  # для manual режима

storage:
  db_path: "~/.duonet/client.db"
  message_retention_days: 30

ui:
  theme: "dark"
  language: "ru"
  
13. Тестирование и отладка
13.1 Тестовая сеть (testnet)

# testnet_config.py
TESTNET_PARAMS = {
    'block_time': 2,  # 2 секунды для быстрого тестирования
    'emission_schedule': [
        (1, 10, 10000),  # ускоренная эмиссия для тестов
    ],
    'min_peers': 2,
    'consensus_threshold': 0.6,  # 60% для тестов
    'rendezvous_servers': [
        "/ip4/127.0.0.1/tcp/9876/p2p/QmTest1"
    ]
}

13.2 Инструменты для отладки

# Проверка соединения с сервером
duonet-cli ping /ip4/95.217.0.1/tcp/9876

# Просмотр баланса
duonet-cli balance

# Отправка тестовой транзакции
duonet-cli send --to 7f3a...b8e2 --amount 10 --memo "test"

# Просмотр логов в реальном времени
duonet-node --log-level debug

14. Заключение
Данная техническая спецификация содержит все необходимые детали для реализации DuoNet:

Криптография: Ed25519, Double Ratchet, Argon2, ChaCha20-Poly1305

Сеть: libp2p, DHT, GossipSub, NAT Traversal

Блокчейн: транзакции, блоки, консенсус Proof-of-Trust

Экономика: прогрессивная эмиссия, распределение 70/10/20

Смарт-контракты: escrow с невозвратом

Клиент: 4 режима выбора сервера, двойной ключ

Хранение: RocksDB (сервер), SQLCipher (клиент)

Проект готов к реализации командой разработчиков.

Версия документа: 1.0
Последнее обновление: Март 2026
Автор: Алексей Николаев
