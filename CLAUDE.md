# podpub — instructions for Claude

This is a lightweight CLI (`podpub.py`) that publishes NotebookLM audio episodes to a private podcast RSS feed hosted on GitHub Pages. The feed URL is `https://synthyclaw.github.io/podpub/feed.xml` and is served via the `itunes:block=yes` directive (hidden from Apple's public directory, but subscribable by URL).

## Publishing workflow

When the user asks to publish, inspect `inbox/` and follow this procedure:

1. **Identify pairs.** For each audio file (`.m4a` / `.mp3` / `.wav`) not already prefixed with `NNN - `, check whether there is a companion file with the same basename:
   - `.md` sidecar present → use it as-is. Skip to step 3.
   - `.pdf` present but no `.md` → generate the `.md` from the PDF (step 2).
   - Neither → the script will auto-generate a generic `"Episode N of …"` description. Ask the user if that's acceptable before running, or if they want to provide a description.

2. **Generate `.md` from PDF.** Read the PDF with the `Read` tool, then write a file at `inbox/<same basename as audio>.md` following the format in the next section. Do not proceed to publish until the `.md` exists.

3. **Preview, then publish.**
   - First run: `.venv/bin/python podpub.py --dry-run` — sanity-check the rename plan, feed XML, and commit message.
   - If everything looks right, run: `.venv/bin/python podpub.py` — this moves files into `audio/`, rebuilds `feed.xml`, commits, and pushes to `origin/main`. GitHub Pages auto-deploys within ~30 seconds.

4. **Leave the PDF alone.** After publish, the PDF stays in `inbox/` (already gitignored). The user can clean it up whenever; the script doesn't depend on it and it won't be re-processed (only audio extensions are scanned).

## Standardized episode description format

The `.md` sidecar becomes the `<description>` in the RSS feed. It is visible in Apple Podcasts and every other podcast app. Use this exact structure:

```
<Paper / book / article title> (<Month Year of publication>)

In this episode we unpack <1–2 sentences: authors, institution if notable, page count if notable, and the central argument or thesis>.

We walk through <specific technical contributions: models, frameworks, methods, benchmarks — cite them by their actual names>, <concrete case studies, datasets, or findings>, and close on <broader implications, open questions, or ethical tensions>.

Reference: <Full APA-style citation>. <DOI or arXiv URL>

Google Scholar citations: <number>
```

### Rules (non-negotiable)

- **Tone**: first-person plural ("we"), present tense. This matches NotebookLM's two-host deep-dive style.
- **Length**: exactly two paragraphs of prose + a Reference line + (optionally) a Google Scholar citations line. No headings, no bullets, no code blocks inside the description.
- **Opening paragraph**: name the authors (or lead author + "et al." for long lists), state where it was published if notable (arXiv, journal, conference), and give the core claim in plain language. Avoid jargon in the thesis sentence.
- **Middle paragraph**: name technical contributions by their real names — model names, benchmark names, framework acronyms. Do not substitute generic phrasing like "various models" where the paper actually names V-JEPA 2-AC, IntPhys 2, etc.
- **Reference line**: APA-ish — "LastName, F., Second, L., Third, L., et al. (Year). Title. Venue or arXiv:ID. URL". For arXiv papers, include both the ID and the full URL.
- **Google Scholar citations**: include this line only if the user explicitly provides the number or if it's easily retrievable. Never guess. If unknown, omit the line entirely.

### Extracting from a PDF

The `Read` tool handles PDFs directly. For long PDFs (>10 pages), pass a `pages` parameter (e.g., `pages: "1-8"`) — the abstract, introduction, and contributions section are usually all that's needed. Pull:

- **Title** → opens the description (with Month/Year in parentheses).
- **Author list + affiliations** → compresses into the opening sentence.
- **Abstract** → primary source for the opening paragraph's thesis.
- **Introduction / Contributions section** → for the middle paragraph's named artifacts.
- **DOI / arXiv ID** → from the front matter for the Reference line.
- **Publication month / year** → from the front matter or the arXiv ID (e.g., `arXiv:2506.22355` → June 2025).

Reference example — a real one from the first episode:
```
Embodied AI Agents: Modeling the World (June 2025)

In this episode we unpack a 40 page position paper from Meta AI Research, led by Pascale Fung with Jitendra Malik and 19 co-authors. The argument: the next generation of AI agents will not live in chat windows. They will be embodied as virtual avatars, as wearables like Meta's AI Glasses, and as robots, and none of them will be useful without a proper world model.

We walk through the three agent types, the case for joint embedding predictive architectures like V-JEPA 2-AC and Vision-Language World Models over pure generative approaches, and the distinction between physical world models (perception, motion, planning) and mental world models (the Theory of Mind layer needed for real collaboration). We also cover the four new benchmarks the paper introduces (MVP, IntPhys 2, CausalVQA, WorldPrediction), and close on lifelong embodied learning, multi agent collaboration, and the ethical tensions around privacy and anthropomorphism.

Reference: Fung, P., Bachrach, Y., Celikyilmaz, A., et al. (2025). Embodied AI Agents: Modeling the World. arXiv:2506.22355. https://arxiv.org/abs/2506.22355

Google Scholar citations: 66
```

## Commands reference

- `.venv/bin/python podpub.py` — publish new inbox items (moves files, rebuilds feed, commits, pushes).
- `.venv/bin/python podpub.py --dry-run` — preview without writing or pushing.
- `.venv/bin/python podpub.py --no-push` — commit locally but skip `git push`.
- `.venv/bin/python podpub.py --rebuild-feed` — re-emit `feed.xml` from existing items without processing the inbox. Use after editing `config.yaml` (show title, description, cover URL) so the feed picks up channel-level changes.

## Layout notes

- **Root (served by GitHub Pages)**: `feed.xml`, `audio/`, `NotebookLM-PodPub-Cover.png`. Don't move these — their URLs are baked into `feed.xml`.
- **`setup/`**: `requirements.txt`, `config.yaml.example`. Setup-only, not touched day-to-day.
- **`inbox/`**: user's drop zone. Contents gitignored (including PDFs).
- **`config.yaml`** (gitignored, at root): paths + podcast metadata. Read by `podpub.py` on every run.
