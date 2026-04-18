# Persuation Profiler To Prevent AI-Driven Fraud

"While others are trying to detect if text is AI, we are detecting the intent of that AI. The Persuasion Profiler identifies the psychological weaponization of language. It transforms a vulnerable user from a 'target' into an 'auditor,' effectively de-weaponizing AI-driven fraud and radicalization at the point of contact."

The Persuasion Profiler is a timely choice because it moves the defense from **"Is this AI?"** (which is becoming a losing battle) to **"What is this AI trying to do to me?"** (which is where the actual harm lives).

To expand this into a viable project, we need to move from a simple classifier to a **Behavioral Firewall**. Here is the expanded blueprint for the **Persuasion Profiler**.

## 1. The Taxonomy: "The Dark NLP Library"

To bridge the "Intent-to-Manipulate" gap, our tool needs a specific vocabulary. Instead of just "negative" or "positive," our model will flag **Tactical Clusters**:

|Tactical Cluster | Signal/Patterns | AI Negativity Application|
|-----------------|-----------------|--------------------------|
|Scarcity/Urgency |"Limited time," "Account lock in 10m," "Only one spot left."|Financial scams, high-pressure crypto "pump-and-dumps."|
|False Intimacy (Loverboy/Friend-Bot)|Over-sharing personal details, "I’m only telling you this," "I care about your safety."|"Pig-Butchering" scams, emotional manipulation, social engineering.|
|Authority Mimicry|	Use of bureaucratic jargon, "Regulatory Compliance Dept," "Security Protocol 404.|"Sophisticated spear-phishing and corporate sabotage.|
|Gaslighting & Isolation|"Your family won't understand," "The mainstream media is lying to you," "Only I have the truth."|Political radicalization and cult-like AI interactions.|

## 2. Technical Architecture: The Dual-Pass Engine
A single NLP model isn't enough to detect "vicious" persuasion. You need a Dual-Pass System:

### Pass A (The Linguistic Surface):
Uses the `GoEmotions` dataset to identify the "emotional temperature." Is the text trying to evoke fear, sadness, or joy?

### Pass B (The Strategic Intent):
This is where you use `AppRay-Dark` and `WEF` Data. This pass looks for "Logical Leapfrogging." It asks: Is the emotional trigger disproportionate to the request? * Example: If a bot says "I am so sorry for your loss" (Empathy) followed immediately by "Click here to verify your inheritance" (Action), the Profiler flags a "Manipulative Pivot."


## 3. The Dashboard: "The X-Ray View"
The goal isn't to block the chat, but to empower the user. The dashboard would provide:

- **The Persuasion Score**: A 1-100 meter (Low to Predatory).
- **Trigger Highlighting**: Color-coded text overlay. Red for Urgency, Purple for False Intimacy, Yellow for Authority Claims.
- **The "Nudge" Suggestion**: Instead of saying "Don't click," it says: "This message uses 'Fear' to drive an 'Immediate Action.' Take 5 minutes to call the actual institution before responding."

## 4. Addressing the Research Deficiency
This project specifically tackles the "Context-Intent Problem." * Current Research Flaw: Most models see "I need your help" as a positive, prosocial sentence.
- **our Solution**: You are building a Temporal Intent Tracker. our tool analyzes the progression of the conversation. It tracks how the AI builds "rapport" over 10 messages before making a "request for data."
- **New Dataset Opportunity**: You can create the "Vicious vs. Virtuous Persuasion Corpus" (VVPC). You take transcripts of helpful AI (therapy bots, teachers) and contrast them with known scam logs.

## 5. Implementation Roadmap
- **Phase 1 (Dark Pattern Detector v0.1)**: Train a multi-label encoder classifier (RoBERTa) on a composed real+synthetic corpus to flag the four Tactical Clusters at the message level.
- **Phase 2 (The Interface)**: Build a Chrome/Edge Extension that injects a small "Safety Shield" icon into chat windows (WhatsApp Web, Telegram, Gmail).
- **Phase 3 (The Feedback Loop)**: Allow users to "Flag for Manipulation." This creates a community-sourced dataset of new AI-driven scam tactics, making the world more resilient in real-time.

## Phase 1: Dark Pattern Detector (v0.1)

**Goal:** A multi-label classifier that flags persuasion tactics at the **message level**. This is the "Pass A + lightweight Pass B" foundation. Conversation-level pivot detection is deferred to Phase 2.

**Deliverable:** A RoBERTa-based classifier + inference engine that takes a single message and returns probabilities for 5 labels:

```python
TACTICS = [
    "urgency_scarcity",
    "false_intimacy",
    "authority_mimicry",
    "gaslighting_isolation",
    "benign",
]
```

### 1.1 Data

No single "dark persuasion" dataset exists — we compose one.

| Source | Role | Notes |
|---|---|---|
| GoEmotions | Emotional signal (weak-label) | Fear / gratitude / grief distributions per cluster |
| Jigsaw Toxic / HateXplain | Coercion & aggression proxy | HateXplain rationales reused for span tagging later |
| Nazario Phishing Corpus | Real urgency + authority mimicry | Primary real scam signal |
| Enron Emails | Benign baseline | Corporate-tone negatives |
| r/scams, r/catfishing, r/relationships | Real false intimacy / pig-butchering | Manual + Pushshift scrape |
| AppRay-Dark | Dark-pattern label schema | Used as taxonomy reference, not training corpus |
| Synthetic (multi-LLM) | Coverage for rare tactics | 500–1000 per cluster; at least 2 different generator LLMs to avoid single-model stylistic bias |

**Why synthetic data at all?** Four roles, ranked by importance:

1. **Coverage for rare tactics.** Real phishing corpora are rich in `urgency_scarcity` and `authority_mimicry` but thin on `false_intimacy` (pig-butchering) and `gaslighting_isolation` (cult / radicalization). Synthesis fills the long tail so no class collapses during training.
2. **Training-only.** Synthetic samples never enter the held-out test set. Reported metrics therefore reflect real-scam performance, not generator mimicry.
3. **Multi-generator to avoid stylistic bias.** At least two different LLMs are used. Training on a single generator teaches the classifier to detect *that model's* scam style rather than the underlying tactic.
4. **Near-miss counter-examples.** The `benign` prompts deliberately produce messages that superficially resemble a tactic but fail its required elements (e.g. a library-due reminder that names a deadline but isn't urgency/scarcity manipulation). These suppress false positives on legitimate real-world traffic.

The trade-off is synthetic data drift (see §1.5) — mitigated by multi-generator synthesis and a real-only test set.

**Held-out test set:** ≥200 samples drawn *only* from real data (r/scams + Nazario). Synthetic data never appears in test.

**Schema (conversation-aware from day one):**

```
{conv_id, turn_idx, text, labels: [multi-hot], source, is_synthetic}
```

Even though the Phase 1 classifier ignores `conv_id` / `turn_idx`, storing them now means Phase 2 (temporal pivot tracker) does not require re-labeling.

### 1.2 Model

- **Baseline:** DistilBERT (fast iteration, sanity check)
- **Primary:** RoBERTa-base, multi-label head (sigmoid + BCE)
- **Stretch:** DeBERTa-v3 if RoBERTa plateaus

We do **not** fine-tune Llama-3 / Mistral-7B in Phase 1 — encoders are the right tool for short-text multi-label classification, and cost/evaluation overhead isn't justified until the dataset is clean.

### 1.3 Evaluation

Multi-label, class-imbalanced. Per-class threshold tuning (not fixed 0.5).

| Metric | Target |
|---|---|
| Macro F1 | ≥ 0.70 |
| Precision (macro) | ≥ 0.80 |
| Recall (macro) | ≥ 0.65 |
| ROC-AUC per class | reported, not gated |

**Precision is the UX metric** — a false positive on a real bank email destroys trust. We optimize thresholds for precision-first, then recover recall in Phase 2 via conversational context.

### 1.4 Code structure

All Phase 1 code lives in `project.ipynb`. Cells are organized top-to-bottom by dependency:

```
project.ipynb
├── Tactic taxonomy        # label enum + per-tactic codebook (definition, cues, examples)
├── Record schema          # SyntheticRecord + append-only JsonlWriter
├── LLM client             # LLMClient base, AnthropicClient (with prompt caching), EchoClient
├── Generator              # SyntheticDataGenerator + prompt builders
└── Runner                 # run_generation(...) entry point + smoke-test and paid-run cells
```

Phase 2 additions (classifier training, inference, evaluation) will be appended as new sections in the same notebook.

### 1.5 Known risks

- **Synthetic data drift.** Classifier learns to detect LLM-generated scam style, not real scams. Mitigation: multi-generator synthesis + real-only test set.
- **Label ambiguity.** "I care about you" is benign *or* false intimacy depending on context Phase 1 cannot see. Accept this ceiling — Phase 2 is where it gets solved.
- **Taxonomy vs. AppRay-Dark mapping.** Decide before labeling begins whether our 4 clusters map 1:1 to AppRay-Dark categories or are a superset.

### 1.6 Phase 1 exit criteria

1. Test-set Macro F1 ≥ 0.70 on real-only held-out data.
2. Inference engine returns per-label probabilities in < 100ms on CPU for a single message.
3. Dataset schema supports conversation context (even if unused).
4. At least one documented failure mode per tactical cluster (for Phase 2 motivation).

