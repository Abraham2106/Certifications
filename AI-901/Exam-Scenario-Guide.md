# AI-901 Exam Scenario Guide
### Based on Real Exam Experience

**Certification Code:** AI-901 | **Issuing Organization:** Microsoft

> [!IMPORTANT]
> This guide is structured exactly like the real exam: **no long definitions, no fill-in-the-blank**.
> Every question is a short scenario. Your job is to *reason* which principle, model, or Foundry service applies.
> If you can answer every block below without looking at the answers, you are ready.

---

## Part 1 — Responsible AI: Scenario Recognition

The exam gives you a 2–3 line business situation and asks which RAI principle is at stake.
The trick: **multiple principles can overlap — pick the most specific one**.

### 🧭 Quick-Reference Card

| Principle | One-Line Trigger | Exam Signal Words |
|:---|:---|:---|
| **Fairness** | The system treats some groups worse than others | "higher rejection rate," "underrepresented," "group A vs. group B," "bias" |
| **Reliability & Safety** | The system behaves inconsistently or causes harm | "incorrect output," "crash," "harmful content," "jailbreak," "hallucination," "monitor after deployment" |
| **Privacy & Security** | Personal data is exposed or misused | "personal data," "training data leak," "GDPR," "model remembers PII," "data at rest" |
| **Inclusiveness** | Some users cannot use the system at all | "disability," "language barrier," "accessibility," "screen reader," "rural users," "low digital literacy" |
| **Transparency** | Users cannot understand why a decision was made | "black box," "cannot explain," "audit trail," "model card," "explainability," "why was I rejected?" |
| **Accountability** | No one is responsible when something goes wrong | "who is liable," "no human review," "oversight," "high-stakes decision," "human-in-the-loop" |

---

### 🎯 Scenario Bank: Which RAI Principle?

<details>
<summary><strong>Scenario 1 — Loan Rejection Rate</strong></summary>

A bank deploys a credit-scoring AI. Overall accuracy is 91%. An audit reveals applicants from one ethnic group are denied at 2× the rate of others with similar income levels.

**Which RAI principle is most at stake?**
- A) Reliability & Safety
- B) Accountability
- C) Fairness
- D) Transparency

**Answer: C — Fairness.**
The system produces systematically different outcomes for a protected group. High overall accuracy does not eliminate group-level disparity. This is a representation or historical bias problem in the training data, not a safety or transparency issue.

</details>

---

<details>
<summary><strong>Scenario 2 — Medical Chatbot Confident Hallucination</strong></summary>

A hospital deploys a chatbot that answers patient questions. A patient asks about drug interactions. The chatbot provides a confident, detailed — but factually wrong — answer that contradicts the prescribing doctor's instructions.

**Which RAI principle is most at stake?**
- A) Fairness
- B) Reliability & Safety
- C) Transparency
- D) Accountability

**Answer: B — Reliability & Safety.**
The system produced harmful output (a wrong medical recommendation) with false confidence. Hallucination is a safety risk. The fix is content safety filters, a system prompt limiting scope, and human-in-the-loop for medical decisions.

</details>

---

<details>
<summary><strong>Scenario 3 — Voice Assistant for Elderly Users</strong></summary>

A government agency builds a benefits-information assistant. A community group reports that elderly users and non-native speakers abandon the app because it only accepts typed input and the UI has tiny text.

**Which RAI principle is most at stake?**
- A) Fairness
- B) Inclusiveness
- C) Reliability & Safety
- D) Transparency

**Answer: B — Inclusiveness.**
The system technically works — it is not producing biased outputs. The problem is that specific groups *cannot access it at all* due to design choices. Adding speech input (Azure AI Speech) and larger text options would address this.

</details>

---

<details>
<summary><strong>Scenario 4 — No Explanation for HR Decision</strong></summary>

An HR system rejects a job application automatically. The candidate asks why they were rejected. The system cannot provide any reasoning — it only outputs "not selected." The HR team also cannot explain the decision because the model is a black box.

**Which RAI principle is most at stake?**
- A) Accountability
- B) Fairness
- C) Transparency
- D) Privacy & Security

**Answer: C — Transparency.**
The system cannot explain its decision-making process to users or operators. The fix is implementing explainability tools (Responsible AI Dashboard, InterpretML) and user-facing explanations. If the question focused on *who is responsible* when harm occurs, the answer would shift to Accountability.

</details>

---

<details>
<summary><strong>Scenario 5 — Training Data Memory Leak</strong></summary>

A language model trained on customer support logs is deployed. Researchers discover that the model can be prompted to reproduce verbatim excerpts from training conversations, including customer email addresses and account numbers.

**Which RAI principle is most at stake?**
- A) Reliability & Safety
- B) Privacy & Security
- C) Transparency
- D) Fairness

**Answer: B — Privacy & Security.**
Training memorization is a well-known AI privacy vulnerability. The model has inadvertently encoded personally identifiable information (PII) from training data and can reproduce it. The fix is differential privacy (SmartNoise) and PII scrubbing before training.

</details>

---

<details>
<summary><strong>Scenario 6 — Autonomous Financial Trades, No Review</strong></summary>

A fintech company deploys an AI that autonomously executes buy/sell orders on behalf of clients with no human review step. When the AI makes a catastrophic error and clients lose money, the company says "the AI decided — we cannot be held responsible."

**Which RAI principle is most at stake?**
- A) Fairness
- B) Reliability & Safety
- C) Transparency
- D) Accountability

**Answer: D — Accountability.**
No person or team was designated as responsible for the AI's decisions. The exam signals are: high-stakes decisions, autonomous action, no human oversight, no one answerable. The fix is human-in-the-loop workflows for high-value decisions and a defined incident response owner.

</details>

---

<details>
<summary><strong>Scenario 7 — Reliability vs. Accountability (Trick Question)</strong></summary>

An AI system that routes 911 emergency calls crashes under peak load on New Year's Eve, dropping hundreds of calls. No monitoring was set up. No escalation plan existed.

**Which RAI principle is most at stake?**
- A) Reliability & Safety only
- B) Accountability only
- C) Both Reliability & Safety AND Accountability
- D) Fairness

**Answer: C — Both.**
The crash is a Reliability & Safety failure (the system did not degrade gracefully). The lack of monitoring and no escalation plan is an Accountability failure (no one was responsible for ensuring the system stayed operational). The exam may ask for the *primary* principle — in that case, Accountability, because the reliability failure was a *consequence* of the accountability gap.

</details>

---

## Part 2 — Model Selection: Multimodal vs. Specialized

The exam gives you a scenario with **mixed input types** (text + image, audio + text, etc.) and asks which model or service handles it.

### 🧭 Quick-Reference Card: What Handles What?

| Input | Output | Use This |
|:---|:---|:---|
| Text only → Text | Text | GPT-4o, Phi-4 (any LLM) |
| Text + Image → Text | Text | **GPT-4o (multimodal)** |
| Text → Image | Image | **DALL-E 3 / gpt-image-1** |
| Text + Image → Image (edit) | Image | **gpt-image-1** |
| Audio → Text | Transcript | **Azure AI Speech (STT) / Whisper** |
| Text → Audio | Spoken audio | **Azure AI Speech (TTS)** |
| Image → Text (reading text in image) | Extracted text | **Azure AI Vision (OCR)** |
| Document/Form → Structured fields | JSON fields | **Azure Content Understanding** |
| Text → Entities / Sentiment | Labels | **Azure AI Language** |

> [!IMPORTANT]
> **Multimodal = model that accepts multiple input types IN THE SAME CALL.**
> GPT-4o is the primary multimodal model on the exam. It accepts text AND images in the same prompt.
> If the task is: "look at this image AND answer a question about it" → GPT-4o multimodal.
> If the task is: "generate a new image from a description" → DALL-E 3 / gpt-image-1 (separate, generative).

---

### 🎯 Scenario Bank: Which Model or Service?

<details>
<summary><strong>Scenario 8 — Product Photo + Review Text</strong></summary>

An e-commerce platform wants to automatically analyze a product photo AND its accompanying customer text review together, then generate a one-sentence quality assessment combining both inputs.

**Which model should they use?**
- A) Azure AI Vision to analyze the image, then Azure AI Language to analyze the text — separately
- B) DALL-E 3 to process both the image and text
- C) GPT-4o in multimodal mode, passing both the image and the text in the same prompt
- D) Azure AI Speech to transcribe any spoken reviews, then GPT-4o for text

**Answer: C — GPT-4o multimodal.**
The scenario requires *combining* two input types (image + text) in a *single reasoning step* to produce a unified output. GPT-4o accepts both in one prompt. Option A would work but requires two separate service calls and cannot naturally *combine* the two signals in one assessment.

</details>

---

<details>
<summary><strong>Scenario 9 — I Want Text AND Images. Who Gets What?</strong></summary>

A travel company wants an AI assistant that:
1. Answers user text questions about destinations
2. Shows a generated image of the destination when asked

**Which architecture is correct?**
- A) One call to GPT-4o for both — it generates text and images
- B) GPT-4o for the text responses; a separate call to DALL-E 3 / gpt-image-1 for image generation
- C) Azure Content Understanding for text; Azure AI Vision for images
- D) DALL-E 3 for everything — it handles text and image generation

**Answer: B.**
GPT-4o generates *text*. It can *receive* images as input (multimodal), but it does not generate new images. Image *generation* is DALL-E 3 or gpt-image-1 — a separate model family. The architecture is: user text question → GPT-4o → text answer; user asks for image → gpt-image-1 → generated image.

</details>

---

<details>
<summary><strong>Scenario 10 — Invoice Processing</strong></summary>

An accounting team receives hundreds of scanned invoice PDFs per day. They need to automatically extract: vendor name, invoice date, line items, and total amount due — and push these into their ERP system.

**Which Azure service should they use?**
- A) Azure AI Language with key phrase extraction
- B) GPT-4o with a structured extraction prompt
- C) Azure Content Understanding (Information Extraction)
- D) Azure AI Vision OCR

**Answer: C — Azure Content Understanding.**
OCR (Option D) would extract raw text from the PDF — but it would not map that text to specific fields like "vendor name" or "total due." Language (Option A) is for unstructured prose. GPT-4o (Option B) could work but is expensive and not the purpose-built solution. Content Understanding is purpose-built for structured field extraction from documents and forms.

</details>

---

<details>
<summary><strong>Scenario 11 — Respond to Spoken Prompts with a Multimodal Model</strong></summary>

A developer wants to build an application where a user speaks a question aloud, the app processes it, and an AI model answers — also in spoken audio. The answer must come from a deployed GPT-4o model.

**Which architecture is correct?**
- A) Feed the audio file directly to GPT-4o — it handles audio natively
- B) Use Azure AI Speech STT to convert the spoken question to text → send text to GPT-4o → use Azure AI Speech TTS to convert the response to audio
- C) Use Azure Content Understanding to transcribe the audio, then GPT-4o
- D) Use DALL-E 3 for voice interaction

**Answer: B.**
GPT-4o (in its standard Azure deployment) accepts text and images — not raw audio in most configurations. The correct architecture wraps it: Speech-to-Text converts spoken input → text goes to GPT-4o → response text goes to Text-to-Speech → audio output. This is the "respond to spoken prompts using a deployed multimodal model" pattern the exam tests directly.

</details>

---

<details>
<summary><strong>Scenario 12 — Interpreting Visual Input in a Prompt</strong></summary>

A manufacturing company wants a quality control assistant. A technician photographs a product on the line and types: "Does this part show any cracks or misalignments?" The assistant must analyze both the photo and the question.

**Which service handles this?**
- A) Azure AI Vision object detection
- B) Azure Content Understanding
- C) GPT-4o in multimodal mode — image + text in the same prompt
- D) Azure AI Language

**Answer: C — GPT-4o multimodal.**
The user is sending an image AND a text question together, expecting a reasoned text answer that combines both. This is the exact definition of visual input in a multimodal prompt. Azure AI Vision (Option A) would classify the image or detect objects — it would not answer a natural language question about what it sees in context.

</details>

---

<details>
<summary><strong>Scenario 13 — Generating a New Image from a Description</strong></summary>

A marketing team wants to generate promotional artwork based on text descriptions like "a sunset over the Andes mountains with the company logo on a cliff."

**Which model should they use?**
- A) GPT-4o — it is multimodal and understands the description
- B) DALL-E 3 or gpt-image-1 — image generation models
- C) Azure AI Vision — it analyzes and generates images
- D) Azure AI Language — it extracts the key visual elements first

**Answer: B — DALL-E 3 / gpt-image-1.**
GPT-4o (Option A) can *read* and *understand* images, but it does not *generate* new images from descriptions. Image generation is a separate capability implemented by DALL-E 3 and gpt-image-1. Azure AI Vision (Option C) analyzes existing images — it is not a generative model.

</details>

---

## Part 3 — Foundry Service Selection by Scenario

This is the section where knowing the **exact Azure service name** matters.

### 🧭 Quick-Reference Card: Azure Service → What It Does

| Azure Service | What It Does | Key Capabilities |
|:---|:---|:---|
| **Azure OpenAI / Foundry Models** | LLM text generation and reasoning | Chat, completion, embeddings |
| **GPT-4o (multimodal)** | Text + image input → text output | Visual QA, document reasoning |
| **DALL-E 3 / gpt-image-1** | Text → new image | Marketing art, illustrations |
| **Azure AI Language** | Analyzes *existing* text | Sentiment, NER, key phrases, summarization |
| **Azure AI Speech** | Audio ↔ Text conversion | STT, TTS, translation, speaker ID |
| **Azure AI Vision** | Image/video → structured info | OCR, object detection, classification |
| **Azure Content Understanding** | Documents/forms → structured fields | Invoice extraction, form processing, audio/video extraction |
| **Foundry Agent Service** | Multi-step autonomous AI | Tools, threads, runs, function calling |

---

### 🎯 Scenario Bank: Which Foundry Service?

<details>
<summary><strong>Scenario 14 — Sentiment from Customer Reviews</strong></summary>

A retail company wants to automatically classify thousands of customer product reviews as positive, negative, or mixed — and identify which product aspects customers mention most.

**Which Azure service?**
- A) GPT-4o with a sentiment prompt
- B) Azure AI Language (sentiment analysis + key phrase extraction)
- C) Azure Content Understanding
- D) Foundry Agent Service

**Answer: B — Azure AI Language.**
Sentiment analysis and key phrase extraction are preconfigured features of Azure AI Language, purpose-built for exactly this workload. GPT-4o (Option A) would also work but is more expensive and less efficient for bulk structured analysis.

</details>

---

<details>
<summary><strong>Scenario 15 — Call Center Audio Analysis</strong></summary>

A telecom company records all customer service calls. They want to: (1) transcribe each call, (2) detect whether the customer was frustrated, and (3) flag calls that ended without resolution.

**Which services?**
- A) Azure AI Vision for audio transcription; Azure AI Language for sentiment
- B) Azure AI Speech (STT) for transcription; Azure AI Language for sentiment and summarization
- C) Azure Content Understanding for everything
- D) GPT-4o multimodal — it handles audio directly

**Answer: B.**
Step 1 (transcription) = Azure AI Speech STT. Steps 2 and 3 (sentiment, resolution detection) = Azure AI Language. Content Understanding (Option C) is for extracting structured fields from documents, not for sentiment on unstructured conversation text.

</details>

---

<details>
<summary><strong>Scenario 16 — Multi-Step Autonomous Research</strong></summary>

A legal firm wants an AI that can receive a client's case description, search legal databases for relevant precedents, summarize the top three findings, and draft a preliminary brief — all in one automated workflow without a human approving each step.

**Which Azure service?**
- A) Azure AI Language
- B) GPT-4o with a detailed system prompt
- C) Foundry Agent Service (Agentic AI)
- D) Azure Content Understanding

**Answer: C — Foundry Agent Service.**
The scenario requires *multi-step reasoning, tool use (database search), and autonomous action* toward a goal. This is the definition of agentic AI. A single GPT-4o call (Option B) cannot search databases or chain multiple tool steps autonomously.

</details>

---

<details>
<summary><strong>Scenario 17 — Extracting Fields from Audio Recordings</strong></summary>

An insurance company receives voicemail claims. They want to automatically extract: claimant name, policy number, and date of incident from each recording — as structured data fields.

**Which Azure service?**
- A) Azure AI Speech STT to transcribe, then Azure AI Language to extract entities
- B) Azure Content Understanding (audio extraction mode)
- C) GPT-4o multimodal
- D) Azure AI Vision

**Answer: B — Azure Content Understanding.**
Content Understanding's audio extraction capability processes audio directly to extract predefined structured fields. It is specifically designed for this pattern. Option A (STT + Language) would also work as a pipeline but is not the purpose-built answer. The exam signals "structured fields from audio" → Content Understanding.

</details>

---

<details>
<summary><strong>Scenario 18 — Text in Images (Menu Board, Poster)</strong></summary>

A restaurant chain wants to digitize their physical menu boards. They photograph each board and need to extract all the item names and prices as a list.

**Which Azure service?**
- A) Azure Content Understanding
- B) Azure AI Vision (OCR)
- C) GPT-4o multimodal
- D) Azure AI Language

**Answer: B — Azure AI Vision (OCR).**
The task is extracting text *printed in an image* without mapping to specific form fields. This is classic OCR — Azure AI Vision. Content Understanding (Option A) is for semi-structured *documents with a known schema* (invoices, forms). A menu board with freeform layout maps to OCR.

> **Key Distinction:** OCR = extract all readable text from an image. Content Understanding = extract specific named fields from a document with a known structure.

</details>

---

## Part 4 — Rapid-Fire Service Matching

For each scenario, identify the correct Azure service. Answers below — try to complete the list first.

| # | Scenario | Answer |
|:--|:---|:---|
| 1 | Convert a podcast episode to a searchable text transcript | Azure AI Speech (STT) |
| 2 | Read a customer's name and address from a scanned form | Azure Content Understanding |
| 3 | Generate a new illustration of a dragon for a game | DALL-E 3 / gpt-image-1 |
| 4 | Classify customer support emails as billing, technical, or general | Azure AI Language |
| 5 | A user uploads a photo of a damaged car and asks "what type of damage is this?" | GPT-4o (multimodal) |
| 6 | Count how many people are in a store entrance video feed | Azure AI Vision (object detection) |
| 7 | An AI reads emails, books meetings, and replies — autonomously | Foundry Agent Service |
| 8 | Read text from a billboard in a photograph | Azure AI Vision (OCR) |
| 9 | Convert a written report to audio for a podcast app | Azure AI Speech (TTS) |
| 10 | Extract invoice number, date, and total from 1,000 PDF invoices | Azure Content Understanding |
| 11 | Detect whether a user's review is positive or negative | Azure AI Language (sentiment) |
| 12 | User speaks a question → AI answers with generated speech | Azure AI Speech STT → GPT-4o → Azure AI Speech TTS |

---

## Part 5 — Mixed Scenario Reasoning (Hardest Type)

These questions combine multiple workload types and require you to pick the *most appropriate* service — not just *a* service that could work.

<details>
<summary><strong>Scenario 19 — Hospital Intake (Multi-Service)</strong></summary>

A hospital wants to modernize intake. Patients speak their symptoms into a kiosk. The system must: transcribe the speech, detect urgency level (mild/moderate/critical) from the transcript, and extract structured fields (patient name, symptom keywords, pain scale rating).

Rank the services involved and map each to its step.

**Answer:**
1. **Azure AI Speech (STT)** — transcribes spoken patient input to text
2. **Azure AI Language** — classifies urgency (sentiment/custom classification) and extracts keyword symptoms (NER/key phrase extraction)
3. **Azure Content Understanding** — extracts structured fields (name, pain scale) if the input comes from a form or structured template; if purely conversational, Language NER handles this

The exam would phrase this as: "Which service handles the urgency classification step?" → Azure AI Language.

</details>

---

<details>
<summary><strong>Scenario 20 — The Architecture Trap</strong></summary>

A company wants to build a chatbot that:
- Accepts text and image input from users ✓
- Generates text responses ✓
- Can also generate product images when the user asks ✓
- Runs autonomous multi-step research when asked to "find the best option" ✓

A developer says: "I'll use GPT-4o for everything." What is wrong with this plan?

**Answer:**
GPT-4o handles: text input ✓, image input (multimodal) ✓, text output ✓.
GPT-4o does **NOT** handle:
- **Image generation** → requires DALL-E 3 / gpt-image-1 (separate service)
- **Autonomous multi-step research with tools** → requires Foundry Agent Service (threads, runs, tool calls)

The correct architecture is:
- GPT-4o → for chat responses with text and image understanding
- DALL-E 3 / gpt-image-1 → for generating product images
- Foundry Agent Service (backed by GPT-4o) → for the autonomous research workflow

</details>

---

## 🔑 Final Cheat Sheet: Exam Reasoning Framework

When you see a scenario, apply this decision tree:

```
1. What is the INPUT?
   → Text only?              → LLM (GPT-4o, Phi-4)
   → Text + Image?           → GPT-4o MULTIMODAL
   → Audio?                  → Azure AI Speech (STT)
   → Document/Form/PDF?      → Azure Content Understanding
   → Image (reading text)?   → Azure AI Vision (OCR)
   → Image (objects/faces)?  → Azure AI Vision (detection)

2. What is the OUTPUT?
   → New text?               → LLM
   → New image?              → DALL-E 3 / gpt-image-1
   → Spoken audio?           → Azure AI Speech (TTS)
   → Structured fields?      → Azure Content Understanding
   → Labels/entities?        → Azure AI Language

3. Is the task AUTONOMOUS (multi-step, tools, no human per step)?
   → YES?                    → Foundry Agent Service

4. Is the task ANALYZING existing text (not generating)?
   → Sentiment / Entities / Key phrases → Azure AI Language

5. RAI Principle: What is the CORE PROBLEM?
   → Group gets worse outcome?    → FAIRNESS
   → System harms or fails?       → RELIABILITY & SAFETY
   → Data is exposed?             → PRIVACY & SECURITY
   → Some users can't access it?  → INCLUSIVENESS
   → No one can explain it?       → TRANSPARENCY
   → No one is responsible?       → ACCOUNTABILITY
```

---

*Guide synthesized from AI-901 exam experience — May 2026.*
*Cross-referenced with: AI-Responsibility-Principles.md, AI-Workload-Types.md, AI-Model-Components.md, Deploying-and-Interacting-with-Models-in-the-Foundry-Portal.md, Lightweight-and-Agent-Clients.md*
