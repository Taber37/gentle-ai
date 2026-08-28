---
name: gentle-ai-document-conversion
description: "Trigger: document extraction, conversion, PDF, Office, OCR, URL, image, EXIF, audio, media. Safely convert supported inputs with local Nix CLIs."
license: Apache-2.0
metadata:
  author: "gentleman-programming"
  version: "1.0"
---

## Activation Contract

Load for document extraction/conversion, PDF/Office, OCR-required, URL, image/EXIF, or audio/media workflows. Ordinary YouTube transcripts belong to `yt-transcript`; use MarkItDown for YouTube only in a broader URL/document workflow.

## Hard Rules

- Never install, use `npx`, provision Taber.Dots, use `-p`/`--use-plugins`, use hosted OCR, MCP, Azure, cloud provisioning, credentials, or automatic external integrations.
- For each invoked converter or helper, set its variable from `realpath "$(command -v <command>)"`; require `/nix/store/...` and its exact version before invoking that same quoted variable. Expected versions: AnyDoc `0.2.4` (`-V`), MarkItDown `markitdown 0.1.7` (`-v`), EXIFTool `13.59`, FFmpeg `9.0`.
- URL/YouTube routes require user intent and a transmission disclosure. Before private or user-supplied audio transcription, disclose MarkItDown's Google recognition service and require confirmation.

## Decision Gates

| Input or result | Route/action |
| --- | --- |
| Local textual PDF, legacy Word/PowerPoint, OpenDocument, RTF, EPUB, Office, spreadsheet, presentation, CSV | AnyDoc |
| MSG, recursive ZIP, HTML/JSON/XML/text, image/EXIF, audio, broader URL/document workflow | MarkItDown |
| Unsupported input | Stop; report unsupported input. |
| AnyDoc exit 1 / 2 / 3 | Stop: conversion failure / usage or format error / OCR required. |
| Non-zero MarkItDown | Stop; report its status. |
| Either converter exits zero but output is empty | Stop; report empty-output failure. |
| Explicit plugin-dependent request | After canonical MarkItDown validation, only run `"$converter" --list-plugins` to diagnose; report plugin execution is outside this skill and stop, whether present or absent. |

## Execution Steps

1. Select and state the route. For a local source, set `source=$(realpath "$requested_source")`. For a remote source, require an explicit HTTP(S) URL and set `source="$requested_source"` only after intent and network disclosure. Set `output_parent=$(realpath "$(dirname "$requested_output")")`; derive `output_name=$(basename "$requested_output")`; reject `.` or `..`; set `output="$output_parent/$output_name"`. For local sources require `"$source" != "$output"`; refuse an existing `"$output"` unless the user explicitly authorized overwrite.
2. Set `converter_name` to the selected `anydoc` or `markitdown`, then set `converter=$(realpath "$(command -v "$converter_name")")`; validate its Nix-store path and route-specific exact version, then execute only `"$converter" "$source" -o "$output"`. Validate and invoke helpers by their own quoted canonical variables. Neither converter documents `--`.
3. After either converter succeeds, require nonempty `"$output"`; otherwise stop. Never claim general partial-output detection.
4. For large output, use `-o` and read bounded sections; otherwise return a bounded preview. Stop on every gate failure; never retry through uploads, cloud OCR, or plugins.

## Output Contract

Return route and reason, source identifier, canonical output path, provenance/version evidence, status, bounded preview or stop reason, and required privacy/network disclosure.

## References

- `../../docs/skill-style-guide.md` — project skill contract.
