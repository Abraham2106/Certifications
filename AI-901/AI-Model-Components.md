# AI Model Components

Detailed study notes, model selection guides, deployment options, and configuration parameters for the AI-901 certification.

---

## 1. How Generative Models Work

Every generative AI model in Azure is a next-token prediction machine. It takes input sequences of tokens and predicts the most statistically probable next token, repeating this process autoregressively to construct complete responses.

### 1.1 Tokenization

**The Core Process:** Tokenization decomposes input text into unique subword units, which are the baseline units of text value handled by the model.

**Key Concepts:**
- A Token — Subword units averaging 3 to 4 characters in English. Punctuation, spaces, and special characters are also unique tokens.
- Context Window — The maximum token input/output boundary a model can process at once. Exceeding this boundary causes the model to lose the beginning of the conversation.
- Token Embeddings — Mathematical vector representations of meaning that translate text tokens into numerical space.

### 1.2 Embeddings

**The Core Process:** An embedding is a high-dimensional vector that represents a token's semantic meaning. Tokens representing similar concepts end up geometrically close to one another within this high-dimensional space.

**Key Concepts:**
- Semantic Relationships — The spatial vector arithmetic capturing abstract relationships (e.g., "King" minus "Man" plus "Woman" yields a vector near "Queen").
- Vector Databases — Databases storing embeddings to query geometrically close documents, forming the foundation of Retrieval-Augmented Generation (RAG).

### 1.3 The Transformer and Attention

**The Core Process:** Transformer models use encoder blocks to generate contextual embeddings by analyzing relationships between tokens.

**Key Concepts:**
- Attention Mechanism — A technique examining each token to calculate weights representing how much it is influenced by surrounding tokens.
- Multi-Head Attention — Parallel evaluation of multiple token features to calculate updated embedding vector values.
- Feed-Forward Networks (FFN) sits between attention layers to act as the primary repository for the model's factual knowledge parameters.

### 1.4 Generation

**The Core Process:** The model runs a loop to evaluate output probability distributions, sample a token, append it to the context, and repeat the process to generate text.

**Key Concepts:**
- Autoregressive Loop — The sequential processing loop where predicting longer outputs results in higher token consumption and latency.
- Temperature Scaling — Scaling applied to the probability distribution to shift the randomness of the selected tokens.

### 1.5 The BERT vs. GPT Architecture Split

**Architecture Types:** Understanding the directional flow of models determines their optimization for understanding or generation.

| Architecture | Direction | Primary Use | Azure Example |
|:---------|:--------:|----------:|--------- |
| Encoder-only (BERT-style) | Bidirectional | Classification, embeddings, search | Azure AI Language models |
| Decoder-only (GPT-style) | Left-to-right | Text generation, chat, completion | GPT-4o, Phi-4 in Foundry |
| Encoder-Decoder (T5-style) | Both | Translation, summarization | Azure Translation models |

---

## 2. Selecting the Right Model

Selecting the appropriate catalog model requires balancing the specific reasoning capacity required against operational efficiency, speed, and cost.

**Selection Criteria by Capability:**
- Text Generation & Reasoning — Large models optimized for complex tasks (e.g., GPT-4o, GPT-5, Phi-4).
- Multimodal Processing — Models integrating multiple input types like text and image prompts simultaneously (e.g., GPT-4o).
- Image Generation — Diffusion and generative visual models (e.g., DALL-E 3, FLUX).
- Speech & Translation — Audio interpretation and language translation services (e.g., Whisper, Azure Translator).
- Cost-Efficient Processing — Small language models (SLMs) optimized for speed and low cost (e.g., Microsoft Phi-4, Phi-3.5).

**Cost vs. Capability Trade-Off:** Reaching for the largest reasoning model by default is an architectural anti-pattern. Smaller models (e.g., Phi-4) should be deployed for simple extraction, classification, or formatting tasks to minimize token consumption costs.

---

## 3. Deployment Options and Configuration Parameters

Deploying models on Azure requires selecting the appropriate billing structure and configuring generation parameters.

### 3.1 Deployment Options

**Available Models:**
- Serverless API (MaaS) — Pay-as-you-go billing based on token consumption without provisioning dedicated GPU infrastructure. Ideal for variable or unpredictable workloads.
- Managed Compute — Provisioning a dedicated virtual machine hosting open-weight models (e.g., Llama, Phi). Billing is hourly, making it ideal for high-volume workloads and fine-tuning.

### 3.2 Configuration Parameters

**Available Controls:**
- Temperature — Controls vocabulary randomness from 0 (fully deterministic) to 2 (highly creative). Use low values for factual tasks and high values for brainstorming.
- Top-p — Cumulative probability thresholding that restricts token selection to the top percentage mass. Avoid modifying temperature and top-p simultaneously.
- Max Tokens — Hard upper bound on completion length designed to manage token costs.
- Penalties — Frequency and presence penalties are used to minimize word and topic repetition in long-form generation.

| Parameter | Controls | Use case |
|:---------|:--------:|----------:|
| Temperature | Randomness / creativity | Low for facts, higher for creativity |
| Top-p | Sampling vocabulary breadth | Alternative to temperature |
| Max tokens | Output length and cost | Always set to prevent runaway outputs |
| Frequency penalty | Word repetition | Long-form content |
| Presence penalty | Topic repetition | Diverse, exploratory outputs |

### 3.3 Exam Questions: Model Components

<details>
<summary><b>Question 1: Chatbot Output Consistency</b></summary>

A developer is building a customer support chatbot that must provide consistent, accurate answers about product return policies. The responses must be deterministic — the same question should always produce the same answer. Which configuration parameter should the developer set, and to what value?  
- **A)** Top-p set to 1.0 to maximize vocabulary breadth  
- **B)** Temperature set to 0 to make the model fully deterministic  
- **C)** Max tokens set to 0 to prevent any generation  
- **D)** Frequency penalty set to 2.0 to eliminate repetition  

*Answer:* **B** — Setting temperature to 0 forces the model to select the highest-probability token every time, making the response generation fully deterministic.
</details>

<details>
<summary><b>Question 2: Dedicated GPU Fine-Tuning</b></summary>

A company needs to deploy an open-weight Llama model that will be fine-tuned on proprietary internal documents. The workload is continuous and high-volume, processing thousands of requests per hour. Which deployment option in Azure AI Foundry is most appropriate?  
- **A)** Serverless API, because it requires no infrastructure management and scales automatically  
- **B)** Managed Compute, because it provides dedicated GPU infrastructure suitable for fine-tuning and high-volume predictable workloads  
- **C)** Standard deployment, because it is the default option in the Foundry portal  
- **D)** Serverless API, because fine-tuning is only supported in the pay-per-token model  

*Answer:* **B** — Managed Compute provides dedicated VM/GPU resources, enabling custom model training (fine-tuning) and handling predictable, high-volume operational workloads cost-effectively.
</details>

<details>
<summary><b>Thinking Challenge: Legal Tech Startup Products</b></summary>

- **Product A (Legal Brief Assistant):**
  - Model Choice — GPT-4o. Legal brief generation requires high-level reasoning, long-range context handling, and sophisticated writing capability.
  - Deployment Option — Serverless API. Workloads for drafting assistant interfaces are typically variable and do not require custom infrastructure.
  - Temperature Setting — Medium-high (0.5 to 0.7) to allow creative argumentation while keeping factual case citations grounded.
- **Product B (PDF Clause Extractor):**
  - Model Choice — Phi-4 or Phi-3.5 (SLM). Structural extraction of text blocks into structured JSON is a low-reasoning task that SLMs handle with high speed and low cost.
  - Deployment Option — Managed Compute. Processing 10,000 contracts daily represents a high, continuous workload where dedicated instances are cheaper than pay-per-token.
  - Temperature Setting — 0. Deterministic extraction is required; any creativity could result in hallucinated dates or names.
</details>
