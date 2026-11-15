# SC-500 Study System

Offline study system for **Microsoft SC-500: Implementing End-to-End Security Controls for Cloud and
AI Workloads** → cert *Microsoft Certified: Cloud and AI Security Engineer Associate*.

Written for someone with a **SOC-analyst background** who is moving into security engineering. Security
operations ground (Sentinel collection, Security Copilot) gets short *quick review* pages. Azure
infrastructure, identity engineering and AI security are taught from scratch. Every lesson opens with
the Azure building block it protects ("Azure first") and links a short **Foundations** primer.

The site builds to static HTML that opens from `file://` with no network. Pages use a plain,
point-first voice. Every factual claim is cited to a Microsoft Learn URL, and every Learn screenshot is
included.

## What's here

- `build.py` — the stdlib-only build engine: parse TOML + markdown → render → lint → emit `site/`.
- `content/domains.toml` — the exam blueprint: 4 domains, 12 subdomains, 87 objective bullets
  (captured 2026-11-15 from the live study guide).
- `content/learn-map.toml` — the official curriculum (course SC-500T00: 12 learning paths, 62 modules,
  412 units) mapped to blueprint IDs.
- `content/study-path.toml` — the suggested 6-week path shown at the bottom of the dashboard.
- `content/foundations/` — Azure, identity and AI primers (30), read before the lessons that link them.
- `content/{objectives,cheatsheets,diagrams,reference,questions,labs,snippets,notes}/` — the study material.
- `assets/` — CSS/JS. `progress/` — local, git-ignored (attempt log, notes, study plan).

## Status

- ✅ Blueprint and official curriculum captured.
- ✅ Foundations track and all 87 objective pages.
- ✅ Cheatsheets and diagram pages for D1–D4.
- ⬜ Practice questions, labs and mock exams. These are generated later.

## Page shape

Every objective page follows the same order:

- **Azure first**
- In one sentence
- Why it exists
- Mental model
- What actually matters
- Where it lives
- Exam traps
- Skip

Quick-review pages use a shorter shape with a "What's new for an engineer" section. Each page gets an
automatic *On this page* contents list and a *Jump to Exam traps* link.

Pages are written in a markdown subset only: headings, lists, tables, fenced code (including `svg`),
images, and `==highlight==` spans with sigils (`@` term, `!` trap, `+` correct). No raw HTML.

## Build

```bash
python3 build.py              # emits site/
python3 build.py --lint       # schema, refs, foundations, word budget, SVG overflow, duplicate stems
python3 build.py --report     # coverage / progress stats as JSON
python3 build.py --check-learn-map   # curriculum coverage (no network)
python3 build.py --fetch-images      # cache remote images for offline use (network)
python3 build.py --check-links       # probe cited URLs for rot (network)
python3 serve.py              # optional: localhost server so page notes save to progress/notes.json
```

Open `site/index.html` from `file://`. `site/` is disposable: delete it and rebuild.

## How to study

1. `python3 build.py && open site/index.html`. The dashboard shows readiness, weak areas and the 6-week path.
2. Read the **Foundations** primers each week links, before the lessons that use them.
3. Work domain by domain. Read each domain's cheatsheet and diagrams page at the end of it.
4. Do hands-on work in your own Azure subscription. Use a dedicated resource group, the smallest SKUs,
   and delete the group when done.
5. Export progress weekly with the footer button. `localStorage` on `file://` is not durable.
