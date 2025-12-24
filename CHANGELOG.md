# Change Log

All notable changes to the "idoapico" extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

> Note: Implementation details, refactors, and test additions are recorded in `DEV_CHANGELOG.md`.
