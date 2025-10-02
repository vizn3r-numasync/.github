# Project Overview

This project provides a secure, distributed, and end-to-end encrypted file storage and synchronization system. It is designed to give users full control over their data while enabling seamless multi-device sync and P2P transfers.

## System Components

### CLI (C)
- Command-line interface for file operations
- Local encryption key management
- Direct system calls to the operating system
- Communicates with the Client Daemon for synchronization

### Client Daemon (Go)
- Long-running background process for file monitoring
- Detects file changes and triggers synchronization
- Handles client-side encryption (CEK/KEK)
- Maintains daemon key cache in memory (optional OS keychain support)
- Communicates with CLI, Central Server, and other Client Daemons for P2P transfers

### Central Server (Elixir)
- Orchestrates file synchronization across devices
- Provides P2P relay and NAT traversal services
- Manages share links and device registration
- Coordinates real-time sync events
- Communicates with Client Daemons and Storage Daemons

### Storage Daemon (Go)
- Stores encrypted file chunks securely
- Blind key-value store: server never sees plaintext
- Tracks encrypted metadata such as filenames and file sizes
- Communicates with Central Server only

### Web Client (SvelteKit)
- Provides a web-based file management interface
- Supports client-side end-to-end encryption
- Allows share link creation and access
- Communicates with the Central Server for file operations

## Deployment Models

### Self-Hosted
Users can deploy all components on their own infrastructure:
```

Client Daemon(s)
Central Server
Storage Daemon(s)

```

### Hybrid Model
Users may deploy their own Storage Daemons while the Central Server coordinates synchronization:
```

User Infrastructure:
└── Storage Daemon(s)

Central Server coordinates user storage

```

## Data Flow Examples

### File Upload
```

CLI/Web → Client Daemon (encrypt) → Central Server → Storage Daemon

```

### P2P Transfer
```

Device A ← → Central Server ← → Device B
(Orchestrates connection, provides relay if needed)

```

### Share Link Access
```

Visitor → Web Client → Central Server → Storage Daemon
(Client-side decryption with share_key from URL fragment)

```

## Security Architecture

### End-to-End Encryption
- File content encrypted with CEK (client-side)
- CEK encrypted with KEK derived from user password
- Servers store only encrypted content and encrypted CEK
- Server never sees plaintext, CEK, KEK, or passwords

### Public vs Private Shares
- Private: decrypted client-side only
- Public: server-assisted previews or embeds (optional convenience)

## Key Features

- **Daemon Key Cache:** memory-only by default, optional OS keychain support, automatic expiration and invalidation
- **Multi-Device Sync:** coordinated by Central Server, P2P transfers where possible, relay fallback for NAT/firewall scenarios
- **P2P Capabilities:** direct device-to-device transfer with Central Server orchestrating NAT traversal
- **Hybrid Security Options:** metadata can optionally be server-encrypted to enable search or previews

## Technical Benefits

- **Polyglot Architecture:** CLI in C, Client and Storage Daemons in Go, Central Server in Elixir, Web Client in SvelteKit
- **Scalability:** Elixir server handles massive concurrency, storage daemons are distributed, P2P reduces server bandwidth requirements
- **Security:** fully auditable, end-to-end encryption with zero-knowledge architecture, client controls all encryption keys

This project provides a fully transparent and secure cloud storage ecosystem, empowering users with control over their data while enabling modern sync and sharing features.
