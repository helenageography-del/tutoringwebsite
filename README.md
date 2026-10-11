# Helena Geography Tutoring — Site

A static marketing and booking site for Helena's private geography tutoring
(GCSE and A-Level, plus exam skills coaching), served via Docker + nginx.
No backend, no database, no build step.

## Running it

```
docker compose up --build
```

Then open http://localhost:8080. That's it — no other setup is required
for the site to load, though booking won't be live until Cal.com is
connected (see below).

## Adding photos and video

Drop files into these folders, then just refresh your browser — no
rebuild or restart needed, since `./media` is bind-mounted into the
container as a live volume:

- `media/hero/hero.jpg` — wide background photo for the hero section
  (1920x1080 or larger recommended)
- `media/portrait/portrait.jpg` — Helena's photo for the About section
  (800x800 or larger, square, recommended)
- `media/intro-video/intro.mp4` — optional short intro video (under
  50MB, H.264 mp4 recommended)

Each folder has its own `README.txt` restating these filenames. Until a
file is added, the page falls back gracefully: a background gradient
shows instead of the hero photo, a placeholder avatar shows instead of
the portrait, and the whole intro-video section stays hidden.

## Turning on booking and payment

The booking section embeds a Cal.com widget, which handles Stripe
checkout itself — nothing custom needs to be built for payment.

Before it can go live, Helena needs to:

1. Create a Cal.com account.
2. Set up an event type for lessons (e.g. "geography-lesson").
3. Connect Stripe to that event type so payment is collected at booking,
   and set the event's currency to GBP (£) in Cal.com/Stripe, since the
   site's pricing is displayed in GBP.
4. Get the Cal.com link for that event type (format: `username/event-name`).

Then open `js/booking-config.js` and replace the placeholder:

```js
const CAL_LINK = "REPLACE-ME/geography-lesson";
```

with the real link, e.g.:

```js
const CAL_LINK = "helena-geography/geography-lesson";
```

Save the file and refresh the browser — the Cal.com booking widget will
load automatically. Until this is done, the booking section shows a
"Booking isn't set up yet" message instead of a broken embed.

## Project structure

```
index.html              Single-page site
css/style.css           All styling
js/booking-config.js    Cal.com link — edit this to go live
js/booking.js           Loads the Cal.com embed / shows fallback message
media/hero/             Drop hero.jpg here
media/portrait/         Drop portrait.jpg here
media/intro-video/      Drop intro.mp4 here (optional)
Dockerfile              nginx:alpine, copies only index.html/css/js
docker-compose.yml      Builds the image, bind-mounts ./media
```
