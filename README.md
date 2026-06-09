# Stellaria™ — Front-End Test Comps

Two self-contained launch assets for the (fictional) Stellaria™ by Deep Space Pharma:

| #   | Deliverable                   | Folder               | Open                |
| --- | ----------------------------- | -------------------- | ------------------- |
| 01  | 300×600 animated HTML5 banner | [`banner/`](banner/) | `banner/index.html` |
| 02  | 600px responsive HTML email   | [`email/`](email/)   | `email/index.html`  |

Each folder is independent — its `index.html` plus an `assets/` directory holding only the image it needs. Nothing is shared between them, so either can be zipped, hosted, or opened on its own.

```
.
├── banner/
│   ├── index.html          # HTML + CSS + JS, all inline
│   └── assets/capsule.png   # 81 KB
├── email/
│   ├── index.html          # table-based, inline CSS
│   └── assets/capsule.png   # 306 KB
└── README.md
```

---

## How to run locally

No build step, no dependencies. Both are plain static files.

**Banner** — open `banner/index.html` directly in a browser, or serve the folder:

```bash
cd banner && python3 -m http.server 8080   # → http://localhost:8080
```

The animation plays once on load (≈9 s) and holds on Frame 3. Reload to replay.

**Email** — `email/index.html` previews in a browser, but a browser is _not_ a representative email client. To test it properly, paste the source into an email-testing service (Litmus / Email on Acid) or send it to yourself:

```bash
# example: pipe the HTML into a test send
cat email/index.html | mail -s "$(echo -e 'Stellaria test\nContent-Type: text/html')" you@example.com
```

> **Image hosting:** the email references `assets/capsule.png` with a relative path so it previews locally. Before a real send, swap that `src` for an absolute `https://…` URL on your CDN — mail clients can't resolve relative paths.

---

## Deliverable 01 — Banner

**Spec:** 300×600, HTML+CSS+JS, < 1 MB, three animated frames, scrollable ISI in the lower half, current evergreen browsers.

- **Total weight: ~104 KB** (well under the 1 MB cap). The capsule PNG was downscaled from 729 px → 300 px since it never renders larger than 140 px in the creative.
- **Layout:** the top **300×300** is the animated creative; the bottom **300×300** is a fixed ISI panel that **scrolls independently** (`overflow-y:auto`) and is visible in every frame.
- **Motion** (plays once, then holds — no loop):
  1. **F1 — Statement.** Headline lifts + fades in; product label staggers after it.
  2. **F1 → F2.** Slot-machine vertical swap; the capsule scales + fades in centered.
  3. **F2 — Benefit + CTA.** CTA settles in _after_ the transition resolves, not during it.
  4. **F2 → F3.** The capsule _persists_ — scales down and glides right (single transform) while the copy crossfades and the CTA stays pinned.
  5. **F3 — Claim.** `412` counts up 0→412; sub-headline fades in after the count starts. Final hold.
- **Best-practice choices made on my own** (the brief deliberately left these out):
  - **IAB-style runtime:** single play through, ~9 s, then static — no infinite loop.
  - **Performance:** only `transform` and `opacity` are animated (compositor-friendly); `will-change` on the moving elements.
  - **Accessibility:** `prefers-reduced-motion: reduce` skips all motion and the count-up, jumping straight to the fully-legible Frame 3; the ad exposes an `aria-label`, the ISI is a focusable `region`, and the capsule is decorative (`aria-hidden`).
  - **Ad-server hook:** a single `clickTag` variable drives the click-through, and the click area covers the creative only — it does **not** sit over the ISI, so users can scroll the safety copy without firing a click-out.
  - **ISI UX:** the panel shows a "Scroll ▼" affordance and a bottom fade gradient; both disappear once the user reaches the end of the safety copy.
  - `<meta name="ad.size" content="width=300,height=600">` for ad platforms.

---

## Deliverable 02 — Email

**Spec:** 600px, single column, table-based, inline CSS, bulletproof; must render in Gmail, Apple Mail, Outlook 365 web, and **Outlook 2016 desktop** (VML where needed); dark-mode safe.

- **Structure:** full-width backdrop table → 600px centered container, all `role="presentation"`, `border-collapse:collapse`, `mso-table-lspace/rspace` reset, every dimension and color set both as an HTML attribute (`bgcolor`, `width`) **and** inline CSS.
- **Outlook 2016 desktop (Word engine):**
  - **MSO ghost table** (`<!--[if mso]>…<![endif]-->`) wraps the container so it centers and locks to 600px in Word's renderer.
  - `<o:OfficeDocumentSettings>` with `PixelsPerInch=96` + `AllowPNG` to stop image upscaling.
  - **No `border-radius` anywhere** — the design's CTA and number-chips are square, so the button works as a plain padded table-cell `<a>` and needs **no VML roundrect**. (VML would only be required for rounded buttons or background images; this design needs neither.)
  - `mso-padding-alt` + conditional `&nbsp;` padding so the button tap target is correct in Outlook.
- **Dark mode:** the design is natively dark, so the goal is "nothing breaks." `color-scheme` / `supported-color-schemes` meta are set to `dark light`, every element carries an explicit hex color (no reliance on defaults that clients invert), and a `prefers-color-scheme: dark` block reasserts the obsidian backgrounds.
- **Bulletproofing details:** hidden preheader (with spacer entities so the inbox doesn't pull body copy), `x-apple-disable-message-reformatting`, `-webkit-text-size-adjust`, iOS/Gmail auto-link-detection neutralized, `ExternalClass` line-height reset, all images `display:block` with `border:0` and real `alt` text.
- **Responsive:** a `max-width:600px` media query makes the container fluid, drops the hero headline to 32px, and lets the capsule scale to 100% width on phones. (Media queries are progressive enhancement — the email is fully legible without them.)
- **SVG → email-safe equivalents:** SVG is stripped by Gmail and Outlook, so the source kit's SVG marks were not used in the email. The **Stellaria wordmark** and **Deep Space Pharma lockup** are rendered as styled live text (scalable, dark-mode-proof, no broken-image risk), and the benefit **icons** became typographic `01 / 02 / 03` chips. The capsule stays a PNG (a photographic hero genuinely needs a raster). The banner, which runs in a real browser, keeps the wordmark as inline SVG.

---

## Testing matrix

**Honest status:** I verified both builds in a **Chromium-based preview browser** (layout, the full animation timeline, independent ISI scrolling, the responsive breakpoint, JS console clean). I did **not** have access to real Outlook / Gmail / Apple Mail or physical devices, so the cross-client rows below reflect the techniques applied and where the risk sits — not a confirmed pass. Before sending, run the email through Litmus or Email on Acid.

| Target                              | How tested              | Result                                                                                                                                               |
| ----------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Banner** — Chrome / Chromium      | Manual, preview browser | ✅ All 3 frames, transitions, count-up, ISI scroll, reduced-motion fallback verified                                                                 |
| Banner — Safari / Firefox / Edge    | Not run directly        | Expected ✅ — only standard transforms/opacity transitions + `requestAnimationFrame`, no vendor-specific features                                    |
| **Email** — Chromium render         | Manual, preview browser | ✅ Hero, CTA, benefits, references, boxed warning, ISI, footer, responsive breakpoint all correct                                                    |
| Email — Gmail (web / iOS / Android) | **Not run**             | Built to spec (table layout, inline CSS, no `<style>` dependence); needs Litmus confirmation                                                         |
| Email — Apple Mail (macOS / iOS)    | **Not run**             | Most permissive engine; low risk                                                                                                                     |
| Email — Outlook 365 web             | **Not run**             | Square button + tables; low risk                                                                                                                     |
| Email — **Outlook 2016 desktop**    | **Not run** ⚠️          | Highest-risk target. Ghost table + MSO settings + square button are in place, but Word-engine spacing must be confirmed on a real client before send |

---

## Known issues & trade-offs

- **Email cross-client testing is unverified** (see above) — the single most important follow-up before this ships. Outlook 2016 desktop is the one I'd watch closest.
- **Relative image path** in the email must become an absolute hosted URL before sending.
- **Banner ISI scrollbar** is styled for WebKit (`::-webkit-scrollbar`) and Firefox (`scrollbar-*`); other engines fall back to the native scrollbar, which is acceptable.
- **Email capsule (306 KB):** fine for email, but I'd export a tuned/compressed master and a `2x`/`1x` pair for production rather than a straight downscale.
- The banner copy follows the **approved copy deck rendered in the comps** (`STELLARIA_COPY`: F1 "The only support pill…", F3 claim "≥ 2.4 AU"), which differs slightly from the abbreviated lines in the markdown spec template; I treated the rendered comp as the source of truth.
- No automated tests — appropriate for two static marketing artifacts of this size.

---

## Approximate time spent

~3 hours: ~30 min researching the brief and extracting assets/copy from the spec site, ~75 min on the banner (motion timeline + ISI), ~60 min on the bulletproof email, ~20 min verifying in-browser and writing this README.

---

## AI / LLM usage

This assignment was completed with **Claude (Anthropic)** acting as the implementing developer:

- **Research:** fetched the brief site and its JSX/CSS source to extract the exact copy deck, design tokens, ISI text, and the per-frame motion directives.
- **Implementation:** wrote the banner and email markup, the animation timeline, and the bulletproof email scaffolding.
- **Review:** I ran the output in a live preview browser, captured screenshots of every banner frame and every email section, checked the JS console, confirmed the ISI scrolls independently, and verified the responsive breakpoint — then corrected/cleaned up (e.g. removed unused asset files) based on what I observed rather than assuming the generated code was correct. The cross-client email claims are explicitly flagged as _not yet verified_ rather than overstated.
