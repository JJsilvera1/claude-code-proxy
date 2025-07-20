# Anthropic API Proxy for OpenAI, Gemini, Groq & Moonshot Models 🔄

**Use Anthropic clients (like Claude Code) with OpenAI, Gemini, Groq, or Moonshot backends.** 🤝

A proxy server that lets you use Anthropic clients with OpenAI, Gemini, Groq, or Moonshot AI models via LiteLLM. 🌉


![Anthropic API Proxy](pic.png)

## Quick Start ⚡

### Prerequisites

- OpenAI API key 🔑
- Google AI Studio (Gemini) API key (if using Google provider) 🔑
- Groq API key (if using Groq provider) 🔑
- Moonshot AI API key (if using Moonshot provider) 🔑
- [uv](https://github.com/astral-sh/uv) installed.

### Setup 🛠️

1. **Clone this repository**:
   ```bash
   git clone https://github.com/1rgs/claude-code-openai.git
   cd claude-code-openai
   ```

2. **Install uv** (if you haven't already):
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
   *(`uv` will handle dependencies based on `pyproject.toml` when you run the server)*

3. **Configure Environment Variables**:
   Copy the example environment file:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and fill in your API keys and model configurations:

   *   `ANTHROPIC_API_KEY`: (Optional) Needed only if proxying *to* Anthropic models.
   *   `OPENAI_API_KEY`: Your OpenAI API key (Required if using the default OpenAI preference or as fallback).
   *   `GEMINI_API_KEY`: Your Google AI Studio (Gemini) API key (Required if PREFERRED_PROVIDER=google).
   *   `GROQ_API_KEY`: Your Groq API key (Required if PREFERRED_PROVIDER=groq).
   *   `MOONSHOT_API_KEY`: Your Moonshot AI API key (Required if PREFERRED_PROVIDER=moonshot).
   *   `PREFERRED_PROVIDER` (Optional): Set to `openai` (default), `google`, `groq`, or `moonshot`. This determines the primary backend for mapping `haiku`/`sonnet`.
   *   `BIG_MODEL` (Optional): The model to map `sonnet` requests to. Defaults vary by provider - see mapping logic below.
   *   `SMALL_MODEL` (Optional): The model to map `haiku` requests to. Defaults vary by provider - see mapping logic below.

   **Mapping Logic:**
   - If `PREFERRED_PROVIDER=openai` (default), `haiku`/`sonnet` map to `SMALL_MODEL`/`BIG_MODEL` (defaults: `gpt-4.1-mini`/`gpt-4.1`) prefixed with `openai/`.
   - If `PREFERRED_PROVIDER=google`, `haiku`/`sonnet` map to `SMALL_MODEL`/`BIG_MODEL` (defaults: `gemini-2.0-flash`/`gemini-2.5-pro-preview-03-25`) prefixed with `gemini/`.
   - If `PREFERRED_PROVIDER=groq`, `haiku`/`sonnet` map to `SMALL_MODEL`/`BIG_MODEL` (defaults: `llama-3.1-8b-instant`/`llama-3.3-70b-versatile`) prefixed with `groq/`.
   - If `PREFERRED_PROVIDER=moonshot`, `haiku`/`sonnet` map to `SMALL_MODEL`/`BIG_MODEL` (defaults: `kimi-k2-base`/`kimi-k2-instruct`) prefixed with `moonshot/`.

4. **Run the server**:
   ```bash
   uv run uvicorn server:app --host 0.0.0.0 --port 8082 --reload
   ```
   *(`--reload` is optional, for development)*

### Using with Claude Code 🎮

1. **Install Claude Code** (if you haven't already):
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **Connect to your proxy**:
   ```bash
   ANTHROPIC_BASE_URL=http://localhost:8082 claude
   ```

3. **That's it!** Your Claude Code client will now use the configured backend models (defaulting to OpenAI) through the proxy. 🎯

## Model Mapping 🗺️

The proxy automatically maps Claude models to OpenAI, Gemini, Groq, or Moonshot models based on the configured provider:

| Claude Model | OpenAI (default) | Google | Groq | Moonshot |
|--------------|------------------|--------|------|----------|
| haiku | openai/gpt-4.1-mini | gemini/gemini-2.0-flash | groq/llama-3.1-8b-instant | moonshot/kimi-k2-base |
| sonnet | openai/gpt-4.1 | gemini/gemini-2.5-pro-preview-03-25 | groq/llama-3.3-70b-versatile | moonshot/kimi-k2-instruct |

### Supported Models

#### OpenAI Models
The following OpenAI models are supported with automatic `openai/` prefix handling:
- o3-mini
- o1
- o1-mini
- o1-pro
- gpt-4.5-preview
- gpt-4o
- gpt-4o-audio-preview
- chatgpt-4o-latest
- gpt-4o-mini
- gpt-4o-mini-audio-preview
- gpt-4.1
- gpt-4.1-mini

#### Gemini Models
The following Gemini models are supported with automatic `gemini/` prefix handling:
- gemini-2.5-pro-preview-03-25
- gemini-2.0-flash

#### Groq Models  
The following Groq models are supported with automatic `groq/` prefix handling:
- llama-3.3-70b-versatile (Production)
- llama-3.1-8b-instant (Production) 
- llama3-70b-8192 (Production)
- llama3-8b-8192 (Production)
- gemma2-9b-it (Production)
- whisper-large-v3 (Production - Audio)
- whisper-large-v3-turbo (Production - Audio)
- distil-whisper-large-v3-en (Production - Audio)
- llama-guard-3-8b (Production - Safety)
- meta-llama/llama-4-scout-17b-16e-instruct (Preview)
- meta-llama/llama-4-maverick-17b-128e-instruct (Preview)
- qwen-qwq-32b (Preview)
- qwen-2.5-coder-32b (Preview)
- qwen-2.5-32b (Preview)
- deepseek-r1-distill-qwen-32b (Preview)
- deepseek-r1-distill-llama-70b (Preview)
- mistral-saba-24b (Preview)
- llama-3.2-1b-preview (Preview)
- llama-3.2-3b-preview (Preview)
- llama-3.2-11b-vision-preview (Preview - Vision)
- llama-3.2-90b-vision-preview (Preview - Vision)
- llama-3.3-70b-specdec (Preview)

#### Moonshot AI (Kimi K2) Models
The following Moonshot models are supported with automatic `moonshot/` prefix handling:
- kimi-k2-base
- kimi-k2-instruct

### Model Prefix Handling
The proxy automatically adds the appropriate prefix to model names:
- OpenAI models get the `openai/` prefix 
- Gemini models get the `gemini/` prefix
- Groq models get the `groq/` prefix
- Moonshot models get the `moonshot/` prefix
- The BIG_MODEL and SMALL_MODEL will get the appropriate prefix based on which model list they're in

For example:
- `gpt-4.1` becomes `openai/gpt-4.1`
- `gemini-2.5-pro-preview-03-25` becomes `gemini/gemini-2.5-pro-preview-03-25`
- `llama-3.3-70b-versatile` becomes `groq/llama-3.3-70b-versatile`
- `kimi-k2-instruct` becomes `moonshot/kimi-k2-instruct`
- When BIG_MODEL is set to a specific provider's model, Claude Sonnet will map to `[provider]/[model-name]`

### Customizing Model Mapping

Control the mapping using environment variables in your `.env` file or directly:

**Example 1: Default (Use OpenAI)**
No changes needed in `.env` beyond API keys, or ensure:
```dotenv
OPENAI_API_KEY="your-openai-key"
GEMINI_API_KEY="your-google-key" # Needed if PREFERRED_PROVIDER=google
# PREFERRED_PROVIDER="openai" # Optional, it's the default
# BIG_MODEL="gpt-4.1" # Optional, it's the default
# SMALL_MODEL="gpt-4.1-mini" # Optional, it's the default
```

**Example 2: Prefer Google**
```dotenv
GEMINI_API_KEY="your-google-key"
OPENAI_API_KEY="your-openai-key" # Needed for fallback
PREFERRED_PROVIDER="google"
# BIG_MODEL="gemini-2.5-pro-preview-03-25" # Optional, it's the default for Google pref
# SMALL_MODEL="gemini-2.0-flash" # Optional, it's the default for Google pref
```

**Example 3: Use Groq Models**
```dotenv
GROQ_API_KEY="your-groq-key"
OPENAI_API_KEY="your-openai-key" # Needed for fallback
PREFERRED_PROVIDER="groq"
# BIG_MODEL="llama-3.3-70b-versatile" # Optional, it's the default for Groq
# SMALL_MODEL="llama-3.1-8b-instant" # Optional, it's the default for Groq
```

**Example 4: Use Moonshot AI Models**
```dotenv
MOONSHOT_API_KEY="your-moonshot-key"
OPENAI_API_KEY="your-openai-key" # Needed for fallback
PREFERRED_PROVIDER="moonshot"
# BIG_MODEL="kimi-k2-instruct" # Optional, it's the default for Moonshot
# SMALL_MODEL="kimi-k2-base" # Optional, it's the default for Moonshot
```

**Example 5: Use Specific OpenAI Models**
```dotenv
OPENAI_API_KEY="your-openai-key"
PREFERRED_PROVIDER="openai"
BIG_MODEL="gpt-4o" # Example specific model
SMALL_MODEL="gpt-4o-mini" # Example specific model
```

## How It Works 🧩

This proxy works by:

1. **Receiving requests** in Anthropic's API format 📥
2. **Translating** the requests to the target provider format via LiteLLM 🔄
3. **Sending** the translated request to OpenAI, Gemini, Groq, or Moonshot 📤
4. **Converting** the response back to Anthropic format 🔄
5. **Returning** the formatted response to the client ✅

The proxy handles both streaming and non-streaming responses, maintaining compatibility with all Claude clients. 🌊

## Contributing 🤝

Contributions are welcome! Please feel free to submit a Pull Request. 🎁
