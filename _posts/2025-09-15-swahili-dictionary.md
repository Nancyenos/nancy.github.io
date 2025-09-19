---
layout: post
title: "Adding Swahili to GNOME Crosswords"
date: 2025-09-15
---

**TL;DR:** With the guidance of my mentor, Jonathan, I gathered a list of Swahili words from Hunspell. I then used our tools to add clear definitions from Wiktionary only for the words on that list. I also created a new workflow that relies on configuration files for each list or language. I separated paths into command line flags to avoid hard-wiring the tool to specific project folders. Finally, we ensured the output remains consistent with the current Wordnik-style importer. Now, anyone can add a new language or word list by creating a simple config file and running a few commands.

**Why Swahili, and why now?**

Swahili is widely spoken in East Africa yet often underrepresented in software. Crosswords are a fun way to learn vocabulary and celebrate language: so adding Swahili to GNOME Crosswords felt both useful and exciting.

![alt text](/assets/images/image.png)

**Data sources at a glance**

- Word list: Hunspell Swahili dictionary (list for what words to include).

- Definitions: Wiktionary (via our raw wiktextract JSONL dump).

- Format goal: Output that stays compatible with the existing Broda‑style importer & GNOME word‑list tooling.

**The tooling**

- **def-extractor.py** (in tools/wiktionary-extractor/): filters Wiktionary entries and produces a clean, structured list with parts of speech, tags, and definitions.

- **Meson/Ninja integration:** convenient targets like build-wordlist-defs for repeatable builds.
![alt text](/assets/images/image-1.png)

Initially, the extractor logic assumed certain directory layouts and enums. To support Swahili and future languages cleanly, Jonathan and I focused on making it configurable and separating paths from the code.

What we changed and why
1) **Config-first workflow (new .conf files)**

We introduced per-language config files so each word list can declare:

Input sources (Hunspell list, Wiktionary dump path)

Filters (minimum length, allowed parts of speech, tags, etc.)

Output targets (where the filtered .jsonl/artifacts go)

This makes adding a new language a configuration task instead of a code change.

Here’s the actual config we wrote for Swahili:
```bash
[Word List]
DisplayName=Swahili
Identifier=swahili
Source=sw_KE.dic
Importer=wordnik
Locale=sw
Definitions=True
Visibility=editor
MinLength=2
MaxLength=21
Threshold=50
Alphabet=ABCDEFGHIJKLMNOPQRSTUVWXYZ
```
2) **Decoupled paths via CLI flags**

The extractor no longer depends on hard-coded project paths. Important information can be passed from the command line. This change makes Meson rules and one-off local runs much easier.

Example (paths are illustrative):
```bash
python3 tools/wiktionary-extractor/def-extractor.py \
        --config word-lists/swahili/swahili.conf \
        --dictionary raw-wiktextract-data.jsonl \
        --word-lists-dir wordlist-tools/swahili \
        --output-dir swahili \
```
![alt text](/assets/images/image-2.png)

**Swahili: end‑to‑end workflow**

Here’s the exact flow we used for Swahili:
![alt text](/assets/images/image-3.png)

- Drop in the Hunspell list \ Place the Swahili .dic  under word-lists/swahili/.

- Point the config at your inputs \ Create configs/swahili.conf with the inputs, filters, and outputs you want.
 -
 
**Build & test**
edit meson.buid to add in the sawhili wordlist
```bash
meson setup _build
ninja -C _build
meson compile build-wordlist-index
meson compile -C _build build-wordlist-defs
meson compile build-wordlist-gresource-xml
meson compile build-wordlist-gresource
```
This builds the definitions and wordlist into one gresource file that can be loaded in to crosswords.
![alt text](/assets/images/image-4.png)

Load the resulting list in GNOME Crosswords and check a few entries.
```bash
crosswords/_build
```

![alt text](/assets/images/myphoto.png)
