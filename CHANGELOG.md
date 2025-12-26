# Change Log

All notable changes to the "idoapico" extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v0.1.3] - 2025-12-26

### Fixed
- Improved reliability for Kimi model tool calls: fixed an issue where tool-calls could be missed when tokens were split across streaming chunks.
- Fixed handling of empty model responses and retry behavior so HTTP errors during retries are handled consistently (reduces inconsistent empty-response retries).
- Reduced excessive retry aggressiveness to avoid generating extra requests against overloaded endpoints.


## [0.1.1] - 2025-12-24

### Added
- Improved DeepSeek model support: more reliable tool-call parsing and handling, reduced partial-token leakage in streaming responses, and a system-prompt transformation that improves tool performance when tools are used.

### Fixed
- Prevented JSON/tool call arguments from leaking into the model reasoning display (DeepSeek, Kimi).
- Improved stream error detection and reporting with clearer "Connection Error" messages and better status extraction.
- Fixed partial tool-token buffering that could emit internal tokens in the chat UI.
- Various parser robustness and reliability fixes.


## [v0.1.2] - 2025-12-25

### Added
- Provider Request Parameters Investigation Test Suite: Komprehensif test untuk menginvestigasi dan memverifikasi bahwa semua parameter request (temperature, top_p, max_tokens, dll) dikirim dengan benar ke API.
	- Test File: `src/test/integration/provider-request-parameters.test.ts`
	- Test Coverage: 5 test cases untuk memverifikasi temperature, multiple parameters, undefined handling, reasoning parameters, dan message format
	- Investigation Results: Verifikasi bahwa semua parameter sudah dikirim dengan benar - tidak ada bug ditemukan
	- Mock Testing: Menggunakan nock untuk mock HTTP requests dan capture request body
	- Documentation: `TEST_PARAMETER_INVESTIGATION.md` berisi detailed report dari investigation

- Edit Tools Capability: Implemented support for `editTools` in the model configuration, allowing models to declare their editing capabilities (e.g., `code-rewrite`, `find-replace`).
	- Type Definition: Added `editTools?: string[]` to the `ModelItem` interface in `src/types.ts`.
	- Schema Update: Updated `package.json` to include the `editTools` array property in the `idoapico.models` configuration schema.
	- Capability Mapping: The `prepareLanguageModelChatInformation` function in `src/provideModel.ts` now maps the configured `editTools` to the `capabilities` object for the language model.
	- Documentation: Updated `README.md` to document the new `editTools` setting with usage examples.

### Files Modified
- `src/test/integration/provider-request-parameters.test.ts`: New comprehensive test suite untuk parameter investigation
- `src/types.ts`: Added `editTools` to `ModelItem` interface.
- `package.json`: Updated extension configuration schema.
- `src/provideModel.ts`: Implemented capability mapping.
- `README.md`: Added user-facing documentation.

### Test Results
- ✅ All 5 parameter tests passing
- ✅ No bugs found in parameter transmission
- ✅ All parameter types (temperature, max_tokens, top_p, frequency_penalty, presence_penalty, min_p, reasoning_effort, etc.) transmitted correctly
- ✅ Undefined parameters handled gracefully (not sent to API)

### Fixed
- Unified Retry Mechanism Implementation: Removed Kimi-specific fallback logic and implemented consistent retry behavior across all parsers.
	- KimiParser Fallback Removal: Eliminated `accumulatedReasoning` state and fallback emission logic in `flush()` method to prevent unwanted text output during reasoning-only streams.
	- OpenAIParser Early Return Fix: Updated `parseResponseChunk()` to process reasoning content before checking for empty deltas, ensuring reasoning-only responses trigger retry instead of being treated as empty.
	- MockProgress Thinking Part Detection: Enhanced `getTextParts()` and `getThinkingParts()` filters to properly separate text and thinking parts, including constructor name variants (`LanguageModelThinkingPart`, `el`).
	- Test Infrastructure Updates: Modified `robust-retry.test.ts` to use `delta.reasoning` field instead of `<think>` tags, and updated test expectations to validate retry triggering for reasoning-only streams.
	- Obsolete Test Removal: Deleted `kimi-fallback.test.ts` as fallback logic is no longer needed with unified retry mechanism.

### Technical Implementation
- KimiParser (`src/parsers/kimi.ts`):
	- Removed `accumulatedReasoning` state variable and related fallback logic in `flush()` method
	- Streamlined reasoning handling to rely on unified retry mechanism
- OpenAIParser (`src/parsers/openai.ts`):
	- Reordered logic in `parseResponseChunk()` to process `incomingReasoning` before empty delta checks
	- Ensures reasoning content is properly emitted as thinking parts before retry evaluation
- MockProgress (`src/test/helpers/vscode-mocks.ts`):
	- Enhanced `getTextParts()` filter to exclude thinking parts with constructor names `'LanguageModelThinkingPart'` and `'el'`
	- Improved `getThinkingParts()` to detect thinking parts across different constructor name variants
- Test Suite (`src/test/integration/robust-retry.test.ts`):
	- Updated test data to use `delta.reasoning` field for proper reasoning emission
	- Modified assertions to validate that reasoning-only streams trigger retry without emitting text

### Files Modified
- `src/parsers/kimi.ts`: Removed fallback logic and accumulatedReasoning state
- `src/parsers/openai.ts`: Fixed early return condition for reasoning processing
- `src/test/helpers/vscode-mocks.ts`: Enhanced thinking part detection filters
- `src/test/integration/robust-retry.test.ts`: Updated test data and expectations
- `src/test/integration/kimi-fallback.test.ts`: Deleted obsolete test file
- `CHANGELOG.md`: Added user-facing documentation for unified retry mechanism
- `DEV_CHANGELOG.md`: Added technical implementation details

### Test Results
- ✅ All 132 tests passing (24 pending)
- ✅ Retry mechanism triggers correctly for reasoning-only streams across all parsers
- ✅ No unwanted text emission during reasoning-only responses
- ✅ Consistent error handling behavior across OpenAI, Kimi, and other parsers
- ✅ No regressions in existing functionality

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
