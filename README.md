# Sorosub

**Recurring payments on Stellar — automated subscriptions and donations, settled in seconds.**

Sorosub is a subscription and recurring-donation protocol built on the Stellar network. Creators, SaaS businesses, and nonprofits define a payment plan; subscribers authorize it once via their wallet; and charges execute automatically on schedule — fully on-chain, auditable, and settled in USDC or XLM for a fraction of a cent per transaction.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Architecture](#architecture)
- [Real Stellar Features Used](#real-stellar-features-used)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Resources & Documentation](#resources--documentation)
- [Roadmap](#roadmap)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)
- [License](#license)

---

## How It Works

1. A **creator** defines a subscription plan (price, asset, billing interval) or a recurring-donation plan (open amount, recipient).
2. A **subscriber** connects a Stellar wallet (Freighter) and authorizes the plan once — this deploys or registers against a **Soroban smart contract** that holds the recurring-payment authorization.
3. On each billing cycle, the contract's `charge()` function executes automatically, transferring funds from subscriber to creator and emitting an on-chain event.
4. Subscribers can cancel at any time; cancellation requires the subscriber's own signature, so creators can't unilaterally extend or lock in a plan.
5. Every charge, cancellation, and plan creation is fully visible on-chain and linkable via a public block explorer.

---

## Architecture

```
┌───────────────┐        authorize once        ┌─────────────────────┐
│   Subscriber    │ ─────────────────────────▶ │  Soroban Subscription │
│   (Freighter)    │                              │      Contract          │
└───────────────┘                              └──────────┬───────────┘
                                                              │ charge() on schedule
                                                              ▼
                                                    ┌───────────────────┐
                                                    │      Creator        │
                                                    │  (receives payment)  │
                                                    └───────────────────┘

        All transactions settle on Stellar's public ledger via Horizon
        and are viewable on Stellar Expert.
```

Key design principle: **authorize once, charge automatically.** Stellar's base ledger has no native recurring-debit primitive, so Sorosub uses a Soroban contract to hold the subscriber's authorization and execute scheduled transfers — while claimable balances handle the case where a recurring-donation recipient doesn't yet have an active wallet.

---

## Real Stellar Features Used

| Feature | Purpose in Sorosub |
|---|---|
| **Soroban smart contracts** | Core recurring-charge logic (`charge()`, `cancel()`, `create_plan()`) |
| **Freighter wallet** | Subscriber and creator wallet connection + transaction signing |
| **Claimable balances** | Recurring donations to recipients without an active wallet yet |
| **Trustlines (`ChangeTrust`)** | Enabling accounts to hold and receive USDC on Stellar |
| **Path payments (`PathPaymentStrictSend`)** | Letting subscriber and creator use different settlement assets |
| **Horizon API** | Live account balances, transaction history, network stats |
| **Multi-sig** | Ensuring cancellation requires the subscriber's own signature |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Ledger | Stellar (Testnet/Mainnet) |
| Smart Contracts | Soroban (Rust) |
| Frontend | React + TypeScript + Tailwind |
| Wallet | Freighter |
| Data fetching | React Query |
| Charts | Recharts |
| Backend (optional indexer) | Node.js — for faster subscription/plan queries than raw Horizon polling |

---

## Getting Started

### Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) + [`soroban-cli`](https://developers.stellar.org/docs/build/smart-contracts/getting-started/setup)
- [Node.js 18+](https://nodejs.org/)
- [Freighter wallet](https://www.freighter.app/) browser extension
- A funded [Stellar Testnet account](https://developers.stellar.org/docs/tools/developer-tools/friendbot) (via Friendbot)

### Setup

```bash
git clone https://github.com/boseshittu2323-design/Sorosub.git
cd Sorosub

# Install frontend dependencies
npm install

# Build and deploy the Soroban subscription contract
cd contracts/subscription
soroban contract build
soroban contract deploy --network testnet --source <your-key>

# Run the frontend
cd ../../frontend
npm run dev
```

### Environment Variables

```env
STELLAR_NETWORK=testnet
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
HORIZON_URL=https://horizon-testnet.stellar.org
SUBSCRIPTION_CONTRACT_ID=your_deployed_contract_id
USDC_ISSUER=GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN
```

---

## Project Structure

```
sorosub/
├── contracts/
│   └── subscription/          # Soroban contract: create_plan, charge, cancel
├── frontend/                  # React app — dashboard, plan creation, subscriber view
│   └── src/
├── docs/
│   └── architecture.md
└── README.md
```

---

## Resources & Documentation

Everything you need to understand the underlying network, tools, and standards this project builds on:

- **Stellar Developer Docs** — https://developers.stellar.org/docs
- **Soroban Smart Contracts (getting started)** — https://developers.stellar.org/docs/build/smart-contracts/getting-started
- **Stellar SDK (JavaScript)** — https://developers.stellar.org/docs/tools/sdks/library
- **Horizon API Reference** — https://developers.stellar.org/docs/data/horizon
- **Freighter Wallet (browser extension + API docs)** — https://www.freighter.app/
- **Stellar Expert (block explorer)** — https://stellar.expert/explorer/testnet
- **Friendbot (Testnet funding)** — https://developers.stellar.org/docs/tools/developer-tools/friendbot
- **Claimable Balances spec** — https://developers.stellar.org/docs/learn/encyclopedia/transactions-specialized/claimable-balances
- **Path Payments spec** — https://developers.stellar.org/docs/learn/encyclopedia/transactions-specialized/path-payments
- **Circle USDC on Stellar** — https://developers.circle.com/stellar/docs/usdc-on-stellar
- **Stellar Quest (interactive learning)** — https://quest.stellar.org/

---

## Roadmap

- [x] Protocol architecture and contract design
- [ ] Soroban subscription contract (create/charge/cancel)
- [ ] Frontend: wallet connect, plan creation, subscriber view
- [ ] Claimable balance flow for walletless recipients
- [ ] Path payment support for cross-asset plans
- [ ] Indexer/backend for faster subscription queries
- [ ] Testnet pilot with a real creator/nonprofit
- [ ] Third-party contract audit
- [ ] Mainnet launch

---

## Security Considerations

- Cancellation must always require the **subscriber's own signature** — never let a creator or the contract owner unilaterally end or extend a plan.
- Contract owner/admin keys should use **multi-sig**, not a single hot wallet.
- All charge amounts and intervals should be **immutable once authorized** unless re-authorized by the subscriber, to prevent silent plan changes.
- Conduct a **third-party Soroban contract audit** before Mainnet deployment, given real funds are involved.
- Rate-limit and validate all plan-creation inputs to prevent spam or malformed billing intervals.

---

## Contributing

Contributions are welcome — open an issue to discuss significant changes before submitting a pull request.

---

## License

MIT License — see `LICENSE` file for details.
