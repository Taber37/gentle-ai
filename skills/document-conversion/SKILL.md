---
name: gentle-ai-document-conversion
description: "Trigger: document extraction, conversion, PDF, Office, OCR, URL, image, EXIF, audio, media. Safely convert supported inputs with local Nix CLIs."
license: Apache-2.0
metadata:
  author: "gentleman-programming"
  version: "1.1"
---

## Activation Contract

Load for document extraction/conversion, PDF/Office, OCR-required, URL, image/EXIF, or audio/media workflows. Ordinary YouTube transcripts belong to `yt-transcript`; use MarkItDown for YouTube only in a broader URL/document workflow.

## Hard Rules

- Use MarkItDown `0.1.7` as the default for every supported local or remote input. Use AnyDoc `0.2.4` only for local formats MarkItDown lacks, or PDF OCR.
- Never install, use `npx`, provision Taber.Dots, use `-p`/`--use-plugins`, MCP, MarkItDown Azure/Document Intelligence/Content Understanding, or automatic cloud fallback.
- For each converter/helper, set its variable from `realpath "$(command -v <command>)"`; require `/nix/store/...` and its exact version before invoking that same quoted variable. Expected versions: AnyDoc `0.2.4` (`-V`), MarkItDown `markitdown 0.1.7` (`-v`), EXIFTool `13.59`, FFmpeg `9.0`.
- A non-zero MarkItDown result stops with no retry or AnyDoc fallback, except that a PDF explicitly known to be scanned/image-only may enter the OCR consent gate. A zero-result PDF output containing only whitespace may enter that gate; do not treat every PDF failure as OCR-required. All other empty output stops.
- URL/YouTube routes require user intent and a transmission disclosure. Before private or user-supplied audio transcription, disclose MarkItDown's Google recognition service and require confirmation.
- Never solicit, print, log, or pass an API key in argv. Hosted OCR may use only preconfigured `FIRECRAWL_API_KEY` from the environment after per-document consent.

## Decision Gates

| Input or result | Route/action |
| --- | --- |
| PDF, DOCX, PPTX, XLS/XLSX, CSV, EPUB, MSG, ZIP, HTML, JSON/XML/text, images/EXIF, audio/media, or supported URL/document workflow | MarkItDown. |
| Local DOC, ODT, RTF, PPT, ODS, ODP, or another locally supported format MarkItDown lacks | AnyDoc. |
| PDF explicitly known to be scanned/image-only, or local MarkItDown extraction exits zero but output contains only whitespace | OCR consent gate; do not upload automatically. A non-zero PDF failure alone is not OCR evidence. |
| OCR consent granted for that exact PDF | AnyDoc `--ocr hosted`; upload only that PDF. |
| OCR consent declined or ambiguous | Stop without upload. |
| Unsupported input | Stop; report unsupported input. |
| AnyDoc exit 1 / 2 / 3 outside the consented hosted-OCR route | Stop: conversion failure / usage or format error / OCR required. |
| AnyDoc exits zero but output is empty, or a non-PDF MarkItDown route exits zero but output is empty | Stop; report empty-output failure. |
| Explicit plugin-dependent request | After canonical MarkItDown validation, only run `"$converter" --list-plugins` to diagnose; report plugin execution is outside this skill and stop. |

## Execution Steps

1. Select and state the route. For a local source, set `source=$(realpath "$requested_source")`. For a remote source, require an explicit HTTP(S) URL and set `source="$requested_source"` only after intent and network disclosure. Set `output_parent=$(realpath "$(dirname "$requested_output")")`; derive `output_name=$(basename "$requested_output")`; reject `.` or `..`; set `output="$output_parent/$output_name"`. For local sources require `"$source" != "$output"`; refuse an existing `"$output"` unless the user explicitly authorized overwrite.
2. For a PDF OCR gate, before each exact upload disclose that the complete PDF leaves the machine for Firecrawl Parse; state the effective endpoint (`FIRECRAWL_API_URL` when configured, otherwise `https://api.firecrawl.dev`), whether preconfigured environment-key or keyless mode is used, and that external handling and OCR accuracy are outside local control. Require explicit authorization for that document only; do not persist or generalize it. When that run created an empty local-PDF output, consent may authorize replacing only that generated empty output; never overwrite a pre-existing or non-empty output without separate explicit authorization.
3. Set `converter_name` to the selected `markitdown` or `anydoc`, then set `converter=$(realpath "$(command -v "$converter_name")")`; validate its Nix-store path and route-specific exact version. Invoke only `"$converter" "$source" -o "$output"`; for consented hosted PDF OCR, invoke AnyDoc only as `"$converter" "$source" -o "$output" --ocr hosted`. Neither converter documents `--`.
4. After MarkItDown, stop on non-zero. Treat output as empty unless `grep -q '[^[:space:]]' "$output"` succeeds. If an empty PDF exits zero, enter the OCR consent gate; otherwise stop on empty output. After consented AnyDoc OCR or any other successful route, require the same non-whitespace check; otherwise stop. For large output, use `-o` and read bounded sections; otherwise return a bounded preview. Stop on every gate failure.

## Output Contract

Return route and reason, source identifier, canonical output path, provenance/version evidence, status, bounded preview or stop reason, and required privacy/network disclosure. For OCR, include the consent outcome, endpoint, and environment-key/keyless mode, never a key.

## References

- No supporting files; this skill is self-contained.
