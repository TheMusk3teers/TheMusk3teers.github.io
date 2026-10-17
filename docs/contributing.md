---
title: The Codex
---

# The Codex

How every musketeer adds a writeup.

## 1. Create the file

Copy `templates/writeup.md` from the repo root into the CTF's folder:

```
docs/ctfs/deadface-2026/osint-ghost-in-the-forum.md
```

Name it `category-challenge-name.md`, all lowercase, words joined by hyphens.

## 2. Fill it in

Keep it short and reproducible: what you were given, what you tried, what worked, and the flag. Put scripts in code blocks and screenshots in the same folder.

Wrap the flag in the flag box so readers can find it fast:

```markdown
!!! flag "Flag"
    `deadface{example_flag}`
```

## 3. Link it

Add a line under the right category tab in the CTF's `index.md`:

```markdown
=== "OSINT"
    - [Ghost in the Forum](osint-ghost-in-the-forum.md) · 150 pts · Aramis
```

## 4. Push

Commit to `main`. The site rebuilds itself in about a minute.

!!! warning "Wait for the CTF to end"
    Never publish a writeup while a competition is still running. Most CTFs ban it, and it can get the team disqualified.
