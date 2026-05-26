# Carbontrace SDK — Integration Guide

> Step-by-step walkthrough for integrating the Carbontrace Wallet SDK into your website or application.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Quick Start (5 Minutes)](#2-quick-start-5-minutes)
3. [Container Layout](#3-container-layout)
4. [Checkout Metadata](#4-checkout-metadata)
5. [Checkout Button Binding](#5-checkout-button-binding)
6. [Manual Capture (Custom Flows)](#6-manual-capture-custom-flows)
7. [Direct Sale Registration](#7-direct-sale-registration)
8. [Redemption Integration](#8-redemption-integration)
9. [Dynamic Checkout Data (SPA)](#9-dynamic-checkout-data-spa)
10. [Event Listeners](#10-event-listeners)
11. [Styling & Customization](#11-styling--customization)
12. [Mobile Integration](#12-mobile-integration)
13. [SPA / React / Next.js](#13-spa--react--nextjs)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. Prerequisites

Before integrating:

- **Client account** — You must be an approved Carbontrace business partner
- **Access token** — A JWT token issued by Carbontrace, permanently linked to your account
- **HTTPS** — The SDK requires a secure origin (HTTPS) for wallet connections
- **Pera Wallet** — Your end-users need Pera Wallet (mobile app or browser extension) to connect

> ⚠️ Tokens cannot be self-generated. Contact Carbontrace to receive yours.

---

## 2. Quick Start (5 Minutes)

Add these 4 elements to your checkout page:

```html
<!-- 1. SDK Bundle -->
<script src="https://admin.carbontrace.in/js/carbon-sdk-bundle.js"></script>

<!-- 2. SDK Containers -->
<div id="ct_wallet"></div>
<div id="ct_onboarding"></div>
<div id="ct_project"></div>

<!-- 3. Checkout Metadata -->
<div id="checkoutdata"
  data-invoice_amount="5000"
  data-carbon_footprint="25"
  style="display:none">
</div>

<!-- 4. Checkout Button -->
<button data-id="ctcheckout">Pay now</button>

<!-- 5. Initialize -->
<script>
  document.addEventListener("DOMContentLoaded", async () => {
    await CarbontraceWallet.init({
      token: "YOUR_CLIENT_TOKEN"
    });
  });
</script>
```

**What happens:**

1. The SDK renders a wallet connection UI in `#ct_wallet`
2. An onboarding flow appears in `#ct_onboarding` guiding the user to connect their wallet
3. Once onboarded, a project selector loads in `#ct_project`
4. When the user clicks the checkout button, the SDK:
   - Validates the checkout data
   - Ensures a project is selected
   - Registers the sale with Carbontrace backend
   - The user earns carbon rewards (ALGO + CTCoins)

---

## 3. Container Layout

### Default Container IDs

The SDK expects these container elements on your page:

```html
<div id="ct_wallet"></div>      <!-- Wallet UI: connection, balance, history -->
<div id="ct_onboarding"></div>  <!-- Onboarding: impact hook, project pick, wallet connect -->
<div id="ct_project"></div>     <!-- Project selector: carbon project cards -->
```

### Recommended Page Structure

```html
<div class="your-checkout-layout">

  <!-- Your order summary, cart items, etc. -->
  <div class="order-summary">
    ...
  </div>

  <!-- Carbontrace SDK section -->
  <div class="carbon-section">
    <div id="ct_wallet"></div>
    <div id="ct_onboarding"></div>
    <div id="ct_project"></div>
  </div>

  <!-- Checkout metadata (hidden) -->
  <div id="checkoutdata"
    data-invoice_amount="5000"
    data-carbon_footprint="25"
    style="display:none">
  </div>

  <!-- Your checkout button -->
  <button data-id="ctcheckout">Complete Purchase</button>

</div>
```

### Custom Container IDs

You can use your own container IDs:

```js
await CarbontraceWallet.init({
  token: "YOUR_CLIENT_TOKEN",
  walletContainer: "my_wallet_section",
  onboardingContainer: "my_onboarding_section",
  projectContainer: "my_project_section"
});
```

```html
<div id="my_wallet_section"></div>
<div id="my_onboarding_section"></div>
<div id="my_project_section"></div>
```

### Auto-Created Containers

If any default container is missing from your page, the SDK creates them automatically and appends them near the `#checkoutdata` element (or at the end of `<body>`). A wrapper `#ct_checkout_sdk` is created with a max-width of 540px and centered.

### Visibility Management

When `autoManageUi: true` (default):

- **Wallet** is always visible
- **Onboarding** hides automatically after completion
- **Project selector** appears only when:
  - Onboarding is complete, OR
  - No onboarding exists and a wallet is already connected

---

## 4. Checkout Metadata

### The `#checkoutdata` Element

The primary way to pass checkout data to the SDK:

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

### Attribute Reference

| Attribute | Required | Type | Description |
|-----------|:--------:|------|-------------|
| `data-invoice_amount` | ✅ | number | Order total before carbon charges |
| `data-carbon_footprint` | ✅ | number | Carbon footprint in kg CO₂ |
| `data-carbon_charges` | — | number | Supplemental carbon charge metadata |
| `data-sale_order_id` | — | string | Your order/invoice reference. Auto-generated as `order_{timestamp}` if missing |
| `data-email` | — | string | Customer email address |
| `data-phone` | — | string | Customer phone number |
| `data-endUserId` | — | string | Your system's customer/user ID |
| `data-customer_id` | — | string | Alias for `endUserId` |

> `data-invoice_value` is also accepted as an alias for `data-invoice_amount`.

### Standalone Data Elements

The SDK also reads `data-*` attributes from any element on the page:

```html
<!-- These work even without #checkoutdata -->
<span data-invoice_amount="5000"></span>
<span data-carbon_footprint="25"></span>
```

### Updating Metadata Dynamically

For Single Page Applications (SPAs) or dynamic carts:

```js
// Update the checkout data programmatically
document.getElementById("checkoutdata").dataset.invoice_amount = "7500";
document.getElementById("checkoutdata").dataset.carbon_footprint = "35";
document.getElementById("checkoutdata").dataset.sale_order_id = "order_new_123";
```

The SDK reads these values at capture time, so they can be updated anytime before the checkout button is clicked.

---

## 5. Checkout Button Binding

### Basic Button

```html
<button data-id="ctcheckout">Pay now</button>
```

The SDK listens for clicks on any element with `data-id="ctcheckout"`. This works on `<button>`, `<a>`, `<div>`, or any clickable element.

### Button With Extra Data

```html
<button
  data-id="ctcheckout"
  data-email="customer@example.com"
  data-sale_order_id="order_28991"
  data-phone="9876543210">
  Complete Purchase
</button>
```

The SDK reads `data-email`, `data-sale_order_id`, and `data-phone` from the trigger element and includes them in the sale payload.

### What Happens on Click

1. **Default action is prevented** — The SDK captures the click before your payment processor
2. **Validation runs** — Invoice amount, carbon footprint, and project selection are checked
3. **If validation fails** — An `alert()` shows the error message and `carbontrace:checkout:invalid` fires
4. **If validation passes** — The SDK calls `registerSale()` internally
5. **On success** — `carbontrace:checkout:captured` event fires with the result
6. **Button is re-enabled** — The button's `disabled` attribute is removed

### Multiple Checkout Buttons

Only the first clicked button triggers capture. The selector is `[data-id="ctcheckout"]`, so all matching elements are bound.

---

## 6. Manual Capture (Custom Flows)

If your checkout flow doesn't use a standard button (AJAX forms, React components, etc.):

### Basic Capture

```js
const result = await CarbontraceWallet.captureData();

if (result) {
  console.log("Sale registered:", result);
  // Proceed with payment
} else {
  console.log("Capture failed:", CarbontraceWallet.lastCheckoutError);
}
```

### Capture With Overrides

```js
const result = await CarbontraceWallet.captureData({
  saleData: {
    sale_order_id: "order_28991",
    email: "customer@example.com",
    phone: "9876543210",
    endUserId: "cust_12345",
    invoice_value: 7500,
    carbon_footprint: 35
  }
});
```

Override values take priority over DOM metadata. Missing values are still auto-filled from DOM and SDK state.

### Validate Without Submitting

```js
const validation = CarbontraceWallet.validateCheckoutData({
  invoice_value: 5000,
  carbon_footprint: 25
});

if (validation.valid) {
  console.log("Ready to capture");
} else {
  console.warn("Not ready:", validation.message);
}
```

The validation result shape:

```js
{
  valid: true | false,
  message: "Error message" | null,
  checkoutData: { /* DOM metadata */ },
  selectedProjectId: "113" | null,
  skipSale: false   // true if project is optional and none selected
}
```

---

## 7. Direct Sale Registration

For server-side integrations, POS systems, or admin panels:

```js
const result = await CarbontraceWallet.registerSale({
  sale_date: new Date().toISOString(),
  sale_order_id: "order_28991",
  project_ids: ["113"],
  invoice_value: 5000,
  carbon_footprint: 25,
  email: "customer@example.com",
  phone: "9876543210",
  endUserId: "cust_12345",
  apply_redemption: false
});
```

### Auto-Resolution

Even in direct mode, the SDK fills missing values:

| Field | Auto-resolved from |
|-------|-------------------|
| `invoice_value` | `#checkoutdata` dataset |
| `carbon_footprint` | `#checkoutdata` dataset |
| `project_ids` | `walletState.selectedProjectId` or `window.CT_SELECTED_PROJECT_ID` |
| `client_id` | JWT token payload |
| `receiver_wallet_address` | Connected wallet address |
| `sessionId` | Auto-generated end-user session |
| `email` | `window.CT_CUSTOMER_EMAIL` (fallback) |
| `phone` | `window.CT_CUSTOMER_PHONE` (fallback) |
| `endUserId` | `window.CT_END_USER_ID` (fallback) |

### Response Shape

```js
{
  success: true,
  sale: { /* sale record */ },
  invoice_breakdown: {
    /* detailed pricing breakdown */
  },
  reward: { /* reward details, if applicable */ }
}
```

---

## 8. Redemption Integration

### How Redemption Works

1. A user earns rewards from previous sales (ALGO, CTCoins)
2. The user can redeem rewards on a future purchase
3. The SDK detects active redemption context and applies it to the next sale
4. After the sale succeeds, `markRedemptionConsumed()` must be called

### Setting Redemption Context

The SDK resolves redemption from multiple sources (in priority order):

**Option A — Global variable:**

```js
window.CT_REDEEM_CONTEXT = {
  ctcoins_required: 100,
  discount_amount: 50,
  redemption_id: "rdm_12345"
};
```

**Option B — Session storage:**

```js
sessionStorage.setItem("ct_redeem_context", JSON.stringify({
  ctcoins_required: 100,
  discount_amount: 50
}));
```

**Option C — Direct in captureData:**

```js
const result = await CarbontraceWallet.captureData({
  saleData: {
    apply_redemption: true,
    redemption_context: {
      ctcoins_required: 100,
      discount_amount: 50
    }
  }
});
```

### Clearing Redemption

**This is mandatory after every successful redemption sale:**

```js
window.addEventListener("carbontrace:checkout:captured", (event) => {
  const result = event.detail.result;
  if (result?.sale?.apply_redemption) {
    CarbontraceWallet.markRedemptionConsumed();
  }
});
```

---

## 9. Dynamic Checkout Data (SPA)

For Single Page Applications where checkout data changes without page reloads:

### Updating DOM Metadata

```js
function updateCheckoutData(cart) {
  const el = document.getElementById("checkoutdata");
  el.dataset.invoice_amount = cart.total;
  el.dataset.carbon_footprint = cart.carbonFootprint;
  el.dataset.sale_order_id = cart.orderId;
  el.dataset.email = cart.customerEmail;
}

// Call whenever cart updates
updateCheckoutData(myCart);
```

### Using Global Variables

```js
window.CT_CUSTOMER_EMAIL = currentUser.email;
window.CT_CUSTOMER_PHONE = currentUser.phone;
window.CT_END_USER_ID = currentUser.id;
```

### Re-Initializing After Navigation

If your SPA navigates between pages:

```js
// On checkout page mount
async function mountCheckout() {
  // Re-init if not already initialized
  if (!window.CarbontraceWallet?.currentWallet) {
    await CarbontraceWallet.init({
      token: getAuthToken()
    });
  }
}
```

---

## 10. Event Listeners

### All Available Events

```js
// Wallet/project state changed
window.addEventListener("carbontrace:state", (e) => {
  const { currentWallet, selectedProjectId, onboardingComplete } = e.detail;
});

// Sale is about to be submitted
window.addEventListener("carbontrace:checkout:capture", (e) => {
  const { saleData, checkoutData } = e.detail;
});

// Sale submitted successfully
window.addEventListener("carbontrace:checkout:captured", (e) => {
  const { result, saleData, checkoutData } = e.detail;
});

// Checkout validation failed
window.addEventListener("carbontrace:checkout:invalid", (e) => {
  const { valid, message, checkoutData, selectedProjectId } = e.detail;
});

// Sale skipped (optional project, none selected)
window.addEventListener("carbontrace:checkout:skipped", (e) => {
  const { checkoutData } = e.detail;
});
```

### Practical Example — Show Custom Success Message

```js
window.addEventListener("carbontrace:checkout:captured", (event) => {
  const result = event.detail.result;

  // Show your custom success UI
  document.getElementById("success-message").textContent =
    `✅ Sale ${result.sale.sale_order_id} registered! Carbon offset: ${result.sale.carbon_footprint} kg CO₂`;
  document.getElementById("success-message").style.display = "block";

  // Clear redemption if used
  if (result.sale.apply_redemption) {
    CarbontraceWallet.markRedemptionConsumed();
  }
});
```

### Practical Example — Block Checkout Until Project Selected

```js
window.addEventListener("carbontrace:checkout:invalid", (event) => {
  if (event.detail.message.includes("Carbon Project")) {
    // Scroll to project selector
    document.getElementById("ct_project").scrollIntoView({ behavior: "smooth" });
  }
});
```

---

## 11. Styling & Customization

### SDK Styling Approach

- **Wallet UI** — Renders inside a Shadow DOM. Host page CSS does not affect it.
- **Onboarding** — Rendered with scoped inline styles. Uses teal/neutral color palette.
- **Project Selector** — Uses scoped inline styles. Cards are interactive with hover effects.
- **Modals** — (Opt-in, Project Details) use fixed-position overlays with backdrop blur.

### Host Container Styling

You can style the outer containers:

```css
#ct_wallet {
  max-width: 500px;
  margin: 0 auto;
}

#ct_onboarding {
  margin-bottom: 20px;
}

#ct_project {
  margin-top: 16px;
}
```

### SDK Root Container

The auto-created checkout root (`#ct_checkout_sdk`) has:

```css
#ct_checkout_sdk {
  width: min(100%, 540px);
  margin: 16px auto 0;
  display: grid;
  gap: 12px;
}
```

---

## 12. Mobile Integration

### Pera Wallet Deep Links

On mobile devices, the SDK automatically deep-links to Pera Wallet:
- If Pera Wallet is installed → opens the app directly
- If not installed → shows a QR code for scanning

### Responsive Design

All SDK UI components are mobile-responsive:
- Project cards stack to single column on screens < 640px
- Touch targets are at least 44px for accessibility
- Onboarding steps are optimized for small screens

### Testing on Mobile

1. Serve your page over HTTPS (required for wallet connections)
2. Open on a mobile device with Pera Wallet installed
3. The wallet connection will use deep-linking instead of QR

---

## 13. SPA / React / Next.js

### React Integration

```jsx
import { useEffect } from "react";

function CarbontraceCheckout({ token, cart }) {
  useEffect(() => {
    // Load SDK script dynamically
    const script = document.createElement("script");
    script.src = "https://admin.carbontrace.in/js/carbon-sdk-bundle.js";
    script.onload = async () => {
      await window.CarbontraceWallet.init({ token });
    };
    document.head.appendChild(script);

    return () => {
      // Cleanup if needed
      script.remove();
    };
  }, [token]);

  return (
    <div>
      <div id="ct_wallet" />
      <div id="ct_onboarding" />
      <div id="ct_project" />

      <div
        id="checkoutdata"
        data-invoice_amount={cart.total}
        data-carbon_footprint={cart.carbonFootprint}
        style={{ display: "none" }}
      />

      <button data-id="ctcheckout">
        Pay ₹{cart.total}
      </button>
    </div>
  );
}
```

### Next.js Integration

```jsx
"use client";
import { useEffect } from "react";
import Script from "next/script";

export default function CheckoutPage() {
  const handleSDKLoad = async () => {
    await window.CarbontraceWallet.init({
      token: "YOUR_CLIENT_TOKEN"
    });
  };

  return (
    <>
      <Script
        src="https://admin.carbontrace.in/js/carbon-sdk-bundle.js"
        onLoad={handleSDKLoad}
        strategy="afterInteractive"
      />

      <div id="ct_wallet" />
      <div id="ct_onboarding" />
      <div id="ct_project" />

      <div
        id="checkoutdata"
        data-invoice_amount="5000"
        data-carbon_footprint="25"
        style={{ display: "none" }}
      />

      <button data-id="ctcheckout">Pay now</button>
    </>
  );
}
```

### Key Considerations for SPAs

1. **Load SDK once** — Don't re-load the script on every navigation
2. **Re-init on page change** — Call `init()` when the checkout page mounts
3. **Update metadata dynamically** — Change `data-*` attributes when the cart updates
4. **Cleanup** — The SDK handles its own cleanup; no explicit teardown needed

---

## 14. Troubleshooting

### Token Issues

| Symptom | Solution |
|---------|----------|
| "Login required" displayed | Pass a valid `token` to `init()` |
| SDK stops working after some time | Token expired — `onSessionExpired` fires |
| 401 errors in console | Token is invalid or expired |

### Container Issues

| Symptom | Solution |
|---------|----------|
| Onboarding not visible | Ensure `<div id="ct_onboarding">` exists |
| Project selector not showing | Appears only after onboarding or wallet connection |
| SDK UI renders at bottom of page | Add container `<div>` elements where you want the UI |

### Checkout Issues

| Symptom | Solution |
|---------|----------|
| "Please select a Carbon Project first!" | User hasn't picked a project in the selector |
| `captureData()` returns null | `data-carbon_footprint` or `data-invoice_amount` missing or zero |
| Button click does nothing | Ensure `data-id="ctcheckout"` is on the button |
| Checkout succeeds but no event | Listen for `carbontrace:checkout:captured` |

### Wallet Issues

| Symptom | Solution |
|---------|----------|
| QR code shown but can't scan | Ensure Pera Wallet app is updated |
| "Wallet connection cancelled" | User rejected the connection in Pera |
| "Wallet not linked to your account" | Wallet address not registered with Carbontrace |
| Balance shows 0 after sale | Wait for blockchain confirmation (20s polling) |

### Network Issues

| Symptom | Solution |
|---------|----------|
| CORS errors | SDK must be loaded from `admin.carbontrace.in` |
| API calls failing | Check network connectivity to `admin.carbontrace.in` |
| Slow project loading | Normal on first load; projects are cached after |
