# Persuasion Profiler — Label Codebook (v0.1)

This codebook defines the labels used for Phase 1 annotation and synthetic data generation. It is the single source of truth for what each tactic means. If an example disagrees with the codebook, the codebook wins — update the codebook if the definition is wrong, not the label.

## How to use

- Unit of labeling = **one message** (single Reddit post, single email body, single chat turn).
- Labeling is **multi-label**: a message may have 0, 1, or many tactic labels.
- A message with no tactic present gets exactly `benign` (and nothing else).
- When uncertain between two tactics, apply both and flag for adjudication (see §4).

## 1. Label definitions

Each label has four parts: **Definition** (what it is), **Required elements** (what must be present to apply the label), **Surface cues** (non-exhaustive lexical/structural hints), and **Examples**. Examples are paired — a positive case and a near-miss that should NOT receive the label.

---

### 1.1 `urgency_scarcity`

**Definition.** The message manufactures time pressure or artificial scarcity to push the reader toward action before they can verify, compare, or consult others.

**Required elements** (all must be present):
1. An explicit or implicit deadline / countdown / limited quantity.
2. A requested or implied action the reader must take because of (1).

**Surface cues.** "within 24 hours", "last chance", "only 3 left", "account will be locked", "before midnight", "act now", "offer expires". Countdown timers, bold red text, all caps deadlines.

**Positive example.**
> "URGENT: Your account will be suspended in 30 minutes unless you verify your identity at the link below."

**Near-miss (label as `benign`).**
> "Reminder: your library book is due next Tuesday."
> — Has a deadline, but no pressure asymmetry and no high-risk action. Factual deadlines are not urgency.

**Edge cases.**
- Real urgency (fire drill, medical) → `benign`. Urgency is only a tactic when the *timeline* is artificial or disproportionate to the request.
- "Limited edition" marketing with no action request → `benign`. Scarcity requires a paired ask.

---

### 1.2 `false_intimacy`

**Definition.** The sender simulates personal closeness, shared identity, or emotional vulnerability to lower the reader's guard. Common in pig-butchering, romance scams, and "friend-bot" manipulation.

**Required elements** (at least one must be present):
1. Unsolicited self-disclosure that is disproportionate to the relationship ("I'm a parent too", "I lost my mother last year").
2. Premature claims of affection, care, or shared fate ("I care about you", "we're in this together") from a party with no established relationship.
3. Secret-sharing framing ("I shouldn't tell you this but…", "between you and me").

**Surface cues.** First-person emotional disclosure paired with second-person care statements, pet names ("dear", "love", "my friend") from strangers, claims of shared identity ("as a fellow investor / parent / veteran…").

**Positive example.**
> "I don't usually open up like this, but you remind me of my late husband. I feel I can trust you with something important about my inheritance."

**Near-miss (label as `benign`).**
> "Hey! Long time no talk — how's the new job going?"
> — Familiarity is warranted by prior relationship. False intimacy requires unearned closeness.

**Edge cases.**
- Therapy-bot / grief-support content: genuine warmth is NOT false intimacy. Distinguish by whether there is an eventual extractive ask. In Phase 1 we cannot see the ask, so lean toward `benign` unless the self-disclosure is clearly outsized for the message context.
- Customer service ("I'm sorry for the trouble") → `benign`. Scripted empathy is not intimacy.

---

### 1.3 `authority_mimicry`

**Definition.** The sender impersonates or invokes institutional authority (government, bank, law enforcement, platform security) to compel compliance.

**Required elements** (at least one must be present):
1. Claim of affiliation with a recognizable authority (named agency, department, case number, badge number).
2. Bureaucratic register designed to mimic official communication (reference numbers, legalistic phrasing, compliance jargon).
3. Citation of rules, laws, or protocols as the basis for the request.

**Surface cues.** "This is Agent/Officer [X]", "Case #", "per Section 5.3", "Compliance Department", "Federal [anything]", "Your SSN has been flagged", official-looking headers, spoofed sender names.

**Positive example.**
> "This is the IRS Criminal Investigation Division. Case #CI-44201 has been opened in your name. Failure to respond within 24 hours constitutes admission of tax fraud."

**Near-miss (label as `benign`).**
> "From: payroll@company.com — Your W-2 for 2025 is attached."
> — Legitimate institutional communication without coercive framing.

**Edge cases.**
- Real authority emails CAN receive this label if they use coercive jargon (some legitimate collections notices do). The label is about *tactic*, not *legitimacy*. A real IRS letter using fear-based compliance pressure still gets labeled — a human would correctly identify the tactic even if the sender is real.
- Co-occurs frequently with `urgency_scarcity`. Label both.

---

### 1.4 `gaslighting_isolation`

**Definition.** The sender undermines the reader's trust in their own judgment or in outside sources of truth (family, mainstream media, experts), positioning themselves as the sole reliable authority.

**Required elements** (at least one must be present):
1. Explicit or implicit discrediting of the reader's existing support network ("your family won't understand", "don't tell anyone").
2. Framing outside information sources as deceptive ("the media lies", "they don't want you to know").
3. Reframing the reader's doubts as evidence of manipulation *by others* ("they've got you convinced you're wrong").

**Surface cues.** "Don't tell [trusted party]", "only I can help you", "trust me, not them", "you've been lied to your whole life", "wake up", "they don't want you to see this".

**Positive example.**
> "Don't mention this conversation to your daughter — she's been conditioned by the media to dismiss anything that threatens her worldview. You and I both know the truth."

**Near-miss (label as `benign`).**
> "I disagree with the article you shared — here's why I think it's wrong."
> — Disagreement with a source is not isolation. Gaslighting requires delegitimizing the reader's *capacity to evaluate*, not a specific claim.

**Edge cases.**
- Political content is a common source of false positives. Strong opinions ≠ gaslighting. The label requires an *isolation mechanic* — directing the reader away from their support network or away from verification.
- Frequently co-occurs with `false_intimacy` (isolate, then bond).

---

### 1.5 `benign`

**Definition.** No persuasion tactic from the above four is present at the level required by the codebook.

**Apply `benign` when.**
- None of the four tactics' required elements are met, OR
- Required elements are technically present but the message is clearly routine (library reminders, legitimate marketing with disclosure, real customer service).

**`benign` is mutually exclusive with all other labels.** A message labeled `benign` receives no other label. A message with any tactic label does not receive `benign`.

## 2. Multi-label guidance

Tactics frequently co-occur. Common combinations:

| Combination | Typical pattern |
|---|---|
| `authority_mimicry` + `urgency_scarcity` | Fake IRS / bank notices with 24h deadlines |
| `false_intimacy` + `urgency_scarcity` | Pig-butchering "we need to act on this investment tonight" |
| `false_intimacy` + `gaslighting_isolation` | Romance scams that isolate the victim from family |
| `authority_mimicry` + `gaslighting_isolation` | Cult-like / conspiracy framings citing fake institutions |

**Rule of thumb.** If two labels both have their required elements met, apply both. Do not choose the "dominant" one.

## 3. AppRay-Dark mapping decision

Our four tactics are a **superset** of AppRay-Dark's categories, not a 1:1 mapping. AppRay-Dark focuses on UI dark patterns (interface-level deception); our taxonomy adds conversational and emotional tactics that have no UI equivalent.

**Consequence for Phase 1:** AppRay-Dark is used as a taxonomy cross-reference only. We do not inherit its labels directly. Annotators trained on AppRay-Dark should be briefed that `false_intimacy` and `gaslighting_isolation` have no direct AppRay counterpart and require reading our definitions from scratch.

## 4. Annotator disagreement resolution

Dual-annotation on at least 10% of the corpus. Disagreements resolved by:

1. **Codebook check.** If one annotator's choice clearly matches the required elements and the other's does not, the codebook wins.
2. **Adjudicator review.** Genuine ambiguity escalates to a third annotator. Resolution is recorded with a one-line reason.
3. **Codebook amendment.** If the same ambiguity recurs (≥3 times), the codebook is updated in a new version. Every annotation records which codebook version it was labeled under.

Target inter-annotator agreement: **Cohen's κ ≥ 0.65 per label**. Labels below this threshold at the midpoint of annotation trigger a codebook review before continuing.

## 5. Out of scope for Phase 1

The following are deferred to Phase 2 and should NOT be labeled in Phase 1:

- **Manipulative Pivot.** Requires a Hook message and an Ask message — spans multiple turns. Phase 1 is single-message.
- **Pivot Score (P).** Conversational metric; needs `conv_id` aggregation.
- **Sycophancy.** Requires prior-turn context to identify the agreement loop.
- **Span-level highlighting.** Phase 1 is message-level. Span labels come in Phase 2 using HateXplain-style rationales.
