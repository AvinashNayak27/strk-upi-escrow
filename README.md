# STRK ↔ UPI P2P Escrow

**Verifiable on-chain escrow for STRK tokens with off-chain UPI payments, verified by an Eigen TEE.**

## TL;DR

- **Seller** locks STRK in an on-chain escrow with their UPI details.
- **Buyer** sends the UPI payment off-chain.
- A **headless browser inside an Eigen TEE** verifies the payment and signs the receipt.
- The signed proof is submitted **on-chain**, validated against the escrow, and **STRK is released to the buyer**.

---

## Architecture

```
┌─────────────┐     deposit STRK + UPI     ┌──────────────────┐
│   Seller    │ ─────────────────────────► │  Starknet        │
│             │                             │  Escrow Contract │
└─────────────┘                             └────────┬─────────┘
                                                    │
┌─────────────┐     UPI payment (off-chain)         │
│   Buyer     │ ─────────────────────────►          │
│             │     Amazon Pay / UPI                │
└──────┬──────┘                                     │
       │                                            │
       │  claim_funds(signature, receipt)           │
       └────────────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Eigen TEE       │
                    │  - Headless      │
                    │    browser       │
                    │  - Verifies UPI  │
                    │  - Signs receipt │
                    └──────────────────┘
```

## Links

| Resource | URL |
|----------|-----|
| **Contract (Voyager)** | [0x0261fca9664d38fbe3c932c27db5d036a74d3a01aafe7f9317ab3f73e7522769](https://voyager.online/contract/0x0261fca9664d38fbe3c932c27db5d036a74d3a01aafe7f9317ab3f73e7522769) |
| **Eigen TEE Verify Dashboard** | [verify-sepolia.eigencloud.xyz](https://verify-sepolia.eigencloud.xyz/app/0xc6b33ddea2c85d027800a81432bb17af2a679ee3) |

---

## Flow

1. **Deposit** – Seller deposits STRK, sets UPI ID and price per STRK (INR).
2. **Signal Intent** – Buyer signals intent to buy (1-hour lock).
3. **Pay UPI** – Buyer pays via UPI to seller’s UPI ID.
4. **Verify & Sign** – TEE opens Amazon Pay, verifies the transaction, and signs the receipt.
5. **Claim** – Buyer submits the signed proof on-chain; contract validates and releases STRK.

---

## Repo Structure

```
hello_starknet/
├── src/           # Cairo smart contracts (Starknet)
├── eigentee/      # Eigen TEE service (Playwright + Express)
├── frontend/      # React + StarkZap frontend
└── README.md
```

---

## Quick Start

### 1. Smart contracts (Cairo)

```bash
scarb build
scarb test
```

### 2. Eigen TEE service

```bash
cd eigentee
npm install
cp .env.example .env   # Set MNEMONIC
npm run build && npm start
```

### 3. Frontend

```bash
cd frontend
npm install
cp .env.example .env   # Set VITE_ESCROW_ADDRESS, VITE_TEE_SERVER
npm run dev
```

---

## Security

- **TEE attestation**: The Eigen TEE is attested; the signing key never leaves the enclave.
- **Nullifiers**: Each UPI transaction ID can be used only once.
- **Intent expiry**: Buyer has 1 hour to complete payment and claim after signaling intent.

---

## License

MIT
