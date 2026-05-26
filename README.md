# 🌿 Carbontrace Wallet SDK

![Version](https://img.shields.io/badge/version-2.0-green)
![License](https://img.shields.io/badge/license-MIT-blue)
![Platform](https://img.shields.io/badge/blockchain-Algorand-black)
![Integration](https://img.shields.io/badge/integration-plug--and--play-teal)

> Plug-and-play carbon wallet + checkout integration for verified climate incentives.

Enable businesses to integrate **wallet onboarding, carbon project selection, redemption, and on-chain rewards (ALGO / CTCoins)** using a secure backend-authoritative infrastructure.

---

## What Carbontrace Handles

| Capability | Owner |
|-----------|-------|
| Carbon footprint tracking | Backend |
| Verified project retirement | Backend |
| Wallet onboarding | SDK |
| CTCoins opt-in guidance | SDK |
| Reward distribution on blockchain | Backend |
| Redemption validation | Backend |
| Ledger integrity and auditability | Backend |

The SDK acts as a **thin client** responsible for wallet connection, UI orchestration, checkout capture, and transaction signing — while all reward validation and carbon logic remain **backend-controlled**.

---

## SDK Features

- ✅ WalletConnect integration (Pera Wallet)
- ✅ Impact-first onboarding flow
- ✅ CTCoins ASA opt-in with visual modal
- ✅ Real-time ALGO + CTCoins balance display
- ✅ Carbon project selector with details modal
- ✅ Automatic checkout metadata capture
- ✅ Automatic sale registration
- ✅ Redemption handling
- ✅ Transaction history with blockchain proof
- ✅ Backend-authoritative validation
- ✅ Session expiry handling
- ✅ Balance auto-polling (20s refresh)
- ✅ Toast notification system
- ✅ Shadow DOM wallet encapsulation
- ✅ Minimal integration effort

---

## Core Principles

### Wallet = Identity
Every end-user is identified solely by their wallet address.

### Backend = Source of Truth
All rewards, balances, redemption eligibility, carbon validation, and ledger state are validated server-side. The SDK never performs reward calculations independently.

### SDK = UI + Signing Layer
The SDK only handles wallet connection, onboarding UI, project selection UI, checkout capture, and transaction signing. No carbon logic or reward authority exists in the frontend.

---

## ⚠️ Token Requirement

The Carbontrace SDK requires a valid **client access token**.

- Tokens are issued only by Carbontrace
- Tokens are permanently linked to approved businesses
- Tokens cannot be self-generated
- Tokens are required for all SDK operations

**Contact Carbontrace to obtain your client token.**

---

## 🚀 Quick Start

### Step 1 — Add the SDK Bundle

```html
<script src="https://admin.carbontrace.in/js/carbon-sdk-bundle.js"></script>
```

### Step 2 — Add SDK Containers

```html
<div id="ct_wallet"></div>
<div id="ct_onboarding"></div>
<div id="ct_project"></div>
```

These are the default container IDs. The SDK mounts its UI into these automatically. If any container is missing from your page, the SDK creates it for you.

### Step 3 — Add Checkout Metadata

```html
<div id="checkoutdata"
  data-invoice_amount="5000"
  data-carbon_footprint="25"
  data-carbon_charges="250"
  data-sale_order_id="order_28991"
  data-email="customer@example.com"
  style="display:none">
</div>
```

### Step 4 — Mark Checkout Button

```html
<button data-id="ctcheckout" data-email="customer@example.com">
  Pay now
</button>
```

### Step 5 — Initialize SDK

```html
<script>
document.addEventListener("DOMContentLoaded", async () => {
  await CarbontraceWallet.init({
    token: "YOUR_CLIENT_TOKEN"
  });
});
</script>
```

**That's it.** The SDK handles the rest — onboarding, wallet connection, project selection, checkout capture, and sale registration.

---

## ⚙️ Configuration Options

Pass these options to `CarbontraceWallet.init(config)`:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `token` | `string` | *required* | Client access token issued by Carbontrace |
| `walletContainer` | `string \| Element` | `"ct_wallet"` | Container for wallet UI |
| `onboardingContainer` | `string \| Element` | `"ct_onboarding"` | Container for onboarding flow |
| `projectContainer` | `string \| Element` | `"ct_project"` | Container for project selector |
| `embedWallet` | `boolean` | `false` | Embed wallet inline instead of floating |
| `autoManageUi` | `boolean` | `true` | Auto-show/hide onboarding and project UI based on state |
| `asaId` | `string` | Auto-detected | CTCoins ASA ID override |
| `chainId` | `number` | `416002` (testnet) | Algorand chain ID |

### Callback Options

| Callback | Signature | Fires When |
|----------|-----------|------------|
| `onConnect` | `(address, snapshot)` | Wallet successfully connected |
| `onDisconnect` | `()` | Wallet disconnected |
| `onSessionExpired` | `()` | JWT token expired |
| `onChainPending` | `()` | Blockchain transaction pending |
| `onChainConfirmed` | `()` | Blockchain transaction confirmed |
| `onChainFailed` | `()` | Blockchain transaction failed |

**Example with callbacks:**

```js
await CarbontraceWallet.init({
  token: "YOUR_CLIENT_TOKEN",
  onConnect: (address, snapshot) => {
    console.log("Wallet connected:", address);
    console.log("ALGO balance:", snapshot?.algo?.algos);
  },
  onDisconnect: () => {
    console.log("Wallet disconnected");
  },
  onChainConfirmed: () => {
    console.log("Blockchain transaction confirmed!");
  }
});
```

---

## Default SDK Hosts

When container configuration is omitted, the SDK automatically mounts into:

| Container ID | Purpose |
|---|---|
| `ct_wallet` | Wallet connection, account status, balances, transaction history, redemption UI |
| `ct_onboarding` | Impact-first onboarding flow + CTCoins opt-in |
| `ct_project` | Carbon project selector with project details |

You may override these:

```js
await CarbontraceWallet.init({
  token: "YOUR_CLIENT_TOKEN",
  walletContainer: "my_wallet",
  onboardingContainer: "my_onboarding",
  projectContainer: "my_projects"
});
```

If no matching container exists on the page, the SDK creates default containers and appends them near the checkout metadata element.

---

## 🛒 Checkout Flow

When the checkout button marked with `data-id="ctcheckout"` is clicked, the SDK automatically:

1. Pauses checkout temporarily (prevents default click action)
2. Validates invoice amount is present and greater than zero
3. Validates carbon footprint is present and greater than zero
4. Validates that a carbon project has been selected
5. Resolves any active redemption state
6. Prepares the complete sale payload (enriched with wallet address, client ID, session ID)
7. Submits sale registration to the backend
8. Fires `carbontrace:checkout:captured` event with the result
9. Resumes checkout flow

**No manual payload construction is required.**

---

## 📋 Checkout Metadata

### Recommended Metadata Container

```html
<div id="checkoutdata"
  data-invoice_amount="5000"
  data-carbon_footprint="25"
  data-carbon_charges="250"
  data-sale_order_id="order_28991"
  data-email="customer@example.com"
  data-phone="9876543210"
  data-customer_id="cust_12345"
  style="display:none">
</div>
```

### Supported Attributes

| Attribute | Required | Description |
|-----------|:--------:|-------------|
| `data-invoice_amount` | ✅ | Order total before carbon charges |
| `data-carbon_footprint` | ✅ | Carbon footprint for the order (kg CO₂) |
| `data-carbon_charges` | — | Supplemental carbon charge metadata |
| `data-sale_order_id` | — | External order/invoice reference. Auto-generated as `order_{timestamp}` if missing |
| `data-email` | — | Customer email address |
| `data-phone` | — | Customer phone or mobile number |
| `data-endUserId` | — | Your system's customer/user ID |
| `data-customer_id` | — | Alias for `endUserId` |

> **Note:** The SDK also reads `data-invoice_value` as an alias for `data-invoice_amount`.

The SDK can also read individual `data-*` attributes from standalone elements elsewhere on the page:

```html
<div data-invoice_amount="5000"></div>
<div data-carbon_footprint="25"></div>
```

### Checkout Button Attributes

The checkout button can carry additional metadata:

```html
<button
  data-id="ctcheckout"
  data-email="customer@example.com"
  data-sale_order_id="order_28991"
  data-phone="9876543210">
  Pay now
</button>
```

The SDK reads `data-email`, `data-sale_order_id`, and `data-phone` from the button when present.

---

## 🧩 Manual Checkout Capture

For custom checkout flows where you don't use the `data-id="ctcheckout"` button binding:

```js
const result = await CarbontraceWallet.captureData();
```

With overrides:

```js
const result = await CarbontraceWallet.captureData({
  saleData: {
    sale_order_id: "order_28991",
    email: "customer@example.com",
    phone: "9876543210",
    endUserId: "cust_12345"
  }
});
```

The SDK automatically fills:

- Selected project (`project_ids`)
- Invoice value (from DOM)
- Carbon footprint (from DOM)
- Redemption context
- Wallet address
- Client identity (from JWT token)
- Session ID (auto-generated)

---

## 🧾 Direct Sale Registration

Advanced integrations may directly call:

```js
const result = await CarbontraceWallet.registerSale({
  sale_date: new Date().toISOString(),
  sale_order_id: "order_28991",
  project_ids: ["113"],
  invoice_value: 5000,
  carbon_footprint: 25,
  email: "customer@example.com",
  phone: "9876543210",
  endUserId: "cust_12345"
});
```

Even in direct mode, the SDK can auto-resolve missing `invoice_value`, `carbon_footprint`, and `project_ids` from DOM metadata and SDK state.

The response includes an `invoice_breakdown` object with detailed pricing data.

---

## 🎯 Carbon Project Selection

The SDK includes a built-in project selection system with interactive cards.

- Users must select a carbon project before checkout capture succeeds
- Each project shows type badges and a details button (`ⓘ`)
- Clicking the details button opens a full **Project Details Modal** with carbon retirement data
- The selected project is stored in `walletState.selectedProjectId` and globally as `window.CT_SELECTED_PROJECT_ID`

To load the project selector into a custom container:

```js
CarbontraceWallet.loadProjects("my_project_container");
```

---

## 🌱 Onboarding Flow

The SDK includes an impact-first onboarding experience:

| Step | Name | Purpose |
|------|------|---------|
| 1 | **Impact Hook** | Inspire action with climate vision and impact metrics |
| 2 | **Project Selection** | Choose a carbon project track |
| 3 | **Wallet Choice** | Connect via Pera Wallet or use managed flow |
| 4 | **Wallet Connection** | QR scan / deep-link to Pera Wallet |
| 5 | **Confirmation** | Verify wallet setup is complete |
| 6 | **All Done** | Onboarding finished, SDK switches to active mode |

The SDK automatically manages onboarding visibility through `autoManageUi`:
- Onboarding hides after completion
- Project selector appears when the user is wallet-ready
- Wallet UI is always visible

**Programmatic control:**

```js
// Mount onboarding into a specific container
CarbontraceWallet.initOnboarding("my_container");

// Restart onboarding
CarbontraceWallet.startOnboarding();

// Skip/dismiss onboarding
CarbontraceWallet.skipOnboarding();
```

---

## 🪙 Rewards

Carbontrace currently supports:

- **ALGO rewards** — native Algorand tokens
- **CTCoins** — Algorand Standard Asset (ASA)

Rewards are:
- Backend-distributed
- On-chain logged
- Fully auditable
- Linked to verified carbon activity

### CTCoins Opt-In

Before a wallet can receive CTCoins, it must opt-in to the ASA. The SDK provides a visual opt-in modal:

```js
await CarbontraceWallet.optInCTCoins();
```

This opens a step-by-step modal:
1. **Opt-In Required** — explanation of ASA
2. **Review Asset Details** — show ASA ID, confirm
3. **Transaction Pending** — waiting for blockchain confirmation
4. **Confirmed** — opt-in successful

---

## 🔁 Redemption Flow

> ⚠️ **Critical Integration Requirement**

After a successful sale registration involving redemption, the frontend **must** immediately call:

```js
CarbontraceWallet.markRedemptionConsumed();
```

This prevents stale redemption reuse and ensures backend-authoritative redemption integrity.

**Applies to:**
- Web checkout
- POS systems
- Admin-triggered sales
- API-based flows

### Redemption Context

The SDK resolves redemption from multiple sources (in priority order):

1. SDK internal state (`getRedeemContext()`)
2. `captureData({ saleData: { redemption_context: {...} } })`
3. `window.CT_REDEEM_CONTEXT` global
4. `sessionStorage.getItem("ct_redeem_context")`

---

## 🔧 Public Methods

| Method | Description |
|--------|-------------|
| `CarbontraceWallet.init(config)` | Initialize the SDK with configuration |
| `CarbontraceWallet.connectWallet()` | Open wallet connection dialog (Pera Wallet QR/deep-link) |
| `CarbontraceWallet.disconnectWallet()` | Disconnect the current wallet session |
| `CarbontraceWallet.captureData(options?)` | Capture checkout data and register sale automatically |
| `CarbontraceWallet.registerSale(saleData)` | Register a sale directly with explicit data |
| `CarbontraceWallet.validateCheckoutData(data?)` | Validate checkout data without submitting |
| `CarbontraceWallet.markRedemptionConsumed()` | Clear active redemption after successful use |
| `CarbontraceWallet.optInCTCoins()` | Trigger CTCoins ASA opt-in modal flow |
| `CarbontraceWallet.loadProjects(container)` | Load the project selector into a container |
| `CarbontraceWallet.initOnboarding(container)` | Mount onboarding flow into a container |
| `CarbontraceWallet.startOnboarding()` | Restart the onboarding flow |
| `CarbontraceWallet.skipOnboarding()` | Skip/dismiss onboarding UI |

> 📖 For detailed method signatures and return types, see [API Reference](docs/API_REFERENCE.md).

---

## 📡 SDK Events

Listen for SDK events using standard `window.addEventListener`:

### State Events

```js
window.addEventListener("carbontrace:state", (event) => {
  console.log("Wallet:", event.detail.currentWallet);
  console.log("Project:", event.detail.selectedProjectId);
  console.log("Onboarding done:", event.detail.onboardingComplete);
});
```

| Field | Type | Description |
|-------|------|-------------|
| `currentWallet` | `string \| null` | Connected wallet address |
| `walletState` | `object \| null` | Full wallet state object |
| `onboardingComplete` | `boolean` | Whether onboarding is finished |
| `selectedProject` | `object \| null` | Selected project metadata |
| `selectedProjectId` | `string \| null` | Selected project ID |

### Checkout Events

```js
// Before sale submission
window.addEventListener("carbontrace:checkout:capture", (event) => {
  console.log("Submitting sale:", event.detail.saleData);
});

// After successful sale
window.addEventListener("carbontrace:checkout:captured", (event) => {
  console.log("Sale result:", event.detail.result);
  // Proceed with your payment flow
});

// Validation failed
window.addEventListener("carbontrace:checkout:invalid", (event) => {
  console.warn("Blocked:", event.detail.message);
});

// Sale skipped (project selection not mandatory + none selected)
window.addEventListener("carbontrace:checkout:skipped", (event) => {
  console.log("Sale skipped:", event.detail.checkoutData);
});
```

---

## 🌐 Global Variables

The SDK exposes and reads these window globals for cross-component communication:

### SDK → Page (readable)

| Variable | Type | Description |
|----------|------|-------------|
| `window.CarbontraceWallet` | `class` | The SDK class itself |
| `window.CT_SELECTED_PROJECT_ID` | `string` | Currently selected project ID |
| `window.CT_ONBOARDING_PROJECT` | `object` | Selected project metadata (`{ projectId, name, description, metadata }`) |

### Page → SDK (writable fallbacks)

| Variable | Type | Description |
|----------|------|-------------|
| `window.CT_CUSTOMER_EMAIL` | `string` | Customer email (used if not in DOM or payload) |
| `window.CT_CUSTOMER_PHONE` | `string` | Customer phone (used if not in DOM or payload) |
| `window.CT_END_USER_ID` | `string` | Your system's customer ID (used if not in DOM or payload) |
| `window.CT_REDEEM_CONTEXT` | `object` | Redemption context object |

**Example — setting customer data globally:**

```js
window.CT_CUSTOMER_EMAIL = "customer@example.com";
window.CT_CUSTOMER_PHONE = "9876543210";
window.CT_END_USER_ID = "cust_12345";
```

These are used as fallbacks if the data isn't available in checkout metadata or `captureData()` overrides.

---

## 🧱 System Architecture

```
┌─────────────────────────────────┐
│     Client Website / App        │
│                                 │
│  ┌───────────────────────────┐  │
│  │   Carbontrace SDK         │  │
│  │   (carbon-sdk-bundle.js)  │  │
│  │                           │  │
│  │  • Wallet UI              │  │
│  │  • Onboarding Flow        │  │
│  │  • Project Selector       │  │
│  │  • Checkout Capture       │  │
│  │  • Transaction Signing    │  │
│  └──────────┬────────────────┘  │
└─────────────┼───────────────────┘
              │ HTTPS (Bearer Token)
              ▼
┌─────────────────────────────────┐
│   Carbontrace Backend APIs      │
│   admin.carbontrace.in/api/v1   │
│                                 │
│  • Sale Registration            │
│  • Reward Calculation           │
│  • Wallet Sessions              │
│  • Carbon Validation            │
│  • Invoice & Billing            │
│  • Certificate Generation       │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Algorand Blockchain           │
│                                 │
│  • ALGO Rewards (native)        │
│  • CTCoins (ASA)                │
│  • On-chain proof               │
│  • IPFS metadata pinning        │
└─────────────────────────────────┘
```

The SDK is intentionally lightweight. All authoritative operations occur server-side.

---

## 🛡️ Security Model

- No private keys are stored by the SDK
- Wallet signing happens only inside official wallet apps (Pera Wallet)
- Session expiry is enforced automatically (JWT-based)
- Backend maintains complete ledger authority
- Rewards cannot be manipulated client-side
- SDK uses Shadow DOM to isolate wallet UI from host page styles

---

## 📦 Complete Integration Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Store - Checkout</title>
</head>
<body>

  <!-- 1. SDK Containers -->
  <div id="ct_wallet"></div>
  <div id="ct_onboarding"></div>
  <div id="ct_project"></div>

  <!-- 2. Checkout Metadata (hidden, SDK reads this automatically) -->
  <div id="checkoutdata"
    data-invoice_amount="5000"
    data-carbon_footprint="25"
    data-carbon_charges="250"
    data-sale_order_id="order_28991"
    data-email="customer@example.com"
    data-phone="9876543210"
    style="display:none">
  </div>

  <!-- 3. Your Checkout Button -->
  <button data-id="ctcheckout" data-email="customer@example.com">
    Pay ₹5,000
  </button>

  <!-- 4. Result Display (optional) -->
  <div id="result_output" style="display:none">
    <strong>Registration Result:</strong>
    <pre id="result_json"></pre>
  </div>

  <!-- 5. Load SDK Bundle -->
  <script src="https://admin.carbontrace.in/js/carbon-sdk-bundle.js"></script>

  <!-- 6. Initialize -->
  <script>
    document.addEventListener("DOMContentLoaded", async () => {
      await CarbontraceWallet.init({
        token: "YOUR_CLIENT_TOKEN",

        // Optional callbacks
        onConnect: (address, snapshot) => {
          console.log("Wallet connected:", address);
          console.log("ALGO balance:", snapshot?.algo?.algos);
        },
        onDisconnect: () => {
          console.log("Wallet disconnected");
        },
        onSessionExpired: () => {
          console.warn("Session expired — redirect to login");
        },
        onChainConfirmed: () => {
          console.log("Blockchain confirmed!");
        }
      });

      // Listen for successful sale registration
      window.addEventListener("carbontrace:checkout:captured", (event) => {
        console.log("Sale registered:", event.detail.result);
        // Proceed with your payment flow
      });

      // Listen for validation errors
      window.addEventListener("carbontrace:checkout:invalid", (event) => {
        console.warn("Checkout blocked:", event.detail.message);
      });
    });
  </script>

</body>
</html>
```

---

## 🔍 Troubleshooting

### Invalid or Missing Token

```js
CarbontraceWallet.init({ token: LOGIN_TOKEN });
```

Ensure the token is a valid JWT issued by Carbontrace. Expired tokens trigger `onSessionExpired`.

### Onboarding Not Visible

Confirm `<div id="ct_onboarding"></div>` exists on the page, or pass `onboardingContainer` explicitly in `init()`.

### Project Selector Missing

Confirm `<div id="ct_project"></div>` exists. The selector appears only after onboarding completion or when a wallet is already connected.

### Checkout Stops With Project Alert

The user must select a carbon project before checkout proceeds. The alert message is: *"Please select a Carbon Project first!"*

### `captureData()` Returns Null

Ensure `data-carbon_footprint` and `data-invoice_amount` are present on the page and their values are greater than zero.

### Wallet Connection Fails

- Check that the user has Pera Wallet installed (mobile) or a supported browser extension
- On desktop, a QR code is shown for scanning with Pera Wallet
- Network errors are shown via toast notifications

### Balance Not Updating

The SDK polls for balance updates every 20 seconds. After a sale, it may take a few seconds for rewards to appear in the wallet UI.

---

## ✅ Integration Checklist

- [ ] Add SDK bundle (`<script src="https://admin.carbontrace.in/js/carbon-sdk-bundle.js">`)
- [ ] Add wallet/onboarding/project containers (`ct_wallet`, `ct_onboarding`, `ct_project`)
- [ ] Add checkout metadata (`#checkoutdata` with `data-invoice_amount` and `data-carbon_footprint`)
- [ ] Mark checkout button with `data-id="ctcheckout"`
- [ ] Initialize SDK with valid token (`CarbontraceWallet.init({ token })`)
- [ ] Ensure project selection occurs before checkout
- [ ] Call `markRedemptionConsumed()` after successful redemption sale
- [ ] Listen for `carbontrace:checkout:captured` event for sale confirmation
- [ ] Test wallet connection on both mobile and desktop

---

## 📖 Documentation

- [Integration Guide](docs/INTEGRATION.md) — Step-by-step walkthrough for every integration scenario
- [API Reference](docs/API_REFERENCE.md) — Complete method signatures, events, and data shapes

---

## 📄 License

MIT License

---

## ✨ Final Note

> Carbontrace is not a wallet product.
>
> It is a **carbon incentive infrastructure** where:
>
> - **wallet** = identity
> - **blockchain** = proof
> - **backend** = authority
> - **carbon action** = measurable value
