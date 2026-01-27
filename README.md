Carbontrace Wallet SDK

Plug-and-play wallet integration for carbon incentives.
Enable businesses to reward verified carbon action with on-chain incentives (ALGO / CTCoins) using a secure, backend-authoritative ledger.

Carbon incentives, on-chain — with one script tag.

🌍 Overview

The Carbontrace Wallet SDK allows approved businesses to integrate carbon incentives into their website or application with minimal effort.

Carbontrace handles:

Carbon footprint tracking

Verified project retirement

Reward distribution on blockchain

Ledger integrity and auditability

The SDK acts as a thin client that connects user wallets and displays verified balances — all decisions are made by the Carbontrace backend.

🚀 Key Capabilities

Wallet connection via WalletConnect (Pera Wallet)

Real-time balance display (ALGO + CTCoins)

Backend-verified reward state

Automatic session expiry handling

No blockchain or smart contract knowledge required

🧠 Design Principles

Wallet = Identity
End-users are identified solely by their wallet address.

Backend = Source of Truth
All rewards, balances, and eligibility are validated server-side.

SDK = UI + Signing Layer
The SDK never contains business logic or carbon calculations.

🔐 Access & Authentication

⚠️ Important

Use of the Carbontrace Wallet SDK requires a client access token.

Tokens are not self-generated

Tokens are issued only after client verification by Carbontrace

Each token is permanently linked to an approved client account

To integrate Carbontrace, businesses must complete verification and obtain an official token from the Carbontrace team.

🔌 Integration Guide (High-Level)

Carbontrace SDK is designed to be embedded, not configured.

Integration Flow

Client completes verification with Carbontrace

Carbontrace issues a client access token

SDK is embedded into the client website/app

End-users connect their wallet

Rewards are distributed and displayed automatically

All reward logic and validation remain strictly on the backend.

🧩 How to Integrate (Minimal Snippet)
1️⃣ Add Wallet Container
<div id="carbontrace-wallet"></div>

2️⃣ Load the SDK
<script src="https://cdn.carbontrace.io/carbontrace-wallet.min.js"></script>

3️⃣ Initialize the SDK
<script>
CarbontraceWallet.init({
  apiBaseUrl: "https://api.carbontrace.io/api/v1",
  token: "CLIENT_ACCESS_TOKEN", // Issued by Carbontrace after verification
  containerId: "carbontrace-wallet"
});
</script>


📌 Notes

CLIENT_ACCESS_TOKEN must be obtained from Carbontrace

SDK automatically handles wallet connection, reconnection, and session expiry

No additional setup is required on the client side

🪙 Rewards

Carbontrace currently supports:

ALGO rewards (default)

CTCoins (Carbontrace Algorand Standard Asset)

All rewards are:

Distributed by the backend

Logged on-chain

Fully auditable

🛡️ Security Model

No private keys are ever handled by the SDK

Wallet signing occurs only inside official wallet applications

Session expiry is enforced automatically

Backend maintains full ledger authority

📄 License

MIT License

✨ Final Note

Carbontrace is not a wallet product.
It is a carbon incentive infrastructure that uses wallets as identity and blockchain as proof.
