# Project Agent Rules

## Scope
- This repository contains the `ralph-rlm` OpenCode plugin implementation at `.opencode/plugins/ralph-rlm.ts`.
- Keep changes focused on plugin behavior, protocol docs, and packaging artifacts.

## Source of truth
- Runtime/plugin logic: `.opencode/plugins/ralph-rlm.ts`
- User-facing docs: `README.md`
- Publish metadata/scripts: `package.json`
- Loop config example for this repo: `.opencode/ralph.json`

## Build and verify
- Install dependencies with `bun install`.
- Typecheck with `bun run typecheck`.
- Build distributable with `bun run build`.
- Preferred verification for this repo: `bun run verify`.

## Coding guidelines
- Use strict TypeScript patterns already present in the file (narrow types, explicit unions, safe defaults).
- Keep plugin behavior deterministic and file-first; avoid hidden state outside protocol files.
- Preserve backward compatibility for config keys and tool names unless a breaking change is explicitly requested.
- Prefer small, targeted edits over broad refactors in the plugin file.

## Docs guidelines
- If behavior changes, update `README.md` in the same change.
- Keep examples copy-pasteable and aligned with actual scripts/config.

## Ralph / RLM workflow note
- In this repository, AGENT.md contains static project guidance.
- Loop-specific and attempt-specific strategy belongs in `RLM_INSTRUCTIONS.md`, `PLAN.md`, `CURRENT_STATE.md`, and `NOTES_AND_LEARNINGS.md`.
