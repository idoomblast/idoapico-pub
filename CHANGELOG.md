# Change Log

All notable changes to the "idoapico" extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.14] - 2025-12-22

### Added
- Token usage display in the status bar (compact format) and optional session accumulation (configurable via `idoapico.showTokenUsage`, `idoapico.showSessionTotal`, `idoapico.enableSessionTokenCounting`).

### Technical
- `src/statusBar.ts`: New `TokenStatusBar` class to display per-request and session token usage.
- `src/utils.ts`: Added `normalizeUsage()` to normalize usage objects from different providers.
- `src/provider.ts`: Detects `usage` info in streaming chunks and updates status bar & session totals once per response.
- Added `idoapico.resetTokenSession` command to reset session totals.

## [0.0.13] - 2025-12-22

### Added
- Robust parsing for XML-style `<function_calls>` emitted in streaming model responses. The parser now assembles extremely fragmented XML blocks across many tiny SSE chunks and emits a single tool call object when the block is complete.
- Tolerant parsing for malformed end tags and missing `</invoke>` closures to avoid partial leakage of XML fragments to the UI.
- Unit and integration tests added to reproduce the DeepSeek extreme-fragmentation case and validate behavior deterministically.
- Guard `MAX_XML_BUFFER` to protect against unbounded buffer growth in streaming edge cases.

### Fixed
- Prevent partial XML fragments from being emitted as text in the chat UI when models stream highly fragmented `<function_calls>` blocks (DeepSeek reproduction).

### Technical
- `src/parsers/openai.ts`: Added assembly logic, tolerant end detection, defensive `<invoke>` parsing, and buffer guards.
- Tests: added/updated tests in `src/test/unit/parsers/openai.test.ts` and `src/test/integration/deepseek-*`.
- Minor: added `src/utils/metrics.ts` and test helpers.

## [0.0.12] - 2025-12-21

### Added
- **Reasoning Effort Parameter Support**: Enhanced `reasoning_effort` configuration dengan support untuk nilai `none`, `xhigh`, dan format object `{effort, summary}`. Kompatibel dengan OpenAI o1 models dan Gemini 3 models.

### Technical
- Updated TypeScript types di `src/types.ts` dengan `ReasoningEffortLevel` dan `ReasoningEffortObject`
- Enhanced VS Code configuration schema di `package.json` dengan `anyOf` untuk string dan object formats
- Automatic pass-through ke API requests tanpa perubahan kode tambahan

### Files Changed
- `src/types.ts`: Added new types untuk reasoning effort support
- `package.json`: Updated schema configuration untuk reasoning_effort parameter

## [0.0.11] - 2025-12-21

### Added
- **Dual-Level Request Delay**: New dual-level delay system untuk granular rate limit control
  - **Global Level**: `idoapico.delay` applies to all models
  - **Model Level**: `request_delay` per model overrides global setting
  - **Priority Cascade**: Model-specific delay takes precedence over global delay

### Technical
- Enhanced provider dengan `waitWithDelay()` helper function dengan CancellationToken support
- Added `getEffectiveDelay()` dengan priority cascade logic (model-specific > global)
- Integrated delay mechanism before `executeWithRetry()` untuk consistent behavior
- Smart logging untuk delay tracking dan debugging

### Configuration
- Added `request_delay?: number` to ModelItem interface untuk model-specific delays
- Updated package.json schema dengan validation (type: number, default: 0, minimum: 0)
- Backward compatibility maintained - all existing configurations work unchanged

### Documentation
- Updated README.md dengan comprehensive delay configuration examples
- Added priority cascade explanation dan usage patterns
- Documented combination strategies (global + model-specific scenarios)

### Files Changed
- `src/types.ts`: Added `request_delay` property to ModelItem interface
- `src/provider.ts`: Implemented dual-level delay logic dengan cancellation support
- `package.json`: Added request_delay schema configuration
- `README.md`: Updated documentation dengan examples dan priority cascade details

## [0.0.10] - 2025-12-21

### Technical
- Updated deployment workflow and version management

## [0.0.9] - 2025-12-11

### Technical
- Minor version bump for release workflow optimization

## [0.0.8] - 2025-12-11

### Added
- **System Replace Configuration**: Regex-based replacements untuk system messages dengan support pattern, replacement, dan flags. Fitur ini memungkinkan modifikasi dinamis pada system prompt sesuai kebutuhan model.

### Technical
- Enhanced provider logic untuk regex-based system message modifications
- Added `systemReplace` array configuration di ModelItem interface
- Comprehensive error handling untuk invalid regex patterns

### File Changes
- `src/types.ts`: Added `systemReplace` type definition
- `src/provider.ts`: Implemented regex replacement logic dengan error handling

## [0.0.7] - 2025-12-04

### Fixed
- **DeepSeek/Qwen UI**: Fixed an issue where leading whitespace in the first response chunk caused indentation problems in the chat UI. Implemented smart trimming in `DeepSeekParser`.

## [0.0.1] - 2025-12-01

### Added
- Initial release.
- Support for OpenAI-compatible API endpoints.
- **Tested Models:**
  - Google Gemini variants (including Gemini 3 via OpenAI compatibility).
  - OpenAI models.
  - Qwen 3 Next (Thinking).
  - Kimi K2 with thinking/reasoning support (Experimental).
  - Minimax M2.
  - Deepseek v3.1.

## [0.0.2] - 2025-12-02

### Added
- **Enhanced Error Detection for Stream Responses**: Added detection for error payloads within stream responses when HTTP status code is 200 but payload contains error (e.g., code 400). This addresses the issue where the API returns {"code":400,"message":"Input should be a valid number"} within the stream data.

### Fixed
- **Stream Error Handling**: Fixed bug where stream errors (like validation errors) were not properly detected and reported. The provider now correctly identifies error patterns and throws appropriate errors with detailed messages.

### Technical
- Enhanced stream processing logic to check for error payloads before parsing response chunks
- Added circuit breaker integration for stream errors

## [0.0.4] - 2025-12-04

### Fixed
- **Kimi Tool Call IDs**: Updated Kimi tool ID generation to strictly follow the `functions.{name}:{index}` format required by the Kimi-K2 model. This ensures correct tool tracking and processing.
- **Safety Net Policy**: Relaxed the "hallucinated tool name" safety net. Instead of blocking tools with generated names (like `call_123`), the system now logs a warning and passes them through. This allows VS Code's natural error handling (e.g., "Tool not found") to provide feedback to the model, enabling self-correction.

## [0.0.3] - 2025-12-03
- **Tool ID Sanitization**: Implemented deterministic sanitization for tool call IDs to resolve compatibility issues when switching between models (e.g., Vertex AI to Kimi/OpenAI). This handles long or non-standard IDs by hashing them into a safe, consistent format (`call_` + 24 hex chars).

## [0.0.3] - 2025-12-03

### Fixed
- **Token Leakage**: Fixed an issue where internal model tokens (e.g., `<|tool_call|>`) were sometimes visible in the chat output for certain models.
- **Kimi Tool Calls**: Resolved an issue where tool calls with hexadecimal IDs were being incorrectly filtered out, causing some tools to fail silently.
