# The Lazy Second Brain — My Personal Setup Guide (v2)

> Obsidian + OpenCode (MiniMax M2.5 free) on Arch Linux.
> Dump notes → AI processes them → knowledge graph builds itself.

> **v2 Update:** Added terminal-first capture. No need to open Obsidian — just type.

---

## What This Is

A second brain setup where you capture everything raw, and an AI agent (OpenCode)
automatically turns your messy notes into structured wiki pages at night.

**Stack:**
- **Obsidian** — vault UI, graph view, Bases (database views)
- **OpenCode** — AI agent that processes notes, model: `opencode/minimax-m2.5-free`
- **Git + GitHub** — auto-backup every 10 minutes via Obsidian Git plugin
- **Kepano's Obsidian Skills** — teaches OpenCode proper Obsidian syntax
- **Graphify** — builds an interactive knowledge graph (optional, add in week 2)
- **brain command** — terminal-first capture from anywhere

---

## Folder Structure

```
~/brain/
├── 00-Inbox/         ← dump zone. everything goes here first
├── 01-Projects/      ← active projects with goals and deadlines
├── 02-Areas/         ← ongoing life responsibilities
├── 03-Resources/     ← reference material by topic
├── 04-Archive/       ← processed/dead notes (NEVER delete, only archive)
├── 05-Wiki/          ← MAIN KNOWLEDGE BASE. structured wiki pages
├── 06-Daily/         ← daily notes by date (YYYY-MM-DD)
├── 07-Agent-Logs/    ← what the AI did each session
├── Templates/        ← note blueprints
├── AGENTS.md         ← AI instruction file (do not delete)
└── .gitignore
```

---

## Fresh Install (from scratch)

### 1. Create folder structure

```bash
mkdir -p ~/brain/{00-Inbox,01-Projects,02-Areas,03-Resources,04-Archive,05-Wiki,06-Daily,07-Agent-Logs,Templates}
```

### 2. Create templates

```bash
cat > ~/brain/Templates/wiki-page.md << 'EOF'
---
created: {{date:YYYY-MM-DD}}
tags: 
status: active
related: 
---

# {{title}}

## What it is
Plain language explanation in your own words.

## Key ideas
- 

## My take
What you actually think. How you'd use this. Open questions.

## Sources
- 

## Related
- [[link-to-related-page]]
EOF
```

```bash
cat > ~/brain/Templates/daily-note.md << 'EOF'
---
date: {{date:YYYY-MM-DD}}
---

# {{date:YYYY-MM-DD}} — {{date:dddd}}

## Focus today
- 

## Captures
<!-- Dump EVERYTHING here throughout the day. No organisation. -->

## Lessons
<!-- Fill in at night -->

## Open loops
<!-- Things that need follow-up -->
EOF
```

```bash
cat > ~/brain/Templates/project.md << 'EOF'
---
created: {{date:YYYY-MM-DD}}
status: active
due: 
area: 
tags:
---

# {{title}}

## Goal
What does done look like?

## Tasks
- [ ] 

## Notes

## Related
- 
EOF
```

### 3. Create AGENTS.md (AI instruction file)

```bash
cat > ~/brain/AGENTS.md << 'EOF'
# Second Brain Vault — Agent Context

This is an Obsidian vault. Work with markdown files, wikilinks, and YAML frontmatter.

## Vault structure
- 00-Inbox/       = raw unprocessed notes. Process these.
- 01-Projects/    = active projects with goals and deadlines
- 02-Areas/       = ongoing life responsibilities
- 03-Resources/   = reference material by topic
- 04-Archive/     = processed/completed notes (never delete, only archive)
- 05-Wiki/        = MAIN KNOWLEDGE BASE. Structured wiki pages.
- 06-Daily/       = daily notes by date (YYYY-MM-DD format)
- 07-Agent-Logs/  = save your session outputs here
- 00-PDFs/       = dropped PDF files (read-only, do not edit)
- 00-PDF-Text/   = extracted text from PDFs. Process these like inbox notes.

## My method
Karpathy LLM wiki: capture raw → extract key ideas → update/create wiki pages → link them.

## How to process inbox notes
STEP 1: Read the raw note in 00-Inbox/
STEP 2: Extract exactly 2-3 key ideas from the note
STEP 3: Check if a matching wiki page exists in 05-Wiki/
  - If YES: open it and add the new ideas
  - If NO: create a new wiki page using the wiki template
STEP 4: Add [[wikilinks]] to at least 2 related pages in 05-Wiki/
STEP 5: Move the raw note from 00-Inbox/ to 04-Archive/
STEP 6: Write a log entry to 07-Agent-Logs/YYYY-MM-DD.md

## Wiki page format
Every wiki page MUST start with this exact frontmatter:
---
created: YYYY-MM-DD
tags: [relevant-tags]
status: active
related:
  - [[link1]]
  - [[link2]]
---

Then use this structure:
# Page Title

## What it is
Plain language explanation.

## Key ideas
- bullet points

## My take
Personal perspective, open questions.

## Sources
- references

## Related
- [[wikilinks]]

## CRITICAL RULES
- NEVER delete raw notes — only move them to 04-Archive/
- ALWAYS use YAML frontmatter (the --- block at the top)
- ALWAYS add at least 2 [[wikilinks]] to every wiki page
- ALWAYS log what you did to 07-Agent-Logs/
- Use today's date for all new pages

## Index Workflow
BEFORE answering any query: read index.md first to identify relevant pages.
Then drill into those pages. This avoids the need for embedding-based RAG at scale.
When creating new wiki pages, update index.md to include them.

## PDF Processing
- 00-PDFs/ = dropped PDF files (read-only, do not edit)
- 00-PDF-Text/ = extracted text from PDFs. Process these like inbox notes.
- For question papers: extract repeated patterns, add to CUSAT-Repeated-Questions
- For formula sheets: add formulas to subject-specific wiki pages
- For textbooks: extract key concepts and formulas only

## Wikilink Format (CRITICAL — DO NOT BREAK)
- related field in YAML MUST use list format:
  related:
    - [[Page Name]]
- NEVER use: related: [[Page1]], [[Page2]] (breaks graph)
- ALWAYS add "## Related" section in body with same links
- Graph connections only work when links exist in body text
EOF
```

### 4. Create OpenCode custom commands

```bash
mkdir -p ~/.config/opencode/commands

cat > ~/.config/opencode/commands/process-inbox.md << 'EOF'
# Process Inbox

Go through ALL notes in 00-Inbox/. For each note:
1. Read it completely
2. Extract exactly 2-3 key ideas
3. Check 05-Wiki/ for a matching page
4. If exists: update it. If not: create one with proper YAML frontmatter
5. Add at least 2 [[wikilinks]] to related pages
6. Move the raw note to 04-Archive/
7. Write summary to 07-Agent-Logs/YYYY-MM-DD.md
Report: notes processed, wiki pages created vs updated.
EOF

cat > ~/.config/opencode/commands/weekly-review.md << 'EOF'
# Weekly Vault Review

1. List all wiki pages in 05-Wiki/ sorted by most recently modified
2. Check 00-Inbox/ for notes older than 3 days
3. Find topics across multiple notes with no wiki page yet
4. Suggest 3 wiki pages to create or expand this week
5. Find 5 pages in 05-Wiki/ that should be linked but aren't
6. Write the full review to 07-Agent-Logs/weekly-YYYY-MM-DD.md
EOF

cat > ~/.config/opencode/commands/nightly-review.md << 'EOF'
# Nightly Review

1. Read today's daily note in 06-Daily/
2. Check 07-Agent-Logs/ for today's session
3. Archive anything in 00-Inbox/ older than today
4. Update project statuses in 01-Projects/ if needed
5. Write a 3-line summary to today's daily note under "Lessons"
6. Write a brief log to 07-Agent-Logs/nightly-YYYY-MM-DD.md
EOF
```

### 5. Configure OpenCode for MiniMax M2.5 free

```bash
cat > ~/.config/opencode/opencode.json << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/minimax-m2.5-free",
  "agent": {
    "build": {
      "temperature": 1.0,
      "top_p": 0.95,
      "top_k": 40
    },
    "plan": {
      "temperature": 1.0,
      "top_p": 0.95,
      "top_k": 40
    }
  }
}
EOF
```

> These are MiniMax's officially recommended inference params. Without them the model loops forever.

### 6. Install Kepano's Obsidian Skills

Teaches OpenCode proper Obsidian syntax (wikilinks, Bases, Canvas, frontmatter).

```bash
git clone https://github.com/kepano/obsidian-skills.git ~/.opencode/skills/obsidian-skills
```

### 7. Initialize Git

```bash
cd ~/brain
git config --global user.name "YourName"
git config --global user.email "you@example.com"
git init
git add .
git commit -m "vault: initial setup"
```

### 8. Install Obsidian

```bash
sudo pacman -S obsidian
# or
paru -S obsidian
# or
flatpak install flathub md.obsidian.Obsidian
```

### 9. Open vault in Obsidian

1. Launch Obsidian
2. Click **Open folder as vault** → select `~/brain`

### 10. Configure Obsidian settings

**Settings → Files & Links:**
- Default location for new notes → `In the folder specified below` → `00-Inbox`

**Settings → Core plugins — enable all of these:**
- Templates
- Bases
- Daily Notes
- Backlinks
- Properties
- Graph view

**Settings → Templates:**
- Template folder location → `Templates`

**Settings → Daily Notes:**
- New file location → `06-Daily`
- Template file location → `Templates/daily-note`

### 11. Install community plugins

Settings → Community plugins → Turn off Restricted Mode → Browse

Install and enable each:

| Plugin | Author | What it does |
|--------|--------|-------------|
| Git | Vinzent (Denis Olehov) | Auto-commit + push every 10 min |
| Calendar | Tony Grosinger | Click dates to open daily notes |
| Templater | SilentVoid | Smarter template insertion |
| Omnisearch | Simon Cambier | Full-text search across vault |

**Configure Git plugin** (Settings → Git):
- Auto commit-and-sync interval: `10`
- Auto commit-and-sync after stopping file edits: `ON`
- Commit message: `vault: {{date}}`

### 12. Install OpenCode

```bash
curl -fsSL https://opencode.ai/install | bash
opencode --version
```

### 13. Connect vault to GitHub (backup)

1. Create a **private** repo on github.com (no README, no .gitignore)
2. Then:

```bash
cd ~/brain
git remote add origin git@github.com:YOURUSERNAME/brain.git
git branch -M main
git push -u origin main
```

Done. Obsidian Git auto-pushes every 10 minutes from now.

### 14. Install brain command (terminal-first capture)

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/brain << 'EOF'
#!/bin/bash

# Quick capture to your second brain
# Usage: brain "note content"     → dump to inbox
#        brain --process "note"  → capture + process immediately

VAULT="$HOME/brain"
INBOX="$VAULT/00-Inbox"

if [ ! -d "$VAULT" ]; then
  echo "Error: ~/brain vault not found"
  exit 1
fi

if [ "$1" = "--process" ]; then
  shift
  CONTENT="$*"
  TIMESTAMP=$(date +%Y%m%d-%H%M%S)
  FILE="$INBOX/capture-$TIMESTAMP.md"
  echo -e "---\ndate: $(date +%Y-%m-%d)\ncaptured: terminal\n---\n\n$CONTENT" > "$FILE"
  echo "Captured: $FILE"
  cd "$VAULT" && opencode run "Process all notes in 00-Inbox/ using my wiki method. Log everything."
else
  CONTENT="$*"
  TIMESTAMP=$(date +%Y%m%d-%H%M%S)
  FILE="$INBOX/capture-$TIMESTAMP.md"
  echo -e "---\ndate: $(date +%Y-%m-%d)\ncaptured: terminal\n---\n\n$CONTENT" > "$FILE"
  echo "Captured: $FILE"
fi
EOF

chmod +x ~/.local/bin/brain
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Done! Now you can capture from anywhere.

---

## Creating Bases (Database Views)

Bases are Obsidian's built-in database views. Create base files with correct syntax:

### Inbox Tracker

```bash
cat > ~/brain/00-Inbox/inbox-tracker.base << 'EOF'
---
filters:
  and:
    - property: file.path
      condition: contains
      value: 00-Inbox/
properties:
  - file.ctime
  - tags
sort:
  - property: file.ctime
    direction: asc
views:
  - name: Oldest First
    type: table
EOF
```

### Wiki Browser

```bash
cat > ~/brain/05-Wiki/wiki-browser.base << 'EOF'
---
filters:
  and:
    - property: file.path
      condition: contains
      value: 05-Wiki/
properties:
  - tags
  - file.mtime
  - related
  - status
views:
  - name: All Wiki
    type: card
  - name: Orphans
    type: card
    filters:
      and:
        - property: file.path
          condition: contains
          value: 05-Wiki/
        - property: related
          condition: is empty
EOF
```

### Project Tracker

```bash
cat > ~/brain/01-Projects/projects.base << 'EOF'
---
filters:
  and:
    - property: file.path
      condition: contains
      value: 01-Projects/
properties:
  - status
  - due
  - area
views:
  - name: Active Projects
    type: card
EOF
```

Restart Obsidian after creating base files for them to load correctly.

---

## Daily Workflow (Terminal-First)

### All day — just type

No need to open Obsidian. Capture from anywhere:

```bash
brain "Meeting with John about the project deadline"
brain "Interesting article about AI workflows"
brain "Need to remember: buy milk, call mom"
```

That's it. Notes go straight to inbox.

### Evening (5 min)

```bash
cd ~/brain
opencode run "Process all notes in 00-Inbox/ using my wiki method. Log everything."
```

Or use the shortcut that does both:

```bash
brain --process "One more thought I had today"
```

### Sunday (20 min)

```bash
cd ~/brain
opencode run "Run weekly vault review. Check orphaned wiki pages, old inbox notes, missing links. Save to 07-Agent-Logs/."
```

---

## OpenCode Commands Reference

```bash
# Process inbox (main command)
cd ~/brain && opencode run "Process all notes in 00-Inbox/ using my wiki method. Log everything."

# Plan mode (safe — see what it WOULD do without touching files)
cd ~/brain && opencode run "PLAN ONLY: Tell me step by step how you would process 00-Inbox/. Do NOT create, edit, move, or delete any files."

# Research what you know
cd ~/brain && opencode run "What do I know about [TOPIC]? Search the whole vault."

# Find missing links
cd ~/brain && opencode run "Find 5 wiki pages in 05-Wiki/ that should be linked but aren't."

# Weekly review
cd ~/brain && opencode run "Run weekly vault review. Save to 07-Agent-Logs/."
```

**Inside the OpenCode TUI** (`opencode` → interactive):
```
user:process-inbox    → process all inbox notes
user:weekly-review    → weekly review
user:nightly-review   → nightly tidy
```

---

## Terminal-First Capture (brain command)

```bash
# Quick capture to inbox (instant)
brain "whatever is on your mind"

# Capture + process immediately (AI turns it into wiki)
brain --process "meeting notes about the new feature"

# Works from anywhere — terminal, anywhere in filesystem
```

The brain command:
- Creates timestamped notes in `00-Inbox/`
- Auto-adds frontmatter with date
- Optional `--process` flag triggers OpenCode to process immediately

---

## Graphify (Add in Week 2)

Builds an interactive HTML graph of your entire vault. Install once you have 10+ wiki pages.

```bash
# Install
pip install graphifyy

# If command not found:
export PATH="$HOME/.local/bin:$PATH"

# Wire to OpenCode
graphify install --platform opencode

# Build your vault graph
cd ~/brain && /graphify . --obsidian
# Opens at: ~/brain/graphify-out/graph.html

# Incremental update (fast)
/graphify . --update

# Query the graph
graphify query "what connects habits to self-improvement?" --graph graphify-out/graph.json
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `command not found: opencode` | Close terminal, open new one |
| OpenCode loops forever | Press `Ctrl+C`, run again. Free model is flaky. |
| `ProviderModelNotFoundError` | Check `~/.config/opencode/opencode.json` — model must be `opencode/minimax-m2.5-free` |
| Bases show wrong files | Wrong syntax. Filters need `and:` key (see Bases section above) |
| Template shows `{{date:YYYY-MM-DD}}` raw | Normal in template file itself. Only expands when inserted into a real note. |
| Daily note duplicated | Clicked calendar button multiple times. Delete duplicate content manually. |
| Git push rejected | Someone pushed from another device. Run `git pull` first, then `git push`. |
| Obsidian Git won't push | Settings → Git → Auto push after commit = ON |
| `brain: command not found` | Run `source ~/.bashrc` or restart terminal |

---

## List all available models

```bash
opencode models
```

Current free options:
```
opencode/minimax-m2.5-free   ← what we use
opencode/ling-2.6-flash-free
opencode/hy3-preview-free
opencode/nemotron-3-super-free
```

---

## File Locations

```
~/brain/                          ← vault
~/brain/AGENTS.md                 ← AI instructions
~/brain/Templates/                ← note blueprints
~/.config/opencode/opencode.json  ← OpenCode config (model, params)
~/.config/opencode/commands/      ← custom slash commands
~/.opencode/skills/obsidian-skills/ ← Kepano's Obsidian skills
~/.local/bin/brain                ← terminal capture script
```

---

## Resources

- [OpenCode docs](https://opencode.ai)
- [Kepano's Obsidian Skills](https://github.com/kepano/obsidian-skills)
- [Obsidian Git plugin](https://github.com/Vinzent03/obsidian-git)
- [Karpathy's LLM wiki method](https://x.com/karpathy) — the philosophy behind this setup
- [The Power of Habit — Charles Duhigg](https://charlesduhigg.com/the-power-of-habit/) — first test note topic
- [GitHub](https://github.com/Baseplayer23893/lazy-second-brain-guide) — this guide