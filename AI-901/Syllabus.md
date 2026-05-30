# Syllabus and Study Guide

**Certification Code:** AI-901  
**Issuing Organization:** Microsoft  

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

## 3. Original Syllabus

Extracted from Microsoft Study Guides for the Microsoft AI-901 Certification (Skills measured as of April 15, 2026).

### 3.1 Audience Profile & Skills at a Glance

**Candidate Profile:** Designed for candidates at the beginning of their career in AI solution development. Candidates should have conceptual knowledge of AI solutions in Azure, Foundational technical skills to work with them, knowledge of Python coding syntax, and familiarity with Azure resources.

**Skills at a Glance:**
- Identify AI concepts and responsibilities (40–45%)
- Implement AI solutions by using Microsoft Foundry (55–60%)

### 3.2 Identify AI Concepts and Capabilities (40–45%)

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

### 3.3 Implement AI Solutions by using Microsoft Foundry (55–60%)

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