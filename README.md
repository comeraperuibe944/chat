# zkchat

A zero-knowledge, peer-to-peer public chat running fully on GitHub. No external servers, no third-party databases, no logins, no tracking.

Live: https://comeraperuibe944.github.io/chat/

## Fully on GitHub Architecture

Every component of the application runs exclusively on GitHub infrastructure:
- Hosting: Static assets served via GitHub Pages.
- Message Ingress: Ephemeral issue submissions via GitHub Issues API.
- Processing Pipeline: Anti-spam and verification workflow executed by GitHub Actions.
- Datastore: Append-only encrypted JSON event log distributed via GitHub CDN (`data/messages.json`).
- Zero-Knowledge: Cryptography runs 100% client-side in the browser. GitHub only ever sees high-entropy ciphertext.

## How It Works

1. Client encrypts the message locally with AES-256-GCM using a PBKDF2-derived key (100,000 iterations).
2. Client signs the payload with an ECDSA P-256 keypair generated in browser memory to prevent identity spoofing.
3. Client solves a WebAssembly Proof-of-Work challenge (Hashcash) to rate-limit spam.
4. Encrypted envelope is posted to repository Issues as a raw JSON payload.
5. A GitHub Action validates the PoW, verifies timestamps (5-minute anti-replay window), and appends the envelope to `data/messages.json`.
6. Peers fetch the static JSON via GitHub Pages CDN, verify the ECDSA signature, and decrypt client-side.

## Wire Format

Every message is stored as an encrypted envelope:

```json
{
  "version": 1,
  "roomId": "public",
  "senderHash": "k7p227zc",
  "color": "#191970",
  "device": "desktop",
  "publicKey": "{\"kty\":\"EC\",\"crv\":\"P-256\",\"x\":\"...\",\"y\":\"...\"}",
  "signature": "MEQCIG...",
  "salt": "iKjf61b8kmYggW4cZMZgpA==",
  "iv": "i3D6w9YwPE2pSKpn",
  "ciphertext": "IvuOCeXL7OfnJbYH8...",
  "timestamp": 1788705523421,
  "pow": {
    "nonce": "2115",
    "difficulty": 12,
    "challenge": "public:k7p227zc:1788705523372"
  }
}
```

## Security Model

- Zero-Knowledge: Plaintext never leaves the browser. The host only sees ciphertext, IV, and salt.
- Anti-Spoofing: Asymmetric ECDSA P-256 signatures stored in localStorage. Forged messages are flagged in red as spoofed.
- Anti-Spam: Wasm-compiled Hashcash Proof-of-Work prevents automated flooding.
- Anti-Replay: 5-minute maximum timestamp drift.
- Tombstone Deletion: Cryptographically signed delete events allow authors to prune messages from peers' local state.
- XSS Immune: DOM rendered strictly via textContent.

## Source

Source code and build pipeline are maintained in a private repository (`zkchat-src`) and compiled/obfuscated before deployment to this showcase.
