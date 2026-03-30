# SwiftWasm Skills Authoring Guide

## Core Principles
- Provide accurate, actionable guidance for Swift developers targeting WebAssembly.
- Keep instructions specific to the SwiftWasm toolchain and its ecosystem libraries.
- Prefer concrete code examples over abstract descriptions.
- Document platform differences between native Swift and Swift on Wasm explicitly.
- Keep each skill focused on its specific domain (JavaScriptKit, BridgeJS, or porting).

## Content Guidelines
- Keep `SKILL.md` procedural and concise — it is the entry point agents read first.
- Put detailed API references, patterns, and examples in `references/`.
- Put helper scripts in `scripts/` within the skill folder.
- Organize content by the tasks an agent performs: setup, coding patterns, troubleshooting.
- Use concrete examples that compile and run under the SwiftWasm toolchain.

## What To Include
- Project setup and toolchain installation steps.
- API usage patterns with paired Swift and JavaScript examples where applicable.
- Known limitations and workarounds for Swift on Wasm.
- Compatibility tables for frameworks and APIs.
- Environment diagnostic scripts and build configuration guidance.

## What To Exclude
- General Swift language tutorials unrelated to Wasm specifics.
- Native-only frameworks that have no Wasm path (unless documenting incompatibility in the porting skill).
- Speculative or unverified toolchain behavior.
- Duplicate content across skill folders — link instead.

## Skill Structure

Each skill directory at the repo root follows this layout:

```
<skill-name>/
  SKILL.md              # Primary skill document agents read
  references/           # Detailed reference material
  scripts/              # Helper scripts (optional)
```

Paths in `SKILL.md` are relative to the skill folder. For example, the JavaScriptKit
`scripts/doctor.py` is referenced as `scripts/doctor.py` within its `SKILL.md`.

## Language And Tone
- Use direct, technical language.
- Use imperative instructions for actionable steps.
- Prefer short paragraphs and bullet lists over prose.
- Keep examples compact and realistic.

## Updating Skills
- Update `SKILL.md` first when workflow or process changes.
- Update only the affected `references/*.md` files when specific guidance changes.
- Preserve section structure so agents can find guidance predictably.
- Keep `references/` filenames stable; add new files only for genuinely new topics.
- Validate that all relative paths in `SKILL.md` resolve correctly after changes.
- Update `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` when adding or removing skills.
