# The Bluest Hour — Full Specification

## What this document is

This is the complete technical and conceptual specification for The Bluest Hour, a static web application deployed via GitHub Pages. It is written so that an AI assistant with no prior context can understand every decision, every dependency, and every line of code in the project. It is also written so that a human reader — a student, a teacher, a curious developer — can follow the thread from astronomy to sentiment analysis without losing the plot.

---

## 1. The idea

Every evening, for a window of roughly 20–40 minutes (depending on latitude and season), the sky turns a specific, saturated blue. This is the "blue hour" — the interval after civil twilight ends but before nautical twilight ends, when the sun sits between 6° and 12° below the horizon.

During this window, direct sunlight no longer reaches the observer. But indirect sunlight still scatters through the upper atmosphere, and because short wavelengths (blue) scatter more than long wavelengths (red), the sky fills with a deep, even blue that has no orange or pink in it. Photographers call it "the blue hour." Joan Didion wrote about it in *Blue Nights* (2011):

> "In certain latitudes there comes a span of time approaching and following the summer solstice, some weeks in all, when the twilights turn long and blue."

The app tells you when to start a 20-minute evening walk in Godfrey, Illinois (38.9556°N, 90.1868°W) so that your walk is centered on the midpoint of this window. It also lets you write a short journal entry about the walk and runs a sentiment analysis model on that entry — entirely in the browser, with no server.

---

## 2. Architecture

### 2.1 Deployment model

This is a **zero-dependency static site**. There is no `package.json`, no `node_modules`, no build step, no bundler, no framework, no backend server, no serverless function, no database.

The entire application is a single file: `index.html`.

It is served by GitHub Pages from the `main` branch of `buildLittleWorlds/bluest-hour`. GitHub Pages serves static files over HTTPS with a global CDN. The site is available at:

```
https://buildlittleworlds.github.io/bluest-hour/
```

### 2.2 Runtime dependencies

The app has exactly two runtime dependencies, both loaded over the network at page load:

| Dependency | What it provides | How it's loaded |
|------------|-----------------|-----------------|
| [Sunrise-Sunset API](https://api.sunrise-sunset.org) | Twilight times for a given latitude/longitude | `fetch()` to a REST endpoint |
| [Transformers.js v4](https://huggingface.co/docs/transformers.js) | In-browser ML inference (sentiment analysis) | ES module `import()` from jsDelivr CDN |

Neither requires an API key. Neither requires a server-side proxy. Both support CORS.

### 2.3 What runs where

Everything runs in the user's browser. The page makes two network requests on load:

1. **Sunrise-Sunset API** → returns JSON with today's twilight times for Godfrey, IL.
2. **jsDelivr CDN** → returns the Transformers.js v4 library (~250KB gzipped), which then downloads a sentiment analysis model (~67MB on first load, cached thereafter) from the Hugging Face Hub.

After these initial loads, the app works entirely offline (the twilight data is for today only, but the model is cached in the browser's Cache API).

---

## 3. The twilight calculation

### 3.1 Astronomical background

The sun's position below the horizon defines three twilight phases:

| Phase | Sun below horizon | What happens |
|-------|-------------------|-------------|
| Civil twilight | 0°–6° | Sky still bright enough to read. Streetlights may not be on yet. |
| Nautical twilight | 6°–12° | The blue hour. Sky is deep blue. Horizon still visible at sea. |
| Astronomical twilight | 12°–18° | Sky effectively dark. Faint stars visible. |

The **blue hour** is the nautical twilight phase — specifically, the evening one (there is a morning blue hour too, but this app ignores it).

### 3.2 The Sunrise-Sunset API

**Endpoint:**
```
https://api.sunrise-sunset.org/json?lat=38.9556&lng=-90.1868&formatted=0
```

The `formatted=0` parameter requests ISO 8601 timestamps in UTC (e.g., `2026-04-10T23:45:12+00:00`) instead of the default 12-hour format. This is critical — without it, the API returns times like `7:45:12 PM` with no timezone information, which is ambiguous and harder to parse.

**Response shape (relevant fields):**
```json
{
  "status": "OK",
  "results": {
    "civil_twilight_end": "2026-04-11T00:42:18+00:00",
    "nautical_twilight_end": "2026-04-11T01:14:33+00:00"
  }
}
```

The API returns many other fields (sunrise, sunset, solar_noon, astronomical_twilight_begin/end, etc.) but the app only uses `civil_twilight_end` and `nautical_twilight_end`.

### 3.3 The walk calculation

The app performs five calculations:

1. **Blue hour duration** = `nautical_twilight_end - civil_twilight_end`, in minutes. Typically 25–40 minutes depending on season and latitude. At Godfrey's latitude (38.96°N), summer blue hours are longer than winter ones.

2. **Midpoint** = `(civil_twilight_end + nautical_twilight_end) / 2`. This is the moment when the blue is deepest — the sun is at approximately 9° below the horizon.

3. **Walk start** = `midpoint - 10 minutes`. The walk is 20 minutes long, centered on the midpoint.

4. **Walk end** = `midpoint + 10 minutes`.

5. **Overlap** = the number of minutes where the walk interval intersects the blue hour interval. If the walk is perfectly centered (which it always is in this version), the overlap equals `min(20, blue_hour_duration)` — i.e., the full walk is inside the blue hour unless the blue hour is shorter than 20 minutes.

### 3.4 Timezone handling

The API returns UTC timestamps. The app converts them to Central Time (America/Chicago) for display using `Intl.DateTimeFormat` — the browser's built-in timezone-aware formatter. This handles CDT/CST transitions automatically. No timezone library is needed.

```javascript
date.toLocaleTimeString('en-US', {
  hour: 'numeric',
  minute: '2-digit',
  timeZone: 'America/Chicago'
});
```

This produces output like `7:42 PM`.

---

## 4. The visual timeline

The app renders a horizontal gradient bar representing the transition from civil twilight (lighter blue, left) to deep nautical twilight (dark navy, right). A gold-bordered overlay marks where the 20-minute walk falls within this window.

### 4.1 How the positioning works

The timeline represents a window that extends 10 minutes before `civil_twilight_end` and 10 minutes after `nautical_twilight_end` — this padding prevents the walk overlay from being clipped at the edges.

```javascript
const windowStart = civilEnd - 10 minutes;
const windowEnd = nauticalEnd + 10 minutes;
const windowDuration = windowEnd - windowStart;

const walkLeftPct = ((walkStart - windowStart) / windowDuration) * 100;
const walkWidthPct = ((walkEnd - walkStart) / windowDuration) * 100;
```

The walk overlay is positioned using `left` and `width` as percentages of the timeline container.

### 4.2 The gradient

The timeline's CSS gradient moves from `#5b9bd5` (a steel blue representing late civil twilight) through `#2e6b9e` and `#1a3a5c` to `#0a1628` (near-black representing deep nautical twilight). This roughly simulates the darkening sky.

---

## 5. Transformers.js v4

This is the core technology that makes the journal feature possible. Here is everything an AI needs to know about it.

### 5.1 What Transformers.js is

Transformers.js is a JavaScript library by Hugging Face that runs machine learning models directly in the browser (or in Node.js/Bun/Deno). It is the JavaScript equivalent of the Python `transformers` library. It does not call any server — the model weights are downloaded to the browser and inference runs locally on the user's device.

**Package name:** `@huggingface/transformers`
**Current version:** 4.0.1 (released February 9, 2026)
**Previous major version:** 3.x (October 2024)
**Historical package name (v1–v2):** `@xenova/transformers` (deprecated — do not use)

### 5.2 How it works under the hood

The execution stack is:

```
Your JavaScript code
        ↓
@huggingface/transformers (pipeline API, tokenizer, pre/post-processing)
        ↓
ONNX Runtime Web (inference engine)
        ↓
WebAssembly (CPU) or WebGPU (GPU)
```

1. **Models must be in ONNX format.** Hugging Face models are typically stored as PyTorch weights. To run in the browser, they must be converted to ONNX using Hugging Face Optimum. Many pre-converted models are available under the `Xenova/` and `onnx-community/` namespaces on the Hugging Face Hub.

2. **ONNX Runtime Web** is the underlying inference engine — a WebAssembly build of Microsoft's ONNX Runtime. Transformers.js wraps it with a high-level `pipeline()` API that handles tokenization, tensor construction, and output decoding.

3. **Two execution backends:**
   - **WASM (WebAssembly):** CPU-only, works in all browsers. This is the default and what this app uses.
   - **WebGPU:** GPU-accelerated, significantly faster for large models. Requires `device: 'webgpu'` in the pipeline options. Supported in Chrome 113+, Firefox 141+ (behind flag), Safari 26+ (behind flag).

### 5.3 What changed in v4

| Feature | v3 | v4 |
|---------|----|----|
| WebGPU runtime | JavaScript implementation | Completely rewritten in C++ with ONNX Runtime team |
| Bundle size (web) | Baseline | 53% smaller |
| Build system | Webpack | esbuild (10x faster builds) |
| BERT model speed | Baseline | ~4x faster (via `MultiHeadAttention` operator) |
| Max model size | Limited | 8B+ parameters (GPT-OSS 20B demonstrated) |
| Cross-platform | Browser-focused | Browser + Node + Bun + Deno |
| Cache management | Manual | `ModelRegistry` API |
| Tokenizer | Built into main package | Also available standalone as `@huggingface/tokenizers` (8.8kB) |
| Logging | Limited | `LogLevel` enum (DEBUG/INFO/WARNING/ERROR/NONE) |
| WASM caching | Not built-in | `env.useWasmCache = true` |
| Custom fetch | Not built-in | `env.fetch` for auth headers, abort controllers |
| Repo structure | Single package | pnpm monorepo with workspaces |
| `models.js` | 8,000+ line monolith | Split into per-model modules |

**New model architectures (v4-exclusive):** GPT-OSS, Chatterbox, GraniteMoeHybrid, LFM2-MoE, HunYuanDenseV1, Apertus, Olmo3, FalconH1, Youtu-LLM. Also: Mamba (state-space models), Multi-head Latent Attention, Mixture of Experts.

**No breaking API changes** from v3 to v4. The `pipeline()` API, `device` option, `dtype` option, and model loading patterns are the same. The upgrade is additive.

### 5.4 The pipeline API

The `pipeline()` function is the primary interface. It abstracts away tokenization, model loading, tensor management, and output decoding behind a single function call:

```javascript
import { pipeline } from '@huggingface/transformers';

// Create a pipeline for a specific task
const classifier = await pipeline(
  'sentiment-analysis',                                    // task name
  'Xenova/distilbert-base-uncased-finetuned-sst-2-english', // model ID on HF Hub
  { device: 'wasm' }                                       // options
);

// Run inference
const result = await classifier('The sky was impossibly blue tonight.');
// → [{ label: 'POSITIVE', score: 0.9997 }]
```

**What happens when you call `pipeline()`:**

1. The library fetches the model's `config.json`, `tokenizer.json`, `tokenizer_config.json`, and ONNX model file(s) from `https://huggingface.co/{model_id}/resolve/main/...`.
2. These files are cached in the browser's Cache API. Subsequent page loads skip the download.
3. The ONNX model is loaded into ONNX Runtime Web.
4. The pipeline returns an async callable function.

**What happens when you call the returned function:**

1. The input text is tokenized (split into subword tokens, converted to integer IDs, padded/truncated to the model's expected length).
2. The token IDs are packed into tensors and passed to ONNX Runtime.
3. ONNX Runtime runs the forward pass (through WASM or WebGPU).
4. The output logits are post-processed (softmax for classification, decoding for generation, etc.).
5. The result is returned as a JavaScript object.

### 5.5 Supported tasks

Transformers.js supports ~30 pipeline tasks across text, vision, audio, and multimodal:

**Text:** `text-classification` / `sentiment-analysis`, `token-classification` / `ner`, `question-answering`, `fill-mask`, `summarization`, `translation`, `text-generation`, `text2text-generation`, `zero-shot-classification`, `feature-extraction`

**Vision:** `image-classification`, `object-detection`, `image-segmentation`, `depth-estimation`, `image-to-image`, `background-removal`, `image-feature-extraction`

**Audio:** `automatic-speech-recognition`, `audio-classification`, `text-to-speech` / `text-to-audio`

**Multimodal:** `document-question-answering`, `image-to-text`, `zero-shot-image-classification`, `zero-shot-audio-classification`, `zero-shot-object-detection`

**Not supported:** text-to-image (Stable Diffusion), visual question answering, table QA, mask generation, video classification, audio-to-audio. Training/fine-tuning is not supported — Transformers.js is inference-only.

### 5.6 Loading from CDN (no build step)

Transformers.js can be loaded directly from a CDN as an ES module. No npm, no bundler, no build step:

```html
<script type="module">
  import { pipeline } from 'https://cdn.jsdelivr.net/npm/@huggingface/transformers@4.0.1';
</script>
```

Or via dynamic import (what this app uses):

```javascript
const { pipeline } = await import(
  'https://cdn.jsdelivr.net/npm/@huggingface/transformers@4.0.1'
);
```

The dynamic import approach lets us show a loading state while the library downloads, and catch import errors gracefully.

The jsDelivr CDN serves the package's `module` entry point, which points to the browser-optimized `transformers.web.js` build (53% smaller than the full bundle in v4).

### 5.7 The model: DistilBERT for sentiment analysis

This app uses `Xenova/distilbert-base-uncased-finetuned-sst-2-english`, a pre-converted ONNX version of the DistilBERT model fine-tuned on the Stanford Sentiment Treebank (SST-2).

**Model details:**
- **Architecture:** DistilBERT (a distilled version of BERT — 40% smaller, 60% faster, retains 97% of BERT's accuracy)
- **Parameters:** ~67M
- **Download size:** ~67MB (ONNX, fp32)
- **Task:** Binary sentiment classification (POSITIVE / NEGATIVE)
- **Training data:** SST-2 (Stanford Sentiment Treebank v2 — ~67K movie review sentences)
- **Output:** `{ label: 'POSITIVE' | 'NEGATIVE', score: 0.0–1.0 }`

**Why this model:**
- Small enough to download in a browser without frustrating the user
- Fast enough to run on CPU (WASM) in under 100ms per inference
- Well-tested with Transformers.js (it's the default model for the `sentiment-analysis` pipeline)
- Binary classification is simple to display (a label + a confidence bar)

### 5.8 The ModelRegistry API (v4)

New in v4, `ModelRegistry` provides programmatic access to model metadata and caching:

```javascript
import { ModelRegistry } from '@huggingface/transformers';

// List files needed for a pipeline
const files = await ModelRegistry.get_pipeline_files(
  'sentiment-analysis', modelId, { dtype: 'fp32' }
);

// Check total download size
const metadata = await Promise.all(
  files.map(f => ModelRegistry.get_file_metadata(modelId, f))
);
const totalBytes = metadata.reduce((sum, m) => sum + m.size, 0);

// Check if already cached
const cached = await ModelRegistry.is_pipeline_cached(
  'sentiment-analysis', modelId, { dtype: 'fp32' }
);

// Get available quantization levels
const dtypes = await ModelRegistry.get_available_dtypes(modelId);
// → ['fp32', 'fp16', 'q8', 'q4', 'q4f16']

// Clear cache
await ModelRegistry.clear_pipeline_cache(
  'sentiment-analysis', modelId, { dtype: 'fp32' }
);
```

This app does not currently use `ModelRegistry`, but it would be useful for showing download progress or offering a "clear cache" button.

### 5.9 Quantization (dtype)

Models can be loaded at different precision levels to trade accuracy for size/speed:

| dtype | Bits per weight | Relative size | Use case |
|-------|----------------|---------------|----------|
| `fp32` | 32 | 1.0x (baseline) | Maximum accuracy |
| `fp16` | 16 | 0.5x | Good default for WebGPU |
| `q8` | 8 | 0.25x | Default for WASM |
| `q4` | 4 | 0.125x | Mobile / constrained devices |
| `q4f16` | 4 (weights) + 16 (compute) | ~0.15x | Best for large models on WebGPU |

Specified in pipeline options:

```javascript
const pipe = await pipeline('sentiment-analysis', modelId, {
  dtype: 'q8',      // 8-bit quantized
  device: 'webgpu'  // GPU acceleration
});
```

For encoder-decoder models (like Whisper, T5), you can specify different dtypes per module:

```javascript
const pipe = await pipeline('automatic-speech-recognition', 'Xenova/whisper-small', {
  dtype: {
    encoder_model: 'fp16',
    decoder_model_merged: 'q4'
  }
});
```

### 5.10 Environment configuration

```javascript
import { env, LogLevel } from '@huggingface/transformers';

// Cache WASM runtime files for offline use
env.useWasmCache = true;

// Silence ONNX Runtime warnings (hidden by default in v4)
env.logLevel = LogLevel.WARNING;

// Custom fetch for authenticated model access
env.fetch = (url, options) => fetch(url, {
  ...options,
  headers: { ...options?.headers, Authorization: `Bearer ${token}` }
});

// In browser context, skip local model path checks
env.allowLocalModels = false;
```

### 5.11 Browser compatibility

| Browser | WASM | WebGPU |
|---------|------|--------|
| Chrome 113+ | Yes | Yes |
| Edge 113+ | Yes | Yes |
| Firefox 89+ | Yes | 141+ (behind `dom.webgpu.enabled` flag) |
| Safari 15.2+ | Yes | 26+ (behind feature flag) |
| Mobile Chrome | Yes | Limited |
| Mobile Safari | Yes | No |

This app uses `device: 'wasm'` (the default) for maximum compatibility. WebGPU would be faster for larger models but is not needed for DistilBERT's ~100ms inference time.

### 5.12 Performance characteristics

**Model download (first visit):** ~67MB for DistilBERT fp32. Takes 5–30 seconds depending on connection. Cached in browser Cache API after first download.

**Library download:** ~250KB gzipped from jsDelivr. Cached by browser HTTP cache.

**WASM runtime download:** ~5MB for ONNX Runtime WASM files. Cached if `env.useWasmCache = true`.

**Inference time (DistilBERT, WASM, CPU):** 50–200ms per sentence, depending on device.

**Inference time (DistilBERT, WebGPU, GPU):** 10–50ms per sentence.

**Memory:** ~150MB peak during DistilBERT inference (model weights + WASM heap + intermediate tensors).

### 5.13 Limitations and gotchas

1. **Main thread blocking.** Model inference blocks the main thread, causing UI jank. For production apps with larger models, run inference in a Web Worker. This app tolerates the brief freeze because DistilBERT is fast (~100ms).

2. **First-load penalty.** The 67MB model download is unavoidable on first visit. There is no way to ship the model with the static site (GitHub Pages has a 100MB repo size limit and model files would bloat the repo). The model must come from Hugging Face Hub.

3. **No offline without priming.** The app requires network access on first visit to download both the twilight data and the model. After that, the model is cached, but the twilight data is fetched fresh each visit (it changes daily).

4. **Inference only.** You cannot fine-tune or train models with Transformers.js. If you want a model that classifies "blue hour quality" instead of generic sentiment, you'd need to fine-tune in Python, convert to ONNX, and host on HF Hub.

5. **ONNX format required.** Not all Hugging Face models have ONNX weights. Look for models tagged `transformers.js` on the Hub, or convert your own using Hugging Face Optimum.

6. **CORS.** The browser must be able to reach `huggingface.co` to download models. Some corporate networks block this.

---

## 6. The journal and sentiment analysis feature

### 6.1 User flow

1. The user types a short journal entry into a `<textarea>`: *"The light was extraordinary tonight — that deep cobalt that makes everything look like a Vermeer."*

2. They click "Analyze Mood."

3. The app passes the text to the DistilBERT classifier, which returns `{ label: 'POSITIVE', score: 0.9834 }`.

4. The app displays:
   - The label ("mood: positive")
   - A poetic interpretation ("a good evening for the blue light" for positive, "even the difficult evenings have their own color" for negative)
   - A confidence bar (98% filled, colored with a blue-to-gold gradient for positive or a warm-to-dark gradient for negative)

### 6.2 Model loading strategy

The model loads immediately on page load (in parallel with the twilight API call). The "Analyze Mood" button is disabled until loading completes. A status message below the textarea tracks progress:

- `"Sentiment model not loaded"` → initial state
- `"Loading sentiment model…"` → library and model downloading
- `"Model ready"` → inference available, button enabled
- `"Model error: {message}"` → something went wrong

The model is loaded eagerly (not lazily on first click) because:
- The download takes several seconds; starting it immediately hides the latency behind the time the user spends reading the page.
- The library import itself is ~250KB, which is fast.
- The model weights (~67MB) are the bottleneck, and they cache after first download.

### 6.3 Why WASM instead of WebGPU

The pipeline is created with `{ device: 'wasm' }` explicitly. Reasons:

1. **Compatibility.** WASM works in all browsers. WebGPU doesn't.
2. **Overkill for this model.** DistilBERT has 67M parameters and runs a forward pass in ~100ms on WASM. WebGPU's advantage is most visible with large models (1B+ parameters).
3. **Simpler error handling.** WebGPU can fail to initialize for many reasons (driver issues, browser flags, GPU memory). WASM just works.

For future features with larger models (image classification, text generation), switching to WebGPU with a WASM fallback would be appropriate:

```javascript
const device = navigator.gpu ? 'webgpu' : 'wasm';
const pipe = await pipeline('image-classification', modelId, { device });
```

---

## 7. The visual design

### 7.1 Design philosophy

The app is designed to feel like the blue hour itself — quiet, deep, contemplative. The visual language borrows from the astronomical phenomenon:

- **Background gradient** simulates a twilight sky, moving from deep navy (top/zenith) through blues to a warm gold-orange at the bottom (horizon).
- **Glassmorphism cards** (`backdrop-filter: blur()` + translucent background + subtle border) float over the sky like windows.
- **Typography** uses Georgia (serif) for body text — literary, unhurried — and system sans-serif for UI labels and stats.
- **Color palette** is derived from the actual spectral characteristics of twilight light.

### 7.2 CSS architecture

All styles are inline in `<style>` within `index.html`. No external CSS files. CSS custom properties (variables) define the palette:

```css
--deep-blue: #0a1628;      /* Near-black navy — deep nautical twilight */
--twilight-blue: #1a3a5c;  /* Dark blue — mid nautical twilight */
--sky-blue: #2e6b9e;       /* Medium blue — early nautical twilight */
--light-blue: #5b9bd5;     /* Steel blue — late civil twilight */
--pale-gold: #f0d68a;      /* Warm gold — accent, walk times */
--warm-white: #f5f0e8;     /* Off-white — text */
--card-bg: rgba(26, 58, 92, 0.35);     /* Translucent card background */
--glass-border: rgba(91, 155, 213, 0.3); /* Subtle card border */
```

### 7.3 Responsive behavior

A single `@media (max-width: 480px)` breakpoint reduces the title font size, walk time font size, and card padding for mobile screens. The layout is inherently responsive (flexbox column, max-width cards) and doesn't need complex breakpoints.

---

## 8. File structure

```
bluest-hour/
├── index.html          # The entire application (HTML + CSS + JS)
├── README.md           # Project overview, architecture notes, future directions
└── SPECIFICATION.md    # This file
```

That's it. No other files. No build artifacts. No configuration files.

---

## 9. Data flow diagram

```
Page Load
    │
    ├─── fetch(sunrise-sunset API) ──→ JSON with civil_twilight_end, nautical_twilight_end
    │       │
    │       └─── Calculate walk time ──→ Render walk card (time, timeline, stats)
    │
    └─── import(@huggingface/transformers from CDN) ──→ Library loaded
            │
            └─── pipeline('sentiment-analysis', model) ──→ Model downloaded from HF Hub
                    │                                       (cached in browser Cache API)
                    └─── Button enabled ("Model ready")

User clicks "Analyze Mood"
    │
    └─── classifier(journal text) ──→ Tokenize → ONNX Runtime (WASM) → Softmax
            │
            └─── { label: 'POSITIVE', score: 0.98 } ──→ Render sentiment result
```

---

## 10. How to run locally

```bash
cd bluest-hour
python -m http.server 8000
# or: npx serve .
```

Open `http://localhost:8000`. A local server is needed because ES module imports (`type="module"`) don't work with `file://` URLs due to CORS restrictions.

---

## 11. How to deploy

The site is deployed automatically by GitHub Pages. Any push to `main` triggers a rebuild (which is instant — there's nothing to build, just files to serve).

To enable GitHub Pages on a new repo:
1. Go to **Settings → Pages**
2. Set **Source** to "Deploy from a branch"
3. Set **Branch** to `main`, folder to `/ (root)`
4. Save

The site appears at `https://{username}.github.io/{repo-name}/` within a few minutes.

---

## 12. Future directions

The README outlines three potential expansions, all enabled by Transformers.js v4's browser-native inference:

### 12.1 Sky photo classification

Use `image-classification` pipeline with a ViT model to classify photos of the sky taken during the walk. The user would upload or capture a photo, and the model would identify the twilight phase or dominant color temperature.

```javascript
const classifier = await pipeline(
  'image-classification',
  'google/vit-base-patch16-224'
);
const result = await classifier(imageBlob);
```

This would require WebGPU for acceptable performance (ViT is larger than DistilBERT).

### 12.2 Poetic text generation

Use `text-generation` pipeline with a small language model to generate Didion-style reflections on the evening's conditions, seeded by the twilight data.

```javascript
const generator = await pipeline(
  'text-generation',
  'onnx-community/Qwen2.5-0.5B-Instruct',
  { dtype: 'q4', device: 'webgpu' }
);
const output = await generator(
  `The blue hour tonight lasted ${minutes} minutes. Write a reflection:`,
  { max_new_tokens: 100 }
);
```

This would require a Web Worker to avoid blocking the UI during generation.

### 12.3 Seasonal mood tracking

Store journal entries and sentiment scores in `localStorage` across visits. Visualize how the user's mood correlates with blue hour duration across the year — do longer blue hours (summer) correspond to more positive entries?

This requires no additional ML models, just persistence and a chart library (or hand-drawn SVG).

---

## 13. Course context

This project is part of the [Level 2: Applied Machine Learning Concepts](https://github.com/buildLittleWorlds/level-2-course-material) curriculum, which teaches students to build and deploy Hugging Face Spaces. The dual deployment (a Gradio app on Hugging Face Spaces and a static app on GitHub Pages) demonstrates:

1. **Server-side vs. client-side ML.** The HF Space runs Python on a server; the GitHub Pages version runs JavaScript in the browser. Same idea, completely different execution models.
2. **Transformers.js v4 as a bridge.** The library makes it possible to bring Hugging Face models to static sites — no server, no API key, no backend infrastructure.
3. **Progressive enhancement.** The app works without ML (the walk calculation is pure math). The sentiment analysis is a layer on top — if it fails to load, the core experience is unaffected.

---

## 14. Transformers.js v4 quick reference for AI assistants

If you are an AI assistant working on this project and need to modify or extend the Transformers.js integration, here is what you need to know:

**CDN import (no build step):**
```javascript
import { pipeline, env, LogLevel } from 'https://cdn.jsdelivr.net/npm/@huggingface/transformers@4.0.1';
```

**Create a pipeline:**
```javascript
const pipe = await pipeline(taskName, modelId, { device, dtype, progress_callback });
```

**Run inference:**
```javascript
const result = await pipe(input);          // text, image URL, audio buffer, etc.
const result = await pipe(input, options); // max_new_tokens, top_k, etc.
```

**Device options:** `'wasm'` (default, CPU, universal), `'webgpu'` (GPU, faster, limited browser support).

**Find compatible models:** https://huggingface.co/models?library=transformers.js

**Model naming convention:** ONNX models are typically under `Xenova/` or `onnx-community/` on HF Hub.

**The library is inference-only.** No training, no fine-tuning, no gradient computation.

**Always consider main-thread blocking.** For models larger than DistilBERT, use a Web Worker.

**Cache behavior:** Models are cached in the browser Cache API after first download. Use `ModelRegistry.is_pipeline_cached()` to check, `ModelRegistry.clear_pipeline_cache()` to clear.
