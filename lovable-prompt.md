# Lovable Build Prompt — NFC Review Cards Landing Page

Paste these prompts into Lovable in order. Prompt 0 sets up the project and design system. Prompts 1 through 11 each build one section as its own React component. Prompt 12 builds the API client and order state. Prompt 13 wires the order section to it. Prompt 14 adds page-level behavior. Prompt 15 is the QA pass. Each prompt is self-contained, so if a section comes out wrong you can re-run only that prompt.

The reference implementation is `index.html` in this folder. If Lovable drifts, paste the matching section of that file and say "match this markup exactly." Pixel parity with that file is the goal, not an interpretation of it.

Upload everything in `assets/` to the Lovable project first (`hero-tap.jpg`, `salon-counter.jpg`, `handoff.jpg`, `card-stack.jpg`, `nfc-card-product.png`, `merchynt-logo.svg`) into `public/assets/`, or host them and swap the paths.

Sibling file: `replit-prompt.md` is the same build written for a plain static site (and for building inside the existing store's React app). This file is the Lovable version.

---

## Prompt 0 — Project setup and design system

```
Create a single-page marketing landing page called "NFC Review Cards". Use React + Vite + Tailwind CSS + TypeScript. No router (one route at "/"), no component library, no icons package (inline SVG only), no state library (React hooks only). Do not add a backend, database, or auth: the page talks to an existing external API described later.

DESIGN TOKENS (theme.extend.colors in tailwind.config):
- navy: #0f007d (primary brand background, headlines on white)
- navydeep: #08004a (footer)
- pink: #dd0cf7 (PRIMARY CTA color and accent; also the color for validation errors and warnings. Never use red anywhere.)
- blue: #0063fd (text links inside forms)
- purple: #8b00cc (unused, keep defined)
- green: #05c168 (success, checkmarks, confirmed states)
- orange: #ff9e2c (star ratings, "offer pending" warnings)
- dark: #1c1f23 (body text)
- muted: #8E8B84 (secondary text)
- cloud: #f6f5ff (alternate section background, input tiles)
- line: #e6e4f2 (borders, dividers)

TYPOGRAPHY: load Google Font "DM Sans" (optical size 9..40, weights 300,400,500,600,700,800, italic 400) in index.html and set it as the default sans font. Body text 16px. Headlines extrabold with tight tracking. H1 is 2.25rem on mobile, 3rem from sm, 3.4rem from lg, line-height 1.05. H2 is 1.875rem on mobile, 2.25rem from sm, line-height tight.

SHADOWS (theme.extend.boxShadow):
- card: 0 10px 30px -10px rgba(15,0,125,0.18)
- float: 0 24px 60px -20px rgba(8,0,74,0.55)
- panel: 0 30px 80px -30px rgba(15,0,125,0.25)

GLOBAL CSS (index.css), add exactly:
- html { scroll-behavior: smooth; }
- .hero-grid: background-image radial-gradient(rgba(255,255,255,.08) 1px, transparent 1px), background-size 22px 22px.
- .fade-up: opacity 0, translateY(22px), transition opacity .6s ease-out and transform .6s ease-out. .fade-up.visible: opacity 1, transform none. (A shared hook adds "visible" once via IntersectionObserver, threshold 0.12.)
- .hero-animate: opacity 0, translateY(16px), animation heroUp .7s ease-out forwards, where heroUp ends at opacity 1 and transform none. Delay utilities .d1 .05s, .d2 .2s, .d3 .35s, .d4 .5s, .d5 .7s.
- .cta-btn: transition transform .15s, box-shadow .15s, background-color .15s, opacity .15s; hover translateY(-1px) scale(1.02); active scale(.99); disabled opacity .55, cursor not-allowed, no transform.
- .cta-pulse: keyframe ctaPulse from box-shadow 0 0 0 0 rgba(221,12,247,.45) to 0 0 0 10px rgba(221,12,247,0), 2.4s ease-in-out, 1.2s delay, 3 iterations.
- .float-a: floats up 8px and back over 6s ease-in-out infinite. .float-b: floats down 7px and back over 7s.
- .strike: text-decoration line-through, thickness 2px, color #dd0cf7.
- .field: width 100%, border 1.5px solid #e6e4f2, radius 12px, padding 12px 14px, font-size 15px, white background, color #1c1f23, no outline; focus border #0f007d with box-shadow 0 0 0 4px rgba(15,0,125,.08); .field.error border #dd0cf7; placeholder #a7a4b3; disabled background #f6f5ff color #8E8B84.
- .label: block, 13px, font-weight 600, color #1c1f23, margin-bottom 6px.
- .qty-chip: border 1.5px solid #e6e4f2, radius 14px, padding 12px 10px, centered, white; hover border navy; .active: navy background, white text, border navy, shadow 0 10px 24px -12px rgba(15,0,125,.5); the inner .qty-sub becomes rgba(255,255,255,.75) when active.
- .choice: border 1.5px solid #e6e4f2, radius 12px, padding 10px 14px, 14px semibold, centered; hover border navy; .active border navy, background rgba(15,0,125,.05), color navy.
- .biz-result: flex, gap 12px, align center, padding 10px 12px, radius 12px, transparent 1.5px border; hover background #f6f5ff and border #e6e4f2.
- .suggest: absolute, left 0 right 0, top calc(100% + 6px), white, border 1.5px solid #e6e4f2, radius 12px, shadow 0 16px 40px -16px rgba(15,0,125,.3), z-index 20, overflow hidden; child buttons full width, left aligned, padding 10px 14px, 14px, hover background #f6f5ff.
- .step-num: 34px circle, navy background, white 700 14px, inline-flex centered, flex-shrink 0; .step-num.done green background.
- .spinner: 18px ring, 2px border rgba(255,255,255,.4) with white top, spinning .7s linear infinite; .spinner.dark uses rgba(15,0,125,.2) and navy top.
- .phone: width 250px, radius 38px, background #0b0b12, padding 10px, box-shadow 0 30px 70px -25px rgba(8,0,74,.6) plus inset 0 0 0 2px #2a2a38. .phone-screen: radius 30px, white, overflow hidden, min-height 500px. .star-fill: transition fill .25s.
- .faq-answer: max-height 0, overflow hidden, transition max-height .35s ease; .faq-answer.open max-height 420px. .faq-chevron: transition transform .3s; .rotated rotate(180deg).
- .sticky-cta: translateY(110%), transition transform .3s; .show translateY(0).
- @media (prefers-reduced-motion: reduce): every class above renders its final state with no animation or transition.

PAGE-LEVEL RULES:
- Max content width 72rem (max-w-6xl) with 1.25rem horizontal padding on every section. Section vertical padding py-16 sm:py-24 unless a prompt says otherwise.
- No site navigation anywhere. The only links are in-page CTAs to "#order", Google's help article about review links (new tab), the mailto for reviewstore@merchynt.com, and Privacy/Terms in the footer.
- Add <meta name="robots" content="noindex, nofollow">.
- Title: "Get More Google Reviews With One Tap | NFC Review Cards from $7 | Merchynt".
- Meta description: "A pre-programmed NFC card for your counter. Customers tap their phone and your Google review page opens. No app, no monthly fee, ships unbranded. 30% off and free shipping for a limited time."
- Open Graph title "Get More Google Reviews With One Tap", description "Pre-programmed NFC review cards. $7 a card with 30% off, free shipping in the USA, ready to use the day they arrive.", type website, image /assets/hero-tap.jpg.
- Leave three commented placeholders in the head for the Meta Pixel base code, Microsoft Clarity, and Vercel Analytics.
- Never use emojis. Never use an em dash. Never write "no credit card". Body text is dark on light backgrounds and white with reduced opacity on navy.
- Every CTA anchor gets className "cta-btn" and a data-cta attribute (values given per section). CTAs are pill buttons: pink background, white bold text, rounded-full, arrow icon on the right where specified.

COMPONENT PLAN (create these files, one component each, and compose them in App.tsx in this order): Header, Hero, StatsBar, Problem, HowItWorks (contains PhoneMockup), WhatYouGet, WhoItsFor, OrderSection (contains QuantityStep, BusinessStep, ShippingStep, OrderSummary), Faq, FinalCta, Footer, StickyCta. Shared: hooks/useFadeUp.ts, lib/api.ts, lib/order.ts (state and pricing), lib/reviewLink.ts.
```

## Prompt 1 — Header

```
Build the HEADER component. It is absolutely positioned over the hero (absolute, top 0, inset-x 0, z-index 30), transparent.

Inside a max-w-6xl mx-auto px-5 py-5 flex items-center justify-between row:
- Left: the Merchynt logo from /assets/merchynt-logo.svg, height 1.75rem on mobile and 2rem from sm, rendered white (className "brightness-0 invert"), wrapped in an anchor to "#top" with aria-label "Merchynt", className "hero-animate d1".
- Right, hidden below sm: an anchor to "#order" with data-cta="header", className "cta-btn hero-animate d2 hidden sm:inline-flex items-center gap-2 rounded-full bg-pink text-white text-sm font-semibold px-5 py-2.5 shadow-card", text "Get 30% Off".
```

## Prompt 2 — Hero

```
Build the HERO component. Section id="top", className "relative bg-navy text-white overflow-hidden".

Background layers (all absolute, pointer-events none): a full-bleed div with className "hero-grid opacity-70"; a 560px circle "rounded-full bg-blue/30 blur-3xl" at -top-40 -right-32; a 520px circle "rounded-full bg-pink/25 blur-3xl" at -bottom-48 -left-32.

Grid container: "relative max-w-6xl mx-auto px-5 pt-28 pb-16 sm:pt-32 sm:pb-24 grid lg:grid-cols-12 gap-10 lg:gap-8 items-center".

LEFT COLUMN (lg:col-span-6):
1. Eyebrow pill, className "hero-animate d1 inline-flex items-center gap-2 rounded-full bg-white/10 border border-white/15 px-3.5 py-1.5 text-xs sm:text-sm font-semibold tracking-wide": a "w-2 h-2 rounded-full bg-green" dot, then "Limited time: 30% off + free shipping".
2. H1, className "hero-animate d2 mt-5 text-4xl sm:text-5xl lg:text-[3.4rem] font-extrabold leading-[1.05] tracking-tight": "Get More Google Reviews With " then a span className "text-pink" containing "One Tap".
3. Paragraph, className "hero-animate d3 mt-5 text-lg sm:text-xl text-white/80 leading-relaxed max-w-xl": "A pre-programmed NFC card for your counter. Customers tap their phone, your Google review page opens, and they write the review while the good experience is still fresh. No app, no monthly fee, no Merchynt branding."
4. CTA row, className "hero-animate d4 mt-7 flex flex-col sm:flex-row sm:items-center gap-4":
   - Anchor href="#order" data-cta="hero", className "cta-btn cta-pulse inline-flex items-center justify-center gap-2 rounded-full bg-pink text-white text-base sm:text-lg font-bold px-8 py-4 shadow-float": text "Get My Cards for 30% Off" then a 20px inline SVG arrow (stroke currentColor, width 2.4, round caps; path "M5 12h14M13 6l6 6-6 6").
   - A div className "text-sm text-white/75 leading-snug": span className "strike text-white/50" "$10", a space, span className "text-white font-bold text-base" "$7 per card", a <br>, then "Free shipping in the USA".
5. Trust row, className "hero-animate d5 mt-8 flex flex-wrap items-center gap-x-6 gap-y-3 text-sm text-white/75". Three spans, each "inline-flex items-center gap-2" with a 16px green check SVG (stroke currentColor, width 3, path "M20 6L9 17l-5-5") and the text: "Arrives programmed and ready to use", "Works with iPhone and Android", "Mailed next business day".

RIGHT COLUMN (className "lg:col-span-6 relative hero-animate d3"):
- Image frame div "relative rounded-3xl overflow-hidden shadow-float border border-white/10 bg-white/5" containing <img src="/assets/hero-tap.jpg" alt="A customer taps their phone on a Google review NFC card at a cafe counter" className="w-full h-[320px] sm:h-[440px] object-cover" width={1536} height={1024}>.
- Floating price card, className "float-a absolute -bottom-5 -left-3 sm:-left-8 bg-white text-dark rounded-2xl shadow-float px-5 py-4 w-[210px]": a label "Per card, this week" in "text-[11px] font-semibold uppercase tracking-wider text-muted"; then a "mt-1 flex items-baseline gap-2" row with "$7" in "text-3xl font-extrabold text-navy" and "$10" in "strike text-muted font-semibold"; then "mt-1 text-xs font-semibold text-green inline-flex items-center gap-1" with a 14px check icon and "Free shipping included".
- Floating review notification, className "float-b absolute -top-4 -right-2 sm:-right-6 bg-white text-dark rounded-2xl shadow-float px-4 py-3 w-[230px]": a "flex items-center gap-3" row with a 36px "rounded-full bg-cloud flex items-center justify-center flex-shrink-0" circle holding the four-color Google "G" SVG (20px; blue #4285F4, green #34A853, yellow #FBBC05, red #EA4335), then a min-w-0 block with "New 5-star review" in "text-[13px] font-bold leading-tight" over a "mt-0.5 flex items-center gap-1 text-orange" row of five 14px filled star SVGs (path "M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01z") followed by "just now" in "text-[11px] text-muted ml-1".
```

## Prompt 3 — Stats bar

```
Build the STATS BAR component. Section className "bg-white border-b border-line". Container "max-w-6xl mx-auto px-5 py-10 sm:py-12".

A "grid sm:grid-cols-3 gap-6 sm:gap-8". Each cell has className "fade-up text-center sm:text-left"; cells 2 and 3 get inline style transitionDelay ".1s" and ".2s". Each cell: the number in "text-4xl sm:text-5xl font-extrabold text-navy tracking-tight" and a line in "mt-2 text-sm sm:text-base text-dark/80 leading-snug".

1. "97%" / "of consumers read reviews before choosing a local business"
2. "83%" / "of people who were asked to leave a review went on to leave one"
3. "47%" / "won't use a business that has fewer than 20 reviews"

Under the grid: <p className="fade-up mt-6 text-xs text-muted text-center sm:text-left">Source: BrightLocal Local Consumer Review Survey 2026, 1,002 US consumers.</p>
```

## Prompt 4 — Problem

```
Build the PROBLEM component. Section className "bg-cloud". Container "max-w-6xl mx-auto px-5 py-16 sm:py-24 grid lg:grid-cols-12 gap-10 items-center".

LEFT (className "lg:col-span-6 fade-up"):
- Eyebrow div "text-xs font-bold uppercase tracking-[0.18em] text-pink": "The problem".
- H2 "mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight": "Your customers would leave a review. Nobody asks them at the right moment."
- P "mt-5 text-base sm:text-lg text-dark/80 leading-relaxed": Most owners ask the hard way. A text the next day. An email that lands in Promotions. A "review us" sign that asks people to type a URL. By then the moment has passed. The customer who was happy at the counter is busy again, and the review never gets written.
- P "mt-4 text-base sm:text-lg text-dark/80 leading-relaxed": The card fixes the timing. It sits where customers already have their phone out, and it turns "would you mind leaving us a review?" into a two-second tap.
- Div "mt-7 rounded-2xl overflow-hidden shadow-card hidden sm:block" containing <img src="/assets/salon-counter.jpg" alt="A Google review NFC card standing on a salon front desk" className="w-full h-56 lg:h-64 object-cover" width={1168} height={880} loading="lazy">.

RIGHT (className "lg:col-span-6 grid gap-4"): three cards, each "fade-up rounded-2xl p-5 shadow-card flex gap-4 items-start". Each has a 40px icon tile "w-10 h-10 rounded-xl flex items-center justify-center flex-shrink-0" with a 20px line-icon SVG (stroke currentColor, width 2, round caps), then a block with a title div "font-bold" and a paragraph "mt-1 text-sm".

1. White card (bg-white), icon tile "bg-cloud text-navy", chat-bubble icon (path "M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"). Title "The next-day text". Body (text-dark/70): "Buried under forty other notifications. Opened, maybe. Acted on, rarely."
2. White card, transitionDelay .1s, icon: a screen on a stand (rect x3 y4 w18 h14 rx2 plus path "M8 21h8M12 18v3"). Title 'The "review us" sign'. Body: "Reading a web address, typing it, finding your listing, finding the button. Every step loses people."
3. Navy card "bg-navy text-white", transitionDelay .2s, icon tile "bg-white/10", contactless-waves icon (paths "M8.5 14.5a5 5 0 0 1 0-5M5.6 17.4a9 9 0 0 1 0-10.8M15.5 9.5a5 5 0 0 1 0 5M18.4 6.6a9 9 0 0 1 0 10.8" and a circle r1.5 at 12,12). Title "The tap". Body (text-white/75): "Phone is already out. Your review form opens on its own. Stars, a sentence, post. Done before the receipt prints."
```

## Prompt 5 — How it works (with phone mockup)

```
Build the HOW IT WORKS component and a PhoneMockup child. Section white. Container "max-w-6xl mx-auto px-5 py-16 sm:py-24 grid lg:grid-cols-12 gap-12 items-center".

LEFT (className "lg:col-span-5 order-2 lg:order-1 flex justify-center fade-up"): <PhoneMockup />.

PhoneMockup: div className "phone" containing div className "phone-screen text-[13px] text-dark" with, in order:
- Status bar: "flex items-center justify-between px-5 pt-4 text-[11px] font-semibold" with "9:41" left and a tiny battery block right ("w-3 h-2 bg-dark rounded-[2px] inline-block").
- Row "flex items-center gap-3 px-4 pt-4": a 24px X icon (stroke width 2, paths "M18 6L6 18M6 6l12 12") and "Rate and review" in "font-semibold text-[14px] leading-tight".
- Row "px-4 pt-4 flex items-center gap-3": a "w-10 h-10 rounded-full bg-gradient-to-br from-blue to-pink text-white font-bold flex items-center justify-center" circle with the letter "Y", then "Your business name" in "font-semibold text-[13px]" over "Posting publicly" in "text-[11px] text-muted".
- Star row id="phoneStars", className "px-4 pt-5 flex items-center gap-2", aria-hidden: five 32px star SVGs with className "star-fill" and fill "#d9d9e3" (same star path as the hero).
- Text box id="phoneText", className "mx-4 mt-5 rounded-xl border border-line p-3 min-h-[92px] text-[12px] text-dark/80", initial content a span "text-muted" reading "Share details of your own experience at this place".
- Button row "px-4 mt-4 flex gap-2": "Add photos" in "flex-1 rounded-full border border-line py-2 text-center text-[12px] font-semibold text-muted" and "Post" in "flex-1 rounded-full bg-blue py-2 text-center text-[12px] font-semibold text-white".
- Caption "px-4 pt-6 pb-5 text-[11px] text-muted leading-snug": "Opened by a tap on your card. No app, no search, no typing a link."

PhoneMockup behavior (useEffect + IntersectionObserver, threshold 0.5, runs once): when the phone enters view, set each star's fill to "#ff9e2c" in sequence starting at 500ms with 180ms between stars; after 1600ms clear the text box and type "Fast, friendly and fair on price. Will be back." one character every 32ms. Under prefers-reduced-motion render the final state immediately.

RIGHT (className "lg:col-span-7 order-1 lg:order-2"):
- Eyebrow "fade-up text-xs font-bold uppercase tracking-[0.18em] text-pink": "How it works".
- H2 "fade-up mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight": "Set up once. Then every tap is a review request."
- Steps container "mt-8 space-y-6". Each step is "fade-up flex gap-4" with a div className "step-num" showing the number and a block with title "text-lg font-bold" and paragraph "mt-1 text-dark/75 leading-relaxed":
  1. "Tell us which Google listing" / "Search your business by city and name below, or paste your Google review link. We program every card to open your review page before it ships. You never touch the settings."
  2. (transitionDelay .1s) "Put the card where customers pay" / "The register, the front desk, the check presenter, the service counter, the truck. Anywhere a phone is already out and the experience is still fresh."
  3. (transitionDelay .2s) "Customers tap, rate, post" / "Their phone opens your Google review form on its own. They pick the stars, write a sentence, and hit Post. No app to download, nothing to type, nothing to find."
- CTA anchor href="#order" data-cta="how", className "fade-up cta-btn mt-9 inline-flex items-center gap-2 rounded-full bg-pink text-white font-bold px-7 py-3.5 shadow-card": "Order Cards for 30% Off" plus the 20px arrow icon.
```

## Prompt 6 — What you get

```
Build the WHAT YOU GET component. Section className "bg-cloud". Container "max-w-6xl mx-auto px-5 py-16 sm:py-24 grid lg:grid-cols-12 gap-10 items-center".

LEFT (className "lg:col-span-6 fade-up"): div "relative rounded-3xl overflow-hidden shadow-card bg-white" containing <img src="/assets/card-stack.jpg" alt="A stack of Google review NFC cards" className="w-full aspect-square object-cover" width={1024} height={1024}> and a badge div "absolute top-4 left-4 bg-navy text-white text-xs font-bold px-3 py-1.5 rounded-full" reading "Ships unbranded".

RIGHT (className "lg:col-span-6"):
- Eyebrow "fade-up text-xs font-bold uppercase tracking-[0.18em] text-pink": "What you get".
- H2 "fade-up mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight": "A premium card that does the asking for you".
- UL "mt-7 space-y-3.5". Each LI "fade-up flex gap-3 items-start" with a 20px green check SVG (className "w-5 h-5 text-green flex-shrink-0 mt-0.5", stroke width 3) and a span where the first sentence is inside <strong>:
  1. <strong>Pre-programmed to your Google review page.</strong> It arrives ready. Set it on the counter and it works.
  2. <strong>Works with iPhones from 2018 on and any Android with NFC.</strong> The same tap your customers already use to pay.
  3. <strong>No app, no account, no monthly fee.</strong> One price, and the card keeps working.
  4. <strong>No Merchynt branding anywhere on it.</strong> Your customers see Google, five stars and "Tap to review us."
  5. <strong>Durable card stock built for a counter.</strong> Survives coffee, keys and a thousand taps.
  6. <strong>Mailed USPS the next business day.</strong> Free shipping anywhere in the USA.
- Price box div "fade-up mt-8 bg-white rounded-2xl p-5 shadow-card border border-line" with an inner "flex flex-wrap items-end justify-between gap-4": left block has "This week's price" in "text-xs font-semibold uppercase tracking-wider text-muted" and a "mt-1 flex items-baseline gap-2" row with "$7" (text-4xl font-extrabold text-navy), "$10" (text-muted font-semibold strike) and "per card" (text-sm text-muted); right block "text-sm text-dark/70 max-w-[260px]": "Similar tap-to-review cards from other sellers run $16 or more each, and several charge a monthly fee on top."
```

## Prompt 7 — Who it's for

```
Build the WHO IT'S FOR component. Section white. Container "max-w-6xl mx-auto px-5 py-16 sm:py-24".

Intro block "max-w-2xl": eyebrow "fade-up text-xs font-bold uppercase tracking-[0.18em] text-pink" "Who it's for"; H2 "fade-up mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight" "Built for businesses that see their customers face to face"; P "fade-up mt-4 text-dark/75 text-lg" "If there's a moment when a happy customer is standing in front of you with a phone in hand, the card belongs in that moment."

Then a "mt-10 grid md:grid-cols-12 gap-6 items-stretch":
- Div "md:col-span-5 fade-up rounded-3xl overflow-hidden shadow-card min-h-[280px]" with <img src="/assets/handoff.jpg" alt="A technician hands a Google review card to a homeowner" className="w-full h-full object-cover" width={1536} height={1024}>.
- Div "md:col-span-7 grid sm:grid-cols-2 gap-4" with six cards, each "fade-up bg-cloud rounded-2xl p-5" (transitionDelay stepping .05s per card from 0 to .25s) containing a title div "font-bold text-navy" and a line "mt-1 text-sm text-dark/70":
  1. Restaurants and cafes / By the register, or tucked into the check presenter.
  2. Salons and barbers / At the front desk while they're settling up and still admiring the cut.
  3. Auto shops / On the service counter when the keys come back.
  4. Dentists and clinics / At the checkout window, where every patient stops on the way out.
  5. Contractors and home services / Hand it over at the door when the job is done and the customer is impressed.
  6. Retail and boutiques / Right next to the card reader, where the phone already is.
```

## Prompt 8 — Order section (markup only)

```
Build the ORDER SECTION component with four children (QuantityStep, BusinessStep, ShippingStep, OrderSummary). This prompt is markup and styling only; all props, state and behavior come in Prompts 12 and 13, so for now render static placeholders and accept the props described there without using them.

Section id="order", className "bg-cloud border-t border-line scroll-mt-4". Container "max-w-6xl mx-auto px-5 py-16 sm:py-24".

Intro "max-w-2xl": eyebrow (pink, same style) "Order"; H2 (same H2 style) "Order your cards"; P "fade-up mt-3 text-dark/75 text-lg" "30% off is applied for you. Free shipping in the USA. Payment happens on a secure Stripe checkout page."

Grid "mt-10 grid lg:grid-cols-12 gap-8 items-start".

LEFT column "lg:col-span-7 space-y-6". Each step is a card "fade-up bg-white rounded-3xl shadow-card border border-line p-6 sm:p-8" that opens with a row "flex items-center gap-3" containing a div className "step-num" (number 1, 2 or 3; add "done" when the step is complete) and an h3 "text-xl font-bold", followed by a helper p "mt-2 text-sm text-dark/70 ml-[46px]".

QuantityStep, h3 "How many cards?", helper "One for the register is the minimum. Add one for each person who talks to customers.":
- Div "mt-5 grid grid-cols-4 gap-2 sm:gap-3" with four buttons className "qty-chip" (data-qty 1, 3, 5, 10; add "active" for the current quantity), each with a div "text-xl font-extrabold" showing the number and a div "qty-sub text-[11px] text-muted font-medium" showing "card" for 1 and "cards" otherwise.
- Div "mt-4 flex items-center gap-3": span "text-sm text-dark/70" "Or pick a number:"; a stepper div "inline-flex items-center border border-line rounded-full overflow-hidden" with a minus button "px-3.5 py-2 text-navy font-bold hover:bg-cloud" (aria-label "Fewer cards"), a number input "w-14 text-center font-bold outline-none py-2" (min 1, max 500, aria-label "Number of cards") and a plus button (aria-label "More cards"); then a span "text-sm font-semibold text-green" for the savings text.

BusinessStep, h3 "Which Google listing should the card open?", helper "We program the cards to your Google review page before they ship." Three blocks, only one visible at a time:
- SEARCH block ("mt-5"): a "grid sm:grid-cols-2 gap-3" with two labeled inputs (label className "label", input className "field", autoComplete off): "City your business is in" placeholder "e.g. Austin, TX", and "Business name" placeholder "e.g. Joe's Plumbing". Then a status div "mt-3 text-sm min-h-[20px]" (text-muted normally, text-pink for errors). Then a results list "mt-2 space-y-1" where each result is a button className "biz-result w-full" containing a 48px photo ("w-12 h-12 rounded-lg object-cover bg-cloud flex-shrink-0", referrerPolicy no-referrer) or a "w-12 h-12 rounded-lg bg-cloud" placeholder, then a "min-w-0 text-left" block with the name ("font-bold truncate"), the address or category ("text-xs text-dark/70 truncate"), and when a rating exists a line "text-xs text-muted mt-0.5" with an orange "★" then "5.0 (10 reviews)". Then a text button "mt-4 text-sm font-semibold text-blue hover:underline": "Service-area business or can't find it? Paste your Google review link instead".
- LINK block ("mt-5"): label "Your Google review link", url input className "field" placeholder "https://g.page/r/... or https://search.google.com/local/writereview?placeid=...", a status div "mt-2 text-sm min-h-[20px]", a checkbox row "mt-2 items-center gap-2 text-sm text-dark/80 cursor-pointer" (shown as flex only when the link is unrecognized) with a checkbox "w-4 h-4 accent-[#0f007d]" and the text "Use this link anyway. I've checked that it opens my Google review form.", then a row "mt-3 flex flex-wrap items-center gap-x-4 gap-y-1 text-sm" with an anchor "font-semibold text-blue hover:underline" to https://support.google.com/business/answer/3474122 (target _blank, rel noreferrer) reading "How do I find my Google review link?" and a text button "Search for my business instead".
- SELECTED block ("mt-5"): a div "flex items-center gap-4 rounded-2xl border-2 border-green/60 bg-green/5 p-4" with a 56px photo ("w-14 h-14 rounded-xl object-cover bg-cloud") or a "w-14 h-14 rounded-xl bg-cloud flex items-center justify-center text-navy" fallback holding a 24px building icon (paths "M3 21h18M5 21V7l7-4 7 4v14M9 21v-6h6v6"); a "min-w-0 flex-1" block with "Selected" in "text-xs font-bold uppercase tracking-wider text-green", the name in "font-bold truncate", and meta in "text-sm text-dark/70 truncate"; and a "Change" text button "text-sm font-semibold text-blue hover:underline flex-shrink-0".

ShippingStep, h3 "Where should we ship them?", helper "We ship within the US only. We'll email your order confirmation and call only if something needs checking.":
- Grid "mt-5 grid sm:grid-cols-2 gap-3" of labeled fields (label className "label", input className "field"): email (sm:col-span-2, "Email address", placeholder you@business.com, autoComplete email), phone ("Cell phone number", placeholder (555) 123-4567, type tel), businessName ("Business name", placeholder Acme Co.), firstName ("First name"), lastName ("Last name"), address1 (sm:col-span-2 and relative, "Street address", placeholder "Start typing your address", autoComplete off, with a div className "suggest" dropdown rendered under it when there are suggestions), address2 (sm:col-span-2, label "Apt, suite, unit" followed by a span "font-normal text-muted" "(optional)"), city ("City"), and a nested "grid grid-cols-2 gap-3" with state ("State", placeholder TX, maxLength 2) and postalCode (label "ZIP", placeholder 78701, inputMode numeric).
- Address status div "mt-3 text-sm min-h-[20px]" and a checkbox row (same style as the link override) reading "Ship to this address exactly as I typed it.", shown only when verification failed.
- A block "mt-6 pt-6 border-t border-line grid sm:grid-cols-2 gap-5":
  - div with a "label" "Who are you buying for?" and a "grid grid-cols-2 gap-2" of two buttons className "choice": "My own business" and "A client".
  - div with a "label" "Are you a Paige customer?" and two "choice" buttons "Yes" and "No".
  - a "sm:col-span-2" div, shown only when the buyer picked "A client", with a "label" "Are you in the Merchynt Agency Partner program?" and a "grid grid-cols-2 gap-2 max-w-xs" of "choice" buttons "Yes" and "No".

RIGHT column "lg:col-span-5 lg:sticky lg:top-6": OrderSummary, a panel "fade-up bg-navy text-white rounded-3xl shadow-panel p-6 sm:p-8":
- Header "flex items-center justify-between": h3 "text-lg font-bold" "Your order" and a badge span "text-[11px] font-bold uppercase tracking-wider px-2.5 py-1 rounded-full" that is "bg-green/20 text-green" when the discount is live ("30% off applied") and "bg-orange/20 text-orange" otherwise ("Offer pending").
- Product row "mt-5 flex items-center gap-4": <img src="/assets/nfc-card-product.png" alt="Google review NFC card" className="w-16 h-16 rounded-xl object-cover bg-white">, then a flex-1 block with "Google Review NFC Card" (font-bold) over the quantity line (text-sm text-white/70, e.g. "3 cards at $10 each"), then a right-aligned block with the list total in "text-sm text-white/50 strike" (hidden when there is no discount) over the net total in "font-bold".
- Totals "mt-5 space-y-2 text-sm border-t border-white/10 pt-4": rows "flex justify-between" for Subtotal (label text-white/70), "30% off (REVIEWS30)" in text-green with the discount as "-$9", "Shipping (USPS, USA)" with "Free" in text-green font-semibold, and a Total row "text-lg font-extrabold pt-3 border-t border-white/10".
- A notice div "mt-3 text-xs rounded-xl bg-orange/15 text-orange px-3 py-2 leading-snug", hidden unless there is a coupon message.
- A <details className="mt-3 text-sm"> with <summary className="cursor-pointer text-white/60 hover:text-white">Have a different promo code?</summary> and inside a "mt-2 flex gap-2" row with an input className "field !bg-white/10 !border-white/20 !text-white placeholder:!text-white/40" placeholder "Code" and a button "rounded-xl bg-white/15 hover:bg-white/25 px-4 font-semibold" "Apply".
- An error box "mt-4 text-sm rounded-xl bg-pink/20 text-white px-4 py-3 leading-snug", hidden unless there are validation errors; when shown it has a "font-bold mb-1" line "A couple of things to finish:" and a "list-disc pl-5 space-y-0.5" list.
- The pay button: <button className="cta-btn mt-5 w-full inline-flex items-center justify-center gap-2 rounded-full bg-pink text-white text-lg font-bold px-6 py-4 shadow-float"> with a 20px lock icon (rect x3 y11 w18 h11 rx2 plus path "M7 11V7a5 5 0 0 1 10 0v4") and the text "Proceed to Secure Checkout".
- Under it "mt-3 flex flex-wrap items-center justify-center gap-x-4 gap-y-1 text-[12px] text-white/60": three spans "Secure payment by Stripe", "Card programmed before shipping", "Unbranded".
- Footnote p "mt-4 text-[12px] text-white/50 leading-snug": "We mail every order USPS ground on the next business day. Delivery can take up to 10 business days depending on where you are. To keep the price this low there's no tracking number. Questions: " followed by an underlined mailto link reviewstore@merchynt.com.
```

## Prompt 9 — FAQ

```
Build the FAQ component. Section white. Container "max-w-3xl mx-auto px-5 py-16 sm:py-24".

Eyebrow "fade-up text-xs font-bold uppercase tracking-[0.18em] text-pink": "Questions". H2 "fade-up mt-3 text-3xl sm:text-4xl font-extrabold text-navy leading-tight tracking-tight": "What owners ask before they order".

List "mt-8 divide-y divide-line". Each item is a div "faq-item py-5" with a button "faq-q w-full flex items-center justify-between text-left gap-4" (the question in a span "font-bold text-lg" and a 20px chevron SVG className "faq-chevron w-5 h-5 text-navy flex-shrink-0", stroke width 2.5, path "M6 9l6 6 6-6") and a div "faq-answer" (add "open" when expanded; the chevron gets "rotated") containing a p "pt-3 text-dark/75 leading-relaxed". Opening one item closes any other. Keep the open state in React (useState of the open index).

1. Will it work with my customers' phones? / Yes for almost everyone. iPhones from 2018 onward (XS, XR and newer) read the card automatically when it's held near the top of the phone. Android phones with NFC do the same. It's the same technology people use to tap and pay, so most customers already know the motion.
2. Do I need an app or a subscription? / No. You pay once for the cards. There's no account to create, no app for you or your customers, and no monthly fee. The card carries your review link on its own and keeps working.
3. Is asking for reviews this way allowed by Google? / Yes. Google encourages businesses to ask customers for reviews and even publishes a shareable review link for exactly this purpose. What Google prohibits is paying or rewarding people for reviews, or only asking the customers you know are happy. The card simply makes the ask easy for everyone.
4. Does the card say Merchynt anywhere? / No. The card shows the Google logo, five stars, a tap symbol and "Tap to review us." Nothing about Merchynt appears on the card or in what your customers see on their phone.
5. How do you know which Google listing to use? / You pick it in step 2 above. Search by city and business name and choose your listing from the results, or paste your Google review link if you're a service-area business without a public address. We program every card to that exact link before it ships.
6. How long until it arrives? / We mail every order USPS ground on the next business day. Most arrive within a week, and it can take up to 10 business days depending on where you are. Shipping is free. To keep the card at this price there's no tracking number.
7. I have more than one location. / Each order is programmed to one Google listing, so place one order per location. The discount applies to every order while the offer is running.
8. What if a customer doesn't know how to tap? / Say "hold your phone against the card." That's the whole instruction, and it's the same motion as tapping to pay. If you'd rather offer a QR code as well, we also print a QR review flyer. Email reviewstore@merchynt.com and we'll point you to it. (render the email as a mailto anchor with className "text-blue font-semibold")
```

## Prompt 10 — Final CTA and footer

```
Build the FINAL CTA and FOOTER components.

FINAL CTA: section "relative bg-navy text-white overflow-hidden" with a full-bleed "hero-grid opacity-60" layer and a 420px "rounded-full bg-pink/25 blur-3xl" circle at -top-32 right-0. Container "relative max-w-4xl mx-auto px-5 py-20 sm:py-28 text-center":
- H2 "fade-up text-3xl sm:text-5xl font-extrabold leading-tight tracking-tight": "Stop hoping for reviews." <br> "Ask with a tap."
- P "fade-up mt-5 text-lg text-white/80 max-w-2xl mx-auto": "$7 a card while the offer lasts. Free shipping. Programmed to your Google listing and ready to use the day it lands in your mailbox."
- Anchor href="#order" data-cta="final", className "fade-up cta-btn mt-8 inline-flex items-center gap-2 rounded-full bg-pink text-white text-lg font-bold px-9 py-4 shadow-float": "Get My Cards for 30% Off" plus the arrow icon.
- Div "fade-up mt-8 flex flex-wrap justify-center gap-x-6 gap-y-2 text-sm text-white/70" with four spans: "No app", "No monthly fee", "No Merchynt branding", "Ships next business day".

FOOTER: footer "bg-navydeep text-white/60". Container "max-w-6xl mx-auto px-5 py-8 flex flex-col sm:flex-row items-center justify-between gap-4 text-sm". Left "flex items-center gap-3": the logo (white via brightness-0 invert, h-5, opacity-70) and "© {new Date().getFullYear()} Merchynt. All rights reserved." Right "flex items-center gap-5": anchors "hover:text-white" for "reviewstore@merchynt.com" (mailto), "Privacy" (https://www.merchynt.com/privacy-policy, new tab) and "Terms" (https://www.merchynt.com/terms-of-service, new tab).
```

## Prompt 11 — Sticky mobile CTA

```
Build the STICKY CTA component: a fixed bar className "sticky-cta fixed bottom-0 inset-x-0 z-40 lg:hidden bg-white/95 backdrop-blur border-t border-line px-4 py-3 flex items-center justify-between gap-3". Left "leading-tight": "NFC review cards" in "text-xs text-muted" over a "font-extrabold text-navy" line containing "$10" in "strike text-muted font-semibold text-sm mr-1" and "$7 per card". Right: anchor href="#order" data-cta="sticky", className "cta-btn inline-flex items-center gap-2 rounded-full bg-pink text-white font-bold px-5 py-3 shadow-card text-sm", "Get 30% Off".

Behavior: an IntersectionObserver (threshold 0.05) watches the hero section (#top) and the order section (#order). Add the "show" class only when the hero is NOT intersecting and the order section is NOT intersecting. Remove it otherwise.
```

## Prompt 12 — API client, pricing and order state

```
Create lib/api.ts, lib/reviewLink.ts and lib/order.ts. These talk to an existing external backend; do not create any server code.

lib/api.ts:
- export const CONFIG = { apiBase: 'https://reviewstore.merchynt.com', productKey: 'nfc_card', listPriceCents: 1000, couponCode: 'REVIEWS30', offerPercent: 30, defaultQty: 3, minNameChars: 4, searchDebounceMs: 450, addrDebounceMs: 250 }.
- A generic request<T>(path, init?) that calls fetch(CONFIG.apiBase + path, { credentials: 'include', ...init, headers: { Accept: 'application/json', ...(body ? {'Content-Type': 'application/json'} : {}), ...init.headers } }), parses JSON when possible, and on a non-2xx response throws an Error whose message is the JSON "error" or "message" field, or "Request failed (status)".
- Typed helpers:
  getProducts(): GET /api/products -> Array<{ id, key, name, priceCents, currency, shippingCents, imageUrl, customizable, active }>.
  checkCoupon(code, quantity): GET /api/orders/coupon?code=&productKey=nfc_card&quantity= -> { valid: boolean, code?: string, discountCents?: number, message?: string }.
  searchBusiness(name, city, signal): GET /api/business-search?name=&city= -> { available: boolean, candidates: Array<{ name, address, placeId, reviewLink, photoUrl, rating, reviewCount, category }> }.
  autocompleteAddress(search): GET /api/address/autocomplete?search= -> { suggestions: Array<{ text, streetLine, secondary, city, state, zipcode }> }.
  verifyAddress({ street, street2?, city, state, zipcode }): POST /api/address/verify -> { verified: boolean, address?: { address1, address2, city, state, postalCode }, message?: string }.
  captureSession(audienceType): POST /api/session with { audienceType }.
  createOrder(payload): POST /api/orders -> { checkoutUrl: string }.
- track(event, params): calls window.fbq('track', event, params) if fbq exists, wrapped in try/catch. trackCustom(event, params) does the same with 'trackCustom'.

lib/reviewLink.ts: export classifyLink(raw: string): 'trusted' | 'profile-only' | 'unrecognized'. Trim; return 'unrecognized' if empty. Parse with new URL, retrying with "https://" prefixed. Lowercase host without "www.". Rules: host givetings.com or any subdomain -> trusted; host g.page -> trusted if pathname starts with "/r/", else profile-only; host google.com or any subdomain -> trusted if pathname contains "writereview", profile-only if pathname contains "/maps/place" or starts with "/maps" or the query contains "cid="; host maps.app.goo.gl -> profile-only; anything else -> unrecognized. Also export normalizePhone(v): strip non-digits, drop a leading "1" when 11 digits, return "+1" + digits when exactly 10 digits, else null. And isEmail(v): /^[^\s@]+@[^\s@]+\.[^\s@]+$/.

lib/order.ts: a useOrder() hook that owns all order state and exposes it to the components:
- state: product (or null), qty (default 3), coupon ({ code, discountCents } | null), couponNotice (string | null), mode ('search' | 'link'), business (candidate | null), reviewLink (string), linkTrust, audienceType ('smb' default | 'agency'), isPaigeCustomer (false), isAgencyPartner (boolean | null), shipping fields (email, phone, businessName, firstName, lastName, address1, address2, city, state, postalCode), addressVerified (false), addressOverride (false), addressStatus ({ kind: 'idle' | 'checking' | 'ok' | 'fail', message }), errors (string[]), submitting (false).
- derived: unitCents = product?.priceCents ?? 1000; subtotalCents = unit * qty; discountCents = coupon ? min(coupon.discountCents, subtotal) : 0; totalCents = subtotal - discount; money(cents) renders "$21" for whole dollars and "$21.50" otherwise.
- on mount: track('ViewContent', { content_name: 'NFC Review Card', content_ids: ['nfc_card'], content_type: 'product', value: 10, currency: 'USD' }); read utm_source, utm_medium, utm_campaign, utm_content, utm_term, fbclid, gclid, ttclid from the URL and store them as JSON in sessionStorage under "nfc_lp_attribution"; load products and keep the one with key "nfc_card"; validate the configured coupon silently.
- validateCoupon(code, silent): guard against out-of-order responses with a counter. On valid: set coupon and clear the notice. On invalid: coupon = null; notice = silent ? "The 30% code REVIEWS30 isn't active yet, so the total below is the full price. Email reviewstore@merchynt.com and we'll sort it out." : (response.message ?? "That promo code isn't valid."). On network error: "We couldn't check the promo code right now. Refresh the page to try again."
- setQty(n, fromUser): clamp 1..500; re-validate the current coupon silently; if fromUser, track('AddToCart', { content_name, content_ids, content_type, num_items: n, value: total/100, currency }) and captureSessionOnce().
- captureSessionOnce(): POST /api/session once per audience choice; ignore failures.
- selectBusiness(candidate): sets business, reviewLink = candidate.reviewLink, linkTrust 'trusted', prefill businessName if empty, captureSessionOnce(). clearBusiness() reverses it. setMode(mode).
- setReviewLinkInput(value): classify; trusted -> accept (reviewLink = value, business = null); profile-only or unrecognized -> reviewLink = ''. setLinkOverride(checked): when checked and trust is unrecognized, accept the value.
- address helpers: setField(name, value) resets addressVerified/override/status when an address field changes; runVerify() posts { street: address1, street2: address2 || undefined, city, state, zipcode: postalCode } and on verified writes the normalized address back and sets status ok "Address confirmed by USPS."; on not verified sets status fail with (message ?? "USPS couldn't confirm that address.") and shows the override checkbox; on error sets status fail "We couldn't reach the address checker. Confirm below to ship as typed." applySuggestion(s) fills address1/address2/city/state/postalCode from streetLine/secondary/city/state/zipcode then runVerify().
- validate(): returns the error list: reviewLink missing -> "Pick your Google listing (or paste your review link) in step 2."; bad email -> "Enter a valid email address."; bad phone -> "Enter a 10-digit US cell phone number."; empty businessName -> "Enter your business name."; missing first or last name -> "Enter the first and last name for shipping."; incomplete address -> "Complete the shipping address."; address complete but neither verified nor override -> 'Wait for the address check, or tick "Ship to this address exactly as I typed it."'; audience agency with isAgencyPartner null -> "Tell us whether you are in the Agency Partner program." Also return the set of field names that failed so inputs can get the "error" class.
- submit(): if submitting return; run validate; if errors, set them and return. Otherwise set submitting, track('AddPaymentInfo', { value, currency }) and track('InitiateCheckout', { content_ids, content_type, num_items, value, currency }), build the payload below with a stable idempotencyKey (crypto.randomUUID(), kept in a ref and reused on retry), call createOrder, and on success set window.location.href = response.checkoutUrl. On error: clear the idempotency key, set errors to [error.message || 'Failed to create the order. Please try again.', 'If it keeps happening, email reviewstore@merchynt.com and we will place it for you.'], submitting = false.

PAYLOAD (must match exactly; the backend validates it):
{
  productKey: 'nfc_card',
  quantity: qty,
  idempotencyKey,
  setupMethod: 'manual',
  email: email.trim(),
  phone: normalizePhone(phone),            // "+15551234567"
  businessName: businessName.trim(),
  reviewLink: reviewLink || undefined,
  audienceType,                            // 'smb' | 'agency'
  shippingMethod: 'standard',
  shipping: { firstName, lastName, address1, address2: address2 || undefined, city, state: state.toUpperCase(), postalCode },
  couponCode: coupon ? coupon.code : undefined,
  isPaigeCustomer: isPaigeCustomer === true,
  isAgencyPartner: audienceType === 'agency' ? isAgencyPartner : null,
  flyer: undefined
}
```

## Prompt 13 — Wire the order section

```
Connect the OrderSection components to useOrder() (call the hook once in OrderSection and pass what each child needs).

QuantityStep: chips call setQty(n, true) and show "active" for the current qty; minus/plus call setQty(qty - 1, true) / setQty(qty + 1, true); typing in the input calls setQty(value, true) on input and re-clamps on blur; the savings span shows "You save $9" when discountCents > 0, else empty. The step-num gets "done" once any quantity is set (always, after mount).

BusinessStep:
- Show the SEARCH block when mode === 'search' and no business is selected, the LINK block when mode === 'link' and no reviewLink is accepted, and the SELECTED block whenever a business is selected.
- Search: keep city and name in local state; a useEffect with a 450ms debounce calls searchBusiness when the name has at least 4 characters and the city is non-empty, skipping when the lowercase "city|name" key is unchanged, and aborting the previous request with an AbortController. Status text: "" when both fields are empty; "Add the city so we can find the right listing." when only the name is typed; "Keep typing the business name." when the city is typed and the name is short; 'Searching Google for "{name}" in {city}...' while loading; "Pick your listing:" when results arrive; "No match yet. Try the city as it appears on Google, or paste your review link below." (pink) for an empty result; "Search isn't available right now. Paste your Google review link below instead." (pink) when available is false or the request fails. Render up to 6 candidates. Clicking one calls selectBusiness.
- Link mode: the input calls setReviewLinkInput. Status: trusted -> green semibold "Looks good. That link opens the Google review form."; profile-only -> pink semibold "That link opens your profile, not the review popup." followed by muted "Use the "Get more reviews" link from your Google Business Profile, or search for your business above."; unrecognized -> muted "We don't recognize that as a Google review link." and show the override checkbox (setLinkOverride).
- Selected block: shows the photo (or building fallback), the name, and meta "{address}  ·  {rating} stars, {reviewCount} reviews" (omit the rating part when there is none). "Change" calls clearBusiness. The step-num gets "done" when reviewLink is accepted.

ShippingStep:
- Every input is controlled by the hook's fields. On phone blur, format the visible value as (555) 123-4567 when it has 10 digits.
- address1: on change, run a 250ms debounce (3+ characters) that calls autocompleteAddress and shows up to 6 suggestions in the dropdown; picking one calls applySuggestion and closes the dropdown; a document-level pointerdown outside the field and dropdown closes it. On blur of address1 (after a 250ms delay, and only if no suggestion was picked) and on blur of address2/city/state/postalCode, call runVerify when address1, city, state and postalCode are all filled and the address is not already verified.
- Address status line renders: checking -> muted "Checking the address with USPS..."; ok -> green semibold message; fail -> pink semibold message plus muted "Check it, or confirm below to ship as typed." with the override checkbox visible (controlled by addressOverride). The step-num gets "done" when addressVerified is true.
- Audience buttons set audienceType (and "active"); the partner block appears only for 'agency'. Paige buttons set isPaigeCustomer. Partner buttons set isAgencyPartner.
- Inputs whose names are in the failed-field set get the "error" class.

OrderSummary:
- Quantity line: "{qty} card(s) at {money(unit)} each". List total = money(subtotal) (shown with strike only when discount > 0). Net = money(total). Subtotal, discount ("-$9" or "$0"), total. Badge: when discount > 0, "{round(discount/subtotal*100)}% off applied" in green; otherwise "Offer pending" in orange. The code label shows coupon?.code ?? CONFIG.couponCode. The notice div shows couponNotice when present.
- The promo input's Apply button calls validateCoupon(typed, false).
- The error box renders the errors list when non-empty and is scrolled into view (smooth, block center) whenever it becomes non-empty.
- The pay button calls submit(); while submitting it is disabled and shows <span className="spinner" /> followed by "Opening secure checkout...".
```

## Prompt 14 — Page behavior

```
Add page-level behavior:

1. useFadeUp hook: on mount, observe every element with class "fade-up" (IntersectionObserver, threshold 0.12); add "visible" the first time it intersects and unobserve it. Call it once in App after the components mount. Elements can carry an inline transitionDelay style for staggering.

2. CTA click hook: on click of any anchor with a data-cta attribute, call trackCustom('OrderCTAClick', { placement: <the data-cta value> }) when window.fbq exists. Never block the scroll.

3. Sticky CTA visibility as described in Prompt 11.

4. Phone mockup animation as described in Prompt 5 (once, on 50% visibility).

5. Smooth scrolling for all "#order" and "#top" anchors (html scroll-behavior smooth; the order section has scroll-mt-4).

6. REDUCED MOTION: under prefers-reduced-motion: reduce, fade-up and hero-animate elements render visible immediately, the floating cards and CTA pulse are static, the phone stars render orange with the review text already typed.

7. Leave the three commented tracking placeholders in index.html (Meta Pixel, Clarity, Vercel Analytics). Do not add any tracking script yourself.
```

## Prompt 15 — QA pass

```
Review the whole page against index.html (the reference) at 1280px, 1024px, 768px and 375px and fix anything that fails:
- Pixel parity: section backgrounds alternate white / cloud / white / cloud / white / cloud (order) / white (FAQ) / navy (final CTA) / navydeep (footer). Hero and final CTA are navy with the dot grid. Type sizes, weights, radii (rounded-2xl cards, rounded-3xl frames and panels, rounded-full CTAs) and shadows match the tokens.
- No horizontal scroll at 375px. Gutters are at least 20px everywhere. Floating hero cards do not push the layout.
- The header CTA is hidden below sm; the sticky bar is hidden at lg and up and only shows between the hero and the order section.
- Changing quantity updates the chips, the stepper, every summary number and the "You save" text, and re-validates the coupon (a network call).
- With no coupon in the backend the badge reads "Offer pending" in orange and the notice explains it; the total shows full price. This is correct behavior, not a bug.
- Typing city "Sacramento" and name "Roofing Sacramento" returns a list within a few seconds; picking one shows the green Selected block with the photo and "5.0 stars, 10 reviews"-style meta.
- Pasting https://g.page/r/abc/review shows the green "Looks good" line; pasting https://maps.app.goo.gl/xyz shows the pink "profile, not the review popup" line and is not accepted; pasting https://example.com shows the override checkbox.
- Typing "1600 Pennsylvania Ave SE" in the street field shows suggestions; picking "Washington, DC 20003" fills the fields and, after a moment, shows "Address confirmed by USPS." with the ZIP as 20003-3228.
- Clicking Proceed with an empty form shows the pink error list and marks the failing fields; with a complete form the button shows the spinner and the browser navigates to a checkout.stripe.com URL.
- FAQ opens one item at a time. The phone stars animate once. prefers-reduced-motion removes every animation.
- No emojis, no em dashes, the phrase "no credit card" does not appear, and "GBP" never appears in visible copy.
- All images have alt text. The phone mockup star row is aria-hidden. Buttons have accessible names (the stepper buttons carry aria-labels).
- Fonts: DM Sans everywhere. Lighthouse accessibility 95 or above.
```
