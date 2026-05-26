# Carbontrace SDK — API Reference

> Complete reference for all public methods, events, global variables, and data shapes.

---

## Table of Contents

1. [Initialization](#initialization)
2. [Wallet Methods](#wallet-methods)
3. [Checkout Methods](#checkout-methods)
4. [Onboarding Methods](#onboarding-methods)
5. [Project Methods](#project-methods)
6. [Redemption Methods](#redemption-methods)
7. [Events](#events)
8. [Global Variables](#global-variables)
9. [Init Config Options](#init-config-options)
10. [Checkout Data Attributes](#checkout-data-attributes)
11. [Wallet State Object](#wallet-state-object)
12. [Response Shapes](#response-shapes)

---

## Initialization

### `CarbontraceWallet.init(config)`

Initialize the SDK. Must be called once before any other method.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `config` | `object` | ✅ | Configuration object (see [Init Config Options](#init-config-options)) |

**Returns:** `Promise<void>`

**Example:**

```js
await CarbontraceWallet.init({
  token: "eyJhbGciOiJIUzI1NiIs...",
  onConnect: (address, snapshot) => {
    console.log("Connected:", address);
  }
});
```

**Behavior:**
1. Sets up API base URL and authentication
2. Creates or resolves SDK containers
3. Mounts wallet UI (floating or embedded)
4. Attempts silent wallet reconnection from previous session
5. If no session, renders disconnected state and shows onboarding
6. Binds checkout button capture (`data-id="ctcheckout"`)
7. Starts JWT token expiry watcher

---

## Wallet Methods

### `CarbontraceWallet.connectWallet()`

Opens the Pera Wallet connection dialog. On desktop, shows a QR code. On mobile, deep-links to the Pera Wallet app.

**Parameters:** None

**Returns:** `Promise<void>`

**Example:**

```js
await CarbontraceWallet.connectWallet();
```

**Behavior:**
- If already connecting, does nothing (debounced)
- Shows a toast on mobile-less desktops: "Scan QR using Pera / Defly / MyAlgo"
- On success: calls `loadWallet()`, renders connected state, triggers `onConnect`
- On failure: shows toast with error message
- On cancel: shows "Wallet connection cancelled" toast

---

### `CarbontraceWallet.disconnectWallet()`

Disconnects the current wallet session.

**Parameters:** None

**Returns:** `Promise<void>`

**Example:**

```js
await CarbontraceWallet.disconnectWallet();
```

**Behavior:**
- Disconnects Pera Wallet session
- Stops balance polling
- Resets all internal state
- Renders disconnected UI
- Triggers `onDisconnect` callback

---

### `CarbontraceWallet.optInCTCoins()`

Opens the CTCoins ASA opt-in modal. This is a multi-step visual flow that guides the user through opting into the CTCoins Algorand Standard Asset.

**Parameters:** None

**Returns:** `Promise<void>`

**Example:**

```js
await CarbontraceWallet.optInCTCoins();
```

**Behavior:**
1. Opens modal: "Opt-In Required" — explains what ASA opt-in means
2. User clicks "Opt-In Now" → shows ASA details and "Approve in Pera Wallet"
3. Creates opt-in transaction via backend (`POST /wallet/optin-txn`)
4. Sends transaction to Pera Wallet for signing
5. Submits signed transaction (`POST /wallet/submit-optin`)
6. Polls for blockchain confirmation (up to 30 attempts, 2s interval)
7. On success: shows confirmation, triggers reward retry, closes modal
8. On failure/cancel: shows error toast, closes modal

**Notes:**
- If already in progress, subsequent calls are ignored
- Wallet must be connected before calling
- Wallet is auto-funded if needed (backend handles minimum ALGO balance)

---

## Checkout Methods

### `CarbontraceWallet.captureData(options?)`

Captures checkout data from the page, validates it, and registers a sale.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `options` | `object` | — | Override options |
| `options.saleData` | `object` | — | Sale data overrides |
| `options.saleData.sale_order_id` | `string` | — | Order ID |
| `options.saleData.email` | `string` | — | Customer email |
| `options.saleData.phone` | `string` | — | Customer phone |
| `options.saleData.endUserId` | `string` | — | Your customer ID |
| `options.saleData.invoice_value` | `number` | — | Invoice amount |
| `options.saleData.carbon_footprint` | `number` | — | Carbon footprint (kg CO₂) |
| `options.saleData.project_ids` | `string[]` | — | Project IDs |
| `options.saleData.apply_redemption` | `boolean` | — | Force apply redemption |
| `options.saleData.redemption_context` | `object` | — | Redemption context |

**Returns:** `Promise<object | null>` — Sale result or `null` on failure

**Example:**

```js
const result = await CarbontraceWallet.captureData({
  saleData: {
    sale_order_id: "order_28991",
    email: "customer@example.com"
  }
});
```

**Auto-resolved fields:**
- `sale_date` — current ISO timestamp
- `sale_order_id` — `order_{timestamp}` if missing
- `project_ids` — from `walletState.selectedProjectId`
- `invoice_value` — from `#checkoutdata` dataset
- `carbon_footprint` — from `#checkoutdata` dataset
- `email` — from trigger element, checkout data, or `window.CT_CUSTOMER_EMAIL`
- `phone` — from trigger element, checkout data, or `window.CT_CUSTOMER_PHONE`
- `endUserId` — from trigger element, checkout data, or `window.CT_END_USER_ID`
- `sessionId` — auto-generated persistent end-user session ID
- `apply_redemption` — `true` if any redemption context is found
- `redemption_context` — resolved from SDK state, payload, globals, or sessionStorage

**Events fired:**
- `carbontrace:checkout:capture` — before submission
- `carbontrace:checkout:captured` — after successful submission
- `carbontrace:checkout:invalid` — if validation fails
- `carbontrace:checkout:skipped` — if project is optional and none selected

---

### `CarbontraceWallet.registerSale(saleData)`

Registers a sale directly with explicit data. Lower-level than `captureData()`.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `saleData` | `object` | ✅ | Sale data object |
| `saleData.sale_date` | `string` | — | ISO date string |
| `saleData.sale_order_id` | `string` | — | Order reference |
| `saleData.project_ids` | `string[]` | — | Carbon project IDs |
| `saleData.invoice_value` | `number` | — | Invoice amount (auto-resolved if missing) |
| `saleData.carbon_footprint` | `number` | ✅ | Carbon footprint in kg CO₂ (must be > 0) |
| `saleData.email` | `string` | — | Customer email |
| `saleData.phone` | `string` | — | Customer phone |
| `saleData.endUserId` | `string` | — | Your customer ID |
| `saleData.apply_redemption` | `boolean` | — | Whether to apply redemption |
| `saleData.redemption_context` | `object` | — | Redemption context |

**Returns:** `Promise<object | null>` — Sale result or `null` on failure

**Example:**

```js
const result = await CarbontraceWallet.registerSale({
  sale_date: new Date().toISOString(),
  sale_order_id: "order_28991",
  project_ids: ["113"],
  invoice_value: 5000,
  carbon_footprint: 25,
  email: "customer@example.com"
});
```

**Returns `null` when:**
- `saleData` is null/undefined
- `carbon_footprint` is missing or ≤ 0
- JWT token cannot be parsed for `client_id`
- API call fails

---

### `CarbontraceWallet.validateCheckoutData(data?)`

Validates checkout data without submitting. Useful for pre-flight checks.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `data` | `object` | — | Data to validate (merged with DOM metadata) |
| `data.invoice_value` | `number` | — | Invoice amount to check |
| `data.carbon_footprint` | `number` | — | Carbon footprint to check |

**Returns:** `object`

```js
{
  valid: boolean,           // true if all checks pass
  message: string | null,   // error message if invalid
  checkoutData: object,     // detected DOM metadata
  selectedProjectId: string | null,  // currently selected project
  skipSale: boolean         // true if project optional + none selected
}
```

**Validation checks (in order):**
1. `invoice_value` must be finite and > 0 → `"Checkout invoice amount is missing."`
2. `carbon_footprint` must be finite and > 0 → `"Checkout carbon footprint is missing."`
3. `selectedProjectId` must exist → `"Please select a Carbon Project first!"`

---

## Onboarding Methods

### `CarbontraceWallet.initOnboarding(containerIdOrEl)`

Mounts the onboarding flow into a container.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `containerIdOrEl` | `string \| Element` | ✅ | Container element or its ID |

**Returns:** `void`

**Example:**

```js
CarbontraceWallet.initOnboarding("my_onboarding_div");
```

**Notes:**
- Automatically called during `init()` if `onboardingContainer` is resolved and wallet is not connected
- Creates `OnboardingFlow` and `OnboardingRenderer` instances
- Fires `carbontrace:state` on each step change

---

### `CarbontraceWallet.startOnboarding()`

Restarts the onboarding flow from the beginning. If onboarding hasn't been initialized yet, calls `initOnboarding()` first.

**Parameters:** None

**Returns:** `void`

---

### `CarbontraceWallet.skipOnboarding()`

Dismisses the onboarding UI by clearing its container.

**Parameters:** None

**Returns:** `void`

---

### `CarbontraceWallet.createGreenWallet()`

Triggers the managed wallet creation branch in onboarding.

**Parameters:** None

**Returns:** `void`

---

## Project Methods

### `CarbontraceWallet.loadProjects(containerIdOrElement)`

Loads and renders the carbon project selector into a container.

**Parameters:**

| Name | Type | Required | Description |
|------|------|:--------:|-------------|
| `containerIdOrElement` | `string \| Element` | ✅ | Container element or its ID |

**Returns:** `Promise<void>`

**Example:**

```js
await CarbontraceWallet.loadProjects("my_project_div");
```

**Behavior:**
1. Shows loading spinner
2. Fetches active projects from `GET /projects/active`
3. Renders interactive project cards with radio selection
4. Each card has a details button (`ⓘ`) that opens a Project Details Modal
5. Selection updates `walletState.selectedProjectId` and `window.CT_SELECTED_PROJECT_ID`
6. If fetch fails, shows mock projects as fallback

---

## Redemption Methods

### `CarbontraceWallet.markRedemptionConsumed()`

Clears the active redemption context after a successful redemption sale.

**Parameters:** None

**Returns:** `void`

**Example:**

```js
// After successful sale with redemption
CarbontraceWallet.markRedemptionConsumed();
```

> ⚠️ **This must be called after every successful sale that involves redemption.** Failing to call this can result in stale redemption reuse.

---

## Events

All events are dispatched on `window` as `CustomEvent` objects.

### `carbontrace:state`

Fires on any SDK state change (wallet connect/disconnect, onboarding step, project selection).

**`event.detail`:**

```js
{
  currentWallet: string | null,
  walletState: object | null,
  onboardingComplete: boolean,
  selectedProject: object | null,
  selectedProjectId: string | null,
  // Additional contextual fields may be present:
  onboardingStep: string    // current onboarding step ID
}
```

---

### `carbontrace:checkout:capture`

Fires immediately before a sale is submitted to the backend.

**`event.detail`:**

```js
{
  saleData: {
    sale_date: string,
    sale_order_id: string,
    project_ids: string[],
    invoice_value: number,
    carbon_footprint: number,
    email: string | null,
    phone: string | null,
    endUserId: string | null,
    sessionId: string,
    apply_redemption: boolean,
    redemption_context: object | null,
    client_id: string,
    receiver_wallet_address: string | null
  },
  checkoutData: object    // raw DOM metadata
}
```

---

### `carbontrace:checkout:captured`

Fires after a sale is successfully registered.

**`event.detail`:**

```js
{
  result: object,           // backend response (see Response Shapes)
  saleData: object,         // submitted sale data
  checkoutData: object      // raw DOM metadata
}
```

---

### `carbontrace:checkout:invalid`

Fires when checkout validation fails.

**`event.detail`:**

```js
{
  valid: false,
  message: string,          // human-readable error
  checkoutData: object,
  selectedProjectId: string | null
}
```

**Possible messages:**
- `"Checkout invoice amount is missing."`
- `"Checkout carbon footprint is missing."`
- `"Please select a Carbon Project first!"`

---

### `carbontrace:checkout:skipped`

Fires when project selection is optional (`isMandatory: false`) and no project is selected.

**`event.detail`:**

```js
{
  checkoutData: object      // raw DOM metadata
}
```

---

## Global Variables

### SDK → Page (readable)

| Variable | Type | Description |
|----------|------|-------------|
| `window.CarbontraceWallet` | `class` | The SDK class |
| `window.CT_SELECTED_PROJECT_ID` | `string` | Currently selected project ID |
| `window.CT_ONBOARDING_PROJECT` | `object` | Selected project (`{ projectId, name, description, metadata }`) |

### Page → SDK (writable, used as fallbacks)

| Variable | Type | Used By | Description |
|----------|------|---------|-------------|
| `window.CT_CUSTOMER_EMAIL` | `string` | `registerSale()` | Customer email fallback |
| `window.CT_CUSTOMER_PHONE` | `string` | `registerSale()` | Customer phone fallback |
| `window.CT_END_USER_ID` | `string` | `registerSale()`, `captureData()` | Your customer ID fallback |
| `window.CT_REDEEM_CONTEXT` | `object` | `captureData()`, `registerSale()` | Redemption context fallback |

---

## Init Config Options

Full reference for `CarbontraceWallet.init(config)`:

```js
{
  // Required
  token: string,                      // JWT access token

  // Container configuration
  walletContainer: string | Element,  // Default: "ct_wallet"
  onboardingContainer: string | Element, // Default: "ct_onboarding"
  projectContainer: string | Element, // Default: "ct_project"
  embedWallet: boolean,               // Default: false

  // UI behavior
  autoManageUi: boolean,              // Default: true

  // Chain configuration
  asaId: string,                      // Default: auto-detected
  chainId: number,                    // Default: 416002 (testnet)
                                      // Mainnet: 416001
                                      // Betanet: 416003

  // Callbacks
  onConnect: (address: string, snapshot: object) => void,
  onDisconnect: () => void,
  onSessionExpired: () => void,
  onChainPending: () => void,
  onChainConfirmed: () => void,
  onChainFailed: () => void
}
```

---

## Checkout Data Attributes

### On `#checkoutdata` element

| Attribute | Type | Required | SDK Field |
|-----------|------|:--------:|-----------|
| `data-invoice_amount` | number | ✅ | `invoice_value` |
| `data-invoice_value` | number | ✅ | `invoice_value` (alias) |
| `data-carbon_footprint` | number | ✅ | `carbon_footprint` |
| `data-carbon_charges` | number | — | `carbon_charges` |
| `data-sale_order_id` | string | — | `sale_order_id` |
| `data-email` | string | — | `email` |
| `data-phone` | string | — | `phone` |
| `data-mobile` | string | — | `phone` (alias) |
| `data-endUserId` | string | — | `endUserId` |
| `data-end_user_id` | string | — | `endUserId` (alias) |
| `data-customer_id` | string | — | `endUserId` (alias) |

### On checkout button (`data-id="ctcheckout"`)

| Attribute | SDK Field |
|-----------|-----------|
| `data-email` | `email` |
| `data-sale_order_id` or `data-saleOrderId` | `sale_order_id` |
| `data-phone` or `data-mobile` | `phone` |
| `data-endUserId` or `data-end_user_id` or `data-customer_id` | `endUserId` |

---

## Wallet State Object

The internal `walletState` object shape:

```js
CarbontraceWallet.walletState = {
  currentWallet: string | null,       // Connected wallet address
  walletSnapshot: {
    connected: boolean,
    walletAddress: string,
    algo: {
      algos: number,                  // ALGO balance
      microAlgos: number
    },
    ctCoins: {
      optedIn: boolean,               // Whether wallet opted into CTCoins ASA
      balance: number,                // CTCoins balance
      asaId: string                   // ASA ID
    },
    chain_status: string | null,      // "PENDING" | "CONFIRMED" | "FAILED"
    redemption: object | null         // Active redemption info
  },
  chainStatus: string | null,
  lastChainStatus: string | null,
  optedIn: boolean,                   // Shortcut for ctCoins.optedIn
  onboardingStep: string | null,      // Current onboarding step ID
  onboardingBranch: string | null,    // "managed" | "scan" | "create"
  onboardingComplete: boolean,
  selectedProject: object | null,     // Full project object
  selectedProjectId: string | null    // Project ID string
}
```

---

## Response Shapes

### `registerSale()` / `captureData()` Response

```js
{
  success: boolean,
  sale: {
    _id: string,
    sale_order_id: string,
    client_id: string,
    project_ids: string[],
    invoice_value: number,
    carbon_footprint: number,
    carbon_charges: number,
    email: string,
    phone: string,
    receiver_wallet_address: string,
    apply_redemption: boolean,
    status: string,
    created_at: string
  },
  invoice_breakdown: {
    // Detailed pricing and carbon charge breakdown
  },
  reward: {
    // Reward details if applicable
    amount_algos: number,
    amount_ctcoins: number,
    status: string
  }
}
```

### `validateCheckoutData()` Response

```js
{
  valid: boolean,
  message: string | null,
  checkoutData: {
    invoice_value: string,
    carbon_footprint: string,
    // ... other detected data-* values
  },
  selectedProjectId: string | null,
  skipSale: boolean               // only when isMandatory is false
}
```
