# The Bluest Hour

> "In certain latitudes there comes a span of time approaching and following the summer solstice, some weeks in all, when the twilights turn long and blue."
> — Joan Didion, *Blue Nights*

A small, personal web app that tells you when to start your evening walk in Godfrey, IL so you're outside as the sky deepens into its bluest hour — the window between civil and nautical twilight, when the sun sits 6°–12° below the horizon and the light approximates, as Didion wrote, "the blue of the glass on a clear day at Chartres."

## Two versions

This project exists in two forms:

**Hugging Face Space** (`bluest-hour-space/`) — A Gradio app that runs server-side using Python. It calls the [Sunrise-Sunset API](https://sunrise-sunset.org/api) to fetch twilight times and renders the result as styled HTML. This is the version deployed at Hugging Face.

**GitHub Pages** (`bluest-hour/` — this repo) — A static, client-side version built with vanilla HTML, CSS, and JavaScript. It runs entirely in the browser with no server, no build step, and no backend. Deployed via GitHub Pages.

## How it works

The blue hour is the period each evening after civil twilight ends but before nautical twilight ends. During this window, the sun is far enough below the horizon that direct and scattered sunlight no longer reaches the sky, but indirect sunlight still scatters preferentially at short (blue) wavelengths through the upper atmosphere. The result is that deep, saturated blue that photographers and painters prize.

The app:

1. Fetches today's twilight data from the Sunrise-Sunset API (free, no key required) for Godfrey, IL (38.9556°N, 90.1868°W).
2. Identifies the blue hour as the window between `civil_twilight_end` and `nautical_twilight_end`.
3. Centers a 20-minute walk on the midpoint of that window.
4. Displays the recommended walk start time, a twilight timeline, and overlap statistics.

## GitHub Pages architecture

The static version is a single `index.html` file. It uses:

- The [Sunrise-Sunset API](https://api.sunrise-sunset.org/json) directly from the browser via `fetch()`.
- The browser's `Intl.DateTimeFormat` for timezone-aware time formatting (no pytz needed).
- Pure CSS for the gradient walk card and timeline — no frameworks, no dependencies.

Because the Sunrise-Sunset API supports CORS, the entire app works as a static page with no proxy or serverless function required. GitHub Pages serves the HTML; the browser does everything else.

## Transformers.js v4 — future directions

[Transformers.js v4](https://huggingface.co/blog/transformersjs-v4) (released February 2026) allows running Hugging Face models directly in the browser via WebGPU. While the current version of this app doesn't use ML inference — it's a straightforward astronomical calculation — Transformers.js v4 opens up interesting possibilities for future iterations:

- **Sentiment-aware walk journaling.** A small text classifier (like `Xenova/distilbert-base-uncased-finetuned-sst-2-english`) could run in-browser to analyze mood entries from evening walks, tracking emotional patterns alongside the shifting blue hour across seasons.
- **Image classification of sky photos.** Users could photograph the sky during their walk and a vision model could classify the twilight phase or dominant color temperature — a kind of empirical check on the astronomical predictions.
- **Poetic text generation.** A small language model could generate Didion-style reflections on the evening's light conditions, seeded by the twilight data.

Transformers.js v4 can be loaded from a CDN with no build step, which makes it a natural fit for this kind of static GitHub Pages project:

```html
<script type="module">
  import { pipeline } from "https://cdn.jsdelivr.net/npm/@huggingface/transformers@4.0.1";

  const classifier = await pipeline(
    "sentiment-analysis",
    "Xenova/distilbert-base-uncased-finetuned-sst-2-english"
  );
  const result = await classifier("The sky was impossibly blue tonight.");
  console.log(result);
</script>
```

Key v4 features relevant to this project:

- **No build step required.** Import directly from jsDelivr CDN via ES modules.
- **WebGPU acceleration.** BERT-class models run up to 4x faster than v3. Supported in Chrome 113+, Firefox 141+, and Safari 26+.
- **53% smaller bundle.** The `transformers.web.js` export is optimized for browser delivery.
- **WASM caching.** Models are cached locally after first download for offline-capable repeat visits.

## Local development

No build step. Just serve the directory:

```bash
python -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`.

## Deployment

1. Push this repo to GitHub.
2. Go to **Settings → Pages → Source** and select the `main` branch.
3. The site will be live at `https://<username>.github.io/bluest-hour/`.

## Course context

This project is part of the [Level 2: Applied Machine Learning Concepts](https://github.com/buildLittleWorlds/level-2-course-material) curriculum, which teaches students to build and deploy Hugging Face Spaces. The dual deployment (HF Space + GitHub Pages) demonstrates how the same idea can be implemented server-side with Python/Gradio and client-side with vanilla JavaScript, and how Transformers.js v4 bridges the gap by bringing ML inference to static sites.

## Credits

- Twilight data: [Sunrise-Sunset API](https://sunrise-sunset.org/api)
- Epigraph: Joan Didion, *Blue Nights* (2011)
- ML inference (future): [Transformers.js v4](https://huggingface.co/blog/transformersjs-v4) by Hugging Face
