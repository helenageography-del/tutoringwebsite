# One-shot build prompt — Helena's Geography Tutoring Site

Paste everything below the line into a fresh Claude Code session (in this
repo) to build the site in one pass.

---

Build a static marketing + booking website for a private geography tutor
(GCSE/IB/A-Level), served from Docker via nginx. No backend, no database,
no build step — plain HTML/CSS/vanilla JS only.

## Pages / sections (single-page site, one `index.html` with anchored sections)

1. **Hero** — full-width background photo, tutor's name, one-line pitch,
   "Book a Lesson" CTA button linking to the booking section.
2. **About** — portrait photo + bio paragraph (placeholder copy is fine,
   editable later), credentials/experience list.
3. **Subjects & Levels** — card grid: GCSE Geography, IB Geography (SL/HL),
   A-Level Geography. Each card: short description, "who it's for".
4. **How it works / Pricing** — simple 3-step explainer (Book → Confirm →
   Learn) plus a pricing table (per-hour rate, package options). Use
   placeholder numbers clearly marked as placeholder.
5. **Testimonials** — 3-4 quote cards, placeholder text, no photos required.
6. **Booking & Payment** — embedded Cal.com booking widget (see "Booking
   integration" below). This is where payment happens — Cal.com handles
   Stripe checkout itself, nothing custom to build here.
7. **Footer** — contact email, social links (placeholders), copyright.

Keep design clean and modern: reasonable font pairing, generous whitespace,
mobile-responsive (test at 400px width). Don't over-engineer — no JS
framework, no CSS framework required unless it's trivially copy-pasted
(e.g. a CDN-linked minimal reset is fine).

## Media — MUST be drop-in, MUST work without a rebuild

Create exactly these three directories at the repo root, each holding a
single expected file (not yet present):

```
media/hero/hero.jpg          <- wide background photo, recommend 1920x1080+
media/portrait/portrait.jpg  <- Helena's photo for the About section, recommend 800x800+ (square)
media/intro-video/intro.mp4  <- optional short intro video, recommend <50MB, H.264 mp4
```

Requirements:
- Put a `README.txt` in each of the three directories stating the exact
  expected filename (including extension/case) and the recommended
  dimensions/format — so the convention is discoverable by opening the
  folder, not just by reading this prompt.
- The `docker-compose.yml` MUST bind-mount `./media` into the container
  (e.g. `./media:/usr/share/nginx/html/media`) as a **volume**, not bake
  it into the image with `COPY`. Dropping a correctly-named file into
  `media/hero/`, `media/portrait/`, or `media/intro-video/` and refreshing
  the browser must show it immediately — no `docker compose build`, no
  restart.
- On first run these directories are empty. Missing images must NOT show
  broken-image icons:
  - The hero section must declare a CSS gradient/solid background on the
    element itself, with `hero.jpg` layered on top via CSS
    `background-image`. If the file is absent, the layer is simply blank
    and the gradient underneath shows through automatically — no JS
    fallback needed for this one.
  - The portrait is an `<img>` tag, so give it an `onerror` handler that
    swaps in a placeholder graphic or hides the `<img>` and reveals a
    fallback avatar/initials block instead.
  - The intro video section is expected to be absent on first run — it's
    optional content, and hiding that whole section (via `onerror`/`onload`
    detection or a simple existence check) when `intro.mp4` is missing is
    correct behavior, not a bug to fix. The page should read fine as a
    6-section site until that file is dropped in.

## Booking integration — Cal.com, explicitly a placeholder

Helena does not yet have a Cal.com account or a Stripe-connected payment
setup — that's a manual step for her, not something this build can create.

- Put the Cal.com config in one small, obviously-named file, e.g.
  `js/booking-config.js`:
  ```js
  // Replace this with Helena's real Cal.com event link once her account
  // and Stripe connection are set up (see README.md).
  const CAL_LINK = "REPLACE-ME/geography-lesson";
  ```
- The booking section must check this value. While it's still
  `"REPLACE-ME/geography-lesson"`, show a clearly visible
  "Booking isn't set up yet" message in place of the widget (not a broken
  embed, not a silent failure). Once `CAL_LINK` is replaced with a real
  Cal.com link, the embed should just work (use Cal.com's standard embed
  snippet/inline widget, pointed at `CAL_LINK`).

## Docker setup

- `Dockerfile`: `FROM nginx:alpine`. `COPY` only the specific static
  assets — `COPY index.html`, `COPY css/ ./css/`, `COPY js/ ./js/` — into
  `/usr/share/nginx/html/`. Do **not** use a blanket `COPY . .` (it would
  pull in `media/` and shadow the bind mount below). Also add a
  `.dockerignore` file containing `media/` as a second safeguard.
- `docker-compose.yml`: one service, build from the Dockerfile, map a host
  port (e.g. `8080:80`), bind-mount `./media` as described above.
- Running `docker compose up --build` from the repo root should serve a
  fully working (if placeholder-content) site at `http://localhost:8080`
  with no further setup.

## Deliverables checklist

- [ ] `index.html`, `css/style.css`, `js/booking-config.js` (+ any other
      small JS needed)
- [ ] `media/hero/README.txt`, `media/portrait/README.txt`,
      `media/intro-video/README.txt`
- [ ] `Dockerfile`, `docker-compose.yml`
- [ ] Top-level `README.md` explaining: how to run it
      (`docker compose up --build`), where to drop photos/video, and that
      Helena needs to create a Cal.com account with Stripe connected and
      update `CAL_LINK` in `js/booking-config.js` before booking/payment
      goes live.
- [ ] Verify in a browser: page loads with placeholder graceful fallbacks,
      resizes cleanly on mobile width, booking section shows the
      "not yet configured" message.

---

## Manual step outside this prompt (for Caleb, not Claude)

Helena needs her own Cal.com account with an event type created and Stripe
connected for payments, before `CAL_LINK` can be set to a real value and
booking/payment goes live end-to-end. Nothing in this repo can create that
account for her.
