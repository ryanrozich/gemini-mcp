# Tool Reference

Full per-tool detail for the 18 tool groups implemented in `src/tools/*.ts`. `AGENTS.md` carries
a condensed index (name, file, one-line purpose) for every-task orientation; this document is the
"read on demand" deep reference for parameters and behavior.

## Tools Implemented

1. **Query** (`query.ts`): Direct queries to Gemini with thinking level control
   - `thinkingLevel`: minimal, low, medium, high

2. **Brainstorm** (`brainstorm.ts`): Collaborative brainstorming between Claude and Gemini

3. **Analyze** (`analyze.ts`): Code and text analysis using Gemini

4. **Summarize** (`summarize.ts`): Content summarization at different detail levels

5. **Image Gen** (`image-gen.ts`):
   - `gemini-generate-image`: Generate images with Nano Banana Pro
     - Up to 4K resolution (1K, 2K, 4K)
     - 10 aspect ratios (1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9)
     - Google Search grounding for real-world accuracy
     - Returns base64 that Claude can SEE!
   - `gemini-image-prompt`: Generate prompts for other image tools (legacy)

6. **Image Edit** (`image-edit.ts`): Multi-turn conversational image editing
   - `gemini-start-image-edit`: Start an editing session
   - `gemini-continue-image-edit`: Continue refining with follow-up prompts
   - `gemini-end-image-edit`: Close a session
   - `gemini-list-image-sessions`: List active sessions

7. **Video Gen** (`video-gen.ts`):
   - `gemini-generate-video`: Start async video generation with Veo
   - `gemini-check-video`: Check video generation status and download

8. **Code Execution** (`code-exec.ts`):
   - `gemini-run-code`: Write and execute Python code
   - Supports: numpy, pandas, matplotlib, scipy, scikit-learn, tensorflow
   - Returns charts as images Claude can see

9. **Google Search** (`search.ts`):
   - `gemini-search`: Real-time web search with inline citations
   - Returns grounded responses with source URLs

10. **Structured Output** (`structured.ts`):
    - `gemini-structured`: JSON responses with schema validation
    - `gemini-extract`: Convenience tool for entities, facts, sentiment, keywords

11. **YouTube Analysis** (`youtube.ts`):
    - `gemini-youtube`: Analyze YouTube videos by URL with clipping
    - `gemini-youtube-summary`: Quick video summarization

12. **Document Analysis** (`document.ts`):
    - `gemini-analyze-document`: Analyze PDFs, DOCX, spreadsheets
    - `gemini-summarize-pdf`: Quick PDF summarization
    - `gemini-extract-tables`: Extract tables from documents

13. **URL Context** (`url-context.ts`):
    - `gemini-analyze-url`: Analyze content from URLs
    - `gemini-compare-urls`: Compare content between two URLs
    - `gemini-extract-from-url`: Extract specific data types from URLs

14. **Context Caching** (`cache.ts`):
    - `gemini-create-cache`: Cache large documents for repeated queries
    - `gemini-query-cache`: Query cached content
    - `gemini-list-caches`: List active caches
    - `gemini-delete-cache`: Delete a cache

15. **Speech/TTS** (`speech.ts`):
    - `gemini-speak`: Text-to-speech with 30 voices
    - `gemini-dialogue`: Multi-speaker dialogue generation
    - `gemini-list-voices`: List available voices

16. **Token Counting** (`token-count.ts`):
    - `gemini-count-tokens`: Count tokens and estimate costs

17. **Deep Research** (`deep-research.ts`):
    - `gemini-deep-research`: Start autonomous multi-step research (async)
    - `gemini-check-research`: Check research status and get results
    - `gemini-research-followup`: Ask follow-up questions on completed research
    - Note: Research typically takes 5-20 minutes, max 60 minutes
    - Full response saved to `GEMINI_OUTPUT_DIR` as JSON

18. **Image Analysis** (`image-analyze.ts`): *Community contribution by @acreeger*
    - `gemini-analyze-image`: Analyze images with object detection and bounding boxes
      - Supports JPEG, PNG, WebP, HEIC, HEIF, GIF
      - Returns `box_2d` (normalized 0-1000) and `bbox_pixels` (pixel coordinates)
      - Structured JSON output with object labels and confidence
      - Thinking level support for complex analysis
      - Handles large files (>20MB) via Files API automatically

## Gemini 3 Specific Features

### Thinking Levels
- `minimal`: Fastest, minimal reasoning (Flash only)
- `low`: Fast responses, basic reasoning
- `medium`: Balanced reasoning (Flash only)
- `high`: Deep reasoning for complex tasks (default)

### Nano Banana Pro (Image Generation)
- Model: `gemini-3-pro-image-preview`
- Resolutions: 1K, 2K (default), 4K
- Google Search grounding for real-world accuracy
- High-fidelity text rendering

### Thought Signatures
- Handled automatically by the SDK when using chat sessions
- Required for multi-turn image editing
- Preserved in conversation history for function calling
