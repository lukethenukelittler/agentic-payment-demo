# Agentic Payment Demo

A demonstration of AI agents discovering paid services via the **x402 Bazaar discovery layer** and completing purchases through the **x402 payment protocol** (HTTP 402 Payment Required). The agent discovers products dynamically, handles payment programmatically, and supports optional real MetaMask signing with on-chain USDC settlement on Base Sepolia.

## Architecture

```
                      GET /products
  ┌─────────┐  ◄───────────────────  ┌──────────┐
  │ Bazaar   │  { productId, name,    │  x402    │
  │ :3001    │    price, payTo, ... } │  Server  │
  └────┬─────┘  ────────────────────▶ │  :3002   │
       │                              └────┬─────┘
       │  Bazaar resource format           │
       │  { type, accepts[],               │ 402 flow:
       │    extensions.bazaar.info }       │ PAYMENT-REQUIRED
       │                                   │ PAYMENT-SIGNATURE
       │                                   │ PAYMENT-RESPONSE
  ┌────┴─────┐                      ┌──────┴─────┐
  │  Agent   │                      │ Facilitator │
  │ (Browser)│ ── settle ──────────▶│ (x402.org)  │
  └──────────┘                      └────────────┘
```

## Project Structure

```
├── x402server/          Express app with product catalog + x402 payment flow
│   ├── app.js           Core server with hardcoded products, facilitator integration
│   └── lib/             x402 facilitator client
├── mcpdiscovery/        Bazaar discovery server
│   └── index.js         Caches x402 /products, serves discovery resources
├── frontend/            Vite + React frontend
│   └── src/
│       ├── App.jsx              Chat UI orchestrator + MetaMask panel + sidebar
│       ├── sodaEngine.js        x402 client (V2 payloads) + Bazaar client + agent engine
│       ├── BazaarDemoPage.jsx   Bazaar discovery data flow trace
│       ├── MerchantPage.jsx     Merchant directory + detail pages
│       └── metamask.js          MetaMask connect/sign utilities
├── docs/
│   ├── x402-architecture.md
│   ├── adr/              Architecture Decision Records
│   ├── issues/           Issue tracking
│   └── prd/              Product Requirements
├── CONTEXT.md            Detailed project context & domain glossary
├── RUNBOOK.md            Step-by-step demo runbook
└── start-all.sh          Single-command launcher for all 3 servers
```

## Quick Start

### Prerequisites

- **Node.js** v18+
- **MetaMask** browser extension
- **Base Sepolia** testnet added to MetaMask
- **Testnet USDC** on Base Sepolia ([faucet.circle.com](https://faucet.circle.com))
- **Testnet ETH** for gas ([alchemy.com/faucets/base-sepolia](https://alchemy.com/faucets/base-sepolia))

### Install & Run

```bash
# Install dependencies
cd x402server && npm install && cd ..
cd mcpdiscovery && npm install && cd ..
cd frontend && npm install && cd ..

# Start all servers (order matters)
bash start-all.sh
```

Or manually in 3 terminals:

```bash
# Terminal 1 — x402 Payment Server (port 3002)
cd x402server && node index.js

# Terminal 2 — Bazaar Discovery (port 3001)
cd mcpdiscovery && node index.js

# Terminal 3 — Frontend (port 5173)
cd frontend && npm run dev
```

Open **http://localhost:5173** and connect MetaMask.

## Demo Pages

| Route | Purpose |
|---|---|
| `/` | Chat interface — agent discovers & pays |
| `/merchant` | Merchant directory + revenue dashboard |
| `/merchant/:walletAddress` | Individual merchant detail |
| `/debug/bazaar` | Bazaar discovery data flow trace |

## Product Catalog

| Product | Price | Merchant |
|---|---|---|
| Espresso | $2.99 | Coffee Provider |
| Latte | $3.49 | Coffee Provider |
| Cappuccino | $3.99 | Coffee Provider |
| Cold Brew | $3.29 | Coffee Provider |
| Coca-Cola Classic | $1.99 | Soft Drink Provider |
| Pepsi Cola | $1.89 | Soft Drink Provider |
| Sprite | $1.79 | Soft Drink Provider |
| Fanta Orange | $1.69 | Soft Drink Provider |
| Dasani Water | $0.99 | Water Provider |
| Smartwater | $1.49 | Water Provider |

## Payment Flow (x402 V2)

1. User clicks **Buy Now** → frontend does `GET /resource/:productId` → server responds **402** with `PAYMENT-REQUIRED`
2. Payment card appears with details (network, amount, merchant wallet)
3. User clicks **Pay** → MetaMask prompts for EIP-712 signature
4. Payment payload (V2 format) sent as `PAYMENT-SIGNATURE` header
5. Server verifies with facilitator, then settles on-chain USDC on Base Sepolia
6. NFT collectible awarded with real transaction hash

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `MERCHANT_COFFEE` | deterministic hash | Coffee provider wallet |
| `MERCHANT_SOFTDRINK` | deterministic hash | Soft drink provider wallet |
| `MERCHANT_WATER` | deterministic hash | Water provider wallet |
| `FACILITATOR_URL` | `https://x402.org/facilitator` | Settlement facilitator |
| `USDC_ADDRESS` | `0x036CbD...dCF7e` | USDC on Base Sepolia |
| `PORT` | `3002` (x402), `3001` (bazaar) | Server ports |

## Running Tests

```bash
cd x402server && npm test     # 14 tests
cd ../mcpdiscovery && npm test # 11 tests
cd ../frontend && npm test     # 31 tests
```

## License

MIT