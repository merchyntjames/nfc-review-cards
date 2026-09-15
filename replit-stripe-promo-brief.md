# Brief for the Review Store (Replit): Stripe promotion codes, applied automatically from a URL parameter

**For:** the owner of the `reviewstore.merchynt.com` Replit app
**From:** James Sowers, Merchynt marketing
**Date:** 2026-09-15
**Goal:** a buyer who lands on the store (or on our landing page) from a link like `https://reviewstore.merchynt.com/?promo=FREENFCCARDSFORBUSINESS` should see the discount applied on step 4 without typing anything, and the discount must actually be charged by Stripe and recorded on the order.

The first half of this document explains what we need and why. The last section is a ready-to-paste prompt for Replit Agent. Read the whole thing first, because the prompt assumes the decisions made here.

---

## 1. What happens today (verified 2026-09-15)

We read the deployed front-end bundle and tested the live API, so these are facts rather than guesses:

1. **Promo codes are validated against the store's own coupon table, not Stripe.** Step 4 calls `GET /api/orders/coupon?code=&productKey=&quantity=` and expects `{ valid, code, discountCents, message }`. The code `FREENFCCARDSFORBUSINESS` was created as a **promotion code in Stripe**, so this endpoint returns `{"valid":false,"message":"That promo code isn't valid."}` for it, and the store's own promo field rejects it.
2. **Nothing reads the URL.** The only query parameters the app ever reads are `orderId` and `token` on `/order/confirmation`. The promo field on step 4 starts empty. There is no `localStorage` or `sessionStorage` use. A link with `?promo=` does nothing.
3. **The order payload carries `couponCode`** to `POST /api/orders`, and the server responds with `{ checkoutUrl }` for a hosted Stripe Checkout Session. We can't see the server, so we don't know whether `couponCode` is currently turned into a Stripe discount on the session, or only used to compute the total the store shows.
4. **Shipping is already $0** in the store UI (the catalog record for the card still says `shippingCents: 500`; please make sure the Stripe session does not add it back).
5. **Merchynt's landing page** at `https://merchynt-review-cards.vercel.app` is a separate front end that calls the same endpoints (`/api/products`, `/api/orders/coupon`, `/api/business-search`, `/api/address/*`, `/api/session`, `/api/orders`). Whatever you build here, keep the request and response shapes backward compatible and the landing page inherits it.

## 2. What we need, in one sentence each

1. The store accepts **Stripe promotion codes** (the human-readable codes you create in the Stripe Dashboard under Product catalog > Coupons > Promotion codes) everywhere it accepts a promo code today.
2. A code arriving in the URL (`?promo=CODE`, with `?coupon=` and `?code=` as aliases) is **remembered across the four steps** and **applied automatically** on step 4, with the discount visible in the summary before the buyer clicks "Proceed to payment".
3. The discount is **delivered to Stripe** on the Checkout Session so the buyer is charged the discounted amount, not shown one number and charged another.
4. The store **listens to Stripe** after payment and records the discount that was actually applied (code, Stripe IDs, amount) on the order, so the admin, the CSV export, HubSpot and the confirmation page all show it.
5. **Free orders work.** The current offer is a free card ($10 value) with free shipping, so a one-card order with this code has a $0 total. That path has to complete cleanly.

## 3. Design decisions (so the agent doesn't have to guess)

### 3.1 Where codes live
Stripe is the source of truth for codes going forward. Keep the existing internal coupon table working for any codes already in it, but look up Stripe **first**:

```
lookup(code):
  promo = stripe.promotionCodes.list({ code, active: true, limit: 1, expand: ['data.coupon'] })
  if promo found and promo.coupon.valid -> Stripe path
  else if internal coupon table has code -> internal path (unchanged behavior)
  else -> invalid
```

Stripe promotion code lookups are case-insensitive on Stripe's side; normalize the input (trim, uppercase) before calling.

### 3.2 Validation response (backward compatible)
`GET /api/orders/coupon?code=&productKey=&quantity=` keeps returning `{ valid, code, discountCents, message }` and adds:

```json
{
  "valid": true,
  "code": "FREENFCCARDSFORBUSINESS",
  "discountCents": 1000,
  "source": "stripe",
  "promotionCodeId": "promo_xxx",
  "couponId": "xxx",
  "description": "$10 off",
  "message": null
}
```

`discountCents` must be computed the same way Stripe will compute it, so the step 4 summary matches the Stripe page:
- `amount_off` coupons: `min(amount_off, subtotalCents)` (Stripe applies a fixed amount once per invoice/session, not per unit).
- `percent_off` coupons: `round(subtotalCents * percent_off / 100)`.
- Honor `restrictions.minimum_amount` (return invalid with a message like "This code needs a $20 minimum order" when the subtotal is under it), `expires_at`, `max_redemptions` vs `times_redeemed`, `restrictions.first_time_transaction`, and `coupon.valid`.
- If the coupon has `applies_to.products`, it only discounts line items that reference those Stripe Product IDs. See 3.4.

### 3.3 URL parameter and persistence (front end)
- On first load of any route, read `promo`, then `coupon`, then `code` from the query string. If present, store it in `sessionStorage` under `reviewstore.promo` and remove nothing from the URL (leave it, it's harmless).
- When step 4 mounts, if there's a stored code and no code is applied yet, run the validation call automatically and, on success, show it as applied (same UI as a typed code, with the existing Remove button). On failure, show the message inline under the promo field and leave the field editable. Do not block checkout on a failed auto-apply.
- Re-validate whenever quantity changes (the existing code already clears the coupon on quantity change; instead, re-run validation with the new quantity so the discount updates rather than disappears).
- Typing a code manually still works exactly as today.
- Our landing page already reads `?promo=` and passes the code through `couponCode`, so it needs no change once the endpoint accepts Stripe codes.

### 3.4 Delivery to Stripe (server, `POST /api/orders`)
When the resolved code is a Stripe promotion code:

```js
const session = await stripe.checkout.sessions.create({
  mode: 'payment',
  line_items: [...],                       // see note below
  discounts: [{ promotion_code: promotionCodeId }],
  // Do NOT also set allow_promotion_codes: Stripe rejects a session that has both.
  metadata: { orderId, promoCode: code, promotionCodeId, couponId, audienceType },
  success_url: ..., cancel_url: ...,
});
```

When no code is applied, set `allow_promotion_codes: true` so a buyer can still type a Stripe code on the Stripe page. Record on the order before redirecting: `couponCode`, `couponSource` (`stripe` | `internal` | `none`), `stripePromotionCodeId`, `stripeCouponId`, `expectedDiscountCents`, `expectedTotalCents`.

**Line items note.** If the session builds line items with inline `price_data` (ad-hoc product data) rather than a real Stripe Price ID, then any coupon restricted to specific products (`applies_to.products`) will silently not apply. Either reference the real Stripe Price for the NFC card in the session, or create promotion codes with no product restriction. Say which one you did in your reply.

When the resolved code is an **internal** coupon (existing behavior), keep whatever the server does today, but make sure the Stripe session total equals the store's displayed total (if it currently doesn't, create a one-off Stripe coupon with `amount_off` and `duration: 'once'` for the session, or reduce the line item unit amount, and note which).

### 3.5 Free orders ($0 total)
With a $10-off code on one $10 card and free shipping, the total is $0. Handle it explicitly:

1. First try Stripe: create the session with the discount and see whether Stripe accepts a zero-total session in `payment` mode for your account configuration. Test this in test mode before assuming either way.
2. If Stripe rejects it (or you'd rather not send buyers to a payment page for $0), take the bypass path: when `expectedTotalCents === 0`, skip Stripe, mark the order `paid` with `paymentProvider: 'none'`, `amountPaidCents: 0`, record the discount fields, send the same confirmation email, and return `{ checkoutUrl: '/order/confirmation?orderId=...&token=...' }` so the front end redirects exactly as it does today. Redeem the promotion code count yourself in this path (Stripe won't), and enforce `max_redemptions` in the lookup so a code can't be used past its limit through the bypass.

Whichever path you take, the confirmation page must show the discount line and a $0 total without looking like an error.

### 3.6 Listening to Stripe (webhook)
On `checkout.session.completed` (retrieve the session with `expand: ['total_details.breakdown', 'line_items']` if the event payload doesn't carry the breakdown):

- Persist `session.amount_subtotal`, `session.amount_total`, `session.total_details.amount_discount`, `session.total_details.amount_shipping`, and the discount identity from `session.total_details.breakdown.discounts[]` (each has `amount` and `discount.coupon` / `discount.promotion_code`). Save as `actualDiscountCents`, `actualTotalCents`, `stripePromotionCodeId`, `stripeCouponId` on the order.
- If a buyer typed a code on the Stripe page (the `allow_promotion_codes` case), this is the only place the store learns about it, so the webhook must write the code fields even when the order was created without one.
- If `actualDiscountCents !== expectedDiscountCents`, keep the Stripe figures as the truth and flag the order in the admin (`discountMismatch: true`) so we can see it.
- Also handle `checkout.session.expired` (mark the order abandoned so promo analytics don't count it) and keep the existing paid/unpaid logic intact.
- Make the webhook idempotent on `event.id`.

### 3.7 Admin, export, HubSpot, confirmation page
- Order detail and the orders list show: code, source, discount amount, total paid, and the mismatch flag.
- CSV export gains columns: `promo_code`, `promo_source`, `discount_cents`, `total_paid_cents`, `stripe_promotion_code_id`.
- HubSpot mapping: add `promo_code` and `discount_amount` to the fields that can be mapped on the deal/contact, default them on if the mapping is one-to-one.
- `/order/confirmation` shows a "Promo code CODE: -$10" line and the correct total.
- Admin stats: count orders by promo code (a simple group-by is enough).

### 3.8 Security and abuse
- Never trust `discountCents` from the client. The server recomputes it from Stripe at order creation.
- Rate-limit `GET /api/orders/coupon` per IP (for example 30 per minute) so the endpoint can't be used to brute-force codes.
- Log lookups that hit Stripe so we can see volume; cache successful lookups for a few minutes keyed by code to keep Stripe API calls low.
- Keep the Stripe secret key server-side only; the front end never sees promotion code IDs beyond the validation response.

## 4. Test plan (do these in Stripe test mode with the Stripe CLI forwarding webhooks)

Create in test mode: (a) `FREENFCCARDSFORBUSINESS` as a promotion code on a $10-off coupon, (b) a 30%-off code, (c) a code with a $20 minimum, (d) an expired code, (e) a code with `max_redemptions: 1`.

| # | Scenario | Expected |
|---|---|---|
| 1 | Open `/?promo=FREENFCCARDSFORBUSINESS`, walk to step 4 with quantity 1 | Code shows as applied with -$10, total $0, no typing |
| 2 | Same, quantity 3 | -$10 once, total $20 (fixed amount applies once) |
| 3 | 30% code, quantity 3 | -$9, total $21, Stripe page shows $21 |
| 4 | Minimum-amount code with 1 card | Inline message naming the minimum, field stays editable, checkout still possible without discount |
| 5 | Expired code | "That promo code isn't valid" (or a more specific message), checkout still possible |
| 6 | Change quantity after a code is applied | Discount recalculates, does not disappear |
| 7 | Remove the applied code | Summary returns to full price; Stripe page shows full price with a promo field available |
| 8 | Type a Stripe code on the Stripe page instead of step 4 | Order record gets the code and discount from the webhook |
| 9 | Complete scenario 1 | Either Stripe accepts the $0 session and the webhook records `amount_total: 0`, or the bypass creates a paid $0 order; confirmation page shows the discount line |
| 10 | Use the `max_redemptions: 1` code twice | Second attempt is rejected at validation, including on the $0 bypass path |
| 11 | Session expires without payment | Order marked abandoned, no redemption counted |
| 12 | Admin and CSV export for orders 1, 3, 8 | Code, source, discount and total present and correct |
| 13 | Internal coupon (existing table) still applies | Unchanged behavior, Stripe total matches the store total |
| 14 | `curl` the validation endpoint 50 times in a minute | Rate limit responds after the threshold |

## 5. What to send back to James when done

- Confirmation of which line-item approach the Stripe session uses (real Price ID or inline `price_data`), because it decides whether product-restricted coupons work.
- Which $0 path you took (Stripe accepted the zero session, or the bypass) and a test order ID for each of scenarios 1, 3 and 8.
- The exact URL parameter names supported (should be `promo`, `coupon`, `code`).
- Any change to the `GET /api/orders/coupon` or `POST /api/orders` shapes beyond the additive fields listed above, since the Merchynt landing page depends on them.

---

## 6. Paste-ready prompt for Replit Agent

Copy everything inside the block into Replit Agent as one message. It's written to be self-sufficient, but attach this document to the project too so the agent can refer to the detailed sections.

```
We're adding Stripe promotion code support to this store. Read the full brief in replit-stripe-promo-brief.md first. Then implement the following, in this order, and stop after each numbered part so I can review before you continue.

CONTEXT
- The store is a React + Vite front end with an Express backend, a coupon table of our own, and hosted Stripe Checkout Sessions created in POST /api/orders. Step 4 of the checkout validates promo codes with GET /api/orders/coupon?code=&productKey=&quantity= and expects { valid, code, discountCents, message }.
- Promo codes are now created in the Stripe Dashboard as promotion codes (for example FREENFCCARDSFORBUSINESS on a $10-off coupon). Today the store rejects them because it only checks its own table.
- A separate marketing landing page calls the same endpoints, so keep every existing request and response field exactly as it is and only ADD fields.

PART 1: Stripe-aware validation (server)
1. In the coupon validation handler, normalize the code (trim, uppercase) and look it up in Stripe first: stripe.promotionCodes.list({ code, active: true, limit: 1, expand: ['data.coupon'] }). If a promotion code is found and its coupon is valid, compute discountCents for the requested product and quantity exactly as Stripe will: amount_off coupons discount min(amount_off, subtotal) once per order; percent_off coupons discount round(subtotal * percent_off / 100). Enforce restrictions.minimum_amount, expires_at, max_redemptions vs times_redeemed, restrictions.first_time_transaction, and coupon.valid, returning valid:false with a specific message when one fails (for a minimum amount, name the amount).
2. If Stripe has no match, fall back to the existing internal coupon table with unchanged behavior.
3. Keep the response shape and add: source ('stripe' | 'internal'), promotionCodeId, couponId, description. Cache successful Stripe lookups in memory for 5 minutes keyed by code. Rate-limit this endpoint to 30 requests per minute per IP.
4. Never trust any discount amount from the client; the server always recomputes.

PART 2: Apply codes from the URL (front end)
5. On first render of the app, read the query parameters promo, then coupon, then code (first one present wins). If present, save it to sessionStorage under 'reviewstore.promo'. Do not strip it from the URL.
6. When the checkout step (step 4) mounts and a stored code exists and no code is currently applied, call the validation endpoint automatically with the current product and quantity. On success, render it exactly like a manually applied code, including the existing Remove control. On failure, show the returned message under the promo field, leave the field editable, and never block checkout.
7. When quantity changes while a code is applied, re-run validation with the new quantity and update discountCents instead of clearing the code. Manual entry keeps working exactly as today.

PART 3: Deliver the discount to Stripe (server, POST /api/orders)
8. When the order payload's couponCode resolves to a Stripe promotion code, create the Checkout Session with discounts: [{ promotion_code: <promotionCodeId> }]. Do not also set allow_promotion_codes on that session (Stripe rejects sessions that set both). When no code is applied, set allow_promotion_codes: true so buyers can enter a Stripe code on the Stripe page.
9. Put orderId, promoCode, promotionCodeId, couponId and audienceType in session.metadata. Before redirecting, store on the order: couponCode, couponSource, stripePromotionCodeId, stripeCouponId, expectedDiscountCents, expectedTotalCents.
10. Check how line items are built. If they use inline price_data instead of a real Stripe Price ID, coupons restricted to specific products will not apply; tell me which it is and, if inline, switch the NFC card line item to the real Stripe Price ID.
11. For internal-table coupons, make sure the Stripe session total equals the store's displayed total (create a one-off Stripe coupon with amount_off and duration 'once' for that session if needed) and tell me what you did.
12. Free orders: when expectedTotalCents is 0, first test in Stripe test mode whether a zero-total payment-mode session is accepted with the discount attached. If it is, use it. If Stripe rejects it, implement a bypass: skip Stripe, mark the order paid with paymentProvider 'none' and amountPaidCents 0, record the discount fields, send the normal confirmation email, increment our own redemption count for the code and enforce max_redemptions in validation, and return { checkoutUrl: '/order/confirmation?orderId=...&token=...' } so the front end redirects as it does today.

PART 4: Listen to Stripe (webhook)
13. On checkout.session.completed, retrieve the session with expand ['total_details.breakdown', 'line_items'] and persist amount_subtotal, amount_total, total_details.amount_discount, total_details.amount_shipping, and each entry of total_details.breakdown.discounts (amount, coupon id, promotion code id) onto the order as actualDiscountCents, actualTotalCents, stripePromotionCodeId, stripeCouponId. This must also capture codes typed on the Stripe page for orders created without one.
14. If actualDiscountCents differs from expectedDiscountCents, keep the Stripe values and set discountMismatch: true on the order.
15. Handle checkout.session.expired by marking the order abandoned. Make webhook processing idempotent on event.id. Do not change the existing paid/unpaid logic otherwise.

PART 5: Admin, export, HubSpot, confirmation
16. Show code, source, discount, total paid and the mismatch flag on the admin order list and detail. Add promo_code, promo_source, discount_cents, total_paid_cents and stripe_promotion_code_id columns to the CSV export. Add promo_code and discount_amount to the HubSpot field mapping options. Add a by-code order count to admin stats.
17. On /order/confirmation, show a "Promo code CODE: -$X" line and the correct total, including a $0 total rendered as a normal successful order.

PART 6: Tests
18. Run the 14 scenarios in section 4 of the brief in Stripe test mode with the Stripe CLI forwarding webhooks, and report the results in a table with test order IDs. Also confirm that the catalog's shippingCents for the NFC card is not added to the Stripe session, since shipping is free.

Do not remove or rename any existing API field. Do not change the order payload the front end sends beyond what is described. Ask me before touching the database schema if migrations are destructive.
```
