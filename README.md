# The P.I.M.P. Report

Front-end for The Proven Insights on Market Predictability Report — predictive market analyses (PMAs) issued as tradable ERC-721 assets.

Single self-contained `index.html`. No build step, no dependencies, no bundler. Open it, host it, done.

---

## Run it

```bash
# any static server works
python3 -m http.server 8000
# then open http://localhost:8000
```

Or just double-click `index.html`.

## Deploy

**GitHub Pages** — already wired. Push to `main`, then in the repo go to
**Settings → Pages → Build and deployment → Source: GitHub Actions**. Next push publishes.

**Anywhere else** — it's one static file. Netlify, Vercel, Cloudflare Pages, S3, or IPFS
(matching the current Pinata setup) all work with zero configuration.

---

## Editing content

Everything an admin would change lives in one place: the `DATA` block near the top of the
`<script>` tag in `index.html`.

| Array | Controls |
|---|---|
| `NFTS` | The NFT gallery and detail modals. `status` is `minted` \| `soon` \| `open`. |
| `NFT_CATS` | Sector filter chips. |
| `VIDEOS` | The video portfolio. `cat` is `2025` \| `2023` \| `market`. |
| `VID_CATS` | Video filter chips. |
| `PMAS` | The "Predictive Market Analyses" coverage cards. |
| `FEATURED` | Which three NFTs appear in "Latest NFTs" — by series id. |
| `KB` | The assistant's knowledge base (used when no backend is connected). |
| `CFG` | Hero video URL and API endpoints. |

To add a video, copy a line in `VIDEOS` and change the title, blurb, thumbnail and YouTube id.
Nothing else needs touching.

### Hero video

`CFG.heroVideo` is empty, so the hero uses an animated poster image. Drop in an MP4 or WebM URL
and it becomes a looping background video automatically, with the image as the poster fallback.

---

## Connecting a backend

Three endpoints are already wired up in `CFG.api`. All three fail gracefully — the forms show a
fallback message and the assistant drops to local retrieval, so nothing breaks while they're missing.

| Endpoint | Method | Body | Purpose |
|---|---|---|---|
| `/api/chat` | POST | `{ message, history }` | Assistant. Responds with a text stream; the client reads it incrementally. |
| `/api/leads` | POST | `{ name, email, phone, message }` | Contact form → CRM. |
| `/api/subscribe` | POST | `{ email }` | Newsletter signup. |

The assistant already speaks the streaming protocol. Point `/api/chat` at a real RAG service and
it works without any front-end changes.

---

## Performance notes

Deliberate choices, worth preserving if you edit the CSS:

- `backdrop-filter` is limited to 5 elements. Each one forces its own GPU compositing pass —
  scattering them across cards is what made the first build lag. Use solid or gradient
  backgrounds on cards instead.
- The ambient background glow is a **static** gradient. Animating a large blurred layer repaints
  every frame and is the single most expensive thing you can add to a page like this.
- Animate `transform` and `opacity` only. Not `background-position`, not `box-shadow`.
- YouTube embeds load on click, not on page load. 27 iframes at once would cost several megabytes.
- Video cards use `content-visibility: auto` so offscreen cards aren't rendered.
- `prefers-reduced-motion` is respected, and the heaviest effects are disabled under 820px.

### Known bottleneck

The NFT artwork and video thumbnails are served at full resolution straight from IPFS. That's the
main remaining cost on mobile. Generating resized WebP versions (roughly 800px wide for gallery
thumbnails) would cut image weight substantially. It's a content-pipeline change, not a code change.

---

## Accessibility

Keyboard navigable throughout, visible focus rings, semantic landmarks, ARIA on tabs, filters,
the modal and the assistant. Skip link to main content. Escape closes the modal and assistant.
Reduced-motion respected.

---

## Roadmap — not yet built

The public site is complete. These require a server, a database and a security review, and are
deliberately not stubbed out here:

1. **Auth** — email/password, bcrypt, JWT with refresh tokens, rate limiting, CSRF, secure cookies.
2. **CRM + leads** — makes `/api/leads` real. Best first step, since the contact form already posts to it.
3. **RAG assistant** — document ingestion, embeddings, vector search, streaming completions behind `/api/chat`.
4. **Admin dashboard** — analytics, knowledge base uploads, NFT and media management.
5. **CMS** — moves the `DATA` arrays out of the file and into a database.

---

© The P.I.M.P. Report. All content, artwork and media rights reserved.
