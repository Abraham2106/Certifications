# Syllabus and Study Guide

**Certification Code:** AI-901  
**Issuing Organization:** Microsoft  

> [!IMPORTANT]
> **Exam Experience Guide (Scenario-Based):** Built from real exam feedback — all content in scenario/multiple-choice format mirroring the actual exam structure.
> → **[Exam Scenario Guide](Exam-Scenario-Guide.md)**

---

## 1. AI Responsibility Principles

Foundational ethical guidelines designed to ensure that AI systems are developed and deployed in a manner that is fair, reliable, safe, and trustworthy.

> [!NOTE]
> All detailed study guides, case studies, exam questions, and architecture challenges for these principles are maintained in the dedicated guide: **[AI Responsibility Principles Guide](AI-Responsibility-Principles.md)**.

### 1.1 Responsible AI - Fairness
- Core concepts: Disadvantages based on personal attributes, context-based behaviors.
- Pipeline biases: Historical bias, representation bias, label bias, design bias.
- Tools: `Fairlearn` open-source integration in Azure ML.

### 1.2 Responsible AI - Reliability & Safety
- Core concepts: Graceful degradation, consistent correctness, harm prevention.
- Generative risks: Unbounded outputs, harmful content, jailbreaking, hallucinations, prompt injections.
- Tools: `Azure AI Content Safety`, Metaprompts/System prompts, Red Teaming, Human-in-the-loop.

### 1.3 Responsible AI - Privacy & Security
- Core concepts: Data protection at rest/in transit, Zero Trust model.
- AI vulnerabilities: Training memorization, data aggregation, model inversion, adversarial examples.
- Tools: `SmartNoise` (differential privacy), `Counterfit` (threat simulation), `Prompt Shields` (injection protection), stateless Foundry processing.

### 1.4 Responsible AI - Inclusiveness
- Core concepts: Empowering everyone regardless of background, physical accessibility, shared responsibility.
- Access dimensions: Inclusive by design (access pathways), inclusive by process (diverse teams).
- Tools: `Azure AI Speech` (TTS/STT), `Azure AI Vision` (descriptions), `Azure AI Translator`, `Azure AI Language` (multilingual).

### 1.5 Responsible AI - Transparency
- Core concepts: Explainable decision-making, black box problem, interpretability vs. explainability.
- Transparency layers: Model transparency, operational transparency, user-facing transparency.
- Tools: `InterpretML`, Responsible AI Dashboard, Transparency Notes, Model Cards, Audit Logs.

### 1.6 Responsible AI - Accountability
- Core concepts: Human oversight and control, blame vs. accountability, shared responsibility model.
- Integration patterns: Human-in-the-loop triggers, incident response protocols, Microsoft AIRT, sensitive uses review.
- Tools: MLOps Model Registry (metadata tracing).

---

## 2. AI Model Components & Configurations

Detailed study guide notes on how generative models work, model selection criteria, and Azure configuration parameters.

> [!NOTE]
> All detailed study guides, configurations, exam questions, and architecture challenges for model components are maintained in the dedicated guide: **[AI Model Components Guide](AI-Model-Components.md)**.

### 2.1 How Generative Models Work
- Core concepts: Tokenization, embeddings, context windows, transformers, attention.
- Architectures: Encoder-only, decoder-only, encoder-decoder models.

### 2.2 Selecting the Right Model
- Dimensions: Capability-based filtering, cost vs. reasoning capability trade-off, multimodal models.
- Azure service mapping: GPT-4o, Phi-4, Whisper, Azure Translator, DALL-E 3.

### 2.3 Deployment Options and Configuration Parameters
- Deployment options: Serverless API (MaaS), Managed Compute.
- Configuration parameters: Temperature, Top-p, Max tokens, Frequency/Presence penalties.

---

## 3. AI Workload Types

Detailed study guide notes on identifying and mapping AI workload types to Azure services.

> [!NOTE]
> All detailed study guides, mappings, exam questions, and architecture challenges for workload types are maintained in the dedicated guide: **[AI Workload Types Guide](AI-Workload-Types.md)**.

### 3.1 Workload Categories & Scenarios
- Core categories: Generative AI, Agentic AI, Text Analysis, Speech, Computer Vision, Information Extraction.
- Service mappings: Azure OpenAI, Azure AI Agent Service, Azure Language, Azure Speech, Azure Vision, Azure Content Understanding.

---

## 4. Prompt Engineering

Detailed study guide notes on system and user prompt design, API formatting, Python code SDK calls, and error handling.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for prompt engineering are maintained in the dedicated guide: **[Prompt Engineering Guide](Prompt-Engineering-System%20and%20User%20Prompts.md)**.

### 4.1 System and User Prompts
- Core concepts: System, user, and assistant roles, token arrays, context memory limits.
- Design principles: Role definitions, behavioral constraints, format templates, in-context zero/few-shot learning, chain-of-thought, grounding.
- Code & errors: `azure-ai-projects` and `openai` clients, zero-shot/few-shot loops, grounded RAG variables, rate limit handlers.

---

## 6. Lightweight Chat Client

Detailed study guide notes on building a minimum viable chat client with the Foundry SDK: authentication, conversation loops, history management, context window handling, and error recovery.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for the lightweight chat client are maintained in the dedicated guide: **[Lightweight & Agent Clients Guide](Lightweight-and-Agent-Clients.md)**.

### 6.1 Lightweight Chat Client — Foundry SDK
- Core architecture: `AIProjectClient` + `get_openai_client()`, `DefaultAzureCredential`.
- History management: Client-side `conversation_history` list, sliding-window trimming to prevent context overflow.
- Error handling: `AuthenticationError`, `RateLimitError`, `APIError`.

---

## 7. Single-Agent Solutions

Detailed study guide notes on designing and coding single-agent solutions in the Foundry portal and with the Python SDK: built-in tools, the agent execution model, threads, runs, and run status lifecycle.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for single-agent solutions are maintained in the dedicated guide: **[Lightweight & Agent Clients Guide](Lightweight-and-Agent-Clients.md)**.

### 7.1 Single-Agent — Concepts and Portal
- Core distinction: chat client (stateless) vs. agent (tools + managed state + orchestration loop).
- Built-in tools: Code Interpreter (Hyper-V sandbox, 1 hr session), File Search (vector search), Web Search, Function Calling.
- Portal path: Agents → New Agent → configure name, model, instructions, tools, knowledge → test in Agent Playground.

### 7.2 Single-Agent — Python SDK
- Core primitives: agent (configuration + tools), thread (server-side conversation state), message, run.
- Run status lifecycle: `queued` → `in_progress` → `requires_action` / `completed` / `failed` / `cancelled`.
- `requires_action`: signals the agent is waiting for a custom function result — does NOT apply to Code Interpreter.

---

## 8. Lightweight Client Application for an Agent

Detailed study guide notes on wiring a lightweight client that delegates all state management to the Foundry Agent Service: polling loop, run lifecycle, thread isolation per session, and comparison with direct chat clients.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for the agent client are maintained in the dedicated guide: **[Lightweight & Agent Clients Guide](Lightweight-and-Agent-Clients.md)**.

### 8.1 Lightweight Agent Client
- State management: server-side Thread vs. client-side `conversation_history` list.
- Polling pattern: `_poll_run()` loop until `COMPLETED`, `FAILED`, or `CANCELLED`.
- Session isolation: one thread per user session; delete thread on session end.
- Cleanup responsibility: delete threads to avoid resource accumulation.

---

## 9. Text Analysis & Speech

Detailed study guide notes on building lightweight text analysis applications with Azure AI Language, the spoken-prompt architecture (STT → GPT-4o → TTS), and Azure AI Speech SDK patterns.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for text analysis and speech are maintained in the dedicated guide: **[Text Analysis & Speech Guide](Text-Analysis-and-Speech.md)**.

### 9.1 Text Analysis — Azure AI Language
- Core capabilities: Key phrase extraction, Named Entity Recognition (NER), sentiment analysis (with opinion mining), abstractive/extractive summarization.
- Exam trap: Text analysis ≠ information extraction. Text analysis = unstructured prose. Information extraction = structured forms/fields.
- SDK pattern: `TextAnalyticsClient`, batch processing, per-document error handling.

### 9.2 Spoken Prompts with a Multimodal Model
- Architecture: Azure AI Speech STT → text to GPT-4o → response text to Azure AI Speech TTS → spoken audio.
- GPT-4o accepts text + image, NOT raw audio — speech wrapping is mandatory.
- `SpeechRecognizer` for input; `SpeechSynthesizer` for output; `en-US-AriaNeural` neural voice.

### 9.3 Lightweight Speech Application
- STT from microphone: `AudioConfig(use_default_microphone=True)`.
- STT from file: `AudioConfig(filename=...)` for pre-recorded audio.
- Real-time translation: `TranslationRecognizer` with source + target language config.
- Batch transcription: REST API for large-volume async processing.

---

## 10. Computer Vision & Image Generation

Detailed study guide notes on GPT-4o multimodal visual input, DALL-E 3 / gpt-image-1 image generation, and Azure AI Vision structured image analysis.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for computer vision and image generation are maintained in the dedicated guide: **[Computer Vision & Image Generation Guide](Computer-Vision-and-Image-Generation.md)**.

### 10.1 Interpreting Visual Input — GPT-4o Multimodal
- Multimodal = text + image in the same prompt → text output.
- Two encoding methods: base64 local file, or public URL.
- `detail` parameter: `"low"` (85 tokens, fast), `"high"` (up to 1,105 tokens, detailed analysis).
- Exam signal: "reason about an image" + natural language question → GPT-4o multimodal.

### 10.2 Image Generation — DALL-E 3 and gpt-image-1
- DALL-E 3: text prompt → new image. Parameters: size, quality (`standard`/`hd`), style (`vivid`/`natural`), n=1 only.
- gpt-image-1: text + optional image → new/edited image (inpainting, variation).
- `revised_prompt` in response: DALL-E 3 rewrites prompts for quality/safety — expected behavior.
- Exam signal: "generate a new image" → DALL-E 3 / gpt-image-1, NOT GPT-4o.

### 10.3 Lightweight Vision Application
- Azure AI Vision: structured output (labels, bounding boxes, OCR text) — NOT natural language reasoning.
- `VisualFeatures.READ` = OCR. `VisualFeatures.OBJECTS` = object detection.
- Decision rule: "reason about what you see" → GPT-4o. "count/locate/extract text" → Azure AI Vision.

---

## 11. Information Extraction

Detailed study guide notes on extracting structured fields from documents, forms, images, audio, and video using Azure Content Understanding.

> [!NOTE]
> All detailed study guides, Python implementations, exam questions, and coding challenges for information extraction are maintained in the dedicated guide: **[Information Extraction Guide](Information-Extraction.md)**.

### 11.1 What Is Information Extraction?
- Purpose: extract specific named fields from structured/semi-structured sources as JSON.
- Service: Azure Content Understanding (successor to Azure Form Recognizer / Document Intelligence).
- Modalities: Documents/PDFs, images (photos of forms), audio recordings, video files.
- Exam trap: OCR = all text verbatim. Information extraction = specific named fields from known structure.

### 11.2 Documents and Forms
- SDK: `DocumentIntelligenceClient` + `begin_analyze_document()`.
- Prebuilt models: `prebuilt-invoice`, `prebuilt-receipt`, `prebuilt-idDocument`, `prebuilt-contract`, `prebuilt-read`.
- Confidence scores: flag low-confidence fields for human review before ERP ingestion.

### 11.3 Images
- Same SDK and API as documents — Content Understanding handles the modality automatically.
- Common use: photographed government IDs (`prebuilt-idDocument`), handwritten expense photos (custom model).

### 11.4 Audio and Video
- REST API pattern: submit job → poll `operation-location` header until `succeeded`.
- Audio: transcribes + extracts fields in one pipeline (no separate STT step).
- Video: transcripts, key moments, chapter segments, visual scene analysis.
- Exam signal: "extract fields from audio/video" → Content Understanding, not STT + Language.

---

## 12. Original Syllabus

Extracted from Microsoft Study Guides for the Microsoft AI-901 Certification (Skills measured as of April 15, 2026).

### 5.1 Audience Profile & Skills at a Glance

**Candidate Profile:** Designed for candidates at the beginning of their career in AI solution development. Candidates should have conceptual knowledge of AI solutions in Azure, Foundational technical skills to work with them, knowledge of Python coding syntax, and familiarity with Azure resources.

**Skills at a Glance:**
- Identify AI concepts and responsibilities (40–45%)
- Implement AI solutions by using Microsoft Foundry (55–60%)

### 5.2 Identify AI Concepts and Capabilities (40–45%)

**Describe principles of responsible AI:**
- Describe considerations for fairness in an AI solution
- Describe considerations for reliability and safety in an AI solution
- Describe considerations for privacy and security in an AI solution
- Describe considerations for inclusiveness in an AI solution
- Describe considerations for transparency in an AI solution
- Describe considerations for accountability in an AI solution

**Identify AI model components and configurations:**
- Describe how generative AI models work
- Identify an appropriate AI model, based on capabilities
- Identify appropriate model deployment options and configuration parameters

**Identify AI workloads:**
- Identify scenarios for common AI workloads, including generative and agentic AI, text analysis, speech, computer vision, and information extraction
- Describe common text analysis techniques, including keyword extraction, entity detection, sentiment analysis, and summarization
- Identify features and capabilities of speech recognition and speech synthesis
- Identify features and capabilities of computer vision and image-generation models
- Identify techniques to extract information from text, images, audio, and videos

### 5.3 Implement AI Solutions by using Microsoft Foundry (55–60%)

**Implement generative AI apps and agents by using Foundry:**
- Create effective system and user prompts for generative AI models
- Deploy a model and interact with it in the Foundry portal
- Create a lightweight chat client application by using the Foundry SDK
- Create and test a single-agent solution in the Foundry portal
- Create a lightweight client application for an agent

**Implement AI solutions for text and speech by using Foundry:**
- Build a lightweight application that includes text analysis
- Respond to spoken prompts by using a deployed multimodal model
- Build a lightweight application by using Azure Speech in Foundry Tools

**Implement AI solutions with computer vision and image-generation capabilities by using Foundry:**
- Interpret visual input in prompts by using a deployed multimodal model
- Create new visual outputs by using generative models
- Build a lightweight application that includes vision capabilities

**Implement AI solutions for information extraction by using Foundry:**
- Extract information from documents and forms by using Azure Content Understanding in Foundry Tools
- Extract information from images by using Content Understanding
- Extract information from audio and video by using Content Understanding
- Build a lightweight application with information extraction capabilities by using Content Understanding

---

2026 - English