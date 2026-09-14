# Replit Build Prompt — NFC Review Cards Landing Page

**How to use this file.** Paste the prompts into Replit Agent one at a time, in order, and let each one finish before sending the next. Prompt 0 sets up the project and the design system. Prompts 1 through 11 each build one section. Prompt 12 wires up the order flow. Prompt 13 is the QA pass. Every prompt is self-contained, but they assume the earlier ones ran.

**Source of truth.** The finished reference is `index.html` in this folder, with images in `assets/`. Upload both to the Replit project before Prompt 0 and tell the agent they exist. When a prompt and the reference disagree, the reference wins. The goal is a pixel-for-pixel match, not an interpretation.

**Two ways to build it.**
- *Standalone static site* (default, matches the reference exactly): a single `index.html` served from the project root. Nothing to install.
- *Inside the existing store app* (`reviewstore.merchynt.com`, React + Vite + Tailwind + shadcn): add a route at `/cards` and rebuild the page as a React page. Replace `CONFIG.apiBase` with relative `/api/...` paths and reuse the existing `createOrder`, `businessSearch`, `addressAutocomplete`, `verifyAddress` and `coupon` hooks from the app's generated API client. Everything else in these prompts still applies. The advantage: same domain, so the Meta Pixel Purchase event on `/order/confirmation` shares a session with the landing page.

---

## Prompt 0 — Project setup and design system

Create a static landing page project. There is no framework and no build step: one file, `index.html`, plus an `assets/` folder that I have uploaded (hero-tap.jpg, salon-counter.jpg, handoff.jpg, card-stack.jpg, nfc-card-product.png, merchynt-logo.svg). Serve `index.html` from the root.

Load Tailwind from the CDN (`https://cdn.tailwindcss.com`) and configure it inline with this exact theme extension:

```js
tailwind.config = {
  theme: { extend: {
    colors: {
      navy: '#0f007d', navydeep: '#08004a', pink: '#dd0cf7', blue: '#0063fd', purple: '#8b00cc',
      green: '#05c168', orange: '#ff9e2c', dark: '#1c1f23', muted: '#8E8B84',
      cloud: '#f6f5ff', line: '#e6e4f2',
    },
    fontFamily: { sans: ['DM Sans', 'system-ui', 'sans-serif'] },
    boxShadow: {
      card: '0 10px 30px -10px rgba(15, 0, 125, 0.18)',
      float: '0 24px 60px -20px rgba(8, 0, 74, 0.55)',
      panel: '0 30px 80px -30px rgba(15, 0, 125, 0.25)',
    },
  } }
}
```

Load the Google font DM Sans with weights 300, 400, 500, 600, 700, 800 and italic 400 (`optical size 9..40`). Body: `bg-white text-dark font-sans`, antialiased.

Add these global styles verbatim:

- `html { scroll-behavior: smooth; }`
- `.fade-up` starts at `opacity:0; transform: translateY(22px)` with a `.6s ease-out` transition on both, and `.fade-up.visible` resets to `opacity:1; transform:none`. An IntersectionObserver at threshold 0.12 adds `.visible` once per element.
- `.hero-animate` is the same idea but keyframe-driven on page load (`heroUp .7s ease-out forwards`, starting from `translateY(16px)`), with delay classes `.d1 .05s`, `.d2 .2s`, `.d3 .35s`, `.d4 .5s`, `.d5 .7s`.
- `.cta-pulse`: box-shadow ring pulse in pink `rgba(221,12,247,.45)` to `0 0 0 10px transparent`, `2.4s ease-in-out`, starts after 1.2s, runs 3 times.
- `.cta-btn`: `transition: transform .15s, box-shadow .15s, background-color .15s, opacity .15s`; hover `translateY(-1px) scale(1.02)`; active `scale(.99)`; disabled `opacity:.55; cursor:not-allowed`.
- `.hero-grid`: `radial-gradient(rgba(255,255,255,.08) 1px, transparent 1px)` at `22px 22px`.
- `.float-a` floats up 8px and back over 6s; `.float-b` floats down 7px and back over 7s; both `ease-in-out infinite`.
- `.faq-answer { max-height:0; overflow:hidden; transition: max-height .35s ease }`, `.faq-answer.open { max-height: 420px }`, `.faq-chevron` rotates 180deg when `.rotated`.
- `.sticky-cta` is translated `110%` down and slides to `0` when `.show`.
- `.strike`: `text-decoration: line-through; text-decoration-thickness: 2px; text-decoration-color: #dd0cf7`.
- Form styles: `.field` is a full-width input with `1.5px solid #e6e4f2` border, `12px` radius, `12px 14px` padding, `15px` text, focus ring `0 0 0 4px rgba(15,0,125,.08)` with navy border, `.field.error` pink border, placeholder `#a7a4b3`. `.label` is `13px` semibold with `6px` bottom margin. `.qty-chip` is a bordered `14px`-radius tile that turns navy with white text when `.active`. `.choice` is a bordered `12px`-radius pill-ish button, `14px` semibold, navy border and 5% navy fill when `.active`. `.biz-result` is a flex row with `12px` radius that gets a cloud background on hover. `.suggest` is an absolutely positioned dropdown under the address field with a `12px` radius, line border and `0 16px 40px -16px rgba(15,0,125,.3)` shadow. `.step-num` is a `34px` navy circle with white bold `14px` text, and `.step-num.done` turns green. `.spinner` is an `18px` white ring spinner; `.spinner.dark` is navy.
- Phone mockup: `.phone` is `250px` wide, `38px` radius, near-black `#0b0b12` body with `10px` padding and an inset `2px #2a2a38` ring plus the float shadow; `.phone-screen` is white, `30px` radius, `min-height: 500px`, overflow hidden.
- Under `prefers-reduced-motion: reduce`, all of the above animations and transitions are disabled and elements render in their final state.

Head: title "Get More Google Reviews With One Tap | NFC Review Cards from $7 | Merchynt", meta description "A pre-programmed NFC card for your counter. Customers tap their phone and your Google review page opens. No app, no monthly fee, ships unbranded. 30% off and free shipping for a limited time.", `robots noindex, nofollow`, Open Graph title/description/type/image (`assets/hero-tap.jpg`). Leave an HTML comment block for tracking placeholders (Meta Pixel, Clarity, Vercel Analytics).

Page content max width is `max-w-6xl` with `px-5` gutters on every section. Section vertical padding is `py-16 sm:py-24` unless stated. No emojis anywhere. No em dashes anywhere in copy.

---

## Prompt 1 — Header

Absolutely positioned over the hero (`absolute top-0 inset-x-0 z-30`), transparent. Inside a `max-w-6xl mx-auto px-5 py-5 flex items-center justify-between` row:

- Left: the Merchynt logo (`assets/merchynt-logo.svg`) rendered white with `brightness-0 invert`, height `h-7 sm:h-8`, wrapped in a link to `#top`, with `hero-animate d1`.
- Right (hidden below `sm`): a pink pill button, text "Get 30% Off", `rounded-full bg-pink text-white text-sm font-semibold px-5 py-2.5 shadow-card`, class `cta-btn hero-animate d2`, `href="#order"`, attribute `data-cta="header"`.

No navigation links of any kind.

---

## Prompt 2 — Hero

Section `id="top"`, `relative bg-navy text-white overflow-hidden`. Background layers: the `.hero-grid` dot pattern at `opacity-70`; a `560px` circle of `bg-blue/30 blur-3xl` at `-top-40 -right-32`; a `520px` circle of `bg-pink/25 blur-3xl` at `-bottom-48 -left-32`.

Grid: `max-w-6xl mx-auto px-5 pt-28 pb-16 sm:pt-32 sm:pb-24 grid lg:grid-cols-12 gap-10 lg:gap-8 items-center`.

**Left column (`lg:col-span-6`):**

1. Eyebrow pill (`hero-animate d1`): `inline-flex items-center gap-2 rounded-full bg-white/10 border border-white/15 px-3.5 py-1.5 text-xs sm:text-sm font-semibold tracking-wide` with a `w-2 h-2 rounded-full bg-green` dot and the text "Limited time: 30% off + free shipping".
2. H1 (`hero-animate d2 mt-5 text-4xl sm:text-5xl lg:text-[3.4rem] font-extrabold leading-[1.05] tracking-tight`): "Get More Google Reviews With One Tap", with "One Tap" wrapped in `text-pink`.
3. Subhead (`hero-animate d3 mt-5 text-lg sm:text-xl text-white/80 leading-relaxed max-w-xl`): "A pre-programmed NFC card for your counter. Customers tap their phone, your Google review page opens, and they write the review while the good experience is still fresh. No app, no monthly fee, no Merchynt branding."
4. CTA row (`hero-animate d4 mt-7 flex flex-col sm:flex-row sm:items-center gap-4`):
   - Button `href="#order" data-cta="hero"`, classes `cta-btn cta-pulse inline-flex items-center justify-center gap-2 rounded-full bg-pink text-white text-base sm:text-lg font-bold px-8 py-4 shadow-float`, text "Get My Cards for 30% Off" followed by a 20px right-arrow line icon.
   - Beside it, `text-sm text-white/75 leading-snug`: "$10" in `.strike text-white/50`, then "$7 per card" in `text-white font-bold text-base`, line break, "Free shipping in the USA".
5. Trust row (`hero-animate d5 mt-8 flex flex-wrap items-center gap-x-6 gap-y-3 text-sm text-white/75`), each item a green check icon (16px, stroke 3) plus text: "Arrives programmed and ready to use", "Works with iPhone and Android", "Mailed next business day".

**Right column (`lg:col-span-6 relative hero-animate d3`):**

- Image frame: `relative rounded-3xl overflow-hidden shadow-float border border-white/10 bg-white/5` containing `assets/hero-tap.jpg` at `w-full h-[320px] sm:h-[440px] object-cover`, alt "A customer taps their phone on a Google review NFC card at a cafe counter".
- Floating price card (`float-a absolute -bottom-5 -left-3 sm:-left-8 bg-white text-dark rounded-2xl shadow-float px-5 py-4 w-[210px]`): label "PER CARD, THIS WEEK" (`text-[11px] font-semibold uppercase tracking-wider text-muted`), then "$7" (`text-3xl font-extrabold text-navy`) beside "$10" (`.strike text-muted font-semibold`), then a green line "Free shipping included" with a small check icon (`text-xs font-semibold text-green`).
- Floating review notification (`float-b absolute -top-4 -right-2 sm:-right-6 bg-white text-dark rounded-2xl shadow-float px-4 py-3 w-[230px]`): a `36px` cloud circle holding the four-color Google "G" SVG, then "New 5-star review" (`text-[13px] font-bold`) over a row of five orange stars (14px each) and "just now" in `text-[11px] text-muted`.

---

## Prompt 3 — Stats bar

White section with `border-b border-line`, padding `py-10 sm:py-12`. A `grid sm:grid-cols-3 gap-6 sm:gap-8`; each cell is `fade-up`, centered on mobile and left-aligned from `sm`:

| Number (`text-4xl sm:text-5xl font-extrabold text-navy tracking-tight`) | Line under it (`mt-2 text-sm sm:text-base text-dark/80 leading-snug`) |
|---|---|
| 97% | of consumers read reviews before choosing a local business |
| 83% | of people who were asked to leave a review went on to leave one |
| 47% | won't use a business that has fewer than 20 reviews |

Cells 2 and 3 get `transition-delay: .1s` and `.2s`. Under the grid: `fade-up mt-6 text-xs text-muted` "Source: BrightLocal Local Consumer Review Survey 2026, 1,002 US consumers."

---

## Prompt 4 — Problem section

Section `bg-cloud`. Grid `lg:grid-cols-12 gap-10 items-center`.

**Left (`lg:col-span-6 fade-up`):**
- Eyebrow `text-xs font-bold uppercase tracking-[0.18em] text-pink`: "The problem".
- H2 `mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight`: "Your customers would leave a review. Nobody asks them at the right moment."
- Paragraph 1 (`mt-5 text-base sm:text-lg text-dark/80 leading-relaxed`): "Most owners ask the hard way. A text the next day. An email that lands in Promotions. A "review us" sign that asks people to type a URL. By then the moment has passed. The customer who was happy at the counter is busy again, and the review never gets written."
- Paragraph 2 (`mt-4`, same classes): "The card fixes the timing. It sits where customers already have their phone out, and it turns "would you mind leaving us a review?" into a two-second tap."
- Below the copy, hidden on mobile (`hidden sm:block`), a `mt-7 rounded-2xl overflow-hidden shadow-card` frame with `assets/salon-counter.jpg` at `w-full h-56 lg:h-64 object-cover`, lazy-loaded, alt "A Google review NFC card standing on a salon front desk".

**Right (`lg:col-span-6 grid gap-4`):** three stacked cards, each `fade-up rounded-2xl p-5 shadow-card flex gap-4 items-start` with a `40px` icon tile on the left (`rounded-xl` cloud background, navy 20px line icon) and a title (`font-bold`) over a `mt-1 text-sm` line:

1. White card, chat-bubble icon. "The next-day text" / "Buried under forty other notifications. Opened, maybe. Acted on, rarely."
2. White card (delay .1s), screen-on-stand icon. "The "review us" sign" / "Reading a web address, typing it, finding your listing, finding the button. Every step loses people."
3. Navy card with white text (delay .2s), icon tile `bg-white/10`, contactless-waves icon. "The tap" / "Phone is already out. Your review form opens on its own. Stars, a sentence, post. Done before the receipt prints." (the body line is `text-white/75`).

---

## Prompt 5 — How it works (with phone mockup)

White section. Grid `lg:grid-cols-12 gap-12 items-center`.

**Left (`lg:col-span-5`, `order-2 lg:order-1`, `flex justify-center fade-up`):** the phone mockup from Prompt 0. Inside `.phone-screen` (`text-[13px] text-dark`):
- Status bar row: "9:41" left, a tiny battery block right, `px-5 pt-4 text-[11px] font-semibold`.
- Row `px-4 pt-4` with an X icon (24px) and "Rate and review" in `font-semibold text-[14px]`.
- Row `px-4 pt-4`: a `40px` circle with a `from-blue to-pink` gradient and a bold white "Y", then "Your business name" (`font-semibold text-[13px]`) over "Posting publicly" (`text-[11px] text-muted`).
- Star row `px-4 pt-5 flex items-center gap-2`, id `phoneStars`: five `32px` star SVGs, initial fill `#d9d9e3`, class `star-fill`.
- Text box `mx-4 mt-5 rounded-xl border border-line p-3 min-h-[92px] text-[12px] text-dark/80`, id `phoneText`, initial content "Share details of your own experience at this place" in `text-muted`.
- Button row `px-4 mt-4 flex gap-2`: "Add photos" (outlined pill, `text-muted`) and "Post" (filled `bg-blue` pill, white text), both `text-[12px] font-semibold text-center py-2 rounded-full flex-1`.
- Caption `px-4 pt-6 pb-5 text-[11px] text-muted leading-snug`: "Opened by a tap on your card. No app, no search, no typing a link."

Behavior: when the phone scrolls 50% into view, the five stars turn `#ff9e2c` one after another (500ms start, 180ms apart), then after 1.6s the text box clears and types out "Fast, friendly and fair on price. Will be back." at 32ms per character. Runs once.

**Right (`lg:col-span-7`, `order-1 lg:order-2`):**
- Eyebrow (pink, same style as before): "How it works".
- H2: "Set up once. Then every tap is a review request."
- Three steps (`mt-8 space-y-6`), each `fade-up flex gap-4` with a `.step-num` circle and a block containing a `text-lg font-bold` title and a `mt-1 text-dark/75 leading-relaxed` paragraph:
  1. "Tell us which Google listing" / "Search your business by city and name below, or paste your Google review link. We program every card to open your review page before it ships. You never touch the settings."
  2. (delay .1s) "Put the card where customers pay" / "The register, the front desk, the check presenter, the service counter, the truck. Anywhere a phone is already out and the experience is still fresh."
  3. (delay .2s) "Customers tap, rate, post" / "Their phone opens your Google review form on its own. They pick the stars, write a sentence, and hit Post. No app to download, nothing to type, nothing to find."
- CTA `href="#order" data-cta="how"`, `fade-up cta-btn mt-9 inline-flex items-center gap-2 rounded-full bg-pink text-white font-bold px-7 py-3.5 shadow-card`, text "Order Cards for 30% Off" plus arrow icon.

---

## Prompt 6 — What you get (product)

Section `bg-cloud`. Grid `lg:grid-cols-12 gap-10 items-center`.

**Left (`lg:col-span-6 fade-up`):** `relative rounded-3xl overflow-hidden shadow-card bg-white` holding `assets/card-stack.jpg` at `w-full aspect-square object-cover`, alt "A stack of Google review NFC cards", with a badge in the top-left corner (`absolute top-4 left-4 bg-navy text-white text-xs font-bold px-3 py-1.5 rounded-full`) reading "Ships unbranded".

**Right (`lg:col-span-6`):**
- Eyebrow: "What you get". H2: "A premium card that does the asking for you".
- Checklist `mt-7 space-y-3.5`, each item `fade-up flex gap-3 items-start` with a green 20px check (stroke 3) and text where the first sentence is bold:
  1. **Pre-programmed to your Google review page.** It arrives ready. Set it on the counter and it works.
  2. **Works with iPhones from 2018 on and any Android with NFC.** The same tap your customers already use to pay.
  3. **No app, no account, no monthly fee.** One price, and the card keeps working.
  4. **No Merchynt branding anywhere on it.** Your customers see Google, five stars and "Tap to review us."
  5. **Durable card stock built for a counter.** Survives coffee, keys and a thousand taps.
  6. **Mailed USPS the next business day.** Free shipping anywhere in the USA.
- Price box `fade-up mt-8 bg-white rounded-2xl p-5 shadow-card border border-line`, inner `flex flex-wrap items-end justify-between gap-4`: left has the label "THIS WEEK'S PRICE" (`text-xs font-semibold uppercase tracking-wider text-muted`) then "$7" (`text-4xl font-extrabold text-navy`), "$10" (`.strike text-muted font-semibold`), "per card" (`text-sm text-muted`); right is `text-sm text-dark/70 max-w-[260px]`: "Similar tap-to-review cards from other sellers run $16 or more each, and several charge a monthly fee on top."

---

## Prompt 7 — Who it's for

White section. Intro block `max-w-2xl`: eyebrow "Who it's for", H2 "Built for businesses that see their customers face to face", paragraph `fade-up mt-4 text-dark/75 text-lg`: "If there's a moment when a happy customer is standing in front of you with a phone in hand, the card belongs in that moment."

Then `mt-10 grid md:grid-cols-12 gap-6 items-stretch`:
- `md:col-span-5 fade-up rounded-3xl overflow-hidden shadow-card min-h-[280px]` holding `assets/handoff.jpg` at `w-full h-full object-cover`, alt "A technician hands a Google review card to a homeowner".
- `md:col-span-7 grid sm:grid-cols-2 gap-4` of six cards, each `fade-up bg-cloud rounded-2xl p-5` with a `font-bold text-navy` title over a `mt-1 text-sm text-dark/70` line, staggered delays of .05s:
  1. Restaurants and cafes / By the register, or tucked into the check presenter.
  2. Salons and barbers / At the front desk while they're settling up and still admiring the cut.
  3. Auto shops / On the service counter when the keys come back.
  4. Dentists and clinics / At the checkout window, where every patient stops on the way out.
  5. Contractors and home services / Hand it over at the door when the job is done and the customer is impressed.
  6. Retail and boutiques / Right next to the card reader, where the phone already is.

---

## Prompt 8 — Order section (layout and static content)

Section `id="order"`, `bg-cloud border-t border-line scroll-mt-4`. Intro `max-w-2xl`: eyebrow "Order", H2 "Order your cards", paragraph `fade-up mt-3 text-dark/75 text-lg`: "30% off is applied for you. Free shipping in the USA. Payment happens on a secure Stripe checkout page."

Then `mt-10 grid lg:grid-cols-12 gap-8 items-start`.

**Left column `lg:col-span-7 space-y-6`, three white cards** (`fade-up bg-white rounded-3xl shadow-card border border-line p-6 sm:p-8`). Each starts with a `flex items-center gap-3` row: a `.step-num` (ids `stepNum1`, `stepNum2`, `stepNum3`) and an `h3 text-xl font-bold`, followed by a `mt-2 text-sm text-dark/70 ml-[46px]` helper line.

*Card 1: "How many cards?"* Helper: "One for the register is the minimum. Add one for each person who talks to customers." Then `mt-5 grid grid-cols-4 gap-2 sm:gap-3` id `qtyChips` of four `.qty-chip` buttons with `data-qty` 1, 3, 5, 10; each shows the number (`text-xl font-extrabold`) over "card"/"cards" (`qty-sub text-[11px] text-muted font-medium`). Under it, `mt-4 flex items-center gap-3`: "Or pick a number:" (`text-sm text-dark/70`), a pill stepper (`inline-flex items-center border border-line rounded-full overflow-hidden`) with a minus button id `qtyMinus`, a number input id `qtyInput` (`w-14 text-center font-bold`, min 1, max 500, value 3) and a plus button id `qtyPlus`, then a `text-sm font-semibold text-green` span id `qtySavings`.

*Card 2: "Which Google listing should the card open?"* Helper: "We program the cards to your Google review page before they ship." Three swappable blocks:
- `bizSearchWrap` (`mt-5`): `grid sm:grid-cols-2 gap-3` with labeled `.field` inputs `bizCity` ("City your business is in", placeholder "e.g. Austin, TX") and `bizName` ("Business name", placeholder "e.g. Joe's Plumbing"); a status line `bizStatus` (`mt-3 text-sm text-muted min-h-[20px]`); a hidden results list `bizResults` (`mt-2 space-y-1`); a text button `toggleLinkMode` (`mt-4 text-sm font-semibold text-blue hover:underline`): "Service-area business or can't find it? Paste your Google review link instead".
- `bizLinkWrap` (hidden): label "Your Google review link", url input `reviewLinkInput` with placeholder "https://g.page/r/... or https://search.google.com/local/writereview?placeid=...", a status line `linkStatus`, a hidden checkbox row `linkOverrideWrap` with checkbox `linkOverride` and the text "Use this link anyway. I've checked that it opens my Google review form.", then a row with the link "How do I find my Google review link?" (opens `https://support.google.com/business/answer/3474122` in a new tab) and a text button `toggleSearchMode` "Search for my business instead".
- `bizSelected` (hidden): a `flex items-center gap-4 rounded-2xl border-2 border-green/60 bg-green/5 p-4` row with a `56px` photo (`selPhoto`, rounded-xl) or a building-icon fallback (`selPhotoFallback`), a block with "SELECTED" (`text-xs font-bold uppercase tracking-wider text-green`), the business name `selName` (`font-bold truncate`) and `selMeta` (`text-sm text-dark/70 truncate`), and a "Change" text button `changeBiz`.

*Card 3: "Where should we ship them?"* Helper: "We ship within the US only. We'll email your order confirmation and call only if something needs checking." Then `mt-5 grid sm:grid-cols-2 gap-3` of labeled `.field` inputs: `email` (full width, "Email address", placeholder you@business.com), `phone` ("Cell phone number", placeholder (555) 123-4567), `businessName` ("Business name", placeholder Acme Co.), `firstName`, `lastName`, `address1` (full width, "Street address", placeholder "Start typing your address", with the hidden `.suggest` dropdown `addrSuggest` under it), `address2` (full width, "Apt, suite, unit (optional)"), `city`, and a two-column pair `state` (placeholder TX, maxlength 2) and `postalCode` (label "ZIP", placeholder 78701, numeric keyboard). Below: status line `addrStatus` and a hidden checkbox row `addrOverrideWrap` with checkbox `addrOverride`: "Ship to this address exactly as I typed it."

Then a `mt-6 pt-6 border-t border-line grid sm:grid-cols-2 gap-5` block:
- "Who are you buying for?" with two `.choice` buttons in `audienceChoices`: "My own business" (`data-aud="smb"`, active) and "A client" (`data-aud="agency"`).
- "Are you a Paige customer?" with `paigeChoices`: "Yes" and "No" (No active).
- Hidden full-width `partnerWrap`: "Are you in the Merchynt Agency Partner program?" with `partnerChoices` Yes / No (`max-w-xs`).

**Right column `lg:col-span-5 lg:sticky lg:top-6`, one navy panel** (`fade-up bg-navy text-white rounded-3xl shadow-panel p-6 sm:p-8`):
- Header row: "Your order" (`text-lg font-bold`) and a badge `offerBadge` (`text-[11px] font-bold uppercase tracking-wider px-2.5 py-1 rounded-full`, green tint) reading "30% off applied".
- Product row `mt-5 flex items-center gap-4`: `assets/nfc-card-product.png` at `w-16 h-16 rounded-xl object-cover bg-white`, then "Google Review NFC Card" (`font-bold`) over `sumQtyLine` (`text-sm text-white/70`), then right-aligned `sumListLine` (`text-sm text-white/50 strike`) over `sumNetLine` (`font-bold`).
- Totals `mt-5 space-y-2 text-sm border-t border-white/10 pt-4`: Subtotal / `sumSubtotal`; a green row "30% off (`sumCode`)" / `sumDiscount`; "Shipping (USPS, USA)" / "Free" in green semibold; then a `text-lg font-extrabold pt-3 border-t border-white/10` row Total / `sumTotal`.
- Hidden notice `couponNotice` (`mt-3 text-xs rounded-xl bg-orange/15 text-orange px-3 py-2`).
- A `details` element: summary "Have a different promo code?" (`text-white/60`), containing a translucent `.field` `couponInput` (placeholder "Code") and an "Apply" button `couponApply` (`rounded-xl bg-white/15 px-4 font-semibold`).
- Hidden error box `formErrors` (`mt-4 text-sm rounded-xl bg-pink/20 text-white px-4 py-3`).
- Button `payBtn`: `cta-btn mt-5 w-full inline-flex items-center justify-center gap-2 rounded-full bg-pink text-white text-lg font-bold px-6 py-4 shadow-float`, a lock icon and span `payBtnText` "Proceed to Secure Checkout".
- Under it `mt-3 flex flex-wrap justify-center gap-x-4 gap-y-1 text-[12px] text-white/60`: "Secure payment by Stripe", "Card programmed before shipping", "Unbranded".
- Footnote `mt-4 text-[12px] text-white/50 leading-snug`: "We mail every order USPS ground on the next business day. Delivery can take up to 10 business days depending on where you are. To keep the price this low there's no tracking number. Questions: reviewstore@merchynt.com" (the email is a mailto link, underlined).

---

## Prompt 9 — FAQ

White section, content `max-w-3xl`. Eyebrow "Questions", H2 "What owners ask before they order". Then `mt-8 divide-y divide-line` id `faq`. Each item is `faq-item py-5` with a full-width `faq-q` button (`flex items-center justify-between text-left gap-4`, question in `font-bold text-lg`, navy chevron `faq-chevron` 20px) and a `faq-answer` div containing `p.pt-3 text-dark/75 leading-relaxed`. Opening one closes the others.

1. **Will it work with my customers' phones?** Yes for almost everyone. iPhones from 2018 onward (XS, XR and newer) read the card automatically when it's held near the top of the phone. Android phones with NFC do the same. It's the same technology people use to tap and pay, so most customers already know the motion.
2. **Do I need an app or a subscription?** No. You pay once for the cards. There's no account to create, no app for you or your customers, and no monthly fee. The card carries your review link on its own and keeps working.
3. **Is asking for reviews this way allowed by Google?** Yes. Google encourages businesses to ask customers for reviews and even publishes a shareable review link for exactly this purpose. What Google prohibits is paying or rewarding people for reviews, or only asking the customers you know are happy. The card simply makes the ask easy for everyone.
4. **Does the card say Merchynt anywhere?** No. The card shows the Google logo, five stars, a tap symbol and "Tap to review us." Nothing about Merchynt appears on the card or in what your customers see on their phone.
5. **How do you know which Google listing to use?** You pick it in step 2 above. Search by city and business name and choose your listing from the results, or paste your Google review link if you're a service-area business without a public address. We program every card to that exact link before it ships.
6. **How long until it arrives?** We mail every order USPS ground on the next business day. Most arrive within a week, and it can take up to 10 business days depending on where you are. Shipping is free. To keep the card at this price there's no tracking number.
7. **I have more than one location.** Each order is programmed to one Google listing, so place one order per location. The discount applies to every order while the offer is running.
8. **What if a customer doesn't know how to tap?** Say "hold your phone against the card." That's the whole instruction, and it's the same motion as tapping to pay. If you'd rather offer a QR code as well, we also print a QR review flyer. Email reviewstore@merchynt.com and we'll point you to it. (email is a blue semibold mailto link)

---

## Prompt 10 — Final CTA and footer

**Final CTA:** `relative bg-navy text-white overflow-hidden`, hero-grid at `opacity-60`, a `420px` `bg-pink/25 blur-3xl` circle at `-top-32 right-0`. Content `max-w-4xl mx-auto px-5 py-20 sm:py-28 text-center`:
- H2 `fade-up text-3xl sm:text-5xl font-extrabold leading-tight tracking-tight`: "Stop hoping for reviews." line break "Ask with a tap."
- Paragraph `fade-up mt-5 text-lg text-white/80 max-w-2xl mx-auto`: "$7 a card while the offer lasts. Free shipping. Programmed to your Google listing and ready to use the day it lands in your mailbox."
- Button `href="#order" data-cta="final"`, `fade-up cta-btn mt-8 inline-flex items-center gap-2 rounded-full bg-pink text-white text-lg font-bold px-9 py-4 shadow-float`: "Get My Cards for 30% Off" plus arrow.
- Row `fade-up mt-8 flex flex-wrap justify-center gap-x-6 gap-y-2 text-sm text-white/70`: "No app", "No monthly fee", "No Merchynt branding", "Ships next business day".

**Footer:** `bg-navydeep text-white/60`, `max-w-6xl mx-auto px-5 py-8 flex flex-col sm:flex-row items-center justify-between gap-4 text-sm`. Left: the logo (white, `h-5`, `opacity-70`) and "© {current year} Merchynt. All rights reserved." Right: links "reviewstore@merchynt.com" (mailto), "Privacy" (https://www.merchynt.com/privacy-policy), "Terms" (https://www.merchynt.com/terms-of-service), each `hover:text-white`.

---

## Prompt 11 — Sticky mobile CTA

A fixed bar `id="stickyCta"`, `sticky-cta fixed bottom-0 inset-x-0 z-40 lg:hidden bg-white/95 backdrop-blur border-t border-line px-4 py-3 flex items-center justify-between gap-3`. Left: "NFC review cards" (`text-xs text-muted`) over "$10" (`.strike text-muted font-semibold text-sm`) and "$7 per card" (`font-extrabold text-navy`). Right: pink pill `href="#order" data-cta="sticky"`, "Get 30% Off", `text-sm font-bold px-5 py-3 shadow-card`.

Behavior: an IntersectionObserver at threshold 0.05 watches the hero and the order section. The bar shows only when the hero has scrolled out of view and the order section is not on screen.

---

## Prompt 12 — Order flow behavior (JavaScript)

Implement the order flow in one inline script (an IIFE). The backend is the existing store; do not build any server code.

**Config**
```js
const CONFIG = {
  apiBase: 'https://reviewstore.merchynt.com',
  productKey: 'nfc_card',
  listPriceCents: 1000,       // fallback until /api/products loads
  couponCode: 'REVIEWS30',    // auto-applied; must exist in the store admin
  offerPercent: 30,
  defaultQty: 3,
  minNameChars: 4,
  searchDebounceMs: 450,
  addrDebounceMs: 250,
};
```
All requests use `fetch` with `credentials: 'include'`, `Accept: application/json`, and `Content-Type: application/json` when there is a body. Non-2xx responses throw an error whose message is the JSON `error` or `message` field.

**State**: `product`, `qty`, `coupon` ({code, discountCents} or null), `mode` ('search' | 'link'), `business`, `reviewLink`, `linkTrust`, `audienceType` ('smb' default), `isPaigeCustomer` (false default), `isAgencyPartner` (null), `addressVerified`, `idempotencyKey`, `submitting`, `sessionCaptured`.

**On load**: fire `fbq('track','ViewContent', …)` if `fbq` exists; store any `utm_*`, `fbclid`, `gclid`, `ttclid` from the URL in `sessionStorage` under `nfc_lp_attribution`; call `setQty(3)` without firing AddToCart; `GET /api/products` and keep the entry whose `key === 'nfc_card'` (use its `priceCents`).

**Pricing**: `unit = product.priceCents || 1000`; `subtotal = unit * qty`; `discount = min(coupon.discountCents, subtotal)` or 0; `total = subtotal - discount`. Shipping is always shown as Free. Money renders as `$21` for whole dollars, `$21.50` otherwise.

**Coupon**: `GET /api/orders/coupon?code=&productKey=nfc_card&quantity=` returns `{valid, code, discountCents, message}`. Validate the configured code on load and after every quantity change (silently). If invalid, set `coupon = null`, show `couponNotice` with "The 30% code REVIEWS30 isn't active yet, so the total below is the full price. Email reviewstore@merchynt.com and we'll sort it out.", and flip the badge to "Offer pending" in orange. The manual "Apply" button validates whatever the user typed and shows the store's `message` on failure. Guard against out-of-order responses with a request counter.

**Summary render**: `sumQtyLine` = "3 cards at $10 each"; `sumSubtotal`, `sumListLine` (hidden when no discount), `sumNetLine`, `sumTotal`, `sumDiscount` ("-$9"), `sumCode`, `offerBadge` ("30% off applied" computed from the actual discount percentage), `qtySavings` ("You save $9").

**Quantity**: chips set qty and highlight; stepper clamps 1..500; typing in the input updates live. Any user-driven change fires `fbq('track','AddToCart', …)` and calls `captureSession()`.

**Business search**: debounce 450ms; require a city and a name of at least 4 characters; skip if the lowercase `city|name` key is unchanged; abort the previous request. `GET /api/business-search?name=&city=` returns `{available, candidates:[{name,address,placeId,reviewLink,photoUrl,rating,reviewCount,category}]}`. Show up to 6 results with photo (`referrerpolicy="no-referrer"`), name, address, and "★ 5.0 (10 reviews)". Status copy: "Searching Google for "{name}" in {city}..." while loading, "Pick your listing:" on results, "No match yet. Try the city as it appears on Google, or paste your review link below." on empty, and "Search isn't available right now. Paste your Google review link below instead." when `available` is false or the request fails. Selecting a result stores it, sets `reviewLink = candidate.reviewLink`, fills the Selected block, hides the search and link blocks, marks step 2 done, prefills `businessName` if empty, and calls `captureSession()`. "Change" reverses it.

**Review link mode**: classify the pasted link exactly like the store does: hosts `givetings.com` are trusted; `g.page` is trusted only when the path starts with `/r/`, otherwise profile-only; `google.com` and subdomains are trusted when the path contains `writereview`, profile-only when it contains `/maps/place`, starts with `/maps`, or the query contains `cid=`; `maps.app.goo.gl` is profile-only; anything else is unrecognized. Trusted: green "Looks good. That link opens the Google review form." and accept. Profile-only: pink "That link opens your profile, not the review popup." plus the hint to use the "Get more reviews" link, and do not accept. Unrecognized: show the override checkbox; accept only when ticked.

**Phone**: strip non-digits, drop a leading 1 on 11 digits, require 10 digits, send as `+1XXXXXXXXXX`. Format the visible field as `(555) 123-4567` on blur.

**Address**: on `address1` input (3+ chars, 250ms debounce) call `GET /api/address/autocomplete?search=` which returns `{suggestions:[{text, streetLine, secondary, city, state, zipcode}]}`; render up to 6 in the dropdown; picking one fills address1/address2/city/state/postalCode and verifies. Also verify on blur once all four required fields are filled. Verification is `POST /api/address/verify` with `{street, street2, city, state, zipcode}` and returns `{verified, address:{address1,address2,city,state,postalCode}, message}`. On success, write the normalized address back, show green "Address confirmed by USPS." and mark step 3 done. On failure show the store's message in pink plus "Check it, or confirm below to ship as typed." and reveal the override checkbox. Close the dropdown on outside pointerdown.

**Toggles**: audience buttons set `audienceType`, show `partnerWrap` only for agency, reset `isAgencyPartner` to null for smb, and re-capture the session. Paige buttons set `isPaigeCustomer`. Partner buttons set `isAgencyPartner`.

**captureSession**: once per audience choice, `POST /api/session` with `{audienceType}`; ignore failures.

**Validation on pay**: reviewLink present; valid email regex; 10-digit phone; business name; first and last name; complete address and (verified or override ticked); for agency, partner answered. Add `.error` to failing fields. Show the list in `formErrors` under the heading "A couple of things to finish:" and scroll it into view.

**Payload** (must match the store's own checkout exactly):
```js
{
  productKey: 'nfc_card', quantity, idempotencyKey,   // crypto.randomUUID(), reused on retry
  setupMethod: 'manual', email, phone, businessName,
  reviewLink, audienceType, shippingMethod: 'standard',
  shipping: { firstName, lastName, address1, address2 | undefined, city, state (uppercase), postalCode },
  couponCode: coupon ? coupon.code : undefined,
  isPaigeCustomer: boolean,
  isAgencyPartner: audienceType === 'agency' ? boolean : null,
  flyer: undefined
}
```
`POST /api/orders`. Fire `AddPaymentInfo` and `InitiateCheckout` pixel events first. While pending, disable the button and show a spinner with "Opening secure checkout...". On success, `window.location.href = response.checkoutUrl`. On error, clear the idempotency key, re-enable the button, and show the error message plus "If it keeps happening, email reviewstore@merchynt.com and we will place it for you."

**CTA clicks**: every `a[data-cta]` fires `fbq('trackCustom','OrderCTAClick',{placement})` when the pixel exists.

---

## Prompt 13 — QA pass

Compare against `index.html` at 1280px, 1024px, 768px and 375px widths and fix any difference in spacing, type size, color, radius or shadow. Check that:

1. There is no horizontal scroll at 375px and every gutter is at least 20px.
2. The header CTA is hidden below `sm`, the sticky bar is hidden at `lg` and above.
3. Changing quantity updates every number in the summary and the "You save" label, and the coupon re-validates.
4. Typing "Sacramento" and "Roofing Sacramento" returns a list within a few seconds and selecting one shows the green Selected block with the photo.
5. Typing "1600 Pennsylvania Ave SE" shows suggestions; picking one fills the fields and shows "Address confirmed by USPS." with the ZIP+4.
6. Clicking Proceed with an empty form shows the error list and marks fields pink; with a complete form it redirects to a `checkout.stripe.com` URL.
7. The FAQ opens one item at a time, the phone stars animate once, and `prefers-reduced-motion` removes every animation.
8. No emojis, no em dashes, and "Google Business Profile" is never abbreviated to GBP in visible copy.
