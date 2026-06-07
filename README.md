# VerdeChain 🌿

> **Verified carbon credits. Permanent retirement. Full provenance.**
> A decentralized carbon credit marketplace on Stellar where carbon projects mint tokenized RWAs, corporations buy and retire them on-chain, and every credit carries an immutable audit trail from issuance to retirement.

[![Stellar](https://img.shields.io/badge/Stellar-Soroban-7C3AED?style=for-the-badge&logo=stellar&logoColor=white)](https://stellar.org)
[![Rust](https://img.shields.io/badge/Rust-Smart_Contracts-orange?style=for-the-badge&logo=rust&logoColor=white)](https://rust-lang.org)
[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org)
[![USDC](https://img.shields.io/badge/Stablecoin-USDC-2775CA?style=for-the-badge)](https://circle.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](./LICENSE)
[![Status](https://img.shields.io/badge/Status-In_Development-yellow?style=for-the-badge)](https://github.com)

---

## Table of Contents

- [The Problem](#-the-problem)
- [The Solution](#-the-solution)
- [How It Works](#️-how-it-works)
- [Architecture](#-architecture)
- [Smart Contracts](#-smart-contracts)
- [Tech Stack](#️-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Contract Deployment](#-contract-deployment)
- [Oracle Setup](#-oracle-setup)
- [Frontend Setup](#-frontend-setup)
- [Running Tests](#-running-tests)
- [User Roles](#-user-roles)
- [Credit Lifecycle](#-credit-lifecycle)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [Security](#-security)
- [License](#-license)

---

## 🎯 New Contributors Start Here!

**Want to contribute?** We've created comprehensive guides to get you started in under 30 minutes:

- 📖 **[New Contributor Guide](./docs/NEW_CONTRIBUTOR_GUIDE.md)** — Complete overview
- 🚀 **[Quick Start Guide](./docs/QUICK_START.md)** — Setup in 15–25 minutes
- ✅ **[Setup Checklist](./docs/SETUP_CHECKLIST.md)** — Verify your environment
- 🔧 **[Troubleshooting](./docs/TROUBLESHOOTING.md)** — Common issues solved
- 📋 **[Quick Reference](./docs/QUICK_REFERENCE.md)** — One-page command reference
- 🔑 **[Configuration Guide](./docs/configuration.md)** — Every environment variable explained
- ♻️ **[Credit Lifecycle](./docs/carbon-credit-lifecycle.md)** — Actors, contracts, data, and error conditions for each stage

**Run this to verify your setup:**

```bash
./scripts/verify-setup.sh    # Linux/macOS
.\scripts\verify-setup.ps1   # Windows
```

---

## The Problem

The voluntary carbon credit market moves over **$2 billion annually** — yet it is riddled with:

- **Fraud** — projects claiming credits for sequestration that never happened
- **Double-counting** — the same tonne of CO₂ sold to multiple buyers
- **Opacity** — corporations have no way to verify what they actually bought
- **Greenwashing** — retired credits with no on-chain proof of retirement
- **Inaccessibility** — small projects cannot afford traditional registry fees

The result is a market where companies pay real money for carbon credits that may not represent real impact — and have no way to prove otherwise to regulators or the public.

---

## The Solution

**VerdeChain** puts the entire carbon credit lifecycle on Stellar:

- Every credit is minted with a **unique serial number** — double counting is mathematically impossible
- Every retirement is **permanently irreversible on-chain** — greenwashing is eliminated
- Every credit carries **full provenance** — from project registration to satellite monitoring to issuance to transfer to retirement
- Every retirement generates a **verifiable certificate** with a permanent public URL
- The entire audit trail is **publicly accessible without a wallet** — regulators, journalists, and the public can verify everything

---

## ⚙️ How It Works

```
PROJECT DEVELOPER          VERDECHAIN                 CORPORATION
       │                        │                           │
       │── Submit project ─────►│                           │
       │   (methodology +       │                           │
       │    coordinates)        │                           │
       │                        │◄── Oracle monitoring ─────│
       │◄── Project verified ───│    (satellite data)       │
       │                        │                           │
       │── Request issuance ───►│                           │
       │   (verified tonnes)    │                           │
       │◄── Credits minted ─────│                           │
       │   (serial numbers      │                           │
       │    assigned)           │                           │
       │                        │◄── Browse marketplace ────│
       │                        │◄── Purchase credits ──────│
       │◄── USDC payment ───────│                           │
       │                        │◄── Retire credits ────────│
       │                        │    (beneficiary + reason) │
       │                        │──► Certificate issued ────►│
       │                        │    (permanent on-chain)   │
```

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    NEXT.JS 14 FRONTEND                       │
│   Public Audit │ Marketplace │ Buy │ Retire │ Dashboard      │
└───────────────────────────┬──────────────────────────────────┘
                            │  @stellar/stellar-sdk
                            │  @stellar/freighter-api
┌───────────────────────────▼──────────────────────────────────┐
│                  SOROBAN CONTRACTS (Rust)                    │
│  verde_registry │ verde_credit │ verde_marketplace           │
│  verde_oracle                                                │
└───────────────────────────┬──────────────────────────────────┘
                            │  py-stellar-base
┌───────────────────────────▼──────────────────────────────────┐
│            ORACLE / VERIFICATION BRIDGE (Python)             │
│  verification_listener │ price_oracle │ satellite_monitor    │
└───────────────────────────┬──────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│          OFF-CHAIN LAYER (PostgreSQL + IPFS)                 │
│  Project docs │ Credit batches │ Retirements │ Certificates  │
└──────────────────────────────────────────────────────────────┘
```

> Key architectural decisions are documented in [docs/adr/](./docs/adr). See the [ADR index](./docs/adr/README.md) for the full list.

---

## Smart Contracts

VerdeChain deploys 4 Soroban contracts written in Rust:

### `verde_registry`

Manages carbon project registration, verification, and lifecycle status.

| Function | Description |
|---|---|
| `register_project()` | Submit a new carbon project for verification |
| `verify_project()` | Accredited verifier approves a project |
| `reject_project()` | Permanently reject a fraudulent project |
| `suspend_project()` | Halt new issuance from project under investigation |
| `update_project_status()` | Oracle pushes monitoring data on-chain |
| `get_project()` | Query full project details |

### `verde_credit`

Mints, transfers, and permanently retires tokenized carbon credits.

| Function | Description |
|---|---|
| `mint_credits()` | Mint credits for verified projects with unique serial numbers |
| `retire_credits()` | Permanently and irreversibly retire credits on-chain |
| `transfer_credits()` | Transfer credits between accounts |
| `verify_serial_range()` | Detect double issuance before minting |
| `get_credit_batch()` | Query a credit batch by ID |
| `get_retirement_certificate()` | Retrieve a permanent retirement certificate |

### `verde_marketplace`

Handles credit listings, purchases, and bulk corporate buying.

| Function | Description |
|---|---|
| `list_credits()` | List credits for sale with price per tonne |
| `delist_credits()` | Remove an active listing |
| `purchase_credits()` | Buy credits — USDC to seller, credits to buyer |
| `bulk_purchase()` | Corporations buy from multiple projects in one tx |
| `get_active_listings()` | Browse all available credits |
| `get_listings_by_vintage()` | Filter credits by vintage year |

### `verde_oracle`

Receives and validates off-chain monitoring and price data.

| Function | Description |
|---|---|
| `submit_monitoring_data()` | Verifier pushes satellite monitoring data |
| `update_credit_price()` | Push benchmark price per methodology and vintage |
| `flag_project()` | Flag a project for investigation |
| `is_monitoring_current()` | Returns false if no data in last 365 days |
| `get_benchmark_price()` | Get current market price per methodology |

### Error Constants

```rust
pub enum VerdeError {
    ProjectNotFound          = 1,
    ProjectNotVerified       = 2,
    ProjectSuspended         = 3,
    InsufficientCredits      = 4,
    AlreadyRetired           = 5,
    SerialNumberConflict     = 6,
    UnauthorizedVerifier     = 7,
    UnauthorizedOracle       = 8,
    InvalidVintageYear       = 9,
    ListingNotFound          = 10,
    InsufficientLiquidity    = 11,
    PriceNotSet              = 12,
    MonitoringDataStale      = 13,
    DoubleCountingDetected   = 14,
    RetirementIrreversible   = 15,
    ZeroAmountNotAllowed     = 16,
    ProjectAlreadyExists     = 17,
    InvalidSerialRange       = 18,
}
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Smart Contracts | Rust + Soroban SDK |
| Blockchain | Stellar Mainnet / Testnet |
| Frontend | Next.js 14 (App Router) + TypeScript |
| Wallet | Freighter (@stellar/freighter-api) |
| Stellar SDK | @stellar/stellar-sdk, soroban-client |
| Payment Token | USDC on Stellar |
| Trading | Stellar DEX (SDEX) |
| Oracle Bridge | Python + py-stellar-base |
| Satellite Data | Google Earth Engine / Planet Labs |
| Price Feeds | Xpansiv CBL + Toucan Protocol |
| Database | PostgreSQL + Prisma ORM |
| File Storage | IPFS via Pinata |
| Auth | JWT + Stellar keypair + SEP-0030 |
| Backend API | NestJS |
| Testing | Rust unit tests + Stellar Testnet |

---

## Project Structure

> For the full annotated reference, see **[docs/folder-structure.md](./docs/folder-structure.md)**.

```
verdechain/
├── .github/          # CI/CD workflows, issue templates, and PR template
├── audit/            # Pre-audit checklist and security review artifacts
├── backend/          # NestJS REST API — auth, projects, credits, retirements, marketplace, oracle
│   └── prisma/
│       └── schema.prisma  # Prisma database schema — all PostgreSQL models and relations
├── components/       # Shared UI components used outside the Next.js app directory
├── contracts/        # Soroban smart contracts written in Rust
│   └── Cargo.toml    # Rust workspace manifest for all Soroban contract crates
├── docs/             # Project documentation: guides, ADRs, runbooks, API references
├── frontend/         # Next.js 14 (App Router) web application
├── hooks/            # Shared React hooks used across the monorepo
├── infra/            # Infrastructure-as-code (Terraform) for cloud provisioning
├── load-tests/       # k6 load test scripts and results for the marketplace API
├── logging/          # Observability stack: Loki, Promtail, Grafana
├── oracle/           # Python oracle bridge: verification listener, price feeds, satellite monitor
├── scripts/          # Developer utility scripts: setup, deploy, test runners, DB backup
├── tests/            # Cross-contract and upgrade path integration tests (Rust)
├── .env.example      # Environment variable template — copy to .env before running locally
├── docker-compose.yml
├── Stellar.toml      # SEP-0001 metadata file for the Stellar network
└── README.md
```

---

## Getting Started

**Estimated setup time: 25–40 minutes**

### 1. Prerequisites

Ensure you have the following installed:

- [Node.js](https://nodejs.org) v18+
- [Rust](https://rustup.rs) v1.74+
- [Python](https://python.org) v3.10+
- [PostgreSQL](https://postgresql.org) v14+
- [Redis](https://redis.io) v6+
- [Git](https://git-scm.com)

Verify your setup:

```bash
./scripts/verify-setup.sh
```

### 2. Clone and Configure

```bash
git clone https://github.com/YOUR_USERNAME/verdechain.git
cd verdechain

cp .env.example .env
# Edit .env — set DATABASE_URL, JWT_SECRET, and REDIS_PASSWORD
```

### 3. Install Rust Toolchain

```bash
rustup target add wasm32-unknown-unknown
cargo install --locked stellar-cli --version 21.0.0
```

### 4. Setup PostgreSQL

```bash
createdb verdechain
# Or manually:
# sudo -u postgres psql -c "CREATE USER verdechain WITH PASSWORD 'changeme';"
# sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE verdechain TO verdechain;"
```

### 5. Setup Redis

```bash
# macOS: brew services start redis
# Ubuntu: sudo systemctl start redis
redis-cli ping   # Should return: PONG
```

### 6. Create a Funded Testnet Account

```bash
stellar keys generate deployer --network testnet --fund
# Save the returned secret key — you'll need it for deployment
```

### 7. Install Freighter Wallet

1. Install [Freighter](https://freighter.app) for your browser
2. Switch to **Testnet** in Freighter settings
3. Optionally fund your wallet from your deployer account

### 8. Install Dependencies

```bash
# Backend
cd backend && npm install && npx prisma generate && npx prisma migrate dev && cd ..

# Frontend
cd frontend && npm install && cd ..

# Oracle
cd oracle && pip3 install -r requirements.txt && cd ..
```

### 9. Build Contracts

```bash
cd contracts
cargo build --target wasm32-unknown-unknown --release
cd ..
```

### 10. Deploy Contracts to Testnet

```bash
cd contracts

stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/verde_registry.wasm \
  --source deployer \
  --network testnet

# Repeat for verde_credit, verde_marketplace, verde_oracle
# Save all returned contract IDs to .env
```

### 11. Start Oracle Services

```bash
cd oracle
python3 verification_listener.py &
python3 price_oracle.py &
python3 satellite_monitor.py &
cd ..
```

### 12. Run Tests

```bash
./scripts/test-all.sh
```

### 13. Start Development Servers

```bash
# Terminal 1
cd backend && npm run start:dev    # → http://localhost:3001

# Terminal 2
cd frontend && npm run dev         # → http://localhost:3000
```

### 🐳 Alternative: Docker

```bash
cp .env.example .env
docker-compose up --build
```

Services started: PostgreSQL (5432), Redis (6379), NestJS Backend (3001), Next.js Frontend (3000), Oracle services (5001), Grafana observability stack.

> You still need to deploy contracts manually (step 10 above).

---

## Contract Deployment

```bash
cd contracts
cargo build --target wasm32-unknown-unknown --release

stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/verde_registry.wasm \
  --source ADMIN_SECRET_KEY \
  --network testnet

stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/verde_credit.wasm \
  --source ADMIN_SECRET_KEY \
  --network testnet

stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/verde_marketplace.wasm \
  --source ADMIN_SECRET_KEY \
  --network testnet

stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/verde_oracle.wasm \
  --source ADMIN_SECRET_KEY \
  --network testnet
```

Save all returned contract IDs to your `.env` file.

---

## Oracle Setup

```bash
cd oracle
pip install -r requirements.txt

python3 verification_listener.py   # polls every 6 hours
python3 price_oracle.py            # runs every 12 hours
python3 satellite_monitor.py       # webhook receiver on port 5001
```

---

## Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). Install [Freighter](https://freighter.app) and switch to **Testnet**.

---

## Running Tests

```bash
# All tests
./scripts/test-all.sh

# Contracts only
cd contracts && cargo test

# Backend only
cd backend && npm test

# Frontend only
cd frontend && npm test
```

### Test Coverage (30 tests across 4 contracts)

| Contract | Tests |
|---|---|
| verde_registry | 7 tests |
| verde_credit | 10 tests |
| verde_marketplace | 7 tests |
| verde_oracle | 6 tests |

---

## User Roles

**Project Developer** — Register projects, submit monitoring data, track issued vs retired credits, receive USDC payments.

**Corporation** — Browse credits by methodology, vintage year, country, and price. Purchase single or bulk credits. Retire credits and download permanent certificates for ESG reporting.

**Verifier** — Accredited verifiers approve projects for credit issuance. Submit on-chain attestations. Earn attestation fees per verified project.

**Public / Auditor** — Browse the full audit trail without a wallet. Look up any serial number. Verify retirement certificates via permanent public URL.

---

## Credit Lifecycle

```
Project Registered → Verifier Approved → Oracle Monitoring →
Credits Minted (serial numbers assigned) → Listed on Marketplace →
Purchased by Corporation → Retired On-Chain (irreversible) →
Certificate Issued (permanent public URL) →
ESG Report Filed
```

> For the full lifecycle reference — actors, contract functions, on-chain/off-chain data, error conditions, and sequence diagrams — see **[docs/carbon-credit-lifecycle.md](./docs/carbon-credit-lifecycle.md)**.

---

## Key Parameters

| Parameter | Value |
|---|---|
| Serial number uniqueness | Globally enforced across all batches |
| Retirement | Permanently irreversible on-chain |
| Oracle freshness | 365 days maximum for monitoring data |
| Price cache TTL | 24 hours |
| Methodology score minimum | 70 / 100 |
| Price deviation alert | 15% single update threshold |
| Protocol fee | 1% of each transaction |

---

## Roadmap

### Phase 1 — Contracts ✅
- [x] `verde_registry` — project registration and verification
- [x] `verde_credit` — mint, retire, transfer with serial numbers
- [x] `verde_marketplace` — list, buy, bulk purchase
- [x] `verde_oracle` — monitoring data and price feeds
- [x] 30 Rust unit tests
- [x] Stellar Testnet deployment

### Phase 2 — Oracle Layer
- [ ] Verification listener service
- [ ] Xpansiv CBL price feed integration
- [ ] Google Earth Engine satellite webhook
- [ ] End-to-end oracle → Soroban test

### Phase 3 — Frontend
- [ ] Freighter wallet integration
- [ ] Public audit explorer (no wallet required)
- [ ] Corporate bulk purchase flow
- [ ] Retirement certificate PDF generator
- [ ] Serial number lookup tool

### Phase 4 — Mainnet
- [ ] Smart contract security audit
- [ ] Gold Standard and Verra VCS methodology validation
- [ ] Mainnet deployment
- [ ] Regulatory compliance review
- [ ] Third-party registry API integrations

---

## Contributing

We welcome contributions! Here's how to get started:

**Quick Links:**
- 📖 [Quick Start Guide](./docs/QUICK_START.md)
- 📝 [Contributing Guide](./CONTRIBUTING.md)
- 🔧 [Troubleshooting](./docs/TROUBLESHOOTING.md)
- 🏷️ [Good First Issues](https://github.com/YOUR_USERNAME/verdechain/labels/good%20first%20issue)

**Development Workflow:**

```bash
# 1. Create a feature branch
git checkout -b feat/your-feature-name

# 2. Make changes and test
./scripts/test-all.sh

# 3. Commit with conventional commits
git commit -m "feat: add serial number validation"

# 4. Push and open a PR
git push origin feat/your-feature-name
```

**Code Guidelines:**
- Follow [Conventional Commits](https://www.conventionalcommits.org/)
- Use `VerdeError` enum for all contract errors
- Follow checks-effects-interactions pattern in Soroban contracts
- Retirement must always be irreversible
- Avoid crypto jargon on buyer-facing pages
- Add tests for new features
- Update documentation

---

## 🔒 Security

We take security seriously. If you discover a vulnerability:

- **Do not** open a public GitHub issue
- **Email** security@verdechain.io with details
- See [SECURITY.md](./SECURITY.md) for our full policy, threat model, and responsible disclosure process

---

## License

MIT License — see [LICENSE](./LICENSE) for details.

---

## Acknowledgements

- [Stellar Development Foundation](https://stellar.org) — Soroban and RWA infrastructure
- [Verra VCS](https://verra.org) — carbon methodology standards
- [Gold Standard](https://goldstandard.org) — verification framework
- [Xpansiv CBL](https://xpansiv.com) — carbon market price data
- [Google Earth Engine](https://earthengine.google.com) — satellite monitoring

---

**Built on Stellar. Built for the planet.**

⭐ Star this repo if VerdeChain matters to you

[Website](#) · [Audit Explorer](#) · [Twitter](#) · [Discord](#)
