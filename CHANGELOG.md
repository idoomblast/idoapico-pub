# Change Log

All notable changes to the "idoapico" extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v0.2.4] - 2026-01-14

### Added
- **MiniMax M2.1 Model Support**: Added comprehensive support for MiniMax M2.1 models with native reasoning capabilities and tool calling.
  - **Parser Support**: New MiniMaxParser that extracts reasoning from native `response_details` field
  - **Configuration**: Full model configuration support with `reasoning_split` parameter for interleaved thinking
  - **Tool Calling**: Excellent tool use capabilities with both OpenAI-compatible and XML tool formats
  - **Documentation**: Complete configuration guide and usage examples in README
  - **Testing**: Comprehensive unit tests covering all parser functionality and edge cases
  - **API Compatibility**: Support for both international and China endpoints

## [v0.2.3] - 2026-01-13

### Fixed
- **GLM-4.7 Vertex AI XML Tag Cleanup**: Fixed XML `<tool_call>` tags leaking into chat UI when using GLM-4.7 models. Tags and surrounding content are now properly removed before display.

## [v0.2.2] - 2026-01-05

### Added
- **Real-Time Token Rate Display**: Added live token rate (tokens/second) during streaming responses. Rate appears in status bar alongside token counts and persists after stream completes.
- **Configurable Rate Update Interval**: Added `tokenRateUpdateInterval` setting to control status bar update frequency (default: 200ms). Set to 0 for immediate updates during debugging.

### Fixed
- **Chutes Proxy Output Cleanup**: Fixed chat output contamination when using Chutes Kimi K2 proxy. Tool call JSON and native tokens were visible in chat UI; now properly cleaned from both streaming and final responses.
- **Tool Call Duplicate Prevention**: Fixed issue where duplicate tool calls could be emitted in certain edge cases during parsing.
- **Reasoning Content Preservation**: Improved handling of reasoning content to ensure it's properly preserved across multi-turn conversations with thinking models.

## [v0.2.1] - 2026-01-02

### Fixed
- **Chutes Proxy API Output Cleanup**: Fixed broken chat output when using Chutes Kimi K2 proxy that mixes thinking content, tool calls, and native tokens in single stream. Enhanced pattern matching to handle ASCII pipe tokens and complete JSON fragments, and applied cleanup to streaming path to prevent tokens/fragments from reaching UI. Eliminated leftover `</think>` tags and tool call arguments visible in chat UI.

## [v0.2.0] - 2026-01-01

### Added
- **Kimi-K2-Thinking Deepinfra Compatibility**: Added automatic detection of OpenAI vs native Kimi token formats, enabling seamless use of Kimi-K2-Thinking with Deepinfra endpoints without manual configuration.

## [v0.1.5] - 2025-12-30

### Added
- **Kimi K2 Reasoning Content Preservation**: Added support for preserving reasoning_content across multi-turn conversations with Kimi K2 Thinking model, improving accuracy and coherence in complex tasks.


## [v0.1.4] - 2025-12-29

### Added
- Improved DeepSeek support: DeepSeek V3 and R-series parser support and **XML-style tool-call parsing** (v3.2 hybrid format).
- Comprehensive unit & integration tests and fixtures covering tool calls and reasoning flows.

### Fixed
- Fixed empty-response retry error handling to respect HTTP errors and avoid inconsistent retries.
- Fixed Kimi tool-call reliability (missed tool-calls when tokens split across streaming chunks).

### Changed
- Reduced empty-response retry aggressiveness and improved provider error reporting.


## [v0.1.3] - 2025-12-26

### Fixed
- Improved reliability for Kimi model tool calls: fixed an issue where tool-calls could be missed when tokens were split across streaming chunks.
- Fixed handling of empty model responses and retry behavior so HTTP errors during retries are handled consistently (reduces inconsistent empty-response retries).
- Reduced excessive retry aggressiveness to avoid generating extra requests against overloaded endpoints.

## [v0.1.2] - 2025-12-25

### Added
- **Edit Tools Configuration**: Models can now declare their editing capabilities (e.g., `code-rewrite`, `find-replace`) via the `editTools` setting in model configuration
- **Parameter Transmission Verification**: Added comprehensive test suite confirming all request parameters (temperature, top_p, max_tokens, etc.) are correctly sent to APIs

### Fixed
- **Unified Retry Mechanism**: Implemented consistent retry behavior across all model parsers
  - Eliminated parser-specific fallback logic for cleaner, more predictable behavior
  - Improved handling of reasoning-only responses to properly trigger retries
  - Better separation of text and thinking parts in progress reporting

## [0.1.1] - 2025-12-24

### Added
- Improved DeepSeek model support: more reliable tool-call parsing and handling, reduced partial-token leakage in streaming responses, and a system-prompt transformation that improves tool performance when tools are used.

### Fixed
- Prevented JSON/tool call arguments from leaking into the model reasoning display (DeepSeek, Kimi).
- Improved stream error detection and reporting with clearer "Connection Error" messages and better status extraction.
- Fixed partial tool-token buffering that could emit internal tokens in the chat UI.
- Various parser robustness and reliability fixes.

## [0.1.0] - 2025-12-22

### Added
- Initial public release.
- Token usage display in the status bar (compact format) and optional session accumulation (configurable via `idoapico.showTokenUsage`, `idoapico.showSessionTotal`, and `idoapico.enableSessionTokenCounting`).
- Robust parsing for XML-style `<function_calls>` emitted in streaming model responses; tolerant parsing for malformed fragments and guarded buffer handling.
- Support for `reasoning_effort` parameter (`none`, `xhigh`, or object `{effort, summary}`).
- Dual-level request delay support: global `idoapico.delay` and per-model `request_delay` (model-specific overrides global).
- System Replace configuration: regex-based replacements for system messages.
- Enhanced error detection for stream responses (detect error payloads embedded in stream data when status is 200).
- Deterministic tool call ID sanitization to handle long or non-standard IDs.
- Initial support for a variety of providers and models (Google Gemini variants, OpenAI, Qwen 3 Next, Kimi K2, Minimax M2, Deepseek v3.1).

### Fixed
- Prevented partial XML fragments from being emitted as text in the chat UI for highly fragmented `<function_calls>` streams.
- Fixed leading-whitespace issue in the first response chunk for DeepSeek/Qwen models.
- Improved stream error detection and reporting for validation/stream errors.
- Fixed Kimi tool call ID generation to use `functions.{name}:{index}` and relaxed hallucinated tool-name handling.
- Fixed token leakage where internal model tokens (e.g., `<|tool_call|>`) were visible in chat output.
- Resolved issue where tool calls with hexadecimal IDs were incorrectly filtered out.
