# Changelog

Version history for `@rlabs-inc/gemini-mcp`. `AGENTS.md` points here rather than carrying the
full history inline — it's read-on-demand context, not something every task needs.

## Key Changes in v0.8.0

- **Image Analysis Tool**: New `gemini-analyze-image` with object detection and bounding boxes (community contribution by @acreeger)
- **Thinking Level Support**: Added thinkingLevel parameter to image analysis for complex visual reasoning
- **Dual Coordinate Output**: Returns both normalized (box_2d) and pixel (bbox_pixels) coordinates

## Previous Versions

- v0.7.x: Published to MCP Registry, CLI renamed to gcli
- v0.6.3: Deep Research Agent, Token Counting
- v0.6.0: TTS with 30 voices, context caching, URL analysis
- v0.5.0: Code execution, Google Search, YouTube analysis, document analysis
- v0.3.0: Gemini 3 models, thinking levels, 4K image gen, multi-turn editing
