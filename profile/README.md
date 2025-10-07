# Distributed Encrypted File Sync / Storage

## Overview

A secure, distributed, **end-to-end encrypted file storage and sync system** giving users full control over their data with P2P transfers and zero-knowledge design.

**Core Principles**

* Zero-knowledge E2E encryption (server never sees plaintext)
* Fully FOSS & self-hostable
* P2P-first communication
* Pay-per-service monetization (infra, not features)

---

## Components

```
 ┌──────────────────────────────┐
 │ CENTRAL SERVER (Go)          │
 │ - Coordination, registry     │
 │ - P2P matchmaking (STUN/TURN)│
 │ - Metadata via PostgreSQL    │
 └─────────┬──────────┬─────────┘
           │ P2P (UDP)
   ┌───────▼──────┐    ┌───────▼────────┐
   │ CLIENTD (Go) │◄──►│ STORAGED (Go)  │
   │ - E2E crypto │    │ - Encrypted KV │
   │ - Sync/watch │    │ - Chunk store  │
   └───────┬──────┘    └────────────────┘
           │ HTTP/WS
   ┌───────▼──────┐
   │ CLI (C) / WEB│
   │  (SvelteKit) │
   └──────────────┘

   ┌────────────────────────────┐
   │ MONETIZATION MIDDLEWARE    │
   │ - Proxy + billing checks   │
   │ - No access to plaintext   │
   └────────────────────────────┘
```

---

## Deployment Models

* **Self-Hosted** – full stack on your infra (free)
* **Managed** – provider hosts servers/storage (paid)
* **Hybrid** – user storage + provider coordination

---

## Data Flow (Simplified)

**Upload**

1. Client encrypts file → ciphertext
2. Central Server assigns storage
3. Client ↔ StorageD via P2P (UDP)
4. Metadata updated centrally

**Sync**

* Devices sync via direct P2P if online
* Fallback to StorageD if offline

**Share Link**

* URL fragment holds share key (`#key`)
* Server never sees plaintext or key

---

## Security Model

* **Keys:** Password → AuthKey (auth) & KEK (encryption)
* **Files:** CEK per file, encrypted by KEK
* **Zero-knowledge:** server stores ciphertext only
* **Recovery:** Optional recovery key decrypts KEK

---

## Protocol

Custom **UDP protocol** with 20-byte header:

```
Type | Flags | Len | SessionID | Seq | Ack | CRC32
```

Supports connection mgmt, metadata, and P2P chunk transfer with reliability and retransmit logic.

---

## Tech Stack

* **Go:** Server, Client, Storage daemons
* **C:** CLI
* **SvelteKit:** Web client
* **PostgreSQL / SQLite:** Metadata & local index
* **XChaCha20-Poly1305, Argon2id:** Encryption primitives

---

## Monetization

Middleware proxy enforces quotas & subscriptions without touching encryption keys.
Open-source core (MIT/Apache 2.0), proprietary billing layer only.

---

## Roadmap

1. Implement protocol & MVP daemons
2. Alpha self-hosted deployment
3. Hybrid/managed hosting
4. Future: differential sync, versioning, PQC

---

**Philosophy:**
*Users own their data. Hosting is optional, trust is not required.*
