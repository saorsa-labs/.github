# Saorsa Labs

**Post-Quantum Secure Decentralized Networking**

Saorsa Labs builds the foundation for privacy-preserving, quantum-resistant peer-to-peer networks. Our ecosystem provides everything needed to create decentralized applications that will remain secure in the post-quantum era.

---

## Quick Links

| Category | Crates |
|----------|--------|
| **Cryptography** | [saorsa-pqc](#saorsa-pqc) |
| **Networking** | [saorsa-core](#saorsa-core) &#124; [saorsa-gossip](#saorsa-gossip) |
| **Security** | [saorsa-mls](#saorsa-mls) &#124; [saorsa-seal](#saorsa-seal) |
| **Storage** | [saorsa-fec](#saorsa-fec) &#124; [saorsa-rsps](#saorsa-rsps) |
| **zkVM** | [saorsa-logic](#saorsa-logic) &#124; [saorsa-attestation-guest](#saorsa-attestation-guest) |
| **Applications** | [saorsa-node](#saorsa-node) &#124; [saorsa-cli](#saorsa-cli) &#124; [saorsa-multiapp](#saorsa-multiapp) |
| **Media** | [saorsa-webrtc](#saorsa-webrtc) |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Applications                                  │
│  saorsa-node  │  saorsa-cli  │  saorsa-multiapp  │  saorsa-robotics │
├─────────────────────────────────────────────────────────────────────┤
│                     Networking Layer                                 │
│         saorsa-core (DHT)     │     saorsa-gossip (Overlay)         │
├─────────────────────────────────────────────────────────────────────┤
│                     Protocol Layer                                   │
│    saorsa-mls (Groups)  │  saorsa-webrtc  │  saorsa-seal (Threshold)│
├─────────────────────────────────────────────────────────────────────┤
│                     Storage & Optimization                           │
│         saorsa-fec (Erasure)    │    saorsa-rsps (DHT Opt)          │
├─────────────────────────────────────────────────────────────────────┤
│                     Attestation Layer                                │
│         saorsa-logic (zkVM)  │  saorsa-attestation-guest (SP1)      │
├─────────────────────────────────────────────────────────────────────┤
│                     Cryptographic Foundation                         │
│                         saorsa-pqc                                   │
│            ML-DSA-65  │  ML-KEM-768  │  ChaCha20-Poly1305           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Two Networking Philosophies

Saorsa Labs provides two complementary approaches to P2P networking:

### saorsa-gossip: Lightweight Gossip Overlay

**DHT-free, contact-graph-based networking for real-time coordination**

```
┌──────────────────────────────────────┐
│  Presence │ PubSub │ CRDT │ Groups   │
├──────────────────────────────────────┤
│     Membership (HyParView + SWIM)    │
├──────────────────────────────────────┤
│         Transport (ant-quic)         │
└──────────────────────────────────────┘
```

| Feature | Implementation |
|---------|----------------|
| Discovery | FOAF queries (3 hops), Rendezvous sharding |
| Membership | HyParView (8-12 active peers) + SWIM failure detection |
| Broadcast | Plumtree epidemic (<500ms P50 latency) |
| State Sync | Delta-CRDTs (OR-Set, LWW-Register) |
| Privacy | MLS-derived presence tags (hourly rotation) |

**Best for**: Real-time messaging, presence, lightweight coordination

### saorsa-core: Full DHT Platform

**DHT-centric with ML-driven adaptive routing for persistent storage**

```
┌──────────────────────────────────────┐
│  Storage │ Identity │ Placement      │
├──────────────────────────────────────┤
│   Adaptive Layer (ML Routing)        │
├──────────────────────────────────────┤
│  Trust-Weighted DHT (Kademlia+Trust) │
├──────────────────────────────────────┤
│         Transport (ant-quic)         │
└──────────────────────────────────────┘
```

| Feature | Implementation |
|---------|----------------|
| DHT | Trust-weighted Kademlia (K=8, EigenTrust) |
| Routing | Thompson Sampling, Hyperbolic, SOM-based |
| Security | Witness protocol, Sybil detection, node age verification |
| Storage | Erasure coding with automatic repair |
| Identity | Four-word addresses, multi-device support |

**Best for**: Distributed storage, Byzantine-tolerant applications, complex routing

### Comparison

| Aspect | saorsa-gossip | saorsa-core |
|--------|---------------|-------------|
| Architecture | DHT-free overlay | DHT-centric |
| Peer count | 8-12 active | 20 per k-bucket |
| Discovery | Contact-graph (FOAF) | Kademlia lookups |
| Trust model | Assumes honest peers | EigenTrust reputation |
| Byzantine tolerance | None | Witness quorum |
| Storage | Transient (CRDTs) | Persistent (erasure-coded) |
| Latency | <500ms broadcast | Adaptive |
| Complexity | 10 crates | 200+ modules |

### Using Together

```
Bootstrap:  saorsa-gossip FOAF  ──→  Fast discovery (3 hops)
                 │
Stabilize:  saorsa-core DHT    ──→  Durable routing table
                 │
Coordinate: saorsa-gossip      ──→  Real-time presence/messaging
                 │
Store:      saorsa-core        ──→  Persistent data with repair
```

---

## Crate Reference

### Cryptographic Foundation

#### saorsa-pqc

**Post-Quantum Cryptography** `v0.3.14` · [GitHub](https://github.com/dirvine/saorsa-pqc) · [crates.io](https://crates.io/crates/saorsa-pqc)

Single source of truth for all cryptographic operations in the Saorsa ecosystem.

```rust
use saorsa_pqc::{ml_dsa, ml_kem, aead};

// Post-quantum signatures (FIPS 204)
let (pk, sk) = ml_dsa::generate_keypair();
let sig = ml_dsa::sign(&sk, message);
assert!(ml_dsa::verify(&pk, message, &sig));

// Post-quantum key encapsulation (FIPS 203)
let (ek, dk) = ml_kem::generate_keypair();
let (ciphertext, shared_secret) = ml_kem::encapsulate(&ek);
let decrypted_secret = ml_kem::decapsulate(&dk, &ciphertext);

// Symmetric encryption
let encrypted = aead::encrypt(&key, &nonce, plaintext, aad);
```

| Algorithm | Standard | Security Level |
|-----------|----------|----------------|
| ML-DSA-65 | FIPS 204 | 128-bit PQ |
| ML-KEM-768 | FIPS 203 | 128-bit PQ |
| SLH-DSA | FIPS 205 | 128-bit PQ |
| ChaCha20-Poly1305 | RFC 8439 | 256-bit |
| BLAKE3 | - | 256-bit |

---

### Networking

#### saorsa-core

**DHT-Based P2P Platform** `v0.9.5` · [GitHub](https://github.com/dirvine/saorsa-core-foundation)

Complete P2P networking foundation with distributed storage, trust-based routing, and quantum-resistant cryptography.

```rust
use saorsa_core::{P2PNode, NodeConfig, api};

// Create and start a node
let config = NodeConfig::default();
let node = P2PNode::new(config).await?;
node.start().await?;

// Register identity with four-word address
let handle = api::register_identity(
    ["alpha", "beta", "gamma", "delta"],
    &keypair
)?;

// Store data with automatic strategy selection
let storage_handle = api::store_data(&handle, data, replica_count)?;
```

**Key Modules**:

| Module | Purpose |
|--------|---------|
| `dht/` | Trust-weighted Kademlia with witness protocol |
| `adaptive/` | ML routing (Thompson Sampling, Q-Learning) |
| `placement/` | EigenTrust-weighted storage orchestration |
| `identity/` | Four-word addresses, multi-device support |
| `persistence/` | RocksDB/SQLite with ChaCha20 encryption |

**DHT Configuration**:
```rust
DHTConfig {
    replication_factor: 8,      // K parameter
    bucket_size: 20,            // Peers per k-bucket
    alpha: 3,                   // Lookup parallelism
    record_ttl: 3600,           // 1 hour
}
```

---

#### saorsa-gossip

**DHT-Free Gossip Overlay** `v0.1.11` · [GitHub](https://github.com/dirvine/saorsa-gossip)

Production-ready gossip protocol for real-time P2P coordination without DHT complexity.

```rust
use saorsa_gossip::{transport, membership, pubsub};

// Join the network
let transport = transport::AntQuicTransport::new(config).await?;
let membership = membership::HyParView::new(transport.clone());
membership.join(bootstrap_addrs).await?;

// Subscribe and publish
let pubsub = pubsub::Plumtree::new(membership.clone());
let mut rx = pubsub.subscribe(topic_id);
pubsub.publish(topic_id, message).await?;
```

**Workspace Crates**:

| Crate | Purpose |
|-------|---------|
| `saorsa-gossip-types` | Core types (TopicId, PeerId, MessageHeader) |
| `saorsa-gossip-identity` | ML-DSA-65 key management |
| `saorsa-gossip-transport` | QUIC with 3 multiplexed streams |
| `saorsa-gossip-membership` | HyParView + SWIM |
| `saorsa-gossip-pubsub` | Plumtree epidemic broadcast |
| `saorsa-gossip-coordinator` | Bootstrap, reflection, relay |
| `saorsa-gossip-rendezvous` | k=16 sharding (65,536 shards) |
| `saorsa-gossip-groups` | MLS group management |
| `saorsa-gossip-presence` | Beacon broadcasting + FOAF |
| `saorsa-gossip-crdt-sync` | Delta-CRDTs |

**Protocol Parameters**:
```
HyParView:  Active=8-12, Passive=64-128, Shuffle=30s
SWIM:       Probe=1s, Suspect=3s, Dead=5s
Plumtree:   Eager=6-12, IHave batch=100ms, Cache=10K×5min
```

---

### Security & Groups

#### saorsa-mls

**Message Layer Security** `v0.3.1` · [GitHub](https://github.com/dirvine/saorsa-mls)

Post-quantum secure group messaging with forward secrecy.

```rust
use saorsa_mls::{Group, Member};

// Create a group
let mut group = Group::new(group_id, &my_keypair)?;

// Add members
group.add_member(member_key_package)?;

// Encrypt for group
let ciphertext = group.encrypt(plaintext)?;

// Process incoming messages
let plaintext = group.decrypt(ciphertext)?;
```

**Features**:
- Post-quantum key exchange (ML-KEM-768)
- Forward secrecy via ratcheting
- Post-compromise security
- Efficient for large groups

---

#### saorsa-seal

**Threshold Sealing** `v0.1.2` · [GitHub](https://github.com/dirvine/saorsa-seal)

Threshold cryptography for group data protection.

```rust
use saorsa_seal::{ThresholdScheme, Shard};

// Create threshold scheme (3-of-5)
let scheme = ThresholdScheme::new(3, 5)?;
let shards = scheme.split(secret)?;

// Reconstruct with any 3 shards
let recovered = scheme.combine(&shards[0..3])?;
```

---

### Storage & Optimization

#### saorsa-fec

**Erasure Coding** `v0.4.11` · [GitHub](https://github.com/dirvine/saorsa-fec)

Quantum-safe erasure coding for distributed storage.

```rust
use saorsa_fec::{Encoder, Decoder};

// Encode data (4 data shards + 2 parity)
let encoder = Encoder::new(4, 2)?;
let shards = encoder.encode(data)?;

// Decode with any 4 of 6 shards
let decoder = Decoder::new(4, 2)?;
let recovered = decoder.decode(&available_shards)?;
```

**Features**:
- Reed-Solomon with SIMD acceleration
- Post-quantum encryption per shard
- Configurable data/parity ratio

---

#### saorsa-rsps

**Root-Scoped Provider Summaries** `v0.2.0` · [GitHub](https://github.com/dirvine/saorsa-rsps-foundation)

DHT optimization using Golomb Coded Sets for efficient content discovery.

```rust
use saorsa_rsps::{ProviderSummary, GolombCodedSet};

// Create provider summary
let summary = ProviderSummary::new(provider_id);
summary.add_content_ids(&content_ids)?;

// Compact representation
let gcs = summary.to_gcs()?;  // ~10 bits per element

// Query
if gcs.maybe_contains(content_id) {
    // Might be present (low false positive rate)
}
```

---

### Attestation & zkVM

#### saorsa-logic

**zkVM-Compatible Logic** `v0.1.0` · [GitHub](https://github.com/dirvine/saorsa-logic)

Pure logic library for zero-knowledge proofs, `no_std` compatible.

```rust
#![no_std]
use saorsa_logic::{derive_attestation, AttestationInput};

// Deterministic computation for zkVM
let input = AttestationInput { /* ... */ };
let attestation = derive_attestation(&input);
```

---

#### saorsa-attestation-guest

**SP1 zkVM Guest** `v0.2.0` · [GitHub](https://github.com/dirvine/saorsa-attestation-guest)

Zero-knowledge proof generator for entangled attestation.

```rust
// Runs inside SP1 zkVM
use saorsa_attestation_guest::prove_attestation;

// Generate proof without revealing private inputs
let proof = prove_attestation(public_input, private_witness)?;
```

---

### Applications

#### saorsa-node

**Network Node** `v0.2.12` · [GitHub](https://github.com/dirvine/saorsa-node)

Production network node with payment verification and auto-upgrade.

```bash
# Generate keys
saorsa-keygen --output ~/.saorsa/keys

# Start node
saorsa-node --config config.toml
```

**Features**:
- Auto-upgrade system
- Payment verification (ant-evm)
- Bootstrap node support
- Optional zkVM attestation

---

#### saorsa-cli

**Command Line Tools** · [GitHub](https://github.com/dirvine/saorsa-cli)

User-facing CLI for network interaction.

```bash
# Store data
saorsa store --file document.pdf

# Retrieve data
saorsa get <address> --output downloaded.pdf

# Check network status
saorsa status
```

---

#### saorsa-multiapp

**Desktop Application** `v0.1.0` · [GitHub](https://github.com/dirvine/saorsa-multiapp)

Quantum-resistant desktop application suite.

**Components**:
- `saorsa-desktop-core` - Core functionality
- `saorsa-desktop-wallet` - Key management
- `saorsa-desktop-sync` - File synchronization
- `saorsa-desktop-media` - Media streaming

---

#### saorsa-webrtc

**WebRTC Framework** `v0.2.1` · [GitHub](https://github.com/dirvine/saorsa-webrtc)

Post-quantum secure WebRTC with pluggable signaling.

```rust
use saorsa_webrtc::{PeerConnection, SignalingChannel};

// Create connection with PQ security
let pc = PeerConnection::new(config)?;

// Add media tracks
pc.add_track(video_track)?;
pc.add_track(audio_track)?;

// Connect via custom signaling
pc.connect(signaling_channel).await?;
```

**Workspace**:
- `saorsa-webrtc-core` - Core WebRTC
- `saorsa-webrtc-cli` - CLI tool
- `saorsa-webrtc-ffi` - FFI bindings
- `saorsa-webrtc-tauri` - Tauri plugin
- `saorsa-webrtc-codecs` - Video/audio codecs

---

## Dependency Graph

```
saorsa-node
├── saorsa-core
│   ├── saorsa-pqc ←──────────────────┐
│   ├── saorsa-rsps                   │
│   ├── saorsa-webrtc                 │
│   │   └── ant-quic (pqc)            │
│   └── saorsa-logic                  │
│       └── (used by attestation)     │
└── ant-evm (payments)                │
                                      │
saorsa-gossip                         │
├── saorsa-pqc ───────────────────────┤
├── saorsa-mls                        │
│   └── saorsa-pqc ───────────────────┤
└── ant-quic                          │
                                      │
saorsa-seal                           │
├── saorsa-core                       │
├── saorsa-fec                        │
│   └── saorsa-pqc ───────────────────┘
└── saorsa-pqc
```

---

## Getting Started

### Prerequisites

```bash
# Rust 1.75+
rustup update stable

# For zkVM features (optional)
cargo install sp1-cli
```

### Basic Node

```bash
# Clone and build
git clone https://github.com/saorsa-labs/saorsa-node
cd saorsa-node
cargo build --release

# Generate identity
./target/release/saorsa-keygen

# Run node
./target/release/saorsa-node
```

### Using as Library

```toml
# Cargo.toml

# For DHT-based networking
[dependencies]
saorsa-core = "0.9"

# For gossip-based networking
[dependencies]
saorsa-gossip = "0.1"

# For post-quantum crypto only
[dependencies]
saorsa-pqc = "0.3"
```

---

## Security

All Saorsa crates follow strict security practices:

- **Post-Quantum**: ML-DSA-65 and ML-KEM-768 (NIST FIPS 203/204)
- **Zero Panics**: No `.unwrap()`, `.expect()`, or `panic!()` in production
- **Memory Safety**: Zeroized memory for cryptographic keys
- **Audit Trail**: All operations can be witnessed and verified

### Reporting Vulnerabilities

Please report security issues to security@saorsa.network (PGP key available).

---

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

```bash
# Run tests
cargo test --all-features

# Check formatting and lints
cargo fmt --check
cargo clippy -- -D warnings

# Build docs
cargo doc --no-deps --open
```

---

## License

Crates are licensed under either:

- **MIT OR Apache-2.0** (permissive): saorsa-pqc, saorsa-logic, saorsa-node, saorsa-cli
- **AGPL-3.0** (copyleft): saorsa-core, saorsa-gossip, saorsa-mls, saorsa-fec, saorsa-rsps, saorsa-webrtc

See individual crate licenses for details.

---

## Roadmap

- [ ] **Q1 2025**: saorsa-core v1.0 stable release
- [ ] **Q2 2025**: saorsa-gossip production hardening
- [ ] **Q3 2025**: Mobile SDK (iOS/Android)
- [ ] **Q4 2025**: Browser WASM support

---

## Contact

- **GitHub**: [github.com/saorsa-labs](https://github.com/saorsa-labs)
- **Documentation**: [docs.saorsa.network](https://docs.saorsa.network)
- **Discord**: [discord.gg/saorsa](https://discord.gg/saorsa)

---

*Building the post-quantum decentralized future.*
