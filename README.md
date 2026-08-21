# ShadowPass

> Privacy-preserving membership verification on the Midnight Network using zero-knowledge proofs.

ShadowPass is a complete Midnight dApp for private allowlist access. It allows users to prove they are members of an authorized allowlist using a zero-knowledge proof without revealing their underlying member identity or salt. A Groth16 ZK proof is generated entirely in the browser and verified on the Midnight Preprod network.

## Quick Start

```bash
# Install dependencies
npm install
npm --prefix frontend install

# Compile the contract
npm run compile

# Copy ZK assets to frontend
npm run copy-zk-assets

# Run unit tests
npm test

# Build the frontend
npm run build:frontend

# Start dev server
npm run dev:frontend
```

Open `http://localhost:5173` and connect a Midnight-compatible wallet (1AM or Lace).

## Architecture

ShadowPass is a complete Midnight dApp with a Compact smart contract, browser-based proving, wallet integration, and a production-style frontend.

```
User
  ↓
Midnight Wallet (1AM / Lace — DApp Connector)
  ↓
ShadowPass dApp (React + Vite — browser only)
  ↓
ZK Proof (Groth16 — browser-delegated proving via wallet)
  ↓
ShadowPass Smart Contract (Compact — on-chain verification)
  ↓
Midnight Preprod Network
```

### Providers

The app uses 7 midnight-js providers assembled in `frontend/src/midnight/providers.ts`:

| Provider | Source |
|----------|--------|
| ZK Config | `FetchZkConfigProvider` (static assets) |
| Proof | `connectedAPI.getProvingProvider()` (browser WASM) |
| Private State | `InMemoryPrivateStateProvider` (browser memory) |
| Public Data | `indexerPublicDataProvider()` (indexer API) |
| Wallet | `connectedAPI` (DApp Connector) |
| Midnight | `connectedAPI` (DApp Connector) |

### Contract

The ShadowPass contract (Compact) supports:

- `constructor(members: Vector<8, Bytes<32>>)` — initializes the allowlist
- `proveMembership(memberId: Bytes<32>, salt: Bytes<32>)` — verifies ZK proof and increments `accessCount`

The ZK circuit computes `persistentCommit(memberId, salt)` and verifies it matches one of the 8 on-chain commitments.

## Privacy Model

| What | On-Chain | Private |
|------|----------|---------|
| Member ID | Hidden (via ZK proof) | User's browser |
| Salt | Hidden (via ZK proof) | User's browser |
| Allowlist commitment | Visible (8 x 32 bytes) | N/A |
| Access count | Visible (increments) | N/A |
| ZK proof | Visible (proof of membership) | N/A |

The zero-knowledge proof proves "I know a valid credential" without revealing which credential. Multiple verifications by the same member cannot be linked on-chain.

## Demo Credentials

This demonstration uses public, documented credentials:

- **Member ID:** `deadbeef000000000000000000000000000000000000000000000000deadbeef`
- **Salt:** `cafebabe000000000000000000000000000000000000000000000000cafebabe`

See `docs/evidence/DEMO-CREDENTIALS.md` for full details.

**Production credential model:** Credentials are generated and distributed privately by an authorized issuer.

## Testing

```bash
# Run all tests
npm test

# Run tests in watch mode
npx vitest
```

Tests validate:
1. CompactContract creation and witness generation
2. Groth16 proof generation and verification
3. ZK circuit constraint satisfaction
4. Invalid credential rejection

## Project Structure

```
ShadowPass/
├── contracts/
│   └── shadowpass.compact          # Compact contract source
├── docs/
│   ├── evidence/
│   │   ├── DOCUMENT.md             # Canonical deployment evidence
│   │   ├── ARCHITECTURE-BLUEPRINT.md  # Full architecture design
│   │   ├── DEMO-CREDENTIALS.md     # Demo credential documentation
│   │   └── DEPLOYMENT-INPUTS.md    # Allowlist configuration
│   └── history/
│       ├── PHASE-0-FEASIBILITY.md  # Feasibility report
│       ├── PHASE-1-RESULT.md       # Phase 1 result
│       ├── PHASE-2-DESIGN.md       # Phase 2 design document
│       ├── PHASE-4-PREDEPLOYMENT-AUDIT.md  # Predeployment audit
│       ├── PHASE-5-DEPLOYMENT-READY.md     # Deployment readiness
│       └── PHASE-6-DEPLOYMENT-PROCEDURE.md # Deployment procedure
├── frontend/
│   ├── src/
│   │   ├── App.tsx                 # Main application
│   │   ├── main.tsx                # Entry point
│   │   ├── config.ts               # Configuration
│   │   ├── compiled-contract.js    # Compiled contract
│   │   ├── components/             # UI components
│   │   ├── hooks/                  # React hooks
│   │   ├── midnight/               # Midnight provider layer
│   │   └── shims/                  # Browser polyfills
│   ├── vite.config.ts              # Vite configuration
│   └── package.json                # Frontend dependencies
├── tests/
│   ├── shadowpass.test.ts          # Contract/circuit tests
│   └── wallet-state.test.ts        # Wallet state tests
├── package.json                    # Root dependencies
├── tsconfig.json                   # TypeScript configuration
├── vitest.config.ts                # Test configuration
└── README.md                       # This file
```

## Tech Stack

- **Compact** — ZK circuit language for Midnight
- **@midnight-ntwrk/compact-runtime** — Runtime for compiled contracts
- **@midnight-ntwrk/midnight-js** — JavaScript SDK for Midnight dApps
- **@midnight-ntwrk/dapp-connector-api** — Wallet connection API
- **React 19** — UI framework
- **Vite** — Build tool and dev server
- **Vitest** — Test framework

## License

MIT
