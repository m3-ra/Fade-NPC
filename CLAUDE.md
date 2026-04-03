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

## Translations

`Fade/tra/` contains language subdirectories (`english/`, `french/`, etc.), each with:
- `FADE.TRA` — main string file (~7080 entries, @0–@7079)
- `SETUP-FADE.TRA` — installer UI strings

**French translation** (`Fade/tra/french/`) is complete as of 2026-04-01. Translation rules used:
- Informal register, **tu** form (never vous)
- All WeiDU tokens preserved verbatim: `<CHARNAME>`, `<PRO_HESHE>`, `<PRO_HISHER>`, `[soundfile]` tags, `~` delimiters, `@N` IDs
- Proper nouns untranslated: character names (Fade, Aran, Imoen, Irenicus, Bodhi, Rasaad, Dorn, etc.), place names (Athkatla, Faerûn, Calimport, etc.), game mechanics labels
- Kit/class names use official French BG translations: Shadowdancer → **Maître des ombres**
- Faction names are translated: Shadow Thieves → **Voleurs de l'ombre**, Shadow Thief (singular member) → **Voleur/Voleuse de l'ombre**
- Title: Shadowmaster (Aran Linvail's title as boss of the Shadow Thieves) → **Maître des ombres**

**Translation workflow:**
- Always translate/review dialogue in bulk using the `.d` files in `Fade/Dialogues/` to understand the full conversation tree — never work on single lines or unrelated lines in isolation
- When Fade refers to the player character and gender agreement is needed, use WeiDU's dual-string syntax: `@ID = ~masculine version~ ~feminine version~`
  - Example: `@0 = ~Alors, tu es prêt à partir, <CHARNAME> ?~ ~Alors, tu es prête à partir, <CHARNAME> ?~`
  - Never use slash/parenthesis notation (`prêt(e)`, `le/la`, `charmeur(se)`) — always use dual-string instead
- Never use `-` or `--` mid-sentence in French — use commas, parentheses, ellipsis, or semicolons instead
- Never abbreviate gold pieces as `po` — always write `pièces d'or` in full (the English `gp` abbreviation was a mistake and should not be carried over)

**Translation review procedure** — follow this every time, without exception:

1. **Read the `.d` file first** — understand every dialogue branch before touching any string. Work one complete conversation chunk at a time, never isolated lines.

2. **Read EN and FR in parallel per chunk** — for every string, ask: does the French sound like something a French person would actually say? If not, rewrite it. "Technically correct" is not enough.

3. **Idiomatic + tone, both simultaneously:**
   - Idiomatic: no calques, no word-for-word structures, no anglicisms
   - Tone: match the speaker's register (Fade is flirty/casual; player lines must match the option's attitude — smirk, sincere, sarcastic, stumbling)

4. **Gender check on every line** — for each string, ask: who is speaking? who are they speaking about?
   - Fade speaking about herself → feminine agreements
   - Fade speaking about a male player (romance files, `E3FADEROMANCEACTIVE=1` or `=2`) → masculine agreements, no dual-string needed
   - Non-romance files with `<CHARNAME>` → dual-string `~masc~ ~fem~` wherever gender agreement is required

5. **Flag all issues before editing** — list every problem found in the chunk before making any edits, so nothing gets silently skipped.

## Key Conventions

- All string literals in `.d` and `.tp2` files are externalized to `.tra` files and referenced as `@NNN` (e.g., `@1`, `@100`).
- `AUTO_TRA ~Fade/Tra/%s~` in the TP2 header means WeiDU auto-selects language TRA files.
- `HANDLE_CHARSETS` with `infer_charsets = 1` manages encoding for different game editions.
- `SHADOWD/` contains creature variants for the optional Shadowdancer class component.
