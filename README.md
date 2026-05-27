# A2F - Analysis to Fake Protocol

[![Rust](https://img.shields.io/badge/rust-1.70+-blue.svg)](https://www.rust-lang.org)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-green.svg)](LICENSE)

**A2F is an asynchronous, out-of-order tolerant, high-latency cryptography protocol.**

> "Send in chaos. Receive in order. Timestamps bind everything."

## How It Works
SENDER:

Generate session key K_t

Wrap K_t with AES-256 + ChaCha20-Poly1305

Encrypt data with K_t

Attach timestamp t to both

Shuffle and send in random order
|
V
UNSTABLE NETWORK:

Out of order

Packet loss

High latency
|
V
RECEIVER:

Receive packets in any order

Buffer by timestamp

When both key and data arrive, decrypt

Old packets expire after timeout

text

## Features

- Out-of-order delivery support (no retransmission)
- Data can arrive before key (buffered)
- High latency tolerance (no timeout issues)
- Dummy packet mixing (traffic analysis resistance)
- Multi-layer encryption (AES-256 + ChaCha20-Poly1305)
- Simple implementation (timestamp + buffer)

## Quick Start

Add to your Cargo.toml:

```toml
[dependencies]
a2f = { git = "https://github.com/kcjsa/a2f" }
Basic usage:

use a2f::{A2FConfig, A2FSender, A2FReceiver, current_timestamp};

fn main() -> anyhow::Result<()> {
    let master_key = [0x42; 32];  // Share this securely!
    let config = A2FConfig::default();
    
    let mut sender = A2FSender::new(master_key, &config);
    let mut receiver = A2FReceiver::new(master_key, &config);
    
    let ts = current_timestamp();
    
    let key_packet = sender.generate_key_packet(ts)?;
    let data_packet = sender.encrypt_data(b"Hello, A2F!", ts)?;
    
    let packets = sender.shuffle_packets(vec![key_packet, data_packet]);
    
    for packet in packets {
        if let Some(decrypted) = receiver.receive_packet(packet)? {
            println!("Decrypted: {}", String::from_utf8_lossy(&decrypted));
        }
    }
    
    Ok(())
}
Test Results
Test	Result
Basic encrypt/decrypt	✅
Key before data	✅
Data before key	✅
Multiple messages mixed	✅
Dummy packets + packet loss	✅
UDP communication	✅
Use Cases
Starlink / LEO satellite networks

Long-distance high-latency networks

Mobile / unstable connections

Mesh networks

Censorship circumvention

Security
Multi-layer: AES-256 + ChaCha20-Poly1305

Traffic analysis resistant

No plaintext key transmission

Ephemeral session keys

License
MIT or Apache-2.0

Author
@kcjsa

---

## 保存してプッシュ

```bash
cd ~/ミュージック/a2f
cat > README.md
# 上記の内容をコピーして貼り付け、Ctrl+Dで保存

git add README.md
git commit -m "Add README"
git push origin master
