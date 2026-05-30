# Prompt Engineering

Detailed study notes, prompt design principles, Python code implementation patterns, and error handling structures for the AI-901 certification.

---

## 1. System and User Prompts

This module shifts from conceptual understanding to practical implementation. Understanding how prompts structure model behavior is essential to deploy working AI applications.

### 1.1 The Message Structure

**The Core Process:** When calling a chat completion model in Azure AI Foundry, the input is structured as an array of message objects, each assigned a specific role rather than a single text string.

**Available Roles:**
- `system` — Instructions that define how the model should behave for the entire conversation. This is set by the developer and is hidden from the end user.
- `user` — The input text or query sent by the person interacting with the model.
- `assistant` — The model's previous generated completions, which are appended back to the messages array to provide context.

> The model has no persistent memory between API calls. Every call must include the full conversation history you want the model to reason over.

```python
# Messages array for a typical single-turn call
messages = [
    {"role": "system", "content": "You are a helpful assistant..."},
    {"role": "user",   "content": "What is the return policy?"}
]

# Messages array for a multi-turn conversation maintaining context history
messages = [
    {"role": "system",    "content": "You are a customer support agent..."},
    {"role": "user",      "content": "I want to return a product."},
    {"role": "assistant", "content": "I can help with that. What is your order number?"},
    {"role": "user",      "content": "Order 12345."}
]
```

### 1.2 System Prompts — Design Principles

**System Message Failure Modes:** System messages can overfit to specific examples, fail on edge cases, contain conflicting instructions (such as "be brief" and "be comprehensive" without prioritization), be overly long (consuming the context window), or hide critical formatting requirements.

**Four Core Components of a System Prompt:**
- Role definition — Assigning a specific, domain-focused identity to the assistant (e.g., "billing platform assistant").
- Behavioral constraints — Explicitly defining out-of-scope queries and expected model rejection behavior.
- Output format — Explaining exact downstream serialization needs (e.g., structured JSON format).
- Safety grounding — Instructing the model how to handle sensitive inputs and avoid jailbreak attempts.

### 1.3 User Prompts — Design Principles

**In-Context Learning:** User prompts leverage the model's in-context reasoning ability using zero-shot, few-shot, or reasoning instructions.

**Core User Prompting Techniques:**
- Zero-shot — Passing prompts without any example completions, relying entirely on the model's pre-trained knowledge to control token costs.
- Few-shot — Embedding one or more input-output pairs in the prompt or conversation history to demonstrate the desired format.
- Chain-of-thought — Directing the model to output its reasoning step-by-step before returning the final solution.
- Grounding — Incorporating factual reference source documents directly into the prompt to limit hallucinations.

**Practical Guidelines:** Be specific, restrict the operational space, use descriptive analogies, repeat critical instructions, and pay attention to order sequence.

---

## 2. Code Implementation with Azure AI Foundry SDK

This section covers hands-on Python implementation using the `azure-ai-projects` SDK.

### 2.1 Authentication & Configuration

The Foundry SDK uses `DefaultAzureCredential` from `azure-identity` to securely authenticate against endpoints without hardcoded keys.

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

credential = DefaultAzureCredential()

client = AIProjectClient(
    endpoint=os.environ["AZURE_AI_FOUNDRY_ENDPOINT"],  # e.g. https://<hub>.api.azureml.ms
    credential=credential,
)
```

For local development or testing, you can use the direct `openai` library setup:

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-12-01-preview"
)
```

### 2.2 Zero-Shot Generation Call

```python
response = client.chat.completions.create(
    model="gpt-4o",  # your deployment name in Foundry
    messages=[
        {
            "role": "system",
            "content": (
                "You are a customer support assistant for Contoso Airlines. "
                "Only answer questions about flight bookings, cancellations, and baggage policy. "
                "If asked about anything else, say: 'I can only help with airline-related questions.' "
                "Always respond in plain, clear English. Keep responses under 150 words."
            )
        },
        {
            "role": "user",
            "content": "Can I bring a guitar as carry-on luggage?"
        }
    ],
    temperature=0.3,
    max_tokens=200
)

print(response.choices[0].message.content)
```

### 2.3 Few-Shot Structured JSON Extraction Call

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "system",
            "content": (
                "You extract sentiment from customer reviews. "
                "Respond only with a JSON object containing 'sentiment' "
                "(positive, negative, or neutral) and 'confidence' (high, medium, or low)."
            )
        },
        # Few-shot examples embedded as prior turns
        {
            "role": "user",
            "content": "The flight was smooth and the crew was wonderful."
        },
        {
            "role": "assistant",
            "content": '{"sentiment": "positive", "confidence": "high"}'
        },
        {
            "role": "user",
            "content": "The delay was unacceptable and nobody explained what was happening."
        },
        {
            "role": "assistant",
            "content": '{"sentiment": "negative", "confidence": "high"}'
        },
        # The actual input
        {
            "role": "user",
            "content": "The food was okay but nothing special."
        }
    ],
    temperature=0.0,  # deterministic — we want consistent JSON
    max_tokens=50
)

print(response.choices[0].message.content)
# Expected: {"sentiment": "neutral", "confidence": "medium"}
```

### 2.4 Grounded Context Prompt Call

```python
policy_document = """
Contoso Airlines Baggage Policy (updated March 2026):
- Carry-on: 1 bag up to 10kg and 55x40x20cm.
- Musical instruments: allowed as carry-on if within size limits.
  Oversized instruments require a separate seat purchase.
- Checked baggage: first bag free on international flights, 
  $35 fee on domestic flights.
"""

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "system",
            "content": (
                "You are a Contoso Airlines support assistant. "
                "Answer questions using ONLY the policy document provided. "
                "If the answer is not in the document, say: "
                "'I don't have that information in our current policy.' "
                "Do not use any outside knowledge."
            )
        },
        {
            "role": "user",
            "content": (
                f"Policy document:\n{policy_document}\n\n"
                "Question: Can I bring my guitar as carry-on?"
            )
        }
    ],
    temperature=0.0,
    max_tokens=150
)

print(response.choices[0].message.content)
```

### 2.5 Error Handling Implementation

```python
from openai import AzureOpenAI, APIError, RateLimitError, AuthenticationError

try:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user",   "content": "Hello"}
        ],
        temperature=0.7,
        max_tokens=100
    )
    print(response.choices[0].message.content)

except AuthenticationError as e:
    print(f"Authentication failed — check your API key or credential: {e}")

except RateLimitError as e:
    print(f"Rate limit hit — implement exponential backoff retry: {e}")

except APIError as e:
    print(f"Azure OpenAI API error: {e.status_code} — {e.message}")
```

---

## 3. Common System Prompt Failure Patterns

Understanding standard system message issues helps secure deployment safety.

| Failure | What happens | Fix |
|:---------|:--------:|----------:|
| No role definition | Model behaves generically, ignores your use case | Add explicit role and domain scope |
| Conflicting instructions | Model picks one instruction and ignores the other | Prioritize explicitly: "If X, do Y; otherwise do Z" |
| Overly long system prompt | Consumes context window, less room for user content | Be concise; move examples to few-shot turns |
| No output format specified | Inconsistent response structure, breaks downstream parsing | State format explicitly: "Respond only in JSON" |
| No out-of-scope handling | Model answers anything, including off-topic requests | Explicitly define the rejection behavior |

---

## 4. Exam Practice: Prompt Engineering

<details>
<summary><b>Question 1: Contradictory Instructions</b></summary>

A developer is building a legal document assistant. The system prompt instructs the model to "be comprehensive and thorough in all responses" and also to "keep responses brief and easy to understand." After deployment, responses are inconsistent — sometimes very long, sometimes very short. What is the most likely cause?  
- **A)** The temperature setting is too high, causing random output length  
- **B)** The system prompt contains conflicting instructions without prioritization  
- **C)** The model does not support long system prompts  
- **D)** Few-shot examples are missing from the user prompts  

*Answer:* **B** — The prompt provides conflicting directives without outlining priority constraints. Setting explicit guidelines is required to resolve such contradictions consistently.
</details>

<details>
<summary><b>Question 2: JSON Formatting Consistency</b></summary>

A developer needs a model to extract product names and prices from customer receipts and return them in a consistent JSON format. In testing, zero-shot prompting produces correct JSON about 60% of the time but inconsistent formats otherwise. What is the most effective next step?  
- **A)** Increase the temperature to make the model more creative with output formats  
- **B)** Switch to a larger model to improve accuracy  
- **C)** Add few-shot examples showing the exact input-output JSON format to the messages array  
- **D)** Add chain-of-thought instructions to help the model reason about the receipt  

*Answer:* **C** — Few-shot examples demonstrating the exact desired formatting are highly effective for teaching the model formatting constraints, improving output structure reliability.
</details>

<details>
<summary><b>Code Challenge: Support Ticket Categorization</b></summary>

Write a Python function `analyze_support_ticket(ticket_text: str) -> dict` that uses Azure OpenAI to classify support tickets.

```python
import os
import json
import time
from openai import AzureOpenAI, RateLimitError, APIError

def analyze_support_ticket(ticket_text: str) -> dict:
    client = AzureOpenAI(
        azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
        api_key=os.environ["AZURE_OPENAI_API_KEY"],
        api_version="2024-12-01-preview"
    )
    
    # Establish a strict system prompt to enforce category formatting
    system_prompt = (
        "You are an AI support ticket routing assistant. "
        "Your task is to classify incoming customer support tickets into one of three categories: "
        "'billing', 'technical', or 'general'. "
        "You must respond ONLY with a valid JSON object containing exactly two keys: "
        "'category' (which must be 'billing', 'technical', or 'general') and "
        "'confidence' (which must be 'high', 'medium', or 'low'). "
        "Do not include any pre-text, markdown fences, or post-text."
    )
    
    # Using few-shot examples inside the message history to establish JSON shape
    messages = [
        {"role": "system", "content": system_prompt},
        # Example 1
        {"role": "user", "content": "I was charged twice on my credit card this month."},
        {"role": "assistant", "content": '{"category": "billing", "confidence": "high"}'},
        # Example 2
        {"role": "user", "content": "The application crashes every time I click save."},
        {"role": "assistant", "content": '{"category": "technical", "confidence": "high"}'},
        # Input Ticket
        {"role": "user", "content": ticket_text}
    ]
    
    max_retries = 3
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                temperature=0.0,  # Zero randomness enforces formatting structure
                max_tokens=50
            )
            raw_content = response.choices[0].message.content.strip()
            return json.loads(raw_content)
            
        except RateLimitError as e:
            if attempt == max_retries - 1:
                print("Rate limit reached. All retries exhausted.")
                raise e
            time.sleep(2 ** attempt)  # Simple exponential backoff retry mechanism
            
        except APIError as e:
            print(f"Azure OpenAI API error occurred: {e.message}")
            raise e

# System Prompt Design Choices Justification:
# 1. Strict Formatting Constraint: Explicitly limits the output to a JSON object without markdown fences, avoiding parsing errors.
# 2. Output Restrictive Vocabulary: Restricts allowed category values to ['billing', 'technical', 'general'] and confidence values to ['high', 'medium', 'low'].
# 3. Few-shot Embedding: Solidifies parsing reliability by showing exact structured responses.
# 4. Temperature Zero: Eliminates output sampling variety to maintain strict format compliance.
```
</details>
