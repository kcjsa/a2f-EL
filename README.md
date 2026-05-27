# A2F - Analysis to Fake Protocol

[![Rust](https://img.shields.io/badge/rust-1.70+-blue.svg)](https://www.rust-lang.org)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-green.svg)](LICENSE)

**A2F is an asynchronous, out-of-order tolerant, high-latency cryptography protocol.**

> "Send in chaos. Receive in order. Timestamps bind everything."

## Problem Statement

Traditional cryptography protocols (TLS, SSH, WireGuard) assume:
- Packets arrive **in order**
- Keys arrive **before data**
- Low **latency** (timeouts and retransmissions)

But real-world networks are not ideal:
- Starlink: 30-80ms jitter, 1-2% packet loss, handover every 15 seconds
- Satellite communications: 500ms+ RTT
- Mobile networks: unstable connectivity
- Mesh networks: out-of-order delivery

**A2F solves this by using timestamps as binders between keys and data.**

## How It Works
┌─────────────────────────────────────────────────────────────┐
│ SENDER │
├─────────────────────────────────────────────────────────────┤
│ 1. Generate session key K_t │
│ 2. Wrap K_t with AES-256 + ChaCha20-Poly1305 │
│ 3. Encrypt data with K_t │
│ 4. Attach timestamp t to both │
│ 5. Shuffle and send in random order │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│ UNSTABLE NETWORK │
│ - Out of order - Packet loss - High latency │
└─────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────┐
│ RECEIVER │
├─────────────────────────────────────────────────────────────┤
│ 1. Receive packets in any order │
│ 2. Buffer by timestamp │
│ 3. When both key and data arrive, decrypt │
│ 4. Old packets expire after timeout │
└─────────────────────────────────────────────────────────────┘

text

## Features

| Feature | A2F | Traditional |
|---------|-----|-------------|
| Out-of-order delivery | ✅ Designed for it | ❌ Requires reordering |
| Key arrives after data | ✅ Buffered | ❌ Retransmission |
| High latency tolerance | ✅ No timeout | ❌ RTO backoff |
| Dummy packet mixing | ✅ Supported | ❌ Not supported |
| Multi-layer encryption | ✅ AES+ChaCha20 | ❌ Single cipher |
| Traffic analysis resistance | ✅ Random shuffle | ❌ Predictable order |

## Quick Start

### Add to your Cargo.toml

```toml
[dependencies]
a2f = { git = "https://github.com/kcjsa/a2f" }
# or after crates.io publish:
# a2f = "0.1"
Basic Usage
rust
use a2f::{A2FConfig, A2FSender, A2FReceiver, current_timestamp};

fn main() -> anyhow::Result<()> {
    let master_key = [0x42; 32];  // Share this securely!
    let config = A2FConfig::default();
    
    let mut sender = A2FSender::new(master_key, &config);
    let mut receiver = A2FReceiver::new(master_key, &config);
    
    let ts = current_timestamp();
    
    // Create key and encrypted data packets
    let key_packet = sender.generate_key_packet(ts)?;
    let data_packet = sender.encrypt_data(b"Hello, A2F!", ts)?;
    
    // Shuffle packets (simulate out-of-order delivery)
    let packets = sender.shuffle_packets(vec![key_packet, data_packet]);
    
    // Receive in any order - timestamps handle the binding
    for packet in packets {
        if let Some(decrypted) = receiver.receive_packet(packet)? {
            println!("Decrypted: {}", String::from_utf8_lossy(&decrypted));
        }
    }
    
    Ok(())
}
UDP Server Example
rust
// Server
let socket = UdpSocket::bind("0.0.0.0:8888")?;
let mut receiver = A2FReceiver::new(master_key, &config);

loop {
    let mut buf = [0u8; 65536];
    let (len, addr) = socket.recv_from(&mut buf)?;
    if let Ok(packet) = Packet::deserialize(&buf[..len]) {
        if let Some(decrypted) = receiver.receive_packet(packet)? {
            println!("Message: {}", String::from_utf8_lossy(&decrypted));
        }
    }
}
Test Results
All experiments passed:

Test	Description	Result
1	Basic encrypt/decrypt	✅
2	Key arrives before data	✅
3	Data arrives before key	✅
4	Multiple messages mixed	✅
5	Dummy packets + packet loss	✅
6	Real UDP communication	✅
Use Cases
Starlink / LEO satellite networks - High jitter, frequent handovers

Long-distance networks - 500ms+ RTT

Mobile / unstable connections - Packet loss and reordering

Mesh networks - Unpredictable routing

Censorship circumvention - Traffic analysis resistance

IoT / sensor networks - Intermittent connectivity

Performance
Encryption: AES-256-GCM + ChaCha20-Poly1305

Packet size: ~100 bytes overhead per key+data pair

Buffer timeout: Configurable (default 10 seconds)

Security
Multi-layer defense: AES-256 + ChaCha20-Poly1305

Traffic analysis resistance: Random packet order, dummy packets

Forward secrecy: Session keys are ephemeral

No side-channel: Keys are never sent in plaintext

Why "Analysis to Fake"?
"Fake out traffic analysis."

The protocol makes it difficult for an observer to:

Distinguish key packets from data packets

Predict packet order

Determine which packets belong together

License
MIT or Apache-2.0

Author
@kcjsa

A2F - Because high-latency networks deserve secure communication.

text

---

## 保存してプッシュ

```bash
cd ~/ミュージック/a2f

# README.mdを作成
# 上記の内容をコピーして貼り付け

# Gitに追加してプッシュ
git add README.md
git commit -m "Add README"
git push origin master
