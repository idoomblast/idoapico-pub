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

- VS Code Insiders, or VS Code started with `--enable-proposed-api=idoomblast.idoapico`
- VS Code 1.106.1 or higher
- An OpenAI-compatible API endpoint or API key (OpenAI, Azure OpenAI, etc.)

This extension uses the proposed `chatProvider` API. If you install the packaged `.vsix` into a regular VS Code session without enabling that proposal, provider registration will fail with `CANNOT use API Proposal chatProvider`.

## Extension Settings

This extension contributes the following configuration options:

### Manage Models (`chatLanguageModels.json`) - Preferred

On current VS Code builds, the preferred configuration path is **Manage Models** and the generated `chatLanguageModels.json` file. The `idoapico` provider supports both of the following shapes:

1. A single-model provider group, where the root entry contains `modelId`.
2. A grouped provider entry, where the root entry contains shared settings plus a `models` array.

The grouped form is recommended when multiple models share the same API root and API key.

#### Supported Shapes

**Single-model provider group**

```json
{
  "name": "IDOOM AI",
  "vendor": "idoapico",
  "provider": "idoomAI",
  "modelId": "go/deepseek-v4-flash",
  "displayName": "DeepSeek v4 Flash GO",
  "baseUrl": "https://ai.idoom.me/v1",
  "apiKey": "${input:chat.lm.secret.6402420c}"
}
```

**Grouped provider entry**

```json
{
  "name": "IdoomAI",
  "vendor": "idoapico",
  "provider": "idoomAI",
  "apiKey": "${input:chat.lm.secret.-3a2aa0b0}",
  "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
  "models": [
    {
      "id": "GLM-4.7",
      "name": "GLM-4.7"
    },
    {
      "id": "GLM-5.1",
      "name": "GLM-5.1"
    }
  ]
}
```

#### Provider Group Fields

The following fields are supported at the root level of an `idoapico` provider-group entry in `chatLanguageModels.json`:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | Yes | The label shown by VS Code for the provider group. |
| `vendor` | `string` | Yes | Must be `idoapico`. |
| `provider` | `string` | No | Optional provider label used for logging, parser defaults, and API key lookup. |
| `apiKey` | `string` | No | API key stored through VS Code secret interpolation, for example `${input:chat.lm.secret.xxx}`. |
| `baseUrl` | `string` | Yes | The API root for the provider group. Use the API root such as `https://api.example.com/v1`, not the final `/chat/completions` endpoint. |
| `modelId` | `string` | Yes for single-model shape | Model identifier sent in the request body when using the single-model shape. |
| `displayName` | `string` | No | Optional display label for the single-model shape. |
| `family` | `string` | No | Default family hint used for parser selection and model grouping. |
| `parser` | `string` | No | Explicit parser override. Supported values: `openai`, `anthropic`, `kimi`, `gemini`, `generic`, `gptoss`, `deepseekrseries`, `deepseekv3`. |
| `vision` | `boolean` | No | Default image-input capability for the single-model shape or for models in a grouped entry. |
| `toolCalling` | `boolean` | No | Default tool-calling capability hint advertised to VS Code. |
| `contextLength` | `number` | No | Default total context window in tokens. |
| `maxCompletionTokens` | `number` | No | Default maximum output tokens. |
| `requestDelay` | `number` | No | Default request delay in milliseconds for this group. |
| `headers` | `object` | No | Default extra HTTP headers. |
| `extra` | `object` | No | Default extra request-body fields. If a key here overlaps with a generated request key, `extra` wins. |
| `systemReplace` | `array` | No | Default system-message replacement rules applied before the request is sent. |
| `thinkingMode` | `string` | No | Default `thinking.type` value. Supported values: `enabled`, `disabled`. |
| `enableThinking` | `boolean` | No | Default `enable_thinking` flag for providers such as Kimi. |
| `thinkingBudget` | `number` | No | Default `thinking_budget` for compatible providers. |
| `editTools` | `array` | No | Default edit-tool hints exposed to VS Code. |
| `models` | `array` | Yes for grouped shape | Array of per-model definitions that inherit the root configuration. |
| `settings` | `object` | No | VS Code-managed per-model override block. Keys must match the returned model `id` or `id::configId`. |

#### `models[]` Fields

Each item in `models[]` represents one model exposed under the same provider group.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | Yes | Model identifier sent as `model` in the API request. |
| `configId` | `string` | No | Optional suffix that makes duplicate model IDs unique in the picker. The final exposed ID becomes `id::configId`. |
| `name` | `string` | No | Display name shown in the picker. |
| `displayName` | `string` | No | Alias for `name`. |
| `provider` | `string` | No | Per-model override for the provider label used by logging, parser defaults, and API key lookup. |
| `baseUrl` | `string` | No | Per-model API root override. This must still be an API root such as `https://api.example.com/v1`. |
| `family` | `string` | No | Per-model family hint used for parser selection. |
| `parser` | `string` | No | Per-model parser override. |
| `vision` | `boolean` | No | Whether the model accepts image input. |
| `toolCalling` | `boolean` | No | Capability hint for whether the model should advertise tool calling. |
| `contextLength` | `number` | No | Total context window in tokens. |
| `maxInputTokens` | `number` | No | Input-token budget. When provided with `maxOutputTokens`, the extension derives `contextLength = maxInputTokens + maxOutputTokens`. |
| `maxOutputTokens` | `number` | No | Output-token budget used to derive `maxCompletionTokens`. |
| `maxCompletionTokens` | `number` | No | Explicit maximum output tokens. |
| `maxTokens` | `number` | No | Legacy alias for `maxCompletionTokens`. |
| `temperature` | `number \| null` | No | Sampling temperature. |
| `topP` | `number \| null` | No | Top-p sampling value. |
| `topK` | `number` | No | Top-k sampling value. |
| `minP` | `number` | No | Minimum probability threshold. |
| `frequencyPenalty` | `number` | No | Frequency penalty sent to the API. |
| `presencePenalty` | `number` | No | Presence penalty sent to the API. |
| `repetitionPenalty` | `number` | No | Repetition penalty sent to the API. |
| `reasoningEffort` | `string` | No | Top-level reasoning effort. Supported values: `none`, `minimal`, `low`, `medium`, `high`, `max`, `xhigh`. |
| `reasoning` | `object` | No | Advanced reasoning object with `effort`, `exclude`, `max_tokens`, and `enabled`. |
| `thinking` | `boolean \| object` | No | Convenience field for thinking mode. `true` maps to `{ "type": "enabled" }`, `false` maps to `{ "type": "disabled" }`. |
| `thinkingMode` | `string` | No | Explicit `thinking.type` override with `enabled` or `disabled`. |
| `enableThinking` | `boolean` | No | Explicit `enable_thinking` override. |
| `thinkingBudget` | `number` | No | Explicit `thinking_budget` override. |
| `headers` | `object` | No | Additional per-model HTTP headers merged on top of root headers. |
| `extra` | `object` | No | Additional per-model request-body fields merged on top of root `extra`. |
| `requestDelay` | `number` | No | Per-model delay before a request is sent. |
| `editTools` | `array` | No | Per-model edit-tool hints. |
| `systemReplace` | `array` | No | Per-model system-message replacement rules appended after root-level rules. |

#### `settings` Overrides

The `settings` object in `chatLanguageModels.json` is the per-model override store that VS Code uses for Manage Models. The keys must match the exposed model `id` or `id::configId` exactly.

Supported override keys:

| Key | Type | Description |
| --- | --- | --- |
| `temperature` | `number \| null` | Override model temperature. |
| `topP` | `number \| null` | Override top-p sampling. |
| `topK` | `number` | Override top-k sampling. |
| `minP` | `number` | Override minimum probability threshold. |
| `frequencyPenalty` | `number` | Override frequency penalty. |
| `presencePenalty` | `number` | Override presence penalty. |
| `repetitionPenalty` | `number` | Override repetition penalty. |
| `maxCompletionTokens` | `number` | Override output-token limit. |
| `requestDelay` | `number` | Override the request delay in milliseconds. |
| `reasoningEffort` | `string` | Override top-level reasoning effort. |
| `thinkingMode` | `string` | Override `thinking.type`. |
| `enableThinking` | `boolean` | Override `enable_thinking`. |
| `thinkingBudget` | `number` | Override `thinking_budget`. |

#### Inheritance and Precedence

Configuration is resolved in the following order:

1. Root provider-group defaults.
2. Per-model values inside `models[]`.
3. Per-model `settings` overrides created by Manage Models.
4. `extra` request-body keys, which always win if they overlap with generated request fields.

Special rules:

- Root-level `systemReplace` is inherited by every model in the group.
- Per-model `systemReplace` rules are appended after the root rules.
- `thinking: true` is treated as `thinkingMode: "enabled"` plus `enableThinking: true` when no explicit thinking overrides are set.
- `maxInputTokens` plus `maxOutputTokens` can be used instead of manually calculating `contextLength`.

#### Grouped Provider Example

```json
[
  {
    "name": "IdoomAI",
    "vendor": "idoapico",
    "provider": "idoomAI",
    "apiKey": "${input:chat.lm.secret.-3a2aa0b0}",
    "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
    "systemReplace": [
      {
        "pattern": "You are an AI assistant",
        "replacement": "You are a coding expert",
        "flags": "g"
      }
    ],
    "models": [
      {
        "id": "GLM-4.7",
        "name": "GLM-4.7",
        "vision": false,
        "toolCalling": false,
        "maxInputTokens": 200000,
        "maxOutputTokens": 128000,
        "thinking": true
      },
      {
        "id": "GLM-5.1",
        "name": "GLM-5.1",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 200000,
        "maxOutputTokens": 128000,
        "thinking": true,
        "systemReplace": [
          {
            "pattern": "helpful assistant",
            "replacement": "rigorous coding assistant",
            "flags": "g"
          }
        ]
      }
    ],
    "settings": {
      "GLM-5.1": {
        "reasoningEffort": "xhigh",
        "thinkingBudget": 12000
      }
    }
  }
]
```

This example exposes two models from one provider group, shares the same root `baseUrl` and `apiKey`, inherits root-level `systemReplace`, and applies a per-model `settings` override to `GLM-5.1`.

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
- `reasoning_effort` - Top-level reasoning effort for compatible providers. DeepSeek chat models support `high` and `max`; `xhigh` remains available as a compatibility alias.
- `vision` - Boolean, whether model supports image input (default: false)
- `toolCalling` - Capability hint for whether the model should advertise tool calling in the picker
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
  "effort": "none|minimal|low|medium|high|max|xhigh|auto",
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

For Manage Models and `chatLanguageModels.json`, the official camelCase equivalents are:

```json
{
  "thinkingMode": "enabled",
  "enableThinking": true,
  "thinkingBudget": 10000,
  "reasoningEffort": "max"
}
```

This covers the common Kimi and DeepSeek variant use cases without needing `extra`. If both official fields and `extra` provide the same request key, `extra` still wins as the final override.

You can also expose multiple models from one provider group by keeping `baseUrl` and `apiKey` at the root and listing the models under `models`. Each `models[]` entry can define its own `name`, token limits, thinking settings, parser/family overrides, tool capability hints, and `systemReplace` rules.

Use the API root in `baseUrl`, for example `https://open.bigmodel.cn/api/coding/paas/v4`, not the final `/chat/completions` URL.

```json
[
  {
    "name": "IdoomAI",
    "vendor": "idoapico",
    "apiKey": "${input:chat.lm.secret.-3a2aa0b0}",
    "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
    "systemReplace": [
      {
        "pattern": "You are an AI assistant",
        "replacement": "You are a coding expert",
        "flags": "g"
      }
    ],
    "models": [
      {
        "id": "GLM-4.7",
        "name": "GLM-4.7",
        "vision": false,
        "toolCalling": false,
        "maxInputTokens": 200000,
        "maxOutputTokens": 128000,
        "thinking": true
      },
      {
        "id": "GLM-5.1",
        "name": "GLM-5.1",
        "toolCalling": true,
        "vision": false,
        "maxInputTokens": 200000,
        "maxOutputTokens": 128000,
        "thinking": true,
        "systemReplace": [
          {
            "pattern": "helpful assistant",
            "replacement": "rigorous coding assistant",
            "flags": "g"
          }
        ]
      }
    ]
  }
]
```

Root-level `systemReplace` is inherited by every model in the group, and each model can append its own extra replacements.

## Commands

- **`idoapico: Set Generic API Key`** - Set API key for all providers
- **`idoapico: Set Provider API Key`** - Set API key for specific provider
- **`idoapico: Check Endpoint Health`** - Verify endpoint connectivity
- **`idoapico: Refresh Models`** - Reload model configuration
- **`idoapico: Reset Token Session`** - Reset session token counter

## Quick Start

### 0. Run With Proposed API Enabled

For development, use the included launch configuration in [.vscode/launch.json](.vscode/launch.json), which starts VS Code Insiders with `--enable-proposed-api=idoomblast.idoapico`.

For a packaged `.vsix`, launch VS Code with the same flag before testing the extension:

```bash
code-insiders --enable-proposed-api=idoomblast.idoapico
```

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

### MiniMax M2.1 Configuration:
```json
{
  "idoapico.models": [
    {
      "id": "MiniMax-M2.1",
      "owned_by": "minimax",
      "displayName": "MiniMax M2.1",
      "baseUrl": "https://api.minimax.io/v1",
      "context_length": 128000,
      "max_completion_tokens": 4096,
      "temperature": 1.0,
      "extra": {
        "reasoning_split": true
      }
    }
  ]
}
```
For more details, see [MiniMax M2.1 Support](#minimax-m21-support) below.

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

### Kimi K2 Thinking Model Support (Moonshot AI)

We now fully support Kimi K2 Thinking models with multi-step reasoning and tool use capabilities. Kimi K2 preserves reasoning history across multi-turn conversations for improved accuracy.

#### Configuration

```json
{
  "idoapico.models": [
    {
      "id": "kimi-k2-thinking",
      "owned_by": "kimi",
      "displayName": "Kimi K2 Thinking",
      "baseUrl": "https://api.moonshot.ai/v1",
      "parser": "kimi",
      "thinking": true,
      "enable_thinking": true,
      "context_length": 256000,
      "max_completion_tokens": 16000,
      "temperature": 1.0,
      "top_p": 0.9,
      "top_k": 40,
      "min_p": 0.0,
      "presence_penalty": 0.0,
      "repetition_penalty": 1.0,
      "request_delay": 2000
    }
  ]
}
```

#### Key Features

- **Reasoning Content Preservation**: `reasoning_content` field is automatically preserved across multi-turn conversations
- **Multi-Step Tool Calls**: Designed for complex tasks requiring sequential tool execution
- **Streaming**: Always enabled for Kimi K2 to avoid timeout issues with large responses
- **Token Budget**: Set `max_completion_tokens` ≥ 16,000 to ensure full reasoning + content fits

#### Best Practices

1. **Always Preserve Context**: Include full conversation history including `reasoning_content`
2. **Enable Streaming**: Use `stream: true` (automatically set by parser)
3. **Set Temperature to 1.0**: Recommended for optimal reasoning performance
4. **Generous Token Limit**: Use 16K+ tokens to prevent truncation of reasoning chains
5. **Rate Limiting**: Add `request_delay` (2000ms recommended) to respect API limits

For more details, see the [Kimi K2 documentation](https://platform.moonshot.ai/docs/guide/use-kimi-k2-thinking-model).

### MiniMax M2.1 Support

We fully support MiniMax M2.1 models with native tool calling and interleaved thinking capabilities. MiniMax M2.1 provides excellent performance for coding and agentic tasks with state-of-the-art results on SWE-bench and other benchmarks.

#### Configuration

```json
{
  "idoapico.models": [
    {
      "id": "MiniMax-M2.1",
      "owned_by": "minimax",
      "displayName": "MiniMax M2.1",
      "baseUrl": "https://api.minimax.io/v1",
      "context_length": 128000,
      "max_completion_tokens": 4096,
      "temperature": 1.0,
      "extra": {
        "reasoning_split": true
      }
    }
  ]
}
```

**International users**: Use `https://api.minimax.io/v1`  
**Users in China**: Use `https://api.minimaxi.com/v1`

#### Key Features

- **Native Reasoning Support**: Automatically extracts reasoning from `response_details` field when `reasoning_split: true`
- **Tool Calling**: Excellent tool use capabilities with XML `<minimax:tool_call>` format support
- **Interleaved Thinking**: Models reason between tool calls for complex multi-step tasks
- **OpenAI Compatibility**: Full OpenAI API format support with automatic parser selection
- **Model Variants**: Support for M2.1 (standard) and M2.1-lightning (faster)

#### Configuration Options

**Enable Separate Reasoning Output:**
```json
{
  "extra": {
    "reasoning_split": true
  }
}
```

**MiniMax M2.1 Lightning (faster, 100 tps):**
```json
{
  "id": "MiniMax-M2.1-lightning",
  "owned_by": "minimax",
  "displayName": "MiniMax M2.1 Lightning"
}
```

#### Best Practices

1. **Enable Reasoning Split**: Set `extra.reasoning_split: true` to separate thinking from content
2. **Preserve Context**: Always include full model responses (including `reasoning_details`) in conversation history for multi-turn interactions
3. **Temperature**: Use `1.0` (recommended by MiniMax for optimal performance)
4. **Tool Use**: MiniMax M2.1 excels at complex tool workflows - no special configuration needed
5. **Rate Limiting**: Add `request_delay` if you encounter rate limits (2000ms recommended)

#### Troubleshooting

**No reasoning output:**
- Ensure `extra.reasoning_split: true` is set in model configuration
- Verify you're using a model that supports interleaved thinking (MiniMax-M2.1)

**Tool calls not working:**
- MiniMax M2.1 supports both OpenAI format and XML tool calls automatically
- Check that tools are properly configured in the requesting extension
- Verify API key has necessary permissions

**Authentication errors:**
- Verify API key is set correctly using `idoapico: Set Provider API Key` command
- Check that `baseUrl` matches your region (international vs China)

For more details, see the [MiniMax Platform Documentation](https://platform.minimax.io/).
