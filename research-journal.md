# Bluest Hour Research Journal

AI + Research Level 2 — exemplar journal built from the real Bluest Hour project history

---

## Week 1 — Sunset Is Not Enough

### This Week's Method
Comparative analysis.

### How I Applied It
Before building anything, I tested the same basic question with three kinds of tools I already had: a weather app, a sunset-time website, and my own eyes. The question was simple: if I want to go outside for the deepest blue part of evening light, is "sunset" actually the right time to leave the house?

I wrote down four quick observations from ordinary walks in Godfrey. On two evenings I left right at sunset. On one evening I left about fifteen minutes later. On one evening I waited until the sky already looked dark. I also looked up the difference between sunset, civil twilight, and nautical twilight because I realized I was using those words loosely.

### What I Expected
I expected sunset to be close enough. I thought the "blue hour" was probably just a poetic name for the period right after sunset.

### What I Found
Sunset was too early. When I left right at sunset, the light was still warm and mixed with orange. The really saturated blue came later, after the sun was already below the horizon. Waiting too long was also wrong, because by then the blue had thinned into a flatter gray-blue and the walk felt late instead of timed.

The useful distinction turned out to be between **sunset** and **twilight phases**. Once I looked up civil and nautical twilight, the timing problem stopped feeling fuzzy. The blue I cared about seemed to live somewhere after civil twilight ended, not at sunset itself.

### Why I Think This Happened
I was starting with the wrong public signal. Sunset is easy to find, but it is not the same thing as "best blue light." The visual effect I care about depends on where the sun is below the horizon, not just on whether it has crossed it.

### Limitations
This was only a few evenings and only my own observation. I was not timing anything carefully yet, and weather made a big difference. I also did not have an app yet, just notes.

### What I Want to Try Next
I want to build the simplest possible predictor: take twilight data for Godfrey, find the blue-hour window, and recommend when to start a walk so I am outside during the middle of it.

---

## Week 2 — First Predictor, First Real Design Choice

### This Week's Method
Prototype comparison.

### How I Applied It
I built the first working version of **Bluest Hour** as a Gradio Space in `bluest-hour-space/`. The app pulled evening twilight data for Godfrey and returned a suggested walk time.

The real decision this week was not "can I make the app run?" It was **how should the app define the walk?** I tested three versions on paper and in the UI:

1. leave at sunset
2. leave when civil twilight ends
3. center a 20-minute walk on the midpoint between civil twilight end and nautical twilight end

I also chose a walk length. I tried thinking through 10, 20, and 30 minutes.

### What I Expected
I expected "leave at civil twilight end" to win because that felt like a clean astronomical rule and a simpler instruction than doing midpoint math.

### What I Found
The centered 20-minute walk was better. A start-time rule felt too sharp. The centered walk gave the user time to get outside, settle into the walk, and actually be walking during the deepest part of the blue window instead of racing toward it.

The 10-minute walk felt too short to feel like a walk. The 30-minute walk was too long because it stretched outside the window on shorter evenings. Twenty minutes was the size that felt realistic and repeatable.

### Why I Think This Happened
The app is not just a clock. It is trying to shape an experience. Centering the walk on the middle of the blue-hour window is more human than telling someone to leave at a single exact minute. It turns the project from "twilight calculator" into "small evening ritual."

### Limitations
This was still mostly a design judgment. I had a working Space, but I had not yet field-tested it across enough real evenings to know whether the midpoint actually matched what looked bluest from the ground.

### What I Want to Try Next
Use the app outside for real walks and keep notes on whether the predicted midpoint matches the moment the light actually looks deepest from Godfrey.

---

## Week 3 — The Local Offset Problem

### This Week's Method
Observation log and calibration.

### How I Applied It
I used the first working predictor outside and kept a small observation sheet on several evenings. I wrote down:

- the app's predicted bluest moment
- when the light actually looked right to me
- what the horizon looked like that night
- whether haze, trees, or terrain seemed to matter

I also updated the code twice this week. One change was a timing adjustment. The second was the bigger one: adding a **local offset** so the app could recommend a time earlier than the pure astronomical midpoint.

### What I Expected
I expected the astronomical midpoint to be close, maybe off by a few minutes.

### What I Found
It was not just a little off. From Godfrey, the best-looking blue consistently arrived earlier than the clean midpoint suggested. The western horizon is not an ideal flat line. There are bluffs, trees, buildings, and atmospheric conditions that make the light feel like it turns sooner.

By the end of the week I settled on a local offset of about **35 minutes earlier** than the astronomical midpoint. That was the first number in the project that felt clearly like a personal research judgment rather than a borrowed constant.

### Why I Think This Happened
The astronomy is right for an ideal horizon. The walk is happening in a real place. Godfrey is not a geometry problem; it is a place with relief, trees, and local light conditions. The app got better as soon as I stopped pretending those things were noise.

### Limitations
This is still my own calibration, not a formal measurement study. Weather was not controlled. I also do not know if the same offset would hold across the whole year.

### What I Want to Try Next
Now that the timing feels better, I want to add a reflection layer. If the point of the walk is not just to catch the light but to notice something about it, the app should make room for that.

---

## Week 4 — Adding the Journal, Testing a Baseline

### This Week's Method
Interface iteration with a baseline model.

### How I Applied It
This week I pushed the project further in the static Pages version in `bluest-hour/`. I added a text box so the user could write a short note after the walk, and I connected that note to a very simple sentiment model in the browser using Transformers.js and DistilBERT SST-2.

I tested the journaling feature on a small set of lines that sounded like what someone might actually write after a walk:

- "The sky was impossibly blue tonight."
- "It was beautiful in a way that made me a little sad."
- "Everything felt still, but not exactly peaceful."
- "I thought this would make me calmer than it did."

### What I Expected
I expected the positive/negative model to be limited, but still directionally helpful. I thought it would at least separate clearly warm entries from clearly low ones.

### What I Found
The simple cases were fine. "The sky was impossibly blue tonight" came back strongly positive, which made sense. But the mixed entries were much less satisfying. "Beautiful in a way that made me a little sad" still landed as basically positive. "Still, but not exactly peaceful" got pulled toward negative even though that is not really what the line means.

The app now technically had an AI journaling layer, but the layer felt like a blunt instrument.

### Why I Think This Happened
SST-2 is trained for ordinary positive/negative sentiment, not for reflective writing about light, memory, calmness, ambiguity, or evening mood. It can classify obvious tone, but it flattens the more interesting writing into a binary choice.

### Limitations
My test set was tiny and mostly my own writing. I also had not compared the baseline to any richer emotion models yet, so I only knew that the binary model felt coarse, not whether a better alternative would actually help.

### What I Want to Try Next
Compare the binary baseline against richer emotion models and see whether the problem is "this model is too simple" or "this kind of writing is just hard to classify cleanly at all."

---

## Week 5 — Reflective Writing Gets Flattened

### This Week's Method
Comparative analysis.

### How I Applied It
I kept the same basic journal-entry test set and compared the current binary baseline against richer emotion-label approaches in notes and scratch testing. I was especially interested in entries that mixed calmness, strangeness, beauty, and slight sadness, because those are exactly the kinds of lines the app invites.

The key question this week was not "which model is smartest?" It was "does a wider vocabulary help the app describe the feeling more honestly?"

### What I Expected
I expected the richer models to be messier. My guess was that a wider set of labels would produce more noise without really solving the problem.

### What I Found
The richer labels were messy, but they were also more believable. The binary baseline kept forcing mixed reflective writing into positive or negative. The wider emotion outputs at least left room for combinations like calm + curiosity, or beauty + sadness, or a mostly neutral note with a little nostalgia.

That did not mean the richer models were "correct." It meant the binary baseline was often obviously too small for the writing style the app creates.

### Why I Think This Happened
The project does not produce review text. It produces small field notes. A field note about evening light is often observational before it is emotional. When emotion does appear, it is often mixed. The positive/negative frame is not wrong so much as too narrow.

### Limitations
This was still not a formal evaluation. I was comparing outputs and judging them qualitatively. I also had not yet looked for published research on nature exposure, reflective writing, or mood tracking to see whether my hunch lined up with a real research area.

### What I Want to Try Next
Turn the hunch into a research question and do a real literature search. If this is going to become more than a nice interface critique, I need to know what nearby researchers actually study.

---

## Week 6 — From Hunch to Research Question

### This Week's Method
Question sharpening and source search.

### How I Applied It
I used the Session 6 broad → medium → narrow question structure and ran a Consensus-style source search around four neighborhoods:

- nature exposure and mood
- evening light / timing / wellbeing
- sentiment analysis on reflective writing
- journaling as a way of measuring feeling over time

I also compared two possible project questions:

1. Does a blue-hour-timed walk change how people feel?
2. When people write reflections after blue-hour-timed walks, does a generic sentiment model capture those reflections or flatten them?

### What I Expected
I expected to find very little on the exact phrase "blue hour," but I hoped there would be nearby literature on nature, mood, and reflective writing.

### What I Found
That is exactly what happened. I did not find a neat literature on "blue hour apps," but I did find that the project clearly lives near real research conversations about nature exposure, positive affect, journaling, and text-based mood measurement.

More importantly, the second question was much better than the first. "Does the walk help?" is interesting, but it is broad and hard to test well in a small class project. "Does the model flatten the writing?" is narrower, directly connected to the existing app, and grounded in something I already observed in Weeks 4 and 5.

The question I wrote down at the end of the week was:

> When people write short reflections after blue-hour-timed walks, does a generic positive/negative sentiment model actually capture their mood, or does it flatten a more mixed reflective style of writing?

### Why I Think This Happened
The project became research as soon as I stopped treating the model as decoration. The interesting thing is not just that the app has an AI layer. The interesting thing is whether the AI layer is valid for the kind of writing the app creates.

### Limitations
I still do not have a large sample of real entries or multiple human raters. My literature search is still early and mostly about finding the right neighborhood.

### What I Want to Try Next
Write the simple paper from the journal evidence I already have, then decide what the next version of the project should be: better storage, better emotion labeling, a richer interface, or all three.

---

## Week 7 — Limits, Fit, and the Next Version

### This Week's Method
Limitation audit and next-version planning.

### How I Applied It
I reviewed the project as a full chain instead of as isolated features:

- timing prediction
- local offset
- journaling box
- binary sentiment model
- research question

I also looked at the newer almanac-style redesign as a possible next step, not as a replacement for the simpler project. That helped me separate what belongs in the first paper from what belongs in a later, more ambitious version.

### What I Expected
I expected the main limitation to be the sentiment model.

### What I Found
The model is a limitation, but it is not the only one.

1. **The timing is Godfrey-specific.** The local offset is the product of observation in one place.
2. **The journal data is tiny.** I do not yet have enough real entries to say much quantitatively.
3. **The sentiment model is a fit problem.** It was trained for ordinary positive/negative text, not reflective field notes.
4. **The writing itself is culturally and personally specific.** A model that reads my kind of quiet, mixed writing one way might read someone else's very differently.
5. **The project is stronger as a journal-to-paper story than as a "finished product."** The real lesson is how the build created a question.

### Why I Think This Happened
The project matured by getting more honest. The simple version does not need to solve everything. It needs to show a believable research path: build a tool, notice a mismatch, ask a question, document the limits, then design the next version on purpose.

That is also why the simpler paper should come first. The almanac redesign and the richer emotion model are valuable, but they make more sense once the journal has already shown the problem the redesign is responding to.

### Limitations
This journal is still a staged exemplar, not a full longitudinal study. The next version also risks becoming too ornate if I forget what the actual question is.

### What I Want to Try Next
Use this journal as the spine for a short paper in plain language, then treat the almanac redesign as the advanced continuation: same project, clearer question, more ambitious form.



Here’s a clean markdown journal entry that captures the problem clearly and analytically:

```markdown
# Journal Entry: Persistence Problem in Bluest Hour Space

## Date
[Insert date]

## Observation

I attempted to use the Bluest Hour Hugging Face Space as a way to log observations from my evening walks. The idea was to enter notes after each walk—ideally building a cumulative record tied to specific dates and predicted “bluest moments.”

However, I am encountering a fundamental problem: **none of my entries persist between sessions**.

Each time I restart or reload the Space, any observational data I previously entered is gone. The interface resets completely, as if no prior interaction ever occurred.

## Initial Interpretation

This suggests that the Space is functioning as a **stateless application**. It calculates and renders results dynamically (based on API calls and input date), but it does not store any user-generated data.

In other words:
- Inputs are processed in real time
- Outputs are rendered immediately
- But **no data is written to disk, memory, or an external database in a persistent way**

## Hypothesis

The likely cause is that the Space:
1. Does not include any backend storage mechanism (e.g., file writing, database, or API endpoint)
2. Runs in an environment where **local state is ephemeral** (i.e., resets on restart or rebuild)
3. Uses Gradio purely for interface rendering, not for data persistence

This aligns with how many Hugging Face Spaces operate by default: they are designed for demonstration and computation, not long-term state retention.

## Consequence for My Experiment

This is a blocking issue for my intended use.

My experiment depends on:
- Repeated observations over multiple days
- The ability to compare predicted vs. experienced “bluest moments”
- Accumulation of qualitative notes over time

Without persistence, the Space cannot function as a logging tool. It can only function as a **prediction tool**.

## Emerging Question

What is the minimal architectural change needed to turn this from a stateless predictor into a persistent observational system?

Possible directions to explore:
- Writing entries to a local file (if the environment allows persistence)
- Using an external storage solution (e.g., Google Sheets, a simple API, or a lightweight database)
- Downloading entries manually after each session (least desirable)

## Reflection

There is an interesting conceptual gap here: the Space is already framed as something grounded in **ongoing local observation** (e.g., the 35-minute offset), yet it does not itself support the continued accumulation of those observations.

In a sense, the epistemology of the project (iterative, observational, local) is not yet matched by its technical infrastructure (stateless, session-bound, non-retentive).

That mismatch is now the central design problem.
```





