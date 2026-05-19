Carbontrace Wallet SDK

Plug-and-play carbon wallet + checkout integration for verified climate incentives.
Enable businesses to integrate wallet onboarding, carbon project selection, redemption, and on-chain rewards (ALGO / CTCoins) using a secure backend-authoritative infrastructure.

Carbon incentives, on-chain — with one script tag.

🌍 Overview

The Carbontrace Wallet SDK allows approved businesses to integrate carbon incentives directly into their website, checkout flow, or application with minimal custom development.

Carbontrace handles:

Carbon footprint tracking
Verified project retirement
Wallet onboarding
CTCoins opt-in guidance
Reward distribution on blockchain
Redemption validation
Ledger integrity and auditability

The SDK acts as a thin client responsible for wallet connection, UI orchestration, checkout capture, and transaction signing — while all reward validation and carbon logic remain backend-controlled.

🚀 Core Features
WalletConnect integration (Pera Wallet)
Wallet onboarding flow
CTCoins opt-in guidance
Real-time ALGO + CTCoins balance display
Carbon project selector
Automatic checkout metadata capture
Automatic sale registration
Redemption handling
Backend-authoritative validation
Session expiry handling
Minimal integration effort
🧠 Design Philosophy
Wallet = Identity

Every end-user is identified solely by their wallet address.

Backend = Source of Truth

All:

rewards
balances
redemption eligibility
carbon validation
ledger state

are validated server-side.

The SDK never performs reward calculations independently.

SDK = UI + Signing Layer

The SDK only handles:

wallet connection
onboarding UI
project selection UI
checkout capture
transaction signing

No carbon logic or reward authority exists in the frontend.

🔐 Access & Authentication

⚠️ Important

The Carbontrace SDK requires a valid client access token.

Tokens:

are issued only by Carbontrace
are permanently linked to approved businesses
cannot be self-generated
are required for all SDK operations
🔌 Minimal Integration
1️⃣ Load SDK Bundle
<script src="/js/carbon-sdk-bundle.js"></script>
2️⃣ Add Default SDK Containers
<div id="ct_wallet"></div>
<div id="ct_onboarding"></div>
<div id="ct_project"></div>

These containers are automatically used when explicit container configuration is omitted.

3️⃣ Add Checkout Metadata
<div
  id="checkoutdata"
  data-invoice_amount="5000"
  data-carbon_footprint="25"
  data-carbon_charges="250"
  data-sale_order_id="order_28991"
  data-email="customer@example.com">
</div>
4️⃣ Mark Checkout Button
<button
  data-id="ctcheckout"
  data-email="customer@example.com">
  Pay now
</button>
5️⃣ Initialize SDK
<script>
document.addEventListener("DOMContentLoaded", async () => {
  await CarbontraceWallet.init({
    token: LOGIN_TOKEN
  });
});
</script>
⚙️ Default SDK Hosts

When container configuration is omitted, the SDK automatically mounts into:

Container ID	Purpose
ct_wallet	Wallet connection, balances, redemption UI
ct_onboarding	Wallet onboarding + CTCoins opt-in
ct_project	Carbon project selector

You may override these:

await CarbontraceWallet.init({
  token: LOGIN_TOKEN,
  walletContainer: "my_wallet",
  onboardingContainer: "my_onboarding",
  projectContainer: "my_projects"
});
🛒 Checkout Flow

When the checkout button marked with:

data-id="ctcheckout"

is clicked, the SDK automatically:

pauses checkout temporarily
validates invoice amount
validates carbon footprint
validates project selection
resolves redemption state
prepares sale payload
submits sale registration
resumes checkout flow

No manual payload construction is required.

📦 Checkout Metadata

Recommended metadata container:

<div
  id="checkoutdata"
  data-invoice_amount="5000"
  data-carbon_footprint="25"
  data-carbon_charges="250"
  data-sale_order_id="order_28991"
  data-email="customer@example.com">
</div>

Supported attributes:

Attribute	Required	Description
data-invoice_amount	✅	Order total before carbon charges
data-carbon_footprint	✅	Carbon footprint for the order
data-carbon_charges	Optional	Supplemental carbon charge metadata
data-sale_order_id	Optional	External order/invoice reference
data-email	Optional	Customer email

The SDK can also read standalone dataset-backed elements elsewhere on the page.

🧩 Manual Checkout Capture

For custom integrations:

const result = await CarbontraceWallet.captureData();

With overrides:

const result = await CarbontraceWallet.captureData({
  saleData: {
    sale_order_id: "order_28991",
    email: "customer@example.com"
  }
});

The SDK automatically fills:

selected project
invoice value
carbon footprint
redemption context
wallet address
client identity
🧾 Direct Sale Registration

Advanced integrations may directly call:

const result = await CarbontraceWallet.registerSale({
  sale_date: new Date().toISOString(),
  sale_order_id: "order_28991",
  project_ids: ["113"],
  invoice_value: 5000,
  carbon_footprint: 25,
  email: "customer@example.com"
});

The SDK can still auto-resolve missing values from DOM metadata and SDK state.

🎯 Carbon Project Selection

The SDK includes a built-in project selection system.

Users must select a carbon project before checkout capture succeeds.

The selected project is stored in:

walletState.selectedProjectId

and globally exposed as:

window.CT_SELECTED_PROJECT_ID
🪙 Rewards

Carbontrace currently supports:

ALGO rewards
CTCoins (Algorand Standard Asset)

Rewards are:

backend-distributed
on-chain logged
fully auditable
linked to verified carbon activity
🌱 CTCoins Opt-In Flow

The SDK includes onboarding support for CTCoins asset opt-in.

The onboarding UI guides users through:

Wallet connection
Asset opt-in
Approval confirmation
Wallet readiness state

The SDK automatically manages onboarding visibility through autoManageUi.

📡 SDK Events
Wallet & Project State
window.addEventListener("carbontrace:state", (event) => {
  console.log(event.detail.selectedProjectId);
});
Checkout Events
window.addEventListener("carbontrace:checkout:invalid", (event) => {
  console.warn(event.detail.message);
});

window.addEventListener("carbontrace:checkout:captured", (event) => {
  console.log(event.detail.result);
});
🔁 Redemption Flow (Mandatory Integration Rule)

⚠️ Critical Integration Requirement

After a successful sale registration involving redemption, the frontend must immediately call:

CarbontraceWallet.markRedemptionConsumed();

This is a mandatory system invariant.

This prevents stale redemption reuse and ensures backend-authoritative redemption integrity.

Applies to:

web checkout
POS systems
admin-triggered sales
API-based flows
🛡️ Security Model
No private keys are stored by the SDK
Wallet signing happens only inside official wallet apps
Session expiry is enforced automatically
Backend maintains complete ledger authority
Rewards cannot be manipulated client-side
🧱 System Architecture
Frontend Website/App
        ↓
Carbontrace SDK
        ↓
Carbontrace Backend APIs
        ↓
Algorand Blockchain

The SDK is intentionally lightweight.

All authoritative operations occur server-side.

🛠 Troubleshooting
Invalid or Missing Token
CarbontraceWallet.init({ token: LOGIN_TOKEN });

Ensure the token is issued by Carbontrace.

Onboarding Not Visible

Confirm:

<div id="ct_onboarding"></div>

exists on the page.

Project Selector Missing

Confirm:

<div id="ct_project"></div>

exists.

The selector appears only after onboarding/wallet readiness.

Checkout Stops With Project Alert

The user must select a carbon project before checkout proceeds.

captureData() Returns Null

Ensure:

data-carbon_footprint
data-invoice_amount

exist and are greater than zero.

✅ Quick Checklist
Add SDK bundle
Add wallet/onboarding/project containers
Add checkout metadata
Mark checkout button with data-id="ctcheckout"
Initialize SDK with valid token
Ensure project selection occurs before checkout
Call markRedemptionConsumed() after successful redemption sale
📄 License

MIT License

✨ Final Note

Carbontrace is not a wallet product.
It is a carbon incentive infrastructure where:

wallet = identity
blockchain = proof
backend = authority
carbon action = measurable value
