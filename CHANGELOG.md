# Changelog

All notable changes to the `idoapico` extension are recorded in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v0.2.6] - 2026-01-28

### Added
- Tests covering SSE comment filtering and Gemini tool-call index collision cases to improve reliability.

### Fixed
- Prevented SSE control/comment lines (e.g. `: ping`) from corrupting streamed tool-call JSON arguments.
- Prevented Gemini parallel tool-call index collisions from overwriting tool-call arguments by isolating buffers per call.

## [v0.2.5] - 2026-01-15

### Added
- MiniMax M2.1 native tool-call support and related parser improvements.

### Fixed
- Stability fixes for the MiniMax parser.

## [v0.2.4] - 2026-01-14

### Added
- MiniMax M2.1 parser with XML-style tool-call parsing and improved DeepSeek support.

## [v0.2.3] - 2026-01-13

### Fixed
- GLM-4.7 XML tag cleanup to prevent raw tags from appearing in chat.

## [v0.2.2] - 2026-01-05

### Added
- Real-time token rate display and configurable update interval.

### Fixed
- Output cleanup for Chutes proxy and improvements to reasoning content handling.

## [v0.2.1] - 2026-01-02

### Fixed
- Chutes Kimi K2 proxy output cleanup and streaming robustness fixes.

## [v0.2.0] - 2026-01-01

### Added
- Kimi-K2-Thinking Deepinfra compatibility and format auto-detection.

## [v0.1.5] - 2025-12-30

### Added
- Kimi K2 reasoning content preservation across multi-turn conversations.

---

For full developer-focused details (tests, files changed, implementation notes), see `DEV_CHANGELOG.md`.
