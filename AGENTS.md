# Gemini MCP Server Project Guide

This document provides essential information for AI coding agents when working with this Gemini MCP server project.

## Project Overview

This project is an MCP (Model Context Protocol) server that connects Claude to Google's Gemini 3 AI models. It enables bidirectional collaboration between Claude and Gemini, allowing them to work together by sharing capabilities and agent tools.

**Version:** 0.8.0
**Package:** @rlabs-inc/gemini-mcp
**MCP Registry:** io.github.rlabs-inc/gemini-mcp

## Key Components

- `src/index.ts`: Dual-mode entry point (MCP server or CLI)
- `src/server.ts`: MCP server implementation
- `src/cli/`: CLI implementation with themes and commands
- `src/gemini-client.ts`: Client for Google's Generative AI API (includes thinking levels, image/video generation)
- `src/utils/logger.ts`: Logging utilities with configurable verbosity
- `src/tools/*.ts`: Various tool implementations for integration with Claude Code

## Tools Implemented

18 tool groups in `src/tools/*.ts`. Full per-tool parameter/behavior detail (and the Gemini-3
thinking-level / Nano Banana Pro / thought-signature specifics): `docs/TOOLS.md`.

1. **Query** (`query.ts`) — direct queries with thinking-level control
2. **Brainstorm** (`brainstorm.ts`) — collaborative Claude↔Gemini brainstorming
3. **Analyze** (`analyze.ts`) — code and text analysis
4. **Summarize** (`summarize.ts`) — content summarization at different detail levels
5. **Image Gen** (`image-gen.ts`) — Nano Banana Pro image generation + legacy prompt tool
6. **Image Edit** (`image-edit.ts`) — multi-turn conversational image editing (start/continue/end/list sessions)
7. **Video Gen** (`video-gen.ts`) — async Veo video generation + status/download
8. **Code Execution** (`code-exec.ts`) — sandboxed Python execution, returns chart images
9. **Google Search** (`search.ts`) — grounded web search with inline citations
10. **Structured Output** (`structured.ts`) — schema-validated JSON + entity/fact/sentiment/keyword extraction
11. **YouTube Analysis** (`youtube.ts`) — video analysis by URL + clipping, quick summaries
12. **Document Analysis** (`document.ts`) — PDF/DOCX/spreadsheet analysis, summarization, table extraction
13. **URL Context** (`url-context.ts`) — analyze/compare/extract from URLs
14. **Context Caching** (`cache.ts`) — create/query/list/delete caches for repeated large-document queries
15. **Speech/TTS** (`speech.ts`) — text-to-speech (30 voices) + multi-speaker dialogue
16. **Token Counting** (`token-count.ts`) — token counts + cost estimates
17. **Deep Research** (`deep-research.ts`) — async autonomous research (5-20 min, max 60 min), saved to `GEMINI_OUTPUT_DIR`
18. **Image Analysis** (`image-analyze.ts`) — object detection/bounding boxes (*community contribution by @acreeger*)

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GEMINI_API_KEY` | Yes | - | Google Gemini API key |
| `GEMINI_MODEL` | No | - | Override model for init test |
| `GEMINI_PRO_MODEL` | No | `gemini-3-pro-preview` | Pro model (Gemini 3) |
| `GEMINI_FLASH_MODEL` | No | `gemini-3-flash-preview` | Flash model (Gemini 3) |
| `GEMINI_IMAGE_MODEL` | No | `gemini-3-pro-image-preview` | Image model (Nano Banana Pro) |
| `GEMINI_VIDEO_MODEL` | No | `veo-2.0-generate-001` | Video model |
| `GEMINI_OUTPUT_DIR` | No | `./gemini-output` | Output directory for generated files |
| `VERBOSE` | No | `false` | Enable verbose logging |
| `QUIET` | No | `false` | Minimize logging |
| `GEMINI_ENABLED_TOOLS` | No | - | Comma-separated list of tool groups to load |
| `GEMINI_TOOL_PRESET` | No | - | Preset profile: minimal, text, image, research, media, full |

## Command Line Options

- `-v, --verbose`: Enable verbose logging
- `-q, --quiet`: Run in quiet mode
- `-h, --help`: Show help message

## Development Commands

```bash
bun install        # Install dependencies
bun run build      # Build the project
bun run dev        # Run in development mode (with watch)
bun run dev -- -v  # Run with verbose logging
bun run typecheck  # Type check without emitting
bun run format     # Format code with Prettier
bun run lint       # Lint code with ESLint
```

## Dependencies

- `@google/genai`: ^1.34.0 - Google Generative AI SDK
- `@modelcontextprotocol/sdk`: 1.22.0 - MCP SDK (pinned; 1.23.0+ causes TypeScript OOM)
- `zod`: 3.24.3 - Schema validation (pinned for compatibility with MCP SDK)
- `zod-to-json-schema`: 3.24.5 - Zod to JSON Schema conversion (pinned; 3.25+ requires zod/v3 export)

## Architecture Notes

- The server uses stdio transport for communication with Claude Code
- Image generation returns base64 data that Claude can render inline
- Video generation is async - returns operation ID for polling
- Generated files are saved to `GEMINI_OUTPUT_DIR`
- Thinking levels control reasoning depth in Gemini 3
- Image editing uses chat sessions with automatic thought signature handling

## Changelog

See `docs/CHANGELOG.md` for version history (latest: v0.8.0 — image analysis tool).

## Future Roadmap

See `docs/ROADMAP.md` for implementation plan. Remaining features:
- **Lyria Music Generation**: Real-time music via WebSocket (complex)
- **Live Streaming API**: Real-time bidirectional streaming
- **File Search**: Search through uploaded file stores

<!-- catalyst-house-rules:begin -->
## Working the Loop (every agent — interactive too, not just skills)

These are house rules for anyone touching this repo's dev / PR / ticket workflow — whether you are
running a slash-command skill **or** working interactively and ad-hoc. They are **default
reflexes, not skill internals**: reach for them without being told, even on a one-off PR you opened
by hand. They defer their mechanism to the `catalyst-dev` plugin, available in every Catalyst-managed
repo. If that plugin is somehow unavailable, that is a broken environment — repair it (reload the
plugin) rather than routing around it. For GitHub state only, a single **bounded** `gh` check is an
acceptable last resort while you do; never a poll loop, and never a raw Linear API read (the
replica-read rule below is absolute).

- **Waiting on GitHub / CI / Linear state → subscribe to the event log, don't poll.** To block on a
  state change (a PR merged, CI turning green, a review posted, a push to a branch, a ticket
  transition), wait on the unified Catalyst event log instead of re-querying in a loop. Reach for
  the `catalyst-dev:wait-for-github` skill for GitHub events and `catalyst-dev:monitor-events` for
  the general wait-for-a-state-change pattern (they own the broker/webhook mechanics — don't
  reimplement them). A `gh` / `linearis` poll loop burns shared-quota API budget and silently misses
  reaction-only signals (next bullet). When the broker / webhook infra is down — or absent on a host
  with no event-log substrate — these skills degrade to a bounded single-event wait and a bounded
  poll becomes acceptable, but that degradation is the fallback, never your opening move.
- **Judging an automated code review → a clean pass is a reaction, not a review object.** The
  automated PR reviewer signals "no issues" with a 👍 reaction (or a terse "no major issues"
  comment) **instead of** opening review threads — detect it via the PR's reactions and issue
  comments, not only the reviews API. Recognizing the clean pass does **not** waive the rule that a
  PR is mergeable only once **every** review thread has been addressed and resolved.
- **Reading one Linear ticket → the freshness-gated local replica, not bare `linearis`.**
  Invoke the `catalyst-dev:linearis` skill and follow its "Reading Linear" contract — it reads the
  local replica behind a freshness gate (via its `linear_read_ticket` helper, run in the plugin's
  skill context) and does the loud stale/absent fallback for you. Don't hand-roll the read yourself:
  an **un-gated** `sqlite3` of the replica skips the freshness check (you may read stale data or
  create an empty DB), and a bare `linearis issues read <ID>` hits the rate-limited API and 429s the
  shared fleet quota — don't reach for it even as a fallback; the skill's helper owns the loud
  stale/absent path. Writes and list/search go through `linearis`.
<!-- catalyst-house-rules:end -->
