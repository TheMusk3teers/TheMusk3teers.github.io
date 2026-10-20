---
title: File Dump 01
tags:
  - DEADFACE CTF 2026
  - Steganography
  - Forensics
---

# File Dump 01

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | Steganography |
| **Points** | 10 |
| **Author** | @RP-01? |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> We have intercepted a DEADFACE hacker's laptop and determined he had stolen a man named Daniel's entire identity. For this first challenge, let's identify which DEADFACE member is responsible!
>
> Submit the answer as `deadface{deadface alias}`

Files given: `File_Dump_Challenge.zip` (26 files: Office documents, PDFs, and images)

## Solution

The zip is everything someone collected while stealing **Daniel Hameser**'s identity: `resume.docx`, `invoice.xlsx`, `notes.docx`, `reminders.docx`, receipt photos and business cards.

Office files are zips too, and `docProps/core.xml` inside them keeps the author. We dumped it for all of them:

```python
import zipfile, xml.etree.ElementTree as ET, glob

for path in sorted(glob.glob("File Dump Challenge/*.docx") + glob.glob("File Dump Challenge/*.xlsx")):
    with zipfile.ZipFile(path) as z:
        core = ET.fromstring(z.read("docProps/core.xml"))
        ns = {"dc": "http://purl.org/dc/elements/1.1/", "cp": "http://schemas.openxmlformats.org/package/2006/metadata/core-properties"}
        creator = core.findtext("dc:creator", namespaces=ns)
        desc = core.findtext("dc:description", namespaces=ns)
        print(f"{path}: creator={creator!r}, description={desc!r}")
```

- `notes.docx`, `reminders.docx`, `resume.docx` and `invoice.xlsx` all have the author `ghostwriter404`.
- `notes.docx` was saved on a machine called `GHOST404-PC`, and it's a set of notes on keeping the fake Daniel accounts consistent across sites.

`ghostwriter404` and `GHOST404-PC` aren't a DEADFACE alias on their own, but the DEADFACE member who goes by `gh0st404` on GhostTown is an obvious match.

!!! tip "Lead for File Dump 03"
    The `dc:description` in `reminders.docx` also has `backup passphrase: n0rth_st4r`. You need it in [File Dump 03](file-dump-03.md).

## Flag

!!! flag "Flag"
    `deadface{gh0st404}`

## Takeaways

Check `docProps/core.xml` on every Office file (`dc:creator`, `cp:lastModifiedBy`, `dc:description`). If the prompt asks for an alias, give the name they use on the forum, not the variant in the metadata.
