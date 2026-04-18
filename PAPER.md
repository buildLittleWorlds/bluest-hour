# The Bluest Hour

## 1. What I Built

I built a place-based app for Godfrey, Illinois that tells the user when to start an evening walk so the middle of the walk lands in the deepest part of twilight. The project exists in two versions: a Gradio Space in `bluest-hour-space/` and a static Pages version in `bluest-hour/`. In the version I am writing about here, the app does three main things: it calculates a blue-hour walk time, adjusts that timing using a local offset based on observation, and lets the user write a short note after the walk.

The most important design choice came out of Weeks 2 and 3 of the journal. Instead of telling the user to leave at sunset or exactly at civil twilight end, the app centers a **20-minute walk** around the bluest part of evening light. It also uses a **local offset** because the mathematically correct midpoint was not the same as the moment that actually looked bluest from the ground in Godfrey.

## 2. My Research Question

My research question came out of Weeks 4 through 6:

**When people write short reflections after blue-hour-timed walks, does a generic positive/negative sentiment model actually capture their mood, or does it flatten a more mixed reflective style of writing?**

I chose this question because it grows directly out of something I observed in the app instead of something I invented after the fact. The journaling feature made the project more interesting, but it also exposed a problem: the writing the app encourages is often calm, mixed, or observational, and that does not fit neatly into positive versus negative.

## 3. Why This Matters to Me

I did not want this project to stop at "a nice twilight calculator." The whole point of the app is that the walk is supposed to change how the evening feels. If I add a journaling layer, then I am making a claim that the reflection matters too.

That is why the model question matters. If the app invites reflective writing but the AI layer can only flatten that writing into simple mood labels, then the most interesting part of the project is not the label itself. It is the mismatch between the kind of writing the app creates and the kind of text the model was trained to read.

## 4. What I Tried

The project changed shape over several weeks:

- In **Week 1**, I compared sunset times to actual evening walks and realized sunset was too early.
- In **Week 2**, I built the first predictor and compared three walk rules. The centered 20-minute walk worked better than a simple "leave at sunset" or "leave at civil twilight end" rule.
- In **Week 3**, I tested the prediction outside and added a local offset because the best-looking blue in Godfrey arrived earlier than the astronomical midpoint.
- In **Week 4**, I added a journal box and a DistilBERT SST-2 sentiment baseline in the browser.
- In **Week 5**, I compared the binary baseline to richer emotion-label ideas and noticed that reflective entries kept getting flattened.

A small example from the journal shows the issue:

| Entry | What I meant | What the binary model tended to do |
|---|---|---|
| "The sky was impossibly blue tonight." | clear enjoyment | worked reasonably well |
| "It was beautiful in a way that made me a little sad." | mixed feeling | pushed it toward positive |
| "Everything felt still, but not exactly peaceful." | quiet, uncertain mood | pulled it toward negative |

This was enough to move the project from interface design into a real research question.

## 5. What I Learned

I learned three things from the journal.

First, **the timing part of the app had to become local before it became believable**. The clean astronomical midpoint was useful, but it was not the best answer for a real place with a real horizon.

Second, **the journaling feature made the project better**, because it gave the app something human to pay attention to instead of ending at a single recommended time.

Third, **the simple sentiment model was too narrow for the writing the app creates**. It was good enough as a baseline, but it kept turning reflective, mixed, or quiet writing into a simpler emotional story than the writing itself supported. That is the strongest finding in the project so far.

## 6. What Still Needs Work / Who It Might Fail For

This project still has real limitations.

- The local offset is based on observation in Godfrey, so the timing is not automatically portable to other places.
- The journal evidence is still small. I do not yet have enough real entries to make strong claims.
- The binary sentiment model is a fit problem. It was trained for ordinary positive/negative text, not for quiet field notes about evening light.
- The app may read different writers unevenly. Someone who writes directly, metaphorically, or in a different emotional register may get labels that feel even less accurate than mine did.

So the honest claim is not "I built a mood detector." The honest claim is "I built a useful baseline, and the baseline revealed a better question."

## 7. Sources I Would Cite Next

These are the source areas that grew out of Week 6 of the journal:

- **Research on nature exposure and mood:** this would help me explain why the walk itself might matter.
- **Research on reflective writing or journaling:** this would help me explain what kind of text the app produces.
- **Research on sentiment analysis for diary-like text:** this would help me evaluate whether the model is a good fit.
- **Model documentation for DistilBERT SST-2 and any richer emotion model:** this would help me describe the baseline honestly.

---

This paper comes directly from the journal:

- **What I built** ← Weeks 2–3
- **My research question** ← Week 6
- **What I tried** ← Weeks 4–5
- **What I learned** ← Weeks 5–7
- **What still needs work** ← Week 7
