# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A **WeiDU mod** for Baldur's Gate II that adds Fade, a joinable NPC (Fey'ri Shadow Thief), romanceable by male characters of any race/alignment. Compatible with BG2:SoA, BG2:ToB, BGT, BG2:EE, and EET.

## Installation (Build)

WeiDU handles mod installation. From the BG2 game directory:

```bash
# Install the mod (interactive)
weidu Fade/Setup-Fade.tp2

# Or use the packaged setup executable (Windows)
Setup-Fade.exe
```

The GitHub Actions workflow (`.github/workflows/InfinityAutoPackager.yaml`) packages releases into `.iemod` and `.zip` formats automatically on publish.

## File Formats

| Extension | Purpose | Tool |
|-----------|---------|------|
| `.tp2` | Main installer script (WeiDU language) | WeiDU |
| `.baf` | AI/event scripts (compiled to `.bcs`) | WeiDU `COMPILE` |
| `.d` | Dialogue trees (compiled to `.dlg`) | WeiDU `COMPILE` |
| `.tra` | Translation strings, referenced as `@number` in `.d`/`.tp2` files | WeiDU |
| `.2da` | Tab-separated data tables | Text editor |
| `.cre` | Creature definitions (binary) | NearInfinity / DLTCEP |
| `.itm` | Item definitions (binary) | NearInfinity / DLTCEP |
| `.bam` | Sprite/animation graphics (binary) | BAMWORKSHOP |

## Architecture

**Entry point:** `Fade/Setup-Fade.tp2` — orchestrates everything. Reads this first to understand what gets installed.

**Naming convention:** All mod-specific resources use the `E3` prefix (e.g., `E3FADE`, `E3DEMON`) to avoid conflicts with base game resources.

**Game version branching:** TP2 uses `GAME_IS ~eet~`, `GAME_IS ~bg2ee~`, `GAME_IS ~bg2 bgt tob~` to conditionally install/compile different files.

**Dialogue system:**
- `Fade/Dialogues/Fade.d` — main Fade dialogue tree
- `Fade/Dialogues/BE3FADE.d` — primary expanded dialogue (largest file, ~143KB)
- `Fade/Dialogues/BE3Fad25.d` — ToB continuation dialogue
- String externalization in `Fade/tra/english/FADE.TRA` (7,000+ strings)

**Script system (`Fade/Scripts/`):**
- `E3Fade.baf` / `E3Fade25.baf` — main character AI (SoA/ToB)
- `E3FadeD.baf` — dialogue trigger scripts
- `E3Cut00*.baf` (10 files) — cutscene scripts
- `AR*.baf` — hooks into specific game areas
- `E3Aerie.baf`, `E3Jaheira.baf`, `E3Viconia.baf` — interactions with vanilla companions

**Chapter tracking:** EET installs offset chapter numbers by 12 (`bg2_chapter = 12 + i`) to avoid conflicts with BG1 chapters.

## Key Conventions

- All string literals in `.d` and `.tp2` files are externalized to `.tra` files and referenced as `@NNN` (e.g., `@1`, `@100`).
- `AUTO_TRA ~Fade/Tra/%s~` in the TP2 header means WeiDU auto-selects language TRA files.
- `HANDLE_CHARSETS` with `infer_charsets = 1` manages encoding for different game editions.
- `SHADOWD/` contains creature variants for the optional Shadowdancer class component.
