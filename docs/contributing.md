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

Set the tags at the top of the file: the CTF name and the challenge category. They feed the [Tags](tags.md) page.

```yaml
tags:
  - DEADFACE CTF 2026
  - OSINT
```

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

## 4. Post news (optional)

Team news goes in `docs/blog/posts/`, one file per post, named `YYYY-MM-DD-short-title.md`. Start it with a date, an author from `docs/blog/.authors.yml` and a category (Announcements, Campaigns or Hackathons), and put `<!-- more -->` after the first paragraph. New posts show up on the News page and in the RSS feed.

## 5. Push

Commit to `main`. The site rebuilds itself in about a minute.

!!! warning "Wait for the CTF to end"
    Never publish a writeup while a competition is still running. Most CTFs ban it, and it can get the team disqualified.
