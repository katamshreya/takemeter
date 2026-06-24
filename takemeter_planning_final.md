# TakeMeter — Planning Document

---

## Community

I chose the online discussion community around the 2026 horror film *Obsession* (dir. Curry Barker), drawing from **r/obsessionmovie** — a dedicated subreddit with over 400K weekly visitors that launched the same week the film released in theaters (May 15, 2026). This community is a strong fit for a classification task because the film is brand new and culturally buzzy — people are actively posting reactions, arguments, and hot takes in real time, which means the discourse is varied in quality and tone. Some posts engage seriously with the film's themes (toxic relationships, supernatural allegory, Gen Z psychology), while others are pure emotional outbursts or unsupported bold claims, making the distinctions between labels meaningful and visible.

---

## Label Taxonomy

### Label 1: `analysis`
**Definition:** The post makes a structured argument about the film using specific evidence — references to cinematography, directing choices, thematic content, character decisions, specific dialogue, or comparisons to other films. The claim could be evaluated or challenged by someone who also saw the movie.

**Example 1:**
> "The night scene where Nikki stands in the corner watching Bear sleep is the best shot in the film. Clemon's decision to keep her face barely visible forces the audience to fill in the horror themselves — it's the same technique Hereditary uses in the attic scene. The ambiguity is the point."

**Example 2:**
> "I noticed on rewatch he says 'What would be...what's so bad about being with me' — it's even darker as it shows he is aware he hasn't been with the Real Nikki, and never will be because of his actions."

---

### Label 2: `reaction`
**Definition:** An immediate emotional response to the film or a specific scene. Little to no argument — the post is expressing a feeling, not making a claim. Usually written in first person and tied to a specific moment or general viewing experience.

**Example 1:**
> "I was NOT okay after the parking lot scene. I had to pause and just sit there for a minute. The way she just smiles?? I cannot."

**Example 2:**
> "I watched this movie two weeks ago and that scene has been living in my head rent free every night since then. I never get this from any horror movie but this movie got to me."

---

### Label 3: `hot_take`
**Definition:** A bold, confident opinion stated without supporting evidence or structured argument. The claim might be true or false, but the post asserts rather than argues. Often comparative ("best horror since X"), hyperbolic, deliberately provocative, or a very short verdict.

**Example 1:**
> "Obsession is the best horror movie of the last five years and it's not even close. Nothing else has come close to making me this uncomfortable."

**Example 2:**
> "Bear was the villain as soon as he made the wish."

---

## Hard Edge Cases

**The hardest anticipated boundary: `analysis` vs. `hot_take`**

Some posts cite specific details (a scene, a performance, a budget fact) but use them decoratively rather than as genuine evidence for a structured argument. For example:

> "Curry Barker made this for $750k and it looks better than most $50M studio horror films. It's clearly the best Blumhouse-adjacent project in years."

This cites a real fact (the budget) but uses it as a rhetorical flourish to support a superlative claim, not as part of a reasoned argument.

**Decision rule:** If removing the "evidence" would collapse the entire post into nothing (i.e., the cited detail IS the argument), label it `analysis`. If the post would still make the same bold claim without the detail — if the detail is decorative rather than load-bearing — label it `hot_take`.

**Secondary edge case: `reaction` vs. `hot_take`**

A post like "This is the most disturbing movie I've ever seen" could be emotional reaction OR a bold comparative claim. Decision rule: if the post is about the viewer's personal experience ("I've ever seen"), label it `reaction`. If it's making a claim about the film's objective standing ("most disturbing horror film of 2026"), label it `hot_take`.

---

## Data Collection Plan

**Source:** r/obsessionmovie exclusively — a dedicated subreddit for the film with 400K+ weekly visitors. Posts and comments were collected manually by browsing top posts and their comment sections.

**Actual collection method:** Manual copy-paste from Reddit into a CSV. Comments were collected in batches from individual posts, labeled with Claude's assistance, then reviewed and corrected. Each batch was labeled before moving to the next post.

**Final dataset:** 200 labeled examples
- `analysis`: 95 (55%)
- `reaction`: 39 (23%)
- `hot_take`: 38 (22%)

**What was excluded:** Comments under ~15 words, pure jokes/memes with no film content, image-only posts, non-English comments, and spam/bot content.

---

## Evaluation Metrics

**Primary metric: per-class F1 score**
Accuracy alone is misleading on a 3-class task — a model that always predicts `analysis` could hit 55%+ accuracy if that class dominates. F1 balances precision and recall per class, which tells me whether the model has actually learned each distinction or is just defaulting to the majority class.

**Secondary metric: confusion matrix**
The confusion matrix shows which specific label pairs the model confuses — e.g., does it mix up `analysis` and `hot_take` more than `reaction` and either other class? This is more actionable than overall accuracy for diagnosing label boundary problems.

**Baseline comparison:**
I'll run the zero-shot Groq baseline (llama-3.3-70b-versatile) on the same test set before fine-tuning, using my label definitions as the prompt. This tells me whether fine-tuning added value and by how much.

**Success criteria:** Fine-tuned model achieves overall accuracy ≥ 0.70 and per-class F1 ≥ 0.65 for all three labels on the test set. The fine-tuned model must meaningfully outperform the zero-shot baseline (at least +10 percentage points on overall accuracy). If the baseline already hits 0.80+, that's a signal my labels may be too easy.

---

## Definition of Success

A classifier would be genuinely useful if it could auto-tag posts in a movie discussion community to surface the most substantive discourse (`analysis` posts) for users who want deeper engagement. For that to work, the `analysis` class needs F1 ≥ 0.70 specifically — false positives (tagging `hot_take` as `analysis`) would pollute the feed, and false negatives (missing real analysis) would make the tool feel broken. I'd accept the model as "good enough for a real tool" if: overall accuracy ≥ 0.72, `analysis` F1 ≥ 0.70, and the confusion matrix shows no single off-diagonal cell above 30% of a class's total examples.

---

## AI Tool Plan

**Label stress-testing (before annotation):**
Gave Claude my three label definitions and the `analysis` vs. `hot_take` edge case description and asked it to generate posts that sit at the boundary between those labels. This helped sharpen the decision rule (decorative vs. load-bearing evidence) before annotating.

**Annotation assistance (during data collection):**
Used Claude to pre-label all examples by providing label definitions and raw post/comment text in batches. Claude assigned one label per example with a one-sentence reason. Every pre-label was reviewed personally before accepting — no batch was accepted without reading each post. Pre-labeling is disclosed here in the AI usage section of the README.

**Failure analysis (after fine-tuning):**
After getting wrong predictions from the notebook, will paste them into Claude and ask it to identify common patterns — e.g., short posts, sarcasm, missing explicit evidence. Will verify patterns personally by re-reading examples before writing up evaluation.

---

## Hard Annotation Decisions (updated during labeling)

1. **"This is the scariest movie scene I've ever seen only behind the last 20 minutes of Hereditary."** — Could be `reaction` (personal emotional claim) or `hot_take` (bold comparative claim). Decided `hot_take` because it makes a comparative quality judgment about the film relative to another film, not just a personal feeling.

2. **"Not only does he not warn Sarah that Nikki was potentially dangerous, he actually tries to downplay it by telling Sarah that Nikki's dad has cancer, which he knows at that point isn't even true."** — Could be `hot_take` (verdict about Bear) or `analysis` (specific evidence). Decided `analysis` because it cites multiple specific plot points as load-bearing evidence for the claim, not decoration.

3. **"Bear is objectively evil. He saw her shit, piss, and vomit on herself waiting for him and didn't stop it."** — Could be `analysis` (lists events as evidence) or `hot_take` (bold verdict). Decided `hot_take` because the list of events is used for emotional force rather than as a structured argument — the conclusion ("objectively evil") doesn't logically follow from the evidence in the way an analysis would construct it.
