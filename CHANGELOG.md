# Changelog

All notable changes to the Carbontrace Wallet SDK are documented here.

---

## [2.0.0] — 2026-05-26

### Added

- **Impact-first onboarding flow** — Redesigned from scratch with 6-step linear flow (Impact Hook → Project Selection → Wallet Choice → Connect → Confirmation → Done)
- **Project Details Modal** — Click the `ⓘ` button on any project card to see full carbon retirement data, project type, and metadata
- **CTCoins opt-in modal** — Step-by-step visual flow for ASA opt-in (Opt-In Required → Review Asset → Pending → Confirmed)
- **Transaction history with proof modal** — View on-chain transactions and click for detailed proof including IPFS CID, UCR, certificate, and explorer links
- **Phone/mobile capture** — `data-phone` and `data-mobile` checkout metadata attributes
- **End-user ID tracking** — `data-endUserId`, `data-customer_id`, `data-end_user_id` attributes for your customer ID
- **End-user session ID** — Auto-generated persistent session ID (`ct_{timestamp}_{random}`) stored in localStorage
- **`carbontrace:checkout:capture` event** — Fires before sale submission with full payload
- **`carbontrace:checkout:skipped` event** — Fires when project selection is optional and none selected
- **Balance auto-polling** — 20-second interval automatic balance refresh with UI update
- **Toast notification system** — Non-intrusive toast messages for wallet events, errors, and confirmations
- **`validateCheckoutData()` method** — Pre-flight validation without submitting
- **`optInCTCoins()` method** — Programmatic trigger for CTCoins ASA opt-in
- **`loadProjects()` method** — Load project selector into any container
- **`initOnboarding()` / `startOnboarding()` / `skipOnboarding()` methods** — Programmatic onboarding control
- **`createGreenWallet()` method** — Trigger managed wallet creation branch
- **Shadow DOM wallet encapsulation** — Wallet UI isolated from host page styles
- **Automatic `sale_order_id` generation** — `order_{timestamp}` when not provided
- **`invoice_breakdown` response** — Detailed pricing data returned from sale registration
- **Global variable fallbacks** — `CT_CUSTOMER_EMAIL`, `CT_CUSTOMER_PHONE`, `CT_END_USER_ID`, `CT_REDEEM_CONTEXT`
- **Auto-created SDK containers** — Missing `ct_wallet`, `ct_onboarding`, `ct_project` are auto-created
- **Checkout button data attributes** — `data-email`, `data-sale_order_id`, `data-phone` read from trigger element
- **SDK host styles** — Responsive grid layout for auto-created checkout root

### Changed

- **Onboarding redesigned** — Minimal Clean theme with teal (`#0f766e`) color palette, Inter/Outfit typography
- **Wallet UI redesigned** — Premium spacing (20px border-radius), softer shadows, teal gradient badge
- **Project selector** — Interactive radio cards with hover effects and green selection state
- **Checkout capture enriched** — Payload now includes `phone`, `endUserId`, `sessionId`, resolved redemption
- **Redemption resolution** — Priority chain: SDK state → payload → `window.CT_REDEEM_CONTEXT` → sessionStorage
- **Container resolution** — Explicit config → default IDs → auto-created

### Removed

- Old 8-step onboarding with wallet creation branching
- Mnemonic save step in onboarding
- Pera install guide step in onboarding
- Dense, compact wallet layout (replaced by clean, spacious design)

---

## [1.0.0] — Initial Release

### Added

- Wallet connection via Pera Wallet (WalletConnect)
- Checkout data capture from DOM metadata
- `captureData()` for manual checkout capture
- `registerSale()` for direct sale registration
- `markRedemptionConsumed()` for redemption state management
- Carbon project selector
- `carbontrace:state` event
- `carbontrace:checkout:captured` event
- `carbontrace:checkout:invalid` event
- JWT-based session management with auto-expiry
- Backend-authoritative validation model
- Basic onboarding flow
- Chain status polling and events (`onChainPending`, `onChainConfirmed`, `onChainFailed`)
