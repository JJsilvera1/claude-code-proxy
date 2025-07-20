# Anthropic API Proxy for OpenAI, Gemini, Groq & Moonshot Models 🔄

**Use Anthropic clients (like Claude Code) with OpenAI, Gemini, Groq, or Moonshot backends.** 🤝

A proxy server that lets you use Anthropic clients with OpenAI, Gemini, Groq, or Moonshot AI models via LiteLLM. 🌉


![Anthropic API Proxy](pic.png)

# Anthropic Universal API Proxy

**Use Anthropic-compatible clients (like the `claude-code` CLI) with virtually any AI backend, including Groq, OpenAI, Google Gemini, and Moonshot AI.**

This project acts as a powerful proxy server that intercepts API calls made to Anthropic's API, translates them, and intelligently routes them to the provider of your choice via LiteLLM. It's designed for maximum flexibility and customization.

![Proxy Flow Diagram](https://raw.githubusercontent.com/JJsilvera1/claude-code-proxy/main/pic.png)

---

## Key Features

*   **Multi-Provider Backend:** Swap between Groq, OpenAI, Google, and Moonshot AI by changing one line in your configuration.
*   **Intelligent Model Mapping:** Automatically maps standard Anthropic model names (`sonnet`, `haiku`) to specific models on your chosen backend (e.g., map `sonnet` to Groq's `moonshotai/kimi-k2-instruct`).
*   **Fully Configurable:** Easily override default model mappings using environment variables.
*   **Extensible:** Designed to be easily edited to support new models and providers.
*   **Full API Emulation:** Supports streaming, tool use, and token counting, providing a seamless experience for Anthropic clients.

---

## ⚡ Quick Start

This guide will get you running in minutes. The primary example uses **Groq** to run the **Moonshot AI** model, as it offers incredible performance.

### Prerequisites

*   **Python 3.10+** and `git`.
*   **API Keys** for the services you want to use (e.g., [Groq](https://console.groq.com/keys)).
*   **[uv](https://github.com/astral-sh/uv):** An extremely fast Python package installer.

### 1. Setup

First, clone the repository and navigate into the project directory.

```bash
git clone https://github.com/JJsilvera1/claude-code-proxy.git
cd claude-code-proxy
```

If you don't have `uv`, install it:
```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (in PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 2. Configuration

The server is configured using a `.env` file. Create one by copying the example:

```bash
cp .env.example .env
```

Now, edit the `.env` file. Below is the recommended configuration for using Groq's high-performance `moonshotai/kimi-k2-instruct` model.

**File: `.env`**
```dotenv
# --- Provider and API Key ---
# Set the preferred provider to "groq".
PREFERRED_PROVIDER="groq"
# Add your Groq API key.
GROQ_API_KEY="gsk_YourActualGroqApiKeyGoesHere"

# --- Model Mapping ---
# Map the powerful "sonnet" model to Groq's hosted Moonshot model.
BIG_MODEL="moonshotai/kimi-k2-instruct"
# Map the fast "haiku" model to Groq's Llama3.1 model.
SMALL_MODEL="llama-3.1-8b-instant"

# --- Optional Keys for Other Providers ---
# OPENAI_API_KEY="sk-..."
# MOONSHOT_API_KEY="sk-..."
# GEMINI_API_KEY="..."
```

### 3. Run the Server

Start the proxy server. `uv` will automatically handle all dependencies. We include a long timeout to prevent errors with slower, more powerful models.

```bash
uv run uvicorn server:app --host 0.0.0.0 --port 8082 --reload --timeout-keep-alive 300
```

If successful, you'll see: `Uvicorn running on http://0.0.0.0:8082`. Your proxy is now live!

### 4. Connect Your Client (e.g., `claude-code`)

1.  **Install the client** (if you haven't already):
    ```bash
    npm install -g @anthropic-ai/claude-code
    ```
2.  **Connect it to your proxy.** Open a **new terminal** and run the appropriate command for your system.

    **For macOS / Linux:**
    ```bash
    ANTHROPIC_BASE_URL=http://localhost:8082 claude
    ```

    **For Windows PowerShell:**
    ```powershell
    $env:ANTHROPIC_BASE_URL="http://localhost:8082"; claude
    ```

That's it! Your `claude` client is now supercharged by Groq. Watch the server logs in your other terminal to see the model mapping in real-time.

---

## 🧩 How It Works: The Internals

Understanding the logic inside `server.py` is key to customizing the proxy. A request flows through these main stages:

1.  **Request Interception (FastAPI):**
    *   The `@app.post("/v1/messages")` decorator catches any incoming requests to the messages API endpoint.

2.  **Model Validation and Mapping (`validate_model_field`):**
    *   This is the core of the routing logic, located inside the `MessagesRequest` Pydantic model.
    *   **Step 1:** It checks if the incoming model name is `sonnet` or `haiku`. If so, it replaces it with the `BIG_MODEL` or `SMALL_MODEL` value from your `.env` file. (e.g., `sonnet` becomes `moonshotai/kimi-k2-instruct`).
    *   **Step 2:** It checks which provider this target model belongs to by looking in the `GROQ_MODELS`, `OPENAI_MODELS`, etc., lists.
    *   **Step 3:** It prepends the correct provider prefix for LiteLLM (e.g., `moonshotai/kimi-k2-instruct` becomes `groq/moonshotai/kimi-k2-instruct`).

3.  **Payload Conversion (`convert_anthropic_to_litellm`):**
    *   This function translates the Anthropic-formatted request (with its unique content blocks) into the standard OpenAI format that LiteLLM and most other providers expect. It also caps the `max_tokens` value to prevent errors with providers like Groq.

4.  **API Key Injection:**
    *   Right before the final call, the code inspects the provider prefix (e.g., `groq/`) on the model name and attaches the corresponding API key (`GROQ_API_KEY`) to the request.

5.  **Backend Call (LiteLLM):**
    *   `litellm.completion()` is called. LiteLLM handles the final API call to the correct provider (e.g., Groq).

6.  **Response Conversion (`convert_litellm_to_anthropic`):**
    *   The response from the backend is translated back into the Anthropic API format before being sent to the client.

---

## 🛠️ Customization Guide

### Changing Model Mapping

The easiest way to customize the proxy is by editing your `.env` file.

**Example: Use OpenAI's `gpt-4o` and `gpt-4o-mini`**
```dotenv
PREFERRED_PROVIDER="openai"
OPENAI_API_KEY="sk-YourOpenAIKeyHere"

BIG_MODEL="gpt-4o"
SMALL_MODEL="gpt-4o-mini"
```

### Adding a New Model

To make the proxy aware of a new model released by a provider:

1.  Open `server.py`.
2.  Find the provider's model list (e.g., `GROQ_MODELS`).
3.  Add the new model's exact name to the list.

**Example: Add a new preview model to Groq**
```python
# server.py

GROQ_MODELS = [
    # ... other models
    "llama-3.3-70b-versatile",
    "new-experimental-model-v1"  # <-- Add the new model here
]
```
4.  You can now set `BIG_MODEL="new-experimental-model-v1"` in your `.env` file.

---

## 🚑 Troubleshooting

*   **Timeout Errors:** If you see timeout errors in your client, it means the model is taking too long to respond. Restart the server with a longer timeout:
    `uv run uvicorn server:app --reload --timeout-keep-alive 300`

*   **AuthenticationError (401):** This means your API key is wrong or missing. Double-check the key in your `.env` file for the provider being called (check the server logs).

*   **Model Not Found (404):** This means the backend provider doesn't recognize the model name.
    1.  Ensure the model name in your `.env` (`BIG_MODEL`/`SMALL_MODEL`) is spelled exactly right.
    2.  Ensure the model name is included in the correct provider list (`GROQ_MODELS`, etc.) in `server.py`.

*   **PowerShell Command Errors:** If you're on Windows, you cannot set environment variables on the same line. Use the correct syntax:
    `$env:ANTHROPIC_BASE_URL="http://localhost:8082"; claude`

## 🤝 Contributing
