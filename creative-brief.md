# NFC Review Cards Landing Page — Creative Brief

**Project:** Direct-response landing page for the Google review NFC card sold at reviewstore.merchynt.com
**Audience:** Small business owners arriving from Facebook and Instagram ads
**Offer:** 30% off (list $10, offer $7 per card) plus free shipping in the USA
**Conversion goal:** Completed Stripe checkout for NFC cards, measured per landing-page session
**Built:** 2026-09-14 (autonomous overnight build)
**Live:** https://merchynt-review-cards.vercel.app (Vercel project `nfc-review-cards`, GitHub `merchyntjames/nfc-review-cards`)
**Deliverables in this folder:** `index.html`, `creative-brief.md` (this file), `replit-prompt.md`, `README.md`, `assets/`

---

## 1. The assignment

Rebuild the front end of the review store as a single high-converting landing page for the NFC card, keep the existing backend untouched, and make sure every order lands in the same place as orders from the current store. James's framing: "promote super cheap NFC cards to help our SMB audience generate more reviews for their business", with a launch offer of 30% off and free shipping.

The existing store page is a four-step wizard (product, setup, quantity, checkout) with no marketing content at all. A visitor from an ad lands on "Which product would you like?" with two cards and no reason to pick either. That is the conversion problem this page solves.

## 2. What I looked at

| Source | What it gave me |
|---|---|
| reviewstore.merchynt.com, JS bundle read end to end | The complete API surface (`/api/products`, `/api/business-search`, `/api/address/autocomplete`, `/api/address/verify`, `/api/orders/coupon`, `/api/orders`, `/api/session`), the exact order payload the store's own checkout builds, the review-link classifier, phone normalization, the address verification contract, and the fact that CORS is open to any origin with credentials. This is what makes an external page possible with zero backend changes. |
| Live walkthrough of the store's four steps | The setup step copy ("Select the Google Business Profile you'd like to get reviews on or paste your review request link"), the quantity step, the checkout fields, the audience and Paige-customer questions, the shipping notice (USPS ground, next business day, up to 10 business days, no tracking), and the two trust lines ("Arrives ready to use", "Shows up unbranded"). |
| `/api/products` | NFC card: $10 per card, `shippingCents: 500` in the catalog but the store's checkout renders shipping as $0 and Free. So free shipping is already the store's reality; the page states it as part of the offer. Confirm the Stripe session also carries no shipping line (see open questions). |
| The store's product photo | White card, Google G, contactless icon, five stars, "Tap to review us", Google wordmark. Used as the reference image for every generated photo so the card on the page is the card that ships. |
| BrightLocal Local Consumer Review Survey 2026 (1,002 US consumers) | The three hero-adjacent stats: 97% read reviews for local businesses, 83% of people asked to leave a review left one, 47% won't use a business with fewer than 20 reviews. Also 74% look for reviews from the last three months (used in the brief's argument for constant asking, not on the page). |
| Competitor pricing (TAPro, Reviews Card, Digifeel, ABC RFID) | Retail tap-to-review cards run roughly $16 to $17 each in small bundles (TAPro: $34 for 2, $79 for 5), several with a subscription. Wholesale RFID blanks are about $4. So $7 programmed, shipped and unbranded is a real price anchor, which the page uses once. |
| `.agents/product-marketing-context.md`, brand memory, CLAUDE.md | Merchynt palette for parent-brand work (navy #0f007d, hot pink CTA), DM Sans, no emojis, no em dashes, never "no credit card required", spell out Google Business Profile, humanization caps, no five-digit percentage stats. Brand voice: confident, direct, numbers-forward, operator not marketer. |
| `08 - Landing Pages/gbp-audit-v2-smb/` | Folder conventions (brief, prompt, README, assets), the Tailwind config and motion system, the finding that SMB winners on Meta are concrete and outcome-led rather than abstract. |
| Senja review export (300 reviews) | Checked for any review that mentions the cards. None do (the three "card" hits are about credit cards). So the page carries no product testimonials rather than borrowing Paige testimonials for a different product. |

## 3. Diagnosis of the current store page

1. **No argument, only a form.** The first screen asks which product you want before saying why you'd want one. A paid visitor has to supply their own motivation.
2. **Two products compete on step one.** NFC card and QR flyer sit side by side with equal weight. Every ad that promotes the card sends half its attention to the flyer.
3. **No price above the fold.** $10 appears only inside the product card's fine print. "Super cheap" is the whole pitch and the page never says it.
4. **The setup step is the first thing after the product pick.** It is the highest-friction step (search Google, or find your review link) and it comes before the visitor has decided to buy. On this page the business lookup sits inside the order section after the full argument, and the search results themselves (photo, rating, review count) act as reassurance that the card will be programmed to the right listing.
5. **Objections are never handled.** Does it work on iPhone? Do I need an app? Is this allowed by Google? Will it say Merchynt? Nothing on the store answers these. The FAQ on this page does, in the owner's words.
6. **No social proof or context.** Not even the reason reviews matter. The stats bar and the problem section supply it.
7. **Progress bar says "Step 1 of 4" before any value is shown.** Announcing four steps is a cost, not a promise.

## 4. Strategy

**One product, one offer, one page.** The QR flyer is mentioned once, inside the last FAQ answer, as an email-us alternative. Every CTA on the page scrolls to the same order section. There is no navigation and no outbound link above the footer other than Google's "find your review link" help article, which opens in a new tab and serves the form.

**Lead with the mechanism and the price.** The headline names the outcome ("Get More Google Reviews") and the mechanism ("One Tap"). The hero photo shows the tap happening. The price is in three places above the fold: the CTA row, the floating price card, and the eyebrow pill with the offer. A visitor who reads nothing else knows what it is, what it does, and that it costs $7.

**Reframe the problem as timing, not effort.** Owners already know they should ask for reviews. The insight the page sells is that the ask fails because it comes too late (the next-day text, the sign nobody types in). The card moves the ask to the moment the phone is already out. This is the argument the three-card block and the phone mockup make, and it is why the page never says "easy".

**Show the customer's side of the tap.** The phone mockup replicates Google's own "Rate and review" sheet, then animates the stars filling and a review typing itself. It answers "what actually happens when someone taps?" without a paragraph of explanation.

**Make the order section the second hero.** The order flow is inline, three numbered cards with a sticky navy summary that recomputes price, discount and savings on every change. The visitor never leaves the page until Stripe. Business search results show the listing's photo, rating and review count so picking the right one feels safe. Address verification and autocomplete reuse the store's USPS endpoints so shipping mistakes stay low.

**Keep the offer honest by making the page check it.** The 30% code is validated against the store on load and on every quantity change. If the code isn't live, the summary shows full price with a visible "Offer pending" badge and an orange notice, rather than promising $7 and charging $10. When the code exists in the admin, the page flips to the discounted state on its own.

**Merchynt palette, not Paige.** This is a parent-brand physical product, so navy hero, hot pink CTAs, green for confirmation, orange for stars and warnings.

## 5. Page structure and copy rationale

| # | Section | Job | Notes |
|---|---|---|---|
| 0 | Header | Logo and one small CTA | No nav. CTA hidden on mobile because the sticky bar covers it. |
| 1 | Hero (navy) | Outcome, mechanism, price, proof of concept | H1 "Get More Google Reviews With One Tap". Sub names the product, the moment, and the three "no"s (app, monthly fee, Merchynt branding). Hero photo is the generated cafe tap shot. Floating price card ($7 vs $10, free shipping) and a "New 5-star review" notification card. Three green-check trust lines. |
| 2 | Stats bar | Why reviews matter, in numbers | Three BrightLocal 2026 figures with the source line. Chosen for the owner's three fears: nobody reads reviews (97% do), asking annoys people (83% comply), a few reviews is enough (47% skip you under 20). |
| 3 | Problem (cloud) | Reframe the problem as timing | Headline "Your customers would leave a review. Nobody asks them at the right moment." Two short paragraphs, then three cards: the next-day text, the "review us" sign, the tap (navy, the only positive one). Salon-counter photo under the copy on tablet and up. |
| 4 | How it works | Reduce perceived effort; show the customer's screen | Three steps with the tell-us / put-it-where-they-pay / they-tap structure. Google review sheet mockup animates once. CTA "Order Cards for 30% Off". |
| 5 | What you get (cloud) | Product truth and the price anchor | Card stack photo with a "Ships unbranded" badge. Six checklist items lifted from the store's feature list, rewritten as benefits. Price box: $7 vs $10 with the one competitor comparison ("$16 or more each, several charge a monthly fee"). |
| 6 | Who it's for | Let the visitor find themselves | Six business types, each with the specific spot the card lives. Handoff photo for service businesses, which Meta's SMB audience skews toward. |
| 7 | Order (cloud) | Convert | Step 1 quantity (chips 1/3/5/10 plus stepper, default 3). Step 2 business lookup or pasted link. Step 3 contact, shipping, buyer type, Paige customer. Sticky summary with line items, promo notice, "Proceed to Secure Checkout", Stripe and shipping notes. |
| 8 | FAQ | Objections in the owner's words | Phones, app or subscription, Google's rules, Merchynt branding, which listing, delivery time, multiple locations, customers who don't know how to tap. |
| 9 | Final CTA (navy) | Loss-frame close | "Stop hoping for reviews. Ask with a tap." Offer restated. Four "no"s. |
| 10 | Footer | Legal and contact | Privacy, Terms, store email. |
| — | Sticky mobile bar | One tap from the order section | Shows after the hero scrolls out, hides while the order section is on screen. |

### Headline options considered

| Option | Why it was or wasn't chosen |
|---|---|
| **Get More Google Reviews With One Tap** (chosen) | Names the outcome and the mechanism in seven words. "One Tap" takes the pink accent. Matches the store's own name ("Get More Reviews Store") so ad-to-page message match is easy. |
| More Google Reviews. One Tap. $7 a Card. | Numbers-forward, very on-brand, but three fragments in a row is a humanization flag and the price already appears three times in the hero. Strong A/B candidate. |
| Turn Happy Customers Into Google Reviews | Good transformation frame, but hides the mechanism, and "happy customers" invites the "I only ask happy ones" behavior Google prohibits. |
| The $7 Card That Gets You Google Reviews | Price-led. Worth testing if the ad creative leads with price. |

### CTA label options

| Option | Notes |
|---|---|
| **Get My Cards for 30% Off** (chosen) | First person, names the object, carries the offer. |
| Order Cards for 30% Off | Used mid-page for variety. |
| Proceed to Secure Checkout | The pay button. "Secure" and the lock icon pre-frame the hop to Stripe. |
| Show Me the $7 Card | Curiosity variant for testing. |

## 6. Design decisions

- **Single file, no framework.** Tailwind CDN, DM Sans, vanilla JS. Same stack as the other landing pages, so anyone on the team can edit it, and the Replit prompt can reproduce it without a build step.
- **Generated photography, reference-matched.** Four images from GPT Image 2.5 through Higgsfield, each with the store's real product photo as the reference so the printed card is exactly what ships. Prompts used emphatic photographic language and explicit negatives per the photoreal-humans rule; faces are cropped or absent by design so no one is misrepresented. Cost: 8 credits on the Ultra plan, effectively $0 of the $25 budget. Originals in `assets/source/`.
- **Phone mockup is code, not an image.** It reproduces Google's review sheet closely enough to be recognized, animates, and reproduces exactly in Replit.
- **No product testimonials.** There are none on record for the cards. Rather than borrow Paige reviews, the page uses third-party survey data for the "why" and the product's own concreteness for the "trust". Add real card testimonials when the first orders come back.
- **Quantity default is 3.** One card is the minimum useful order; three (register plus two staff) lifts average order value without feeling pushy. The chip labels and the helper line ("Add one for each person who talks to customers") give a reason.
- **Address override checkbox.** The store blocks checkout until USPS verifies the address. This page mirrors the verification but lets the buyer ship "as typed" if USPS can't confirm, because a hard block on a paid landing page loses orders over apartment numbers. The payload sent is identical either way.
- **Motion is restrained.** Fade-ups, hero stagger, two floating cards, a three-pulse on the hero CTA, and the phone animation once. All off under `prefers-reduced-motion`.
- **noindex, nofollow.** Paid-traffic page.

## 7. Integration notes

The page is a client of the existing store backend. Nothing on the store changed.

| Call | Purpose |
|---|---|
| `GET /api/products` | Live price for `nfc_card` (falls back to $10 if unreachable). |
| `GET /api/orders/coupon?code&productKey&quantity` | Validates `REVIEWS30` and returns `discountCents`. |
| `GET /api/business-search?name&city` | Google listing candidates with `reviewLink`. Takes 3 to 6 seconds in testing; the page shows a "Searching Google..." status meanwhile. |
| `GET /api/address/autocomplete?search=` | Address suggestions. |
| `POST /api/address/verify` | USPS verification and normalization (returns ZIP+4). |
| `POST /api/session` | Records the audience choice, same as the store does. |
| `POST /api/orders` | Creates the order and returns the Stripe `checkoutUrl`. The payload is field-for-field what the store's own checkout sends, including `setupMethod: "manual"`, a client-generated `idempotencyKey`, `shippingMethod: "standard"`, `isPaigeCustomer`, `isAgencyPartner` and `couponCode`. |

**Verified in this session against the live API:** product load, coupon validation (returns the store's own "isn't valid" message until the code exists), business search with real results and photos, address autocomplete, USPS verification with ZIP+4 normalization, and the full form validation. Cross-origin calls from both `localhost` and the live Vercel origin were accepted.

**Not verified:** the final `POST /api/orders` and the hop to Stripe. The automated browser blocked the pay click as a real-world transaction. The payload matches the store's bundle exactly, but please place one test order from the live page tomorrow and confirm (a) the Stripe page opens, (b) the total is $7 per card once the code exists, (c) no shipping line appears, and (d) the order shows in the admin with the review link attached. Cancel it from the admin afterwards.

## 8. Tracking and attribution

- **Landing page events (Meta Pixel, once installed):** `ViewContent` on load, `AddToCart` on any quantity change, `AddPaymentInfo` and `InitiateCheckout` on the pay click, plus a custom `OrderCTAClick` with the CTA placement (`header`, `hero`, `how`, `final`, `sticky`).
- **Purchase has to fire on the store.** Stripe returns buyers to `reviewstore.merchynt.com/order/confirmation?orderId=...&token=...`. That page needs the same pixel with a `Purchase` event. Because the domains differ, the pixel will treat it as a new session unless the same pixel ID is on both and `fbclid` survives, which Meta usually handles through first-party cookie matching on the same pixel. The cleaner fix is to host this page on the store's domain (see the Replit prompt's "inside the existing store app" option) or ask the Replit dev to make the Stripe `success_url` configurable so buyers return to this page.
- **UTMs** from the ad URL are stored in `sessionStorage` under `nfc_lp_attribution`. The order payload has no UTM field, so attribution beyond the pixel needs either the store's confirmation page to read Meta's cookie, or a small backend change to accept a `source` field and pass it into Stripe metadata.
- Vercel Analytics, Clarity and the pixel are commented placeholders in `<head>`.

## 9. Copy QA

- Humanization pre-flight (`trope-check.py --channel landing` on the extracted body copy, 1,023 words): 0 em dashes, 0 "here's the", 0 unicode decoration, 0 negative parallelism, 0 announced counts, 0 bold-first bullets. Two caps flagged and documented as overrides under A1 precedence: "standalone short paragraphs" (32) and "consecutive short paragraphs" (18 pairs). Both are an artifact of extracting UI components as paragraphs: the six checklist items, six business-type cards, three problem cards, three steps and eight FAQ answers are each short by channel convention. The actual prose paragraphs (problem section, FAQ answers) are three to five sentences.
- No emojis. No "no credit card". "Google Business Profile" is never abbreviated in visible copy; the page mostly says "Google listing" or "Google review page", which is the owner's language.
- Claims and their sources:
  - 97%, 83%, 47%: BrightLocal Local Consumer Review Survey 2026, 1,002 US adults, cited on the page.
  - "$10 per card", "arrives pre-programmed", "no app, no setup", "100% white-label", "durable premium card", "ships fast within the USA", USPS ground next business day, up to 10 business days, no tracking: the store's product record and checkout copy.
  - "Free shipping": the store's checkout shows Shipping: Free and charges $0 today. Confirm the Stripe session agrees.
  - "iPhones from 2018 on (XS, XR and newer) read the card automatically": Apple's background NFC tag reading shipped with iPhone XS and XR. Older iPhones need an app, which the FAQ implies with "almost everyone".
  - "Similar cards run $16 or more each, several charge a monthly fee": TAPro $34/2 and $79/5 (about $16 to $17 each); subscription models seen at several sellers. Soften to "often $15 or more" if you'd rather not anchor on one competitor's bundle.
  - "Google encourages businesses to ask for reviews and publishes a shareable link; prohibits paying or gating": Google Business Profile policies and the "Get more reviews" share link. Kept general on purpose.
  - "Each order is programmed to one Google listing": the order payload carries one `reviewLink`.

## 10. Test ideas after launch

1. Hero headline: chosen vs "More Google Reviews. One Tap. $7 a Card."
2. Default quantity: 3 vs 1 (conversion rate vs average order value).
3. Order section position: after the argument (current) vs a compact order card in the hero right column with the photo moved down.
4. Stats bar: BrightLocal figures vs a single bold line ("83% of customers asked for a review leave one").
5. Price anchor sentence: competitor comparison vs "$7 is less than one cup of coffee a card" (softer, no competitor claim).
6. Business lookup default: search (current) vs "paste your link" first, for service-area-heavy audiences.
7. Sticky bar copy: "$7 per card" vs "Save 30% today".

## 11. Open questions and requirements for James

1. **Create the promo code.** `REVIEWS30`, 30% off, product `nfc_card`, no minimum. The page is built to auto-apply it and shows "Offer pending" until it exists. If you'd prefer a different code, change `CONFIG.couponCode` at the top of the script.
2. **Place one test order** from https://merchynt-review-cards.vercel.app through to the Stripe page (don't pay), then cancel it in the admin. This is the one step the automated test couldn't complete.
3. **Confirm free shipping in Stripe.** The catalog has `shippingCents: 500` on the card but the store charges $0. Make sure the Stripe session doesn't add it back.
4. **Vercel account.** The CLI on this machine is logged in as `jamesrsowers-9743` (the personal account), which is not where `paige` and `gbp-audit` live. The page is live there with Vercel Authentication turned off for this project. If you want it under the Merchynt account, import `github.com/merchyntjames/nfc-review-cards` from the Vercel dashboard and it will deploy on push. I also tried to connect the GitHub repo for auto-deploys from the CLI and that failed because the account doesn't have GitHub access to the org; deploys currently go through `vercel --prod`.
5. **Domain.** `merchynt-review-cards.vercel.app` is live. `nfc-review-cards.vercel.app` was already taken by someone else. A custom subdomain (`reviews.merchynt.com` or `cards.merchynt.com`) needs a CNAME and would also let the Meta Pixel share a first-party context with the store if the store moves under merchynt.com too.
6. **Purchase pixel on the store's confirmation page**, or a configurable Stripe return URL. Without one, Facebook can't see completed orders.
7. **Testimonials.** Any quotes, photos or numbers from existing card buyers? The page has a natural slot between Who It's For and Order.
8. **Multi-location buyers.** The FAQ says one listing per order. If the store can take several listings in one order, tell me and I'll change the copy and the form.
9. **Repo visibility.** I made the GitHub repo public to match `paige-agencies` and `gbp-audit` (the page is noindex and contains no secrets). Say the word and I'll flip it private.
