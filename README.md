# NFC Review Cards Landing Page

Direct-response landing page for Facebook traffic that sells Merchynt's Google review NFC card at 30% off with free shipping. The page contains the whole order flow and posts orders to the existing store backend at `reviewstore.merchynt.com`, so every order lands in the same admin, HubSpot mapping and Stripe account as the current store. Built 2026-09-14.

| File | What it is |
|---|---|
| `index.html` | The page. Open it directly in a browser, or run the `nfc-review-cards` config in `.claude/launch.json` (port 3461). Single file: Tailwind CDN, DM Sans, vanilla JS. |
| `creative-brief.md` | Research, strategy, page structure, copy rationale, claims and sources, tracking plan, test ideas, open questions. |
| `replit-prompt.md` | Sectioned prompts to rebuild the page pixel for pixel in Replit (or inside the existing store app). |
| `replit-stripe-promo-brief.md` | Brief plus paste-ready prompt for the store's Replit owner: accept Stripe promotion codes, auto-apply from `?promo=`, deliver the discount to the Stripe session, record it from the webhook. |
| `lovable-prompt.md` | Sectioned prompts to rebuild the page pixel for pixel in Lovable as a React + Vite + Tailwind project (16 prompts: design system, one per section, API client and order state, wiring, behavior, QA). |
| `assets/hero-tap.jpg` | Hero photo: customer tapping a phone on the card at a cafe counter (generated, GPT Image 2.5 via Higgsfield, reference-matched to the real card). |
| `assets/salon-counter.jpg` | Card on a salon front desk (generated). Used in the Problem section. |
| `assets/handoff.jpg` | Technician handing the card to a homeowner (generated). Used in Who It's For. |
| `assets/card-stack.jpg` | Fanned stack of cards, studio shot (generated). Used in What You Get. |
| `assets/nfc-card-product.png` | The store's own product image (copied from reviewstore.merchynt.com). Used in the order summary. |
| `assets/qr-flyer-product.png` | The store's QR flyer image, kept for a future QR variant. Not used on the page. |
| `assets/merchynt-logo.svg` | Logo. |
| `assets/og-image.jpg` | Open Graph / social preview, 1200x630, free-card offer (Gemini 3 Pro Image, reference-matched to the card; logo composited in code). `og-image-square.jpg` is the 1200x1200 variant, `og-image-alt.jpg` leads with the One Tap headline instead. Candidates, the retired 30%-off versions and the white logo PNG live in `assets/og/`. |
| `assets/source/` | Original PNG renders from the image generator. |

## How the order flow works

1. Quantity chips (1, 3, 5, 10) or a stepper. Price and savings update live.
2. Business lookup by city and name (`GET /api/business-search`), or paste a Google review link. The link classifier is a copy of the store's, so the same links are accepted.
3. Contact and shipping. Address autocomplete (`GET /api/address/autocomplete?search=`) and USPS verification (`POST /api/address/verify`) use the store's endpoints. If USPS can't confirm, the buyer can tick "ship as typed".
4. The promo code in `CONFIG.couponCode` (`REVIEWS30`), or one passed in the URL as `?promo=` / `?coupon=` / `?code=` (kept in sessionStorage), is validated against `GET /api/orders/coupon` on load and every quantity change. If the store says it is invalid, the summary shows full price and an orange notice instead of silently over-promising.
5. "Proceed to Secure Checkout" posts the same payload the store's own checkout builds to `POST /api/orders`, then redirects to the returned Stripe `checkoutUrl`. Stripe returns the buyer to the store's `/order/confirmation` page.

## Before ads run

1. **Create the promo code in the store admin:** code `REVIEWS30`, 30% off, product `nfc_card`, no minimum quantity. Until it exists the page shows "Offer pending" and full price.
2. **Place one test order** from the live page through to the Stripe page and confirm the total shows $7 per card and no shipping line. (The automated test in this session stopped at the pay button; see the brief.)
3. Paste the Meta Pixel, Clarity and Vercel Analytics tags into the commented placeholders in `<head>`. The Purchase event has to fire on the store's confirmation page, not here.
4. Decide the domain. Live now at https://merchynt-review-cards.vercel.app (Vercel project `nfc-review-cards` under the `jamesrsowers-9743` account); a custom domain like `reviews.merchynt.com` needs a CNAME.

## Deploying

The folder is its own git repo and Vercel project (`nfc-review-cards`). After editing:

```bash
cd "08 - Landing Pages/nfc-review-cards" && git add -A && git commit -m "describe change" && git push && vercel --prod --yes
```
