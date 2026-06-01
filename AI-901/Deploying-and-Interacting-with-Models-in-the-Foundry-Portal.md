# Deploying and Interacting with Models in the Foundry Portal

Detailed study notes on Microsoft Foundry architecture hierarchy, model deployment steps, playground usage, and programmatic authentication patterns for the AI-901 certification.

---

## 1. The Foundry Portal Architecture

Microsoft Foundry consolidates several previous Azure AI services and tools into a unified platform. Understanding the resource hierarchy is essential for managing model deployment endpoints.

**The Resource Hierarchy:**
- Foundry Resource — The top-level Azure resource that owns model inference endpoints, API keys, and billing.
- Foundry Project — A logical workspace inside a Foundry Resource organizing deployments, connections, and tools for a specific application or team.
- Hub (classic only) — An AI Hub resource type used for open-source model hosting, fine-tuning, and Azure Machine Learning integrations.

> The endpoint URL and API keys live at the Foundry Resource level and are shared across all projects connected to it, while individual model deployments live inside the project.

---

## 2. Model Deployment and the Playground

### 2.1 Model Deployment Steps

To deploy a model, execute the following steps in the Foundry Portal:
- Navigate to the Model Catalog — Located in the left navigation pane inside your project to browse models filtered by provider, modality, and task.
- Select a Model — Review the model card for supported deployment types, regions, pricing tier, and SLA support (models sold by Azure are backed by Microsoft support; partner/community models are supported by their providers).
- Choose Deployment Type — Select either Serverless API (pay-per-token MaaS) or Managed Compute (dedicated VM/GPU).
- Name Your Deployment — Establish a deployment name (e.g., `contoso-support-bot`) to be passed in API client calls.
- Configure Parameters — Set tokens-per-minute rate limit, model version, and version upgrade policy.
- Launch Playground — LAND automatically on the interactive test playground when the status displays Succeeded.

### 2.2 Playground Components

The playground is a web-based testing interface consisting of three primary components:
- System Message Panel — A text box used to provide prompts guiding the assistant, with options to append safety system templates.
- Chat Pane — An interactive window to send user messages and view model outputs, where the playground manages the conversation message array behind the scenes.
- Code Tab — Displays language-specific SDK connection snippets and enables opening VS Code for the Web with endpoints and keys pre-imported.

---

## 3. From Portal to Code

The Code tab automatically generates the starter SDK template. Below are the implementation patterns for key-based and Entra ID authentication.

```python
import os
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

# Option 1: API Key authentication (quick start, not recommended for production)
client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-12-01-preview"
)

# Option 2: Microsoft Entra ID authentication (recommended for production)
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(),
    "https://cognitiveservices.azure.com/.default"
)

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    azure_ad_token_provider=token_provider,
    api_version="2024-12-01-preview"
)

# The model parameter must match your DEPLOYMENT NAME, not the model name
response = client.chat.completions.create(
    model="contoso-support-bot",  # your deployment name
    messages=[
        {
            "role": "system",
            "content": "You are a helpful customer support assistant."
        },
        {
            "role": "user",
            "content": "What are your business hours?"
        }
    ],
    temperature=0.7,
    max_tokens=150
)

print(response.choices[0].message.content)
```

**Authentication Scenarios:**

| Pattern | When to use | Security level |
|:---------|:--------:|----------:|
| API Key | Local development, quick prototyping | Low — key can be leaked |
| `DefaultAzureCredential` | Production, CI/CD, managed identity | High — no secrets in code |

---

## 4. Exam Practice: Foundry Portal

<details>
<summary><b>Question 1: Inference Model Parameter</b></summary>

A developer deploys a GPT-4o model in the Foundry portal and names the deployment `hr-assistant-v1`. When writing Python code to call this deployment, what value should be passed in the `model` parameter of the API call?  
- **A)** `gpt-4o`, because the model parameter always refers to the underlying model name  
- **B)** `hr-assistant-v1`, because the model parameter must match the deployment name  
- **C)** The Foundry Resource name, because it hosts all deployments  
- **D)** The project name, because deployments are scoped to projects  

*Answer:* **B** — During inference, the API routes requests based on the deployment name, not the base model name. The deployment name is the correct value for the `model` parameter.
</details>

<details>
<summary><b>Question 2: Prompt Iteration Tooling</b></summary>

A solutions architect needs to quickly iterate on system prompt designs for a new customer support application without writing or redeploying any code. Which Foundry feature is designed specifically for this workflow?  
- **A)** The Model Catalog, to browse and compare different model capabilities  
- **B)** The Foundry Playground, to interactively test and refine prompts against a deployed model  
- **C)** The Code tab, to generate and modify SDK code directly in the browser  
- **D)** The Management Center, to configure deployment parameters and rate limits  

*Answer:* **B** — The Playground is an interactive web-based sandbox built to test system prompts, safety filters, and parameter adjustments against deployed endpoints in real time.
</details>

<details>
<summary><b>Code Challenge: PDF Summary Pipeline</b></summary>

Write a Python function `summarize_document(text: str) -> str` using the production authentication pattern.

```python
import os
import time
from openai import AzureOpenAI, RateLimitError, AuthenticationError, APIError
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

def summarize_document(text: str) -> str:
    # Set up secure Microsoft Entra ID Token Provider
    token_provider = get_bearer_token_provider(
        DefaultAzureCredential(),
        "https://cognitiveservices.azure.com/.default"
    )
    
    # Initialize the client using the Azure AD Token Provider
    client = AzureOpenAI(
        azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
        azure_ad_token_provider=token_provider,
        api_version="2024-12-01-preview"
    )
    
    system_prompt = (
        "You are a document summarization assistant. "
        "Your task is to summarize the document text provided by the user. "
        "The summary must be 3 sentences or less, factual, and strictly grounded in the text."
    )
    
    max_retries = 3
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="document-summarizer",  # Must match the portal deployment name
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": f"Document text:\n{text}"}
                ],
                temperature=0.0,  # Zero temperature ensures grounded, deterministic summaries
                max_tokens=150
            )
            return response.choices[0].message.content.strip()
            
        except AuthenticationError as e:
            print(f"Authentication failed. Verify credentials or role assignments: {e}")
            raise e
            
        except RateLimitError as e:
            if attempt == max_retries - 1:
                print("Rate limits exceeded. All retries failed.")
                raise e
            time.sleep(2 ** attempt)  # Exponential backoff
            
        except APIError as e:
            print(f"Azure OpenAI API error occurred: {e.status_code} - {e.message}")
            raise e

# Why use DefaultAzureCredential over API keys in production?
# 1. No Secrets in Code: Prevents developers from accidentally checking API keys or credentials into version control.
# 2. Role-Based Access Control (RBAC): Relies on Entra ID to define exact permissions, making auditing and key rotation seamless.
# 3. Environment Portability: Automatically resolves credential types across local development (CLI login), CI/CD pipelines, and cloud environments (Managed Identities) without configuration changes.
```
</details>
