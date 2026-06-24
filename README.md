# TakeMeter

**AI 201 — Project 3**
A text classifier for discourse quality in r/obsessionmovie, trained to distinguish structured film analysis from hot takes and emotional reactions.

---

## Community Choice

I chose **r/obsessionmovie**, a dedicated subreddit for the 2026 Blumhouse horror film *Obsession* (dir. Curry Barker). The subreddit launched the same week the film released in theaters (May 15, 2026) and grew to 400K+ weekly visitors within weeks, making it one of the fastest-growing film-specific communities on Reddit. I chose it for three reasons:

1. The film is genuinely divisive — it tackles consent, obsession, and "nice guy" psychology in ways that generate wildly different types of discourse, from careful thematic analysis to one-sentence verdicts.
2. The community was brand new, meaning all posts were fresh and uninfluenced by years of established fandom consensus.
3. The film's subject matter produces a natural spectrum of responses: people analyzing the cinematography, people posting hot takes about whether Bear is a villain, and people posting about how they couldn't sleep after watching it.

---

## Label Taxonomy

### `analysis`
The post makes a structured argument about the film using specific evidence — references to scenes, dialogue, directing choices, character decisions, or comparisons to other films. The claim could be evaluated or challenged by someone who also saw the movie. The evidence must be load-bearing: if you removed it, the argument would collapse.

**Example 1:**
> "The necklace pendant changes throughout the movie — it starts as a star before Bear's wish and eventually turns into a B, then a swirl once Nikki starts battling with herself."

**Example 2:**
> "I noticed on rewatch he says 'What would be...what's so bad about being with me' — it's even darker as it shows he is aware he hasn't been with the Real Nikki and never will be because of his actions."

---

### `hot_take`
A bold, confident opinion stated without supporting evidence or structured argument. The post asserts rather than argues. Often a short verdict, hyperbolic comparison, deliberately provocative claim, or a statement that uses evidence decoratively rather than as load-bearing support.

**Example 1:**
> "Bear was the villain as soon as he made the wish."

**Example 2:**
> "This is the most disturbing movie I've seen since Hereditary and it's not even close."

---

### `reaction`
An immediate emotional response to the film or a specific scene. Little to no argument — the post expresses a feeling, usually written in first person, tied to the viewing experience rather than making a claim about the film.

**Example 1:**
> "I watched this movie two weeks ago and that scene has been living in my head rent free every night since then. I never get this from any horror movie but this movie got to me."

**Example 2:**
> "I was NOT okay after the parking lot scene. I had to pause and just sit there for a minute."

---

## Data Collection

**Source:** r/obsessionmovie exclusively. Posts and comments were collected manually by browsing top posts and their comment sections across the subreddit's first month of activity.

**Process:** I copy-pasted comments and post bodies into a CSV in batches. Each batch was pre-labeled using Claude (see AI Usage section), and I reviewed every pre-label before accepting it. No batch was accepted without reading each individual example. Short comments under ~15 words, pure jokes/memes with no film content, image-only posts, and non-English comments were excluded.

**Label distribution:**

| Label | Count | % |
|-------|-------|---|
| analysis | 112 | 56% |
| hot_take | 45 | 22.5% |
| reaction | 43 | 21.5% |
| **Total** | **200** | **100%** |

`analysis` is the majority class because the subreddit attracts many users who want to dig into the film's themes — the film's ambiguity around Bear's culpability and the nature of the wish generates a lot of structured debate.

---

### Difficult-to-Label Examples

**Example 1:**
> "Bear is objectively evil. He saw her shit, piss, and vomit on herself waiting for him and didn't stop it. He saw her smash a bottle in her head and didn't stop it. She literally begged him to kill her and he didn't stop it."

**Decision: `hot_take`**
This lists specific plot events, which looks like evidence. But the list is used for emotional force rather than as a structured argument — the conclusion ("objectively evil") doesn't logically follow from the evidence in the way an analysis post would construct it. Removing the list wouldn't change the verdict. Evidence is decorative, not load-bearing → `hot_take`.

**Example 2:**
> "This is the scariest movie scene I've ever seen only behind the last 20 minutes of Hereditary."

**Decision: `hot_take`**
Could be `reaction` (personal claim) or `hot_take` (comparative quality claim). I decided `hot_take` because it makes a comparative quality judgment about the film relative to another specific film, not expressing a personal emotional experience. It's a bold superlative claim, not a feeling.

**Example 3:**
> "She didn't go crazy until he questioned her at the restaurant, so as long as he stayed a perfect boyfriend to her they would live happily ever after."

**Decision: `hot_take`**
The sarcastic framing ("would live happily ever after") signals this is a sardonic bold claim rather than genuine analysis. Despite referencing a specific scene, the post isn't building an argument — it's making a pointed observation with rhetorical flair. No evidence is load-bearing here.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` — a smaller, faster distilled version of BERT that handles short-to-medium text well and is well-suited for classification tasks.

**Training setup:**
- 3 epochs
- Learning rate: 2e-5
- Batch size: 16 (train), 32 (eval)
- Weight decay: 0.01
- Warmup steps: 50
- Evaluation strategy: per epoch
- Best model loaded at end based on validation accuracy

**Hyperparameter decision:** I kept the default 3 epochs rather than increasing them. With only 140 training examples, more epochs would risk overfitting — the model would memorize the training set rather than learning generalizable patterns. The validation accuracy plateaued at epoch 2 (56.7%) with no improvement in epoch 3, confirming that more training would not have helped.

---

## Baseline Description

**Model:** `llama-3.3-70b-versatile` via Groq API, zero-shot (no fine-tuning, no examples from my dataset).

**Prompt used:**
```
You are classifying posts and comments from r/obsessionmovie, a subreddit dedicated to the 2026 horror film Obsession.
Assign each post to exactly one of the following categories.

analysis: The post makes a structured argument about the film using specific evidence — references to scenes, dialogue, directing choices, character decisions, or comparisons to other films. The claim could be evaluated by someone who also saw the movie.
Example: "The necklace pendant changes throughout the movie — it starts as a star before Bear's wish and eventually turns into a B, then a swirl once Nikki starts battling with herself."

hot_take: A bold, confident opinion stated without supporting evidence or structured argument. Often a short verdict, hyperbolic claim, or deliberately provocative statement.
Example: "Bear was the villain as soon as he made the wish."

reaction: An immediate emotional response to the film or a specific scene. Little to no argument — the post expresses a feeling, usually written in first person.
Example: "I watched this movie two weeks ago and that scene has been living in my head rent free every night since then."

Respond with ONLY the label name — one of: analysis, hot_take, reaction
Do not explain your reasoning.
```

**Collection:** The baseline was run on the same 30-example test set as the fine-tuned model. Temperature was set to 0 for deterministic outputs. A 0.1s delay was added between requests to respect Groq free-tier rate limits. All 30 responses were parseable.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **0.933** |
| Fine-tuned DistilBERT | 0.567 |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.57 | 1.00 | 0.72 | 17 |
| hot_take | 0.00 | 0.00 | 0.00 | 7 |
| reaction | 0.00 | 0.00 | 0.00 | 6 |
| **accuracy** | | | **0.57** | **30** |
| macro avg | 0.19 | 0.33 | 0.24 | 30 |
| weighted avg | 0.32 | 0.57 | 0.41 | 30 |

### Per-Class Metrics — Zero-Shot Baseline

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.94 | 1.00 | 0.97 | 17 |
| hot_take | 1.00 | 0.71 | 0.83 | 7 |
| reaction | 0.86 | 1.00 | 0.92 | 6 |
| **accuracy** | | | **0.93** | **30** |
| macro avg | 0.93 | 0.90 | 0.91 | 30 |
| weighted avg | 0.94 | 0.93 | 0.93 | 30 |

### Confusion Matrix — Fine-Tuned Model

(See `confusion_matrix.png` for the visual.)

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|---------------------|---------------------|---------------------|
| **True: analysis** | 17 | 0 | 0 |
| **True: hot_take** | 7 | 0 | 0 |
| **True: reaction** | 6 | 0 | 0 |

The fine-tuned model predicted `analysis` for every single example. This is majority-class collapse — with only 140 training examples and `analysis` comprising 56% of the dataset, DistilBERT found the lowest-loss strategy was to always predict the dominant class.

---

### Wrong Prediction Analysis (Fine-Tuned Model)

**Wrong prediction #1:**
> "He knew almost immediately — he was just ignoring it to believe what he wanted to believe."
- **True label:** `hot_take` | **Predicted:** `analysis` (confidence: 0.37)
- **Analysis:** This reads like an analytical claim because it describes Bear's internal state and implies a causal sequence. But it offers no specific scene reference or evidence — it's a bold character verdict. The model likely latched onto the declarative sentence structure ("He knew...he was...") which resembles analytical reasoning, without understanding that it's unsupported.

**Wrong prediction #2:**
> "This scene invokes so much anxiety in me. Time to go watch the movie for the third time."
- **True label:** `reaction` | **Predicted:** `analysis` (confidence: 0.38)
- **Analysis:** This is pure emotional reaction but the model predicted `analysis`. With only 30 reaction examples in training, the model had barely enough signal to learn what reaction posts look like. The word "invokes" may have pattern-matched to analytical vocabulary, pushing it toward the majority class.

**Wrong prediction #3:**
> "She didn't go crazy until he questioned her at the restaurant, so as long as he stayed a perfect boyfriend to her they would live happily ever after."
- **True label:** `hot_take` | **Predicted:** `analysis` (confidence: 0.36)
- **Analysis:** This references a specific scene (the restaurant), which is characteristic of analysis posts. The model couldn't detect the sarcasm ("happily ever after") that signals this is a sardonic hot take rather than a structured argument. Sarcasm and irony are genuinely hard for a small model trained on limited data.

---

### Sample Classifications

| Text (truncated to 100 chars) | True Label | Predicted Label | Confidence | Correct? |
|-------------------------------|------------|-----------------|------------|----------|
| "The necklace pendant changes throughout the movie — it starts as a star before Bear's wish..." | analysis | analysis | 0.41 | ✅ |
| "Bear is a rapist." | hot_take | analysis | 0.36 | ❌ |
| "I just hate how selfish he is." | reaction | analysis | 0.36 | ❌ |
| "Using the words 'more than anyone' changes it from an emotion into a competition with the world..." | analysis | analysis | 0.39 | ✅ |
| "No we CANNOT talk about it. I'm still sleeping with a lamp on three weeks later." | reaction | analysis | 0.37 | ❌ |

**Correct prediction explained:** The necklace pendant example was correctly classified as `analysis`. It references a specific observable prop detail and traces its symbolic change across multiple points in the film — exactly the kind of load-bearing evidence the label definition requires. It's also a longer, more structured post, which likely helped. All correctly classified examples were `analysis` posts, since the model predicted nothing else.

---

## Reflection

**What the model learned vs. what I intended:**
I intended the model to learn three meaningfully different types of film discourse. What it actually learned was to predict `analysis` for everything. The fine-tuned model achieved 56.7% accuracy — exactly the proportion of `analysis` examples in the test set — confirming it never learned to distinguish `hot_take` or `reaction` at all.

The core problem was dataset size. With 140 training examples split across three classes (~78 analysis, ~32 hot_take, ~30 reaction), DistilBERT didn't have enough signal to learn subtle stylistic distinctions. The boundaries between my labels are genuinely hard: `hot_take` and `analysis` both make claims about the film; `reaction` and `hot_take` both use short, declarative language. A model this small needs far more examples to internalize the difference between "evidence is load-bearing" vs. "evidence is decorative."

The baseline told the more interesting story. llama-3.3-70b-versatile achieved 93.3% zero-shot, including 0.83 F1 on `hot_take` and 0.92 F1 on `reaction`. This shows the label definitions were sound — a large LLM could apply them reliably with just a written prompt. The failure was not in the taxonomy but in trying to teach those distinctions to a small model on a small dataset.

**What I would do differently:**
- Collect 500–1000 examples to give the minority classes enough training signal
- Apply class weights during training to penalize majority-class collapse
- Try a larger model (BERT-base or RoBERTa) which might handle subtle stylistic distinctions better
- Consider augmenting minority class examples to balance the dataset before training

---

## Spec Reflection

**One way the spec helped:** The requirement to define a hard edge case rule before annotating forced me to think carefully about `analysis` vs. `hot_take` before labeling a single example. The rule I settled on — if the evidence is "load-bearing" (argument collapses without it) label as `analysis`; if decorative, label as `hot_take` — made borderline cases much more consistent. Without the spec pushing me to define this upfront, I would have made inconsistent decisions throughout collection.

**One way implementation diverged from the spec:** The spec suggested collecting from 2–3 communities for diversity. I collected from r/obsessionmovie exclusively. I made this decision because the film was brand new and the dedicated subreddit had an unusually high density of all three discourse types within a single community — I didn't need to go elsewhere to find variety. Collecting from r/horror or r/movies would have introduced a different audience with different baseline knowledge of the film, adding noise to a task that was already hard enough to define cleanly.

---

## AI Usage

**Instance 1 — Pre-labeling assistance:**
I used Claude to pre-label all 200 examples. The process: I pasted batches of Reddit comments along with my three label definitions and the load-bearing evidence edge case rule, and Claude assigned one label per example with a one-sentence justification. I reviewed every single pre-label before accepting it — I overrode roughly 15–20% of them, mostly in cases where Claude labeled a short bold claim as `analysis` because it referenced a scene (the decorative vs. load-bearing distinction was harder for it to apply consistently than for me). I also used Claude to generate boundary-case examples during label design to stress-test the definitions before annotating.

**Instance 2 — README drafting:**
I used Claude to draft this README based on the actual metrics, wrong predictions, and decisions from the labeling process. I provided all the raw data (label distributions, confusion matrix numbers, wrong prediction examples with confidences) and reviewed the draft for accuracy. I revised the reflection section — Claude's first draft framed the fine-tuning results too apologetically. I changed the framing to treat the baseline outperforming fine-tuning as a genuinely informative finding that validates the taxonomy while revealing the dataset size problem.

**Instance 3 — Groq prompt writing:**
I used Claude to help write the SYSTEM_PROMPT for the zero-shot baseline, starting from my planning.md label definitions. The final prompt uses my label definitions with Claude's help formatting them into a clean instruction structure and selecting one strong example per label from my actual dataset.
