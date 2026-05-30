# AI Workload Types

This module is conceptual but high-stakes on the exam. The questions are typically scenario-based: given a business problem, which AI workload type applies? Getting this wrong means misidentifying the right Azure service in Phase 2 and 3. The workload taxonomy is also how Microsoft organizes the entire Foundry toolset.

The exam requires you to identify scenarios for common AI workloads including generative and agentic AI, text analysis, speech, computer vision, and information extraction.

There are six workload categories. We cover each one below: what it is, what problem it solves, what Azure service implements it, and the exam scenario signals to recognize.

---

## 1. Workload Categories & Scenarios

### 1.1 Generative AI

**The Core Idea:** Generative AI workloads produce new content — text, images, code, audio — rather than classifying or analyzing existing content. The defining characteristic is that the output did not previously exist and is created by the model in response to a prompt.

**Common Scenarios:** Drafting documents, writing code, summarizing long content, answering open-ended questions, generating images from text descriptions, translating between languages with context awareness.

**Azure Implementation:** `Azure OpenAI` models (GPT-4o, GPT-5) and the broader Foundry model catalog. Deployed via the Foundry portal and accessed through the Chat Completions API or the Foundry SDK.

**Exam Signal:** If the scenario involves *creating* something new from a natural language description or prompt, it is a generative AI workload.

### 1.2 Agentic AI

**The Core Idea:** Agentic AI is a specialization of generative AI where the model does not just respond to a single prompt — it reasons across multiple steps, uses tools, takes actions, and pursues a goal autonomously.

**Key Distinctions:**
- Tools — Functions it can call, APIs it can invoke, and data sources it can query.
- Memory — Context it maintains across execution steps.
- Planning Loop — Decides what to do next based on intermediate results.

> By combining live voice capabilities with agent orchestration, Azure enables developers to move from simple conversational bots to interactive AI systems capable of reasoning and acting in real time.

**Common Scenarios:** An agent that reads your email, summarizes action items, creates calendar events, and sends replies — all as part of a single goal. A research agent that searches the web, reads documents, synthesizes findings, and produces a report. A customer service agent that looks up order status in a database, processes a refund, and sends a confirmation email.

**Azure Implementation:** `Azure AI Foundry Agent Service`.

**Exam Signal:** If the scenario involves *multi-step autonomous reasoning, tool use, or taking actions across systems*, it is an agentic AI workload — not just a generative AI one.

### 1.3 Text Analysis

**The Core Idea:** Text analysis workloads process existing text to extract structured information or insights. The model does not generate new content — it reads and analyzes what is already there.

**Azure Implementation:** `Azure AI Language` in Foundry Tools is a cloud-based service that provides natural language processing features for understanding and analyzing text, including named entity recognition, sentiment analysis, language detection, summarization, and question answering.

**Core Techniques:**
- Keyword extraction — Key phrase extraction is a preconfigured feature that evaluates and returns the main concepts in unstructured text as a list. Use case: automatically tagging support tickets, indexing documents.
- Entity detection (Named Entity Recognition) — Named entity recognition identifies different entries in text and categorizes them into predefined types such as people, organizations, locations, and dates. Use case: extracting company names from news articles, identifying patients in clinical notes.
- Sentiment analysis — Sentiment analysis and opinion mining are preconfigured features that help you understand public perception of your brand or topic. These features analyze text to identify positive or negative sentiments and can link them to specific elements within the text. Use case: analyzing customer reviews, monitoring social media mentions.
- Summarization — Text summarization supports two approaches: extractive summarization creates a summary by selecting key sentences from the document and preserving their original positions. Abstractive summarization generates a summary by producing new, concise, and coherent sentences or phrases that are not directly copied from the original document. Use case: condensing call center transcripts, summarizing long reports.

**Exam Signal:** If the scenario involves understanding or extracting insights *from existing text without generating new content*, it is a text analysis workload.

### 1.4 Speech

**The Core Idea:** Speech workloads convert between audio and text, or perform analysis on spoken content. `Azure Speech` in Foundry Tools provides APIs for speech-to-text, text-to-speech, translation, and speaker recognition.

**Core Capabilities:**
- Speech-to-text (STT) — Converts audio into text. Supports real-time transcription for streaming audio, fast transcription for pre-recorded audio files, and batch transcription for processing large volumes of audio asynchronously.
- Text-to-speech (TTS) — Converts written text into spoken audio using neural voices. Microsoft uses Azure Speech for scenarios such as captioning in Microsoft Teams, dictation in Microsoft Office 365, and Read Aloud in the Microsoft Edge browser.
- Speech translation — Converts spoken audio in one language into text or speech in another language in real time.
- Speaker recognition — Identifies *who* is speaking based on voice characteristics, distinct from recognizing *what* is being said.

**Common Scenarios:** Call center transcription and analysis, voice-enabled assistants, meeting transcription, accessibility features for hearing-impaired users, pronunciation assessment for language learning.

**Exam Signal:** If the scenario involves audio input or output — converting speech to text, text to speech, or analyzing spoken content — it is a speech workload.

### 1.5 Computer Vision

**The Core Idea:** Computer vision workloads process visual input — images or video — to extract information or generate insights. `Azure Vision` in Foundry Tools provides advanced algorithms that process images and return information based on visual features. It includes optical character recognition, image analysis, and face detection capabilities.

**Core Capabilities:**
- Image classification — Assigns a label to an entire image. Is this a cat or a dog? Is this a defective product or a good one? Use case: automated quality control on a manufacturing line.
- Object detection — Identifies and locates specific objects within an image, returning bounding box coordinates. Unlike classification, it can find multiple objects in a single image. Use case: detecting pedestrians in autonomous vehicle feeds, finding products on retail shelves.
- OCR (Optical Character Recognition) — Extracts text from images including business documents, invoices, receipts, posters, business cards, letters, and whiteboards, supporting both printed and handwritten text in several languages.
- Facial detection and analysis — Face detection determines important regions in an image. The standard Image Analysis service does not involve distinguishing one face from another, predicting facial attributes, or creating facial templates. The dedicated Azure AI Face service handles identity verification and liveness detection.
- Image generation — Multimodal models like DALL-E 3 and gpt-image-1 generate new images from text prompts. This is the generative AI workload applied to visual output rather than text.

**Exam Signal:** If the scenario involves processing images or video to extract structured information (labels, coordinates, text, faces), it is a computer vision workload. If it involves *creating* a new image from a prompt, it is generative AI applied to vision.

### 1.6 Information Extraction

**The Core Idea:** Information extraction workloads process structured and semi-structured documents (forms, invoices, contracts, PDFs) to extract specific fields and their values in a machine-readable format.

> The distinction from text analysis is important for the exam. Text analysis works on unstructured prose. Information extraction works on documents that have a structure — even if that structure is visual rather than coded in markup.

**Azure Implementation:** `Azure Content Understanding` in Foundry Tools. This is the new, unified service for AI-901 that handles documents, forms, images, audio, and video.

**Common Scenarios:** Extracting line items from invoices, reading fields from scanned government forms, pulling dates and parties from contracts, indexing large document archives.

**Exam Signal:** If the scenario involves *pulling specific structured fields from documents or forms*, it is an information extraction workload, not general text analysis.

---

## 2. The Workload Map

This table maps the core workload types, input/output structures, and primary Azure services.

| Workload | Input | Output | Azure Service |
|:---------|:--------:|----------:| ---| 
| Generative AI | Prompt (text) | Generated content | Azure OpenAI / Foundry Models |
| Agentic AI | Goal + tools | Actions + results | Foundry Agent Service |
| Text Analysis | Existing text | Insights, labels, entities | Azure Language in Foundry Tools |
| Speech | Audio / Text | Text / Audio | Azure Speech in Foundry Tools |
| Computer Vision | Images / Video | Labels, coordinates, text | Azure Vision in Foundry Tools |
| Information Extraction | Documents / Forms | Structured fields | Azure Content Understanding |

---

## 3. Exam Questions: AI Workload Types

<details>
<summary><b>Question 1: Shipping Label Processing</b></summary>

A logistics company wants to automatically read the sender address, recipient address, and tracking number from photographs of shipping labels and store them in a database. Which AI workload type best describes this scenario?  
- **A)** Text analysis, because it involves extracting text from content  
- **B)** Computer vision with OCR, because it involves extracting text from images  
- **C)** Information extraction, because it involves pulling structured fields from a document  
- **D)** Generative AI, because the system must interpret visual content  

*Answer:* **C** — While OCR is the underlying mechanism, the scenario describes extracting specific structured fields (sender, recipient, tracking number) from a semi-structured document (a shipping label). That maps to information extraction. OCR alone would just return all the text; information extraction maps that text to specific fields. In Azure, this is the Content Understanding workload.
</details>

<details>
<summary><b>Question 2: Sentiment and Customer Draft replies</b></summary>

A financial services company deploys an AI system that monitors customer support emails, automatically identifies the sentiment and key topics, routes negative-sentiment messages to senior agents, and drafts a suggested reply for the agent to review. Which combination of workload types does this system use?  
- **A)** Generative AI and Computer Vision  
- **B)** Text Analysis and Generative AI  
- **C)** Agentic AI and Speech  
- **D)** Information Extraction and Text Analysis  

*Answer:* **B** — Analyzing sentiment and extracting key topics from emails is text analysis. Drafting a suggested reply is generative AI. The routing logic is standard application code, not an AI workload. Note: if the system were doing all of this autonomously across multiple tools without human review, it would be agentic AI — but the scenario specifies human review of the draft, making it generative AI plus text analysis.
</details>

<details>
<summary><b>Thinking Challenge: Hospital, Retail, Legal, and Tax Scenarios</b></summary>

For each of the following scenarios, identify the primary AI workload type and the specific Azure service in Foundry that would implement it. If multiple workload types apply, rank them by primacy.

**Scenario A:** A hospital transcribes doctor-patient consultations in real time and automatically extracts medication names, dosages, and diagnoses to populate the patient record.
* *Primacy:* Speech (Speech-to-Text for real-time transcription) is primary. Text Analysis (NER for medical entities) is secondary.
* *Azure Service:* `Azure Speech` and `Azure Language` in Foundry Tools.

**Scenario B:** A retail chain wants to count how many customers are in each store section at any given time using existing CCTV cameras.
* *Primacy:* Computer Vision (Object Detection to locate and count "person" classes).
* *Azure Service:* `Azure Vision` in Foundry Tools.

**Scenario C:** A law firm wants an AI that can receive a client's question by voice, research relevant case law, draft a response, and read it back to the client — all without human intervention between steps.
* *Primacy:* Agentic AI (orchestrating search tools, plan execution, and memory). Speech (STT voice input, TTS read back) and Generative AI (brief drafting) are supporting.
* *Azure Service:* `Azure AI Foundry Agent Service` orchestrating `Azure OpenAI` and `Azure Speech`.

**Scenario D:** A government agency scans thousands of paper tax forms and needs to extract the taxpayer ID, declared income, and filing date from each one.
* *Primacy:* Information Extraction (pulling predefined key-value fields from a visual document layout).
* *Azure Service:* `Azure Content Understanding` in Foundry Tools.
</details>
