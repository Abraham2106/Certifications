# Lightweight & Agent Clients — Study Guide

**Certification Code:** AI-901
**Phase:** 2 — Implementing AI Solutions with Microsoft Foundry

> [!NOTE]
> This guide covers Modules 6, 7, and 8: building a lightweight chat client, designing and coding single-agent solutions, and wiring a lightweight client application for a Foundry agent.

---

# 6.1 Lightweight Chat Client — Foundry SDK

## The Architecture of a Chat Client

A lightweight chat client is the minimum viable application that wraps a deployed model and handles multi-turn conversation. The key word is "lightweight" — the exam distinguishes this from a full production application. You need: authentication, a conversation loop, history management, and error handling. Nothing more.

Use the OpenAI SDK when maximum OpenAI compatibility is required, when generating embeddings, or using Foundry direct models via Chat Completions. Use the Foundry Tools SDKs when working with specific AI services like Vision, Speech, and Language.

For a chat client, the OpenAI SDK via `AIProjectClient` is the correct path.

### Installation

```bash
pip install azure-ai-projects azure-identity openai
```

### The Complete Lightweight Chat Client

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from openai import AzureOpenAI, RateLimitError, APIError, AuthenticationError

# --- Authentication ---
# AIProjectClient is the entry point for all Foundry SDK operations.
# It manages the connection to your Foundry project and exposes
# sub-clients for models, agents, and tools.
project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    # Format: https://<resource>.ai.azure.com/api/projects/<project>
    credential=DefaultAzureCredential(),
)

# get_openai_client() returns a fully authenticated AzureOpenAI client
# scoped to your Foundry project endpoint.
# This is the preferred pattern — credentials are inherited from
# AIProjectClient, so you don't manage tokens separately.
openai_client = project.get_openai_client(api_version="2024-12-01-preview")

DEPLOYMENT_NAME = os.environ["FOUNDRY_MODEL_DEPLOYMENT"]
SYSTEM_PROMPT = (
    "You are a helpful assistant for Contoso Inc. "
    "Answer questions about our products and services. "
    "If you don't know the answer, say so clearly."
)

def chat():
    """
    Multi-turn chat loop.
    Conversation history is maintained as a list of message dicts
    and passed in full on every API call — the model has no memory
    between calls, so history must be managed client-side.
    """
    conversation_history = [
        {"role": "system", "content": SYSTEM_PROMPT}
    ]

    print("Chat client ready. Type 'quit' to exit.\n")

    while True:
        user_input = input("You: ").strip()

        if user_input.lower() == "quit":
            break

        if not user_input:
            continue

        # Append user message to history
        conversation_history.append(
            {"role": "user", "content": user_input}
        )

        try:
            response = openai_client.chat.completions.create(
                model=DEPLOYMENT_NAME,
                messages=conversation_history,
                temperature=0.7,
                max_tokens=500,
            )

            assistant_message = response.choices[0].message.content

            # Append assistant response to maintain context
            conversation_history.append(
                {"role": "assistant", "content": assistant_message}
            )

            print(f"\nAssistant: {assistant_message}\n")

        except AuthenticationError:
            print("Authentication failed. Check your credentials or Foundry endpoint.")
            break

        except RateLimitError:
            print("Rate limit reached. Wait a moment before retrying.")

        except APIError as e:
            print(f"API error {e.status_code}: {e.message}")

if __name__ == "__main__":
    chat()
```

### The Critical Detail — Context Window Management

Every turn appends two messages to `conversation_history` — one user, one assistant. In long conversations this grows indefinitely, eventually exceeding the model's context window and causing a `400` error. A production-ready lightweight client handles this with a sliding window:

```python
MAX_HISTORY_TURNS = 10  # keep last 10 user+assistant pairs

def trim_history(history: list) -> list:
    """Keep system message + last N turns."""
    system_msg = history[0]  # always preserve system prompt
    turns = history[1:]      # everything after system
    if len(turns) > MAX_HISTORY_TURNS * 2:
        turns = turns[-(MAX_HISTORY_TURNS * 2):]
    return [system_msg] + turns
```

Call `conversation_history = trim_history(conversation_history)` before each API call.

---

## 🎯 Exam Practice: Chat Client

<details>
<summary><strong>Q1.</strong> A developer builds a multi-turn chat client using the Foundry SDK. After extended conversations, users report that the application starts returning errors and the model seems to forget earlier context. What are the two most likely causes?</summary>

- A) The system prompt is too long and the temperature is too high
- B) The context window is being exceeded due to unbounded history, and the model has no persistent memory between API calls requiring full history on each request
- C) The `DefaultAzureCredential` token has expired and needs to be refreshed
- D) The deployment name changed after the model was updated in Foundry

**Answer: B** — Using chat completions you send a list of messages and get a response from the model. The model has no memory between calls — history is managed client-side. Unbounded history growth causes context window overflow, producing errors and degraded responses.

</details>

---

<details>
<summary><strong>Q2.</strong> In the Foundry SDK for Python, what is the purpose of calling <code>project.get_openai_client()</code> on an <code>AIProjectClient</code> instance?</summary>

- A) It creates a new model deployment inside your Foundry project
- B) It returns an authenticated AzureOpenAI client scoped to your project endpoint, inheriting credentials from the AIProjectClient
- C) It opens a connection to the Foundry playground from Python code
- D) It registers your application with Azure Active Directory for API access

**Answer: B** — `project.get_openai_client()` returns an authenticated OpenAI client configured to run operations on your Foundry Project endpoint. Credentials are inherited — you don't manage tokens separately.

</details>

---

# 7.1 Single-Agent Solutions — Concepts and Portal

## What Makes an Agent Different from a Chat Client

A chat client is a stateless request-response loop. The application manages history. The model generates text. Nothing else happens.

An agent is fundamentally different: Foundry Agent Service supports agents defined entirely through configuration — instructions, model selection, and tools. Agent Service handles the orchestration and hosting automatically. It is best for rapid prototyping, internal tools, and agents that don't need custom orchestration logic.

The three things that distinguish an agent from a chat client:

**Tools** — The agent can call functions, execute code, search files, query the web, or call external APIs. The model decides when and how to use them based on the user's request.

**Managed state** — Unlike a chat client where you manage the `conversation_history` list yourself, the Agent Service maintains conversation state server-side in a **Thread**. You create a thread once and append messages — the service handles persistence.

**Orchestration loop** — When you submit a message to an agent thread, a **Run** is created. The service executes an internal loop: generate response → check if a tool call is needed → execute tool → feed result back → continue until the agent produces a final answer.

### The Agent Execution Model

```
User Message
     ↓
Thread (server-side conversation state)
     ↓
Run (execution instance)
     ↓
┌─── Agent Loop ────────────────────────────────┐
│  Generate response                            │
│  → Tool call needed? → Execute tool          │
│  → Feed tool result back → Continue          │
│  → No more tool calls → Final answer         │
└───────────────────────────────────────────────┘
     ↓
Response Message in Thread
```

## Built-in Tools — What the Exam Tests

Foundry Agent Service allows you to create AI agents tailored to your needs through custom instructions and augmented by advanced tools like code interpreter and custom functions.

The tools you must know for the exam:

| Tool | Description | Use Case |
|:---|:---|:---|
| **Code Interpreter** | Runs Python code in a Microsoft-managed sandbox. Isolated by Hyper-V boundary. Session active up to 1 hour; idle timeout 30 min. | Data analysis, chart generation, mathematical computation |
| **File Search** | Searches over uploaded documents using vector search. Retrieves relevant chunks to ground responses. | Q&A over internal documents, policy lookup |
| **Web Search** | Searches the internet for current information. | Research tasks, current events |
| **Function Calling** | You define custom Python functions exposed as tools. The agent decides when to call them and passes correct arguments. | Calling your own APIs, database lookups, business logic |

## Creating an Agent in the Portal

The portal path is the no-code option: navigate to **Agents** in the left pane of your Foundry project → **New Agent** → configure:

| Field | Purpose |
|:---|:---|
| **Name** | Identifier for the agent |
| **Model** | Which deployed model powers the agent |
| **Instructions** | The equivalent of a system prompt; defines the agent's role, behavior, and constraints |
| **Tools** | Toggle Code Interpreter, File Search, Web Search, or define custom functions |
| **Knowledge** | Attach files the agent can search over |

Test in the **Agent Playground** — an interactive interface where you send messages and observe the agent's reasoning steps, tool calls, and final responses. The playground shows the full execution trace, not just the final answer.

---

# 7.2 Single-Agent — Python SDK

## The Core Primitives

The four core objects are: the agent definition (configuration and tools), the thread (server-side conversation state), the message (user input appended to a thread), and the run (execution of the agent against a thread).

```python
import os
import time
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import (
    CodeInterpreterTool,
    MessageRole,
    RunStatus,
)

project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)

# Step 1 — Create the agent
# The agent is a persistent resource in your Foundry project.
# You create it once and reuse it across many conversations.
agent = project.agents.create_agent(
    model=os.environ["FOUNDRY_MODEL_DEPLOYMENT"],
    name="data-analyst",
    instructions=(
        "You are a data analysis assistant. "
        "When asked to analyze data or produce charts, use the Code Interpreter tool. "
        "Always explain your results in plain language after running code."
    ),
    tools=CodeInterpreterTool().definitions,
)
print(f"Agent created: {agent.id}")

# Step 2 — Create a thread
# A thread represents one conversation session.
# Create a new thread per user session. Server-side state is maintained automatically.
thread = project.agents.threads.create()
print(f"Thread created: {thread.id}")

# Step 3 — Add a user message to the thread
project.agents.messages.create(
    thread_id=thread.id,
    role=MessageRole.USER,
    content="What is the square root of 144? Show your calculation.",
)

# Step 4 — Create and run the agent against the thread
run = project.agents.runs.create(
    thread_id=thread.id,
    agent_id=agent.id,
)

# Step 5 — Poll until the run completes
# Runs are asynchronous. You poll status until it reaches a terminal state.
while run.status in (RunStatus.QUEUED, RunStatus.IN_PROGRESS):
    time.sleep(1)
    run = project.agents.runs.get(
        thread_id=thread.id,
        run_id=run.id,
    )
    print(f"Run status: {run.status}")

if run.status == RunStatus.FAILED:
    print(f"Run failed: {run.last_error}")
else:
    # Step 6 — Retrieve messages from the thread
    messages = project.agents.messages.list(thread_id=thread.id)
    for msg in messages:
        if msg.role == MessageRole.ASSISTANT:
            for block in msg.content:
                if hasattr(block, "text"):
                    print(f"\nAgent: {block.text.value}")

# Step 7 — Cleanup (important to avoid ongoing costs)
project.agents.delete_agent(agent.id)
```

### Run Status Values — Exam Critical

| Status | Meaning |
|:---|:---|
| `queued` | Run is waiting to start |
| `in_progress` | Agent is actively executing |
| `requires_action` | Agent called a tool and is waiting for results (function calling) |
| `completed` | Run finished successfully |
| `failed` | Run encountered an error |
| `cancelled` | Run was manually cancelled |

> [!IMPORTANT]
> The `requires_action` status is what you handle when implementing **function calling** — the agent has decided to call your custom function and is waiting for you to execute it and return the result. This is specific to custom function tools — Code Interpreter runs in the managed sandbox and does NOT produce `requires_action`.

---

## 🎯 Exam Practice: Single-Agent

<details>
<summary><strong>Q1.</strong> A developer creates a Foundry agent with the Code Interpreter tool. After submitting a user message to the agent's thread, they create a run and check its status. The status shows <code>requires_action</code>. What does this mean and what should the developer do next?</summary>

- A) The run failed and needs to be restarted with different parameters
- B) The agent has decided to call a custom function tool and is waiting for the developer to execute it and return the result
- C) The Code Interpreter sandbox has timed out and needs to be reinitialized
- D) The thread has too many messages and needs to be trimmed before the run can continue

**Answer: B** — `requires_action` is the status that indicates the agent has invoked a function calling tool and is suspended, waiting for the application to execute the function and submit the result back. This is specific to custom function tools — Code Interpreter runs in the managed sandbox and does not produce `requires_action`.

</details>

---

<details>
<summary><strong>Q2.</strong> What is the correct relationship between agents, threads, and runs in Foundry Agent Service?</summary>

- A) One agent per thread, one thread per run — each conversation requires a new agent
- B) An agent is a reusable configuration; a thread is a persistent conversation session; a run is a single execution of the agent against a thread
- C) A thread contains multiple agents; a run selects which agent responds to each message
- D) Runs are permanent resources that persist across user sessions; threads are temporary

**Answer: B** — Agents are defined through configuration and are reusable. A thread represents a conversation session with server-side state. A run is the execution of an agent against a specific thread. You create the agent once, create a new thread per user session, and create a run each time the user sends a message.

</details>

---

# 8.1 Lightweight Client Application for an Agent

## The Difference from a Direct Chat Client

In Module 6 you called the model directly via Chat Completions — your application owned the conversation loop and history. A lightweight agent client is different: the Agent Service owns the conversation state in the thread. Your client's job is to submit messages, trigger runs, poll for completion, and extract responses.

This is a thinner client in terms of state management, but it requires handling the polling loop and understanding run lifecycle.

## The Complete Lightweight Agent Client

```python
import os
import time
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import MessageRole, RunStatus

class FoundryAgentClient:
    """
    Lightweight client for a pre-existing Foundry agent.
    The agent is created once in the portal or via setup script.
    This client connects to it, manages threads per session,
    and handles the run lifecycle.
    """

    def __init__(self, agent_id: str):
        self.project = AIProjectClient(
            endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
            credential=DefaultAzureCredential(),
        )
        self.agent_id = agent_id
        # One thread per session — server manages conversation state
        self.thread = self.project.agents.threads.create()

    def send_message(self, user_input: str) -> str:
        """
        Submit a user message, run the agent, and return the response.
        Handles the full run lifecycle including polling.
        """
        # Append user message to the server-side thread
        self.project.agents.messages.create(
            thread_id=self.thread.id,
            role=MessageRole.USER,
            content=user_input,
        )

        # Create a run — this triggers agent execution
        run = self.project.agents.runs.create(
            thread_id=self.thread.id,
            agent_id=self.agent_id,
        )

        # Poll until terminal state
        run = self._poll_run(run)

        if run.status == RunStatus.FAILED:
            return f"Error: {run.last_error.message if run.last_error else 'Unknown error'}"

        return self._get_last_assistant_message()

    def _poll_run(self, run, interval: float = 1.0):
        """Poll run status until it reaches a terminal state."""
        terminal = {RunStatus.COMPLETED, RunStatus.FAILED, RunStatus.CANCELLED}
        while run.status not in terminal:
            time.sleep(interval)
            run = self.project.agents.runs.get(
                thread_id=self.thread.id,
                run_id=run.id,
            )
        return run

    def _get_last_assistant_message(self) -> str:
        """Retrieve the most recent assistant message from the thread."""
        messages = self.project.agents.messages.list(
            thread_id=self.thread.id
        )
        for msg in messages:
            if msg.role == MessageRole.ASSISTANT:
                for block in msg.content:
                    if hasattr(block, "text"):
                        return block.text.value
        return "No response received."

    def close(self):
        """Clean up the thread when the session ends."""
        self.project.agents.threads.delete(self.thread.id)


def main():
    # Assumes agent was pre-created in portal or setup script
    agent_id = os.environ["FOUNDRY_AGENT_ID"]
    client = FoundryAgentClient(agent_id=agent_id)

    print("Agent client ready. Type 'quit' to exit.\n")
    try:
        while True:
            user_input = input("You: ").strip()
            if user_input.lower() == "quit":
                break
            if not user_input:
                continue
            response = client.send_message(user_input)
            print(f"\nAgent: {response}\n")
    finally:
        client.close()

if __name__ == "__main__":
    main()
```

### Chat Client vs Agent Client — Side by Side

| Concern | Chat Client (Module 6) | Agent Client (Module 8) |
|:---|:---|:---|
| State management | Client-side `conversation_history` list | Server-side Thread |
| History ownership | Your application | Foundry Agent Service |
| Tool execution | Not supported | Supported (Code Interpreter, etc.) |
| Polling required | No — synchronous response | Yes — run is asynchronous |
| Multi-turn context | Manual append to list | Automatic in thread |
| Cleanup needed | None | Delete thread on session end |

---

## 🎯 Exam Practice: Agent Client

<details>
<summary><strong>Q1.</strong> A developer is building a client application for a Foundry agent. They notice that after each user message, the application must wait for the run to reach a completed status before retrieving the response. Why is polling required in this architecture?</summary>

- A) The Foundry SDK does not support synchronous API calls for security reasons
- B) Agent runs are asynchronous — the agent may need to execute multiple tool calls and reasoning steps before producing a final response
- C) The thread is locked during execution and cannot be read until the run completes
- D) Polling is only required when the Code Interpreter tool is enabled

**Answer: B** — Agent runs are inherently asynchronous because the agent may execute multiple tool calls, process results, and reason over multiple steps before producing a final answer. A simple chat completion is one model call; an agent run may involve an unknown number of steps.

</details>

---

<details>
<summary><strong>Q2.</strong> A developer builds a customer support application backed by a Foundry agent. Each user session should be isolated — users should not see each other's conversation history. What is the correct architectural pattern?</summary>

- A) Create one thread shared across all users and filter messages by user ID
- B) Create a new agent for each user session to ensure isolation
- C) Create a new thread per user session and delete it when the session ends
- D) Pass the user ID as part of the system prompt to isolate context

**Answer: C** — For each session or conversation, a thread is required. One thread per user session is the correct pattern. Threads are cheap to create and delete. Sharing a thread across users would expose one user's conversation history to another.

</details>

---

## 💡 Phase 2 Final Code Challenge

You are building a document Q&A agent for a law firm. The agent should:

1. Accept PDF documents uploaded by the user
2. Answer questions about those documents using File Search
3. Always cite the source document and page in its response
4. Refuse to answer questions outside the scope of the uploaded documents

**Design the following — no need to run it, but write complete code:**

- The agent creation code with the correct tool configuration
- The `FoundryAgentClient` class adapted to accept a file upload before starting a session
- The system prompt that enforces citation and scope constraints

Justify each design decision.

---

## Sources

- [Microsoft Foundry Quickstart — SDK — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/quickstarts/get-started-code)
- [Get Started with Foundry SDKs and Endpoints — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [What is Microsoft Foundry Agent Service — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/overview)
- [Foundry Agent Service Quickstart — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/agents/quickstart)
- [Use Code Interpreter with Foundry Agents — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/code-interpreter)
- [Azure AI Agents Python Client Library — Microsoft Learn](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-agents-readme)
- [Azure AI Projects Python Client Library — PyPI](https://pypi.org/project/azure-ai-projects/)
- [Hosted Agents in Foundry Agent Service — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agents)
- [AI-901 Official Study Guide — Microsoft Learn](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-901)
