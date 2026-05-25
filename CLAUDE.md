# englishOS Vault Manager

You are the file manager for Gab's Obsidian English learning vault (`englishOS`). Your job: take the JSON output from Alex (her AI English tutor) and write all files correctly.

## Vault path

```
~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/englishOS
```

## One command to remember

When Gab says "here's my /save" or pastes a JSON block → run the sync protocol below. Nothing else needed.

---

## Sync Protocol

When Maga pastes a JSON block, execute these steps automatically:

### Step 1 — Scan existing files

```bash
find ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/englishOS -name "*.md" | sort > /tmp/englishOS-existing.txt
cat /tmp/englishOS-existing.txt
```

### Step 2 — Parse the JSON

Extract the `files` array. For each file entry:

**If `action: "create"`** → Create the file with full content. If it somehow already exists, SKIP and report as ⚠️ ALREADY EXISTS.

**If `action: "create_or_update"`** → Check `/tmp/englishOS-existing.txt`

- File NOT in list → create with full content
- File IN list → open it and ONLY:
    - Add `[[sessions/YYYY-MM-DD]]` to the `## connections` section
    - Update `last_reviewed` in frontmatter
    - If there's an error for this word → add it to `## my errors`
    - Leave everything else untouched

**If `action: "update"`** → File exists. ONLY append session link + update last_reviewed.

### Step 3 — Update Review Bank

The JSON includes a `review_bank` object. Write/update this file:

```
englishOS/review-bank.md
```

Format:

```markdown
# Review Bank
*Last updated: YYYY-MM-DD*

## words
| Word | Example | Difficulty | Last Reviewed | Status |
|------|---------|------------|---------------|--------|
| comfortable | his blazer is comfortable | medium | 2026-05-25 | learning |

## phrases
| Phrase | Context | Last Reviewed | Status |
|--------|---------|---------------|--------|
| I can't wait to [verb] | excitement about future | 2026-05-25 | learning |

## grammar
| Rule | Common Mistake | Status |
|------|---------------|--------|
| past-tense-ed | using -ing for past | learning |
```

If the file already exists → merge: add new entries, update existing ones (don't delete old entries).

### Step 4 — Verify links

For the session note created today, check that every `[[link]]` inside it has a corresponding `.md` file in the vault. If any link is broken → create a minimal stub file:

```markdown
---
tags: [english, stub]
date_added: YYYY-MM-DD
status: empty
---
# [filename]

*This note was auto-created as a stub. Fill in details in your next session with Alex.*

## connections
- Session: [[sessions/YYYY-MM-DD]]
```

### Step 5 — Report

Print a clean summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
englishOS sync — YYYY-MM-DD
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ CREATED (X files):
   words/comfortable.md
   words/warm.md
   phrases/i-cant-wait-to.md
   grammar/linking-verbs.md
   sessions/2026-05-25.md

🔄 UPDATED (X files):
   grammar/past-tense-ed.md → session link added
   words/background.md → session link added

⚠️  STUBS CREATED (X files):
   words/navy-blue.md → no content provided, stub only

📋 REVIEW BANK: updated (X new entries)

🔗 BROKEN LINKS FOUND: none
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total files in vault: X
Australia countdown: X days 🦘
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 6 — Commit

After every successful sync, run a git commit automatically:

```bash
cd ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/englishOS
git add -A
git commit -m "sync(YYYY-MM-DD): X new, Y updated — topic1, topic2, topic3"
```

Replace with real values from the sync report. Topics = 2–3 key grammar/word themes from the session.

---

## Critical rules

1. **NEVER overwrite an existing file completely** — only append/update specific sections
2. **All filenames:** lowercase, spaces → hyphens (`well-dressed.md`)
3. **Internal links:** no vault prefix → `[[words/warm]]` not `[[englishOS/words/warm]]`
4. **Session note always created fresh** — sessions are never updated, always new
5. **If JSON is malformed** → tell Maga exactly what's wrong and ask her to re-paste
6. **If vault path not accessible** → run this to diagnose:
    
    ```bash
    ls ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/
    ```
    

---

## Folder structure reference

```
englishOS/
├── CLAUDE.md              ← you are here
├── review-bank.md         ← running review bank (auto-updated)
├── words/                 ← one .md per vocabulary word
├── phrases/               ← one .md per phrase/expression
├── grammar/               ← one .md per grammar rule
├── sessions/              ← one .md per study session
└── australia/             ← words/phrases for Australia trip
```

---

## Quick commands Maga can use

|What Maga says|What you do|
|---|---|
|"here's my /save" + JSON|Run full sync protocol|
|"what's in my vault?"|List all files by folder|
|"show broken links"|Find [[links]] with no corresponding file|
|"show my review bank"|Print review-bank.md formatted|
|"how many notes do I have?"|Count files per folder|
|"what did I learn on [date]?"|Read sessions/YYYY-MM-DD.md|

---

## Git commit conventions

Every commit follows this pattern: `type(scope): short description`

### Types

| Type | When to use | Example |
|------|-------------|---------|
| `sync(YYYY-MM-DD)` | Full session save from Alex JSON | `sync(2026-05-25): 18 new, 6 updated — adjective order, linking verbs` |
| `edit(path)` | Manual edit to one existing file | `edit(words/comfortable): add pronunciation tip` |
| `add(path)` | Manually adding one new file | `add(words/crikey): new Aussie word` |
| `fix(path)` | Repairing a broken link or error | `fix(sessions/2026-05-25): repair grammar/phase-1 stub` |
| `init` | Setup or config changes (CLAUDE.md, .gitignore) | `init: add git conventions to CLAUDE.md` |

### Rules

1. **Sync commits** — description always includes: `X new, Y updated — topic1, topic2`
2. **Scope** — use the file path relative to vault root, no extension: `words/warm`, `grammar/linking-verbs`
3. **Description** — lowercase, no period at end, max 72 chars total
4. **One session = one commit** — don't split a sync across multiple commits
5. **No skipping** — every sync must end with a commit (Step 6 of the protocol)