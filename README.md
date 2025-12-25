# idoapico - OpenAI Compatible Models for VS Code

A VS Code extension that integrates OpenAI-compatible models (including Kimi K2, Claude, Qwen3 Coder, Qwen3 next, Minimax M2, DeepSeek V3, Gemini, and custom LLM endpoints) into VS Code's Language Model API. Use any OpenAI API-compatible endpoint directly within VS Code's chat interface.

## Features

- **Multi-Provider Support**: Seamlessly switch between OpenAI, Azure OpenAI, OpenRouter, Kimi/Moonshot, DeepSeek, Gemini, and any OpenAI-compatible endpoint
- **Custom Model Configuration**: Add unlimited models with customizable settings (temperature, top_p, max tokens, etc.)
- **Tool Calling**: Full support for function calling and tool integration with advanced streaming support
- **Vision Support**: Process images with vision-capable models
- **Advanced Reasoning**: Configure reasoning budgets, thinking modes, and reasoning effort for models that support it
- **Secure API Key Management**: Store API keys securely in VS Code's Secret Storage
- **Circuit Breaker Protection**: Automatic endpoint failure detection with 10-second cooldown
- **Proxy Support**: Configure HTTP/HTTPS proxies with authentication
- **Enhanced Retry Mechanism**: Configurable automatic retry with exponential backoff, jitter, and comprehensive error handling
- **Token Counting**: Smart token calculation supporting Chinese characters and images
- **Token Usage Display**: Shows per-request token usage in the status bar with multiple format options (compact/total/detailed) and supports optional session accumulation with persistence
- **System Prompt Transformation**: Regex-based system message replacements for model-specific optimizations
- **Empty Response Handling**: Configurable fallback behavior when models return empty responses
- **Health Checks**: Monitor endpoint availability with built-in health check command

## Requirements

- VS Code 1.106.1 or higher
- An OpenAI-compatible API endpoint or API key (OpenAI, Azure OpenAI, etc.)

## Extension Settings

This extension contributes the following configuration options:

### Core Settings

#### `idoapico.models` (array)
Configure OpenAI-compatible models. Each model requires:
- `id` - Unique model identifier (e.g., "gpt-4o", "claude-3-5-sonnet")
- `owned_by` - Provider name used for API key lookup (e.g., "openai", "anthropic")
- `baseUrl` - API endpoint URL (e.g., "https://api.openai.com/v1")

Optional fields:
- `configId` - Unique suffix for same model with different settings
- `displayName` - Display name in model picker
- `context_length` - Max context in tokens (default: 128000)
- `max_completion_tokens` - Max output tokens (default: 4096)
- `max_tokens` - Alternative name for max_completion_tokens (legacy)
- `vision` - Boolean, whether model supports image input (default: false)
- `headers` - Custom HTTP headers as key-value pairs
- `extra` - Additional fields to send with API requests
- `parser` - Override parser: "openai", "kimi", "anthropic", "gemini", "generic" (auto-detected by default)
- `family` - Model family for parser auto-detection (e.g., 'openai', 'kimi', 'deepseek', 'gemini', 'generic')
- `systemReplace` - Array of regex-based replacements for system messages:
  - `pattern` - Regex pattern to match
  - `replacement` - Replacement string
  - `flags` - Regex flags (default: 'g')
- `request_delay` - Delay in milliseconds before each request to this specific model (overrides global delay setting)
- `editTools` - Array of supported edit modes for file editing in VS Code. This allows you to specify which edit tools the model can use for code or text editing tasks. Recognized values (as of VS Code 1.106.1+):
  - `code-rewrite`: General-purpose code rewrite tool (model rewrites a code snippet and provides only the replacement).
  - `find-replace`: Find and replace text in a document.
  - `multi-find-replace`: Find and replace multiple text snippets across documents.
  - `apply-patch`: File-oriented diff format (used by some OpenAI models).

  **Example:**
  ```json
  {
    "idoapico.models": [
      {
        "id": "gpt-4o",
        "owned_by": "openai",
        "baseUrl": "https://api.openai.com/v1",
        "editTools": ["code-rewrite", "find-replace"]
      }
    ]
  }
  ```
  If not set, VS Code will try all available edit tools and pick the best one. The order of tools in the array does not matter.

#### `idoapico.retry`
Configure automatic retry behavior with enhanced exponential backoff:
- `enabled` - Enable retry on failures (default: true)
- `max_attempts` - Number of retry attempts (default: 3)
- `backoff` - Exponential backoff configuration:
  - `initial_delay_ms` - Initial delay before first retry (default: 1000ms)
  - `max_delay_ms` - Maximum delay between retries (default: 30000ms)
  - `multiplier` - Exponential backoff multiplier (default: 2)
  - `jitter` - Add random jitter to delay ±25% (default: true)
- `policy` - Retry policy configuration:
  - `total_timeout_ms` - Maximum total time for all retries (default: 120000ms)
  - `retryable_status_codes` - HTTP status codes that trigger retry (default: [429, 500, 502, 503, 504])
  - `idempotency_strategy` - Idempotent retry strategy: 'always', 'never', 'safe' (default: 'safe')

#### `idoapico.timeout`
Request timeout in milliseconds (default: 120000, minimum: 1000)

#### `idoapico.delay`
Global artificial delay between requests in milliseconds (default: 0, minimum: 0)

**Priority Cascade**: You can configure delays at two levels:
1. **Global Level**: `idoapico.delay` applies to all models
2. **Model Level**: `request_delay` in individual model config overrides global delay

**Priority**: Model-specific delay takes precedence over global delay. If no delays are set, no delay is applied.

#### `idoapico.proxy`
Configure HTTP proxy:
- `url` - Proxy URL (e.g., "http://proxy.example.com:8080")
- `username` - Proxy username (optional)
- `password` - Proxy password (optional)

#### `idoapico.debug`
Enable debug logging to Output > idoapico (default: false)

#### `idoapico.emptyResponse`
Configure behavior when models return empty responses:
- `retry` - Attempt follow-up retry for empty streams (default: false)
- `maxRetries` - Maximum number of follow-up retries (default: 1)
- `placeholder` - Placeholder message for empty responses (default: "[Model returned no content]")

#### `idoapico.showTokenUsage`
Show token usage for the last response in the status bar (default: true)

#### `idoapico.tokenUsageFormat`
Format for token usage display:
- `compact` - Show total tokens only (default)
- `total` - Show total tokens
- `detailed` - Show prompt, completion, and total tokens

#### `idoapico.showSessionTotal`
Show cumulative session token total in the status bar tooltip (default: false)

#### `idoapico.enableSessionTokenCounting`
Enable accumulation of token totals during the session (default: true)

#### `idoapico.tokenSessionPersistent`
Persist session token total across VS Code restarts (default: false)

### Sampling Parameters

Available for models that support them:
- `temperature` - Sampling temperature (0-2)
- `top_p` - Top-p sampling (0-1)
- `top_k` - Top-k sampling
- `min_p` - Minimum probability threshold
- `frequency_penalty` - Penalize frequent tokens (-2 to 2)
- `presence_penalty` - Penalize repeated tokens (-2 to 2)
- `repetition_penalty` - Alternative repetition penalty

### Advanced Features

#### Reasoning Configuration
```json
"reasoning": {
  "effort": "high|medium|low|minimal|auto",
  "exclude": false,
  "max_tokens": 10000,
  "enabled": true
}
```

#### Thinking/Internal Monologue
```json
"thinking": {
  "type": "enabled|disabled"
},
"enable_thinking": true,
"thinking_budget": 10000
```

## Commands

- **`idoapico: Set Generic API Key`** - Set API key for all providers
- **`idoapico: Set Provider API Key`** - Set API key for specific provider
- **`idoapico: Check Endpoint Health`** - Verify endpoint connectivity
- **`idoapico: Refresh Models`** - Reload model configuration
- **`idoapico: Reset Token Session`** - Reset session token counter

## Quick Start

### 1. Configure Models

Open VS Code settings and add your models to `idoapico.models`:

```json
{
  "idoapico.models": [
    {
      "id": "gpt-4o",
      "owned_by": "openai",
      "displayName": "GPT-4o",
      "baseUrl": "https://api.openai.com/v1",
      "context_length": 128000,
      "max_completion_tokens": 4096,
      "vision": true
    },
    {
      "id": "claude-3-5-sonnet",
      "owned_by": "anthropic",
      "displayName": "Claude 3.5 Sonnet",
      "baseUrl": "https://api.anthropic.com/v1",
      "parser": "anthropic",
      "context_length": 200000
    },
    {
      "id": "moonshot-v1-8k",
      "owned_by": "kimi",
      "displayName": "Kimi (Moonshot)",
      "baseUrl": "https://api.moonshot.cn/v1",
      "parser": "kimi",
      "context_length": 8000,
      "request_delay": 2000
    },
    {
      "id": "deepseek-chat",
      "owned_by": "deepseek",
      "displayName": "DeepSeek V3",
      "baseUrl": "https://api.deepseek.com/v1",
      "family": "deepseek",
      "context_length": 128000,
      "max_completion_tokens": 4096,
      "temperature": 0.7
    }
  ]
}
```

### Delay Configuration Examples

**Global delay for all models:**
```json
{
  "idoapico.delay": 1000
}
```

**Model-specific delay (overrides global):**
```json
{
  "idoapico.models": [
    {
      "id": "expensive-api",
      "owned_by": "custom",
      "displayName": "Expensive API Model",
      "baseUrl": "https://expensive-api.example.com/v1",
      "request_delay": 5000
    }
  ]
}
```

**Combined global + model-specific:**
```json
{
  "idoapico.delay": 1000,
  "idoapico.models": [
    {
      "id": "rate-limited",
      "owned_by": "limited",
      "baseUrl": "https://limited-api.example.com/v1",
      "request_delay": 3000
    }
  ]
}
```
### System Prompt Transformation:
```json
{
  "idoapico.models": [
    {
      "id": "deepseek-coder",
      "owned_by": "deepseek",
      "baseUrl": "https://api.deepseek.com/v1",
      "family": "deepseek",
      "systemReplace": [
        {
          "pattern": "You are an AI assistant",
          "replacement": "You are a coding expert",
          "flags": "g"
        }
      ]
    }
  ]
}
```

### DeepSeek with Enhanced Tool Support:
```json
{
  "idoapico.models": [
    {
      "id": "deepseek-chat",
      "owned_by": "deepseek",
      "baseUrl": "https://api.deepseek.com/v1",
      "family": "deepseek",
      "parser": "deepseek",
      "max_completion_tokens": 4096,
      "temperature": 0.7
    }
  ]
}
```

### GPT-OSS (Vertex AI MaaS) Support
We now support GPT-OSS models (Vertex AI MaaS). You can add GPT-OSS models to your `idoapico.models` configuration the same way you would any other model — no special configuration is required for end users. For developer-level implementation details, see the developer changelog (`DEV_CHANGELOG.md`).

### Gemini Configuration:
```json
{
  "idoapico.models": [
    {
      "id": "gemini-1.5-pro",
      "owned_by": "google",
      "baseUrl": "https://generativelanguage.googleapis.com/v1beta",
      "parser": "gemini",
      "context_length": 2000000,
      "vision": true
    }
  ]
}
```
### 2. Set API Keys

Use the command palette to set API keys:

- **`idoapico: Set Generic API Key`** - Sets a fallback API key for all providers
- **`idoapico: Set Provider API Key`** - Sets a provider-specific API key (recommended for multiple providers)

Keys are stored securely in VS Code's Secret Storage.

### 3. Start Using Models

Open VS Code's Chat interface and select an idoapico model from the model picker dropdown.

## Troubleshooting

### Models not appearing in VS Code Chat
1. Ensure VS Code version is 1.106.1 or higher
2. Check that models have valid `id`, `owned_by`, and `baseUrl` in settings
3. Verify API keys are set for the provider
4. Check Output > idoapico for error messages

### "Connection Error" messages
1. Verify `baseUrl` is correct and includes `/v1` for OpenAI endpoints
2. Check API key is valid and has required permissions
3. Verify network connectivity (check proxy settings if applicable)
4. Use `idoapico: Check Endpoint Health` command

### Timeouts
1. Increase `idoapico.timeout` setting (default: 30000ms)
2. Check network latency to the endpoint
3. Verify endpoint is responsive with health check command

### Tool calls not working
1. Ensure model supports function calling
2. Check tool schema is valid JSON
3. Verify tool names don't contain invalid characters (automatically sanitized)
4. Review VS Code output for parsing errors

## License

See LICENSE file for details.

## Support

- **Issues**: Report bugs and feature requests on GitHub
- **Debug Output**: Enable `idoapico.debug` and check Output > idoapico channel
- **API Compatibility**: Ensure your endpoint is OpenAI-compatible
