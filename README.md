# podpub

A tiny CLI that publishes local audio files (e.g., NotebookLM downloads) to a private podcast RSS feed hosted on GitHub Pages, so you can subscribe in Apple Podcasts or any podcast app.

Drop audio into an inbox folder. Run `python podpub.py`. Episodes get renamed, moved into your repo, appended to `feed.xml`, committed, and pushed. Done.

---

## One-time setup

### 1. GitHub repo + Pages
1. Create a GitHub repo (e.g., `yourname/podpub`) and clone it locally — this directory is the clone.
2. In the repo's GitHub **Settings -> Pages**, set **Source** to `Deploy from a branch`, pick **main** / **`/ (root)`**, and save. After the first push, your feed URL will be:
   ```
   https://YOUR_GITHUB_USER.github.io/podpub/feed.xml
   ```
3. Make sure `git` can push without prompting — either an SSH key or an HTTPS credential helper / personal access token.

### 2. Cover image
Place a square podcast cover image (JPG or PNG, 1400x1400 or larger recommended) at the repo root. This repo already ships with `NotebookLM-PodPub-Cover.png`. Replace it if you want your own.

### 3. Python deps
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 4. First run creates `config.yaml`
```bash
python podpub.py
```
It prompts you for:

- `inbox_dir` — absolute path to the folder where new audio lands (e.g., `/Users/you/Downloads/NotebookLM`)
- `repo_dir` — absolute path to this repo (default: script's own folder)
- `audio_subdir` — default `audio`
- `feed_path` — default `feed.xml`
- `base_url` — e.g., `https://YOUR_GITHUB_USER.github.io/podpub`
- podcast metadata: title, description, author name, author email, language, category
- `cover_image_url` — auto-suggested as `{base_url}/NotebookLM-PodPub-Cover.png`

The file is saved and gitignored — see `config.yaml.example`.

---

## Everyday use

```bash
# Drop audio files into inbox_dir, then:
python podpub.py
```

### Per-episode descriptions (optional)
Alongside an audio file in the inbox, drop a Markdown file with the **same basename** and `.md` extension. Its contents become that episode's `<description>`. Example:

```
inbox/
  My Cool Chat.m4a
  My Cool Chat.md
```

If there's no sidecar, podpub auto-generates `"Episode N of {podcast title}"`.

### Flags
- `--dry-run` - preview the rename plan, feed XML, and commit message without moving files, writing the feed, or pushing.
- `--no-push` - commit locally but skip `git push`.

### What podpub does
1. Scans `inbox_dir` for `.m4a`, `.mp3`, `.wav` files not already prefixed `NNN - `.
2. Parses `feed.xml` to find the highest existing episode number.
3. Sorts new files by mtime (oldest first) and numbers them sequentially.
4. Renames `some_file.m4a` -> `007 - Some File.m4a`, moves into `audio/` (sidecar `.md` moves too).
5. Rebuilds `feed.xml` with all old items + new ones, sorted newest-first.
6. Commits and pushes.

### Subscribing in Apple Podcasts
On iPhone: **Library** tab -> **...** (top right) -> **Follow a Show by URL** -> paste the feed URL:
```
https://YOUR_GITHUB_USER.github.io/podpub/feed.xml
```

The feed sets `<itunes:block>Yes</itunes:block>` so it won't show up in Apple's public directory - only people with the URL can follow it.

Apple Podcasts may take up to an hour to refresh after you push new episodes. To force a refresh: open **Library**, pull down, and release.

---

## Files

```
podpub.py              # the script
config.yaml            # (gitignored) your local config, created on first run
config.yaml.example    # reference template
requirements.txt       # feedgen, PyYAML
feed.xml               # RSS feed (committed, served by Pages)
audio/                 # published audio files (committed, served by Pages)
inbox/                 # local test-drop folder (contents gitignored)
podpub.log             # (gitignored) run log
.gitignore
```

## Testing

```bash
# Put a test file in inbox/ (or your real inbox_dir), then:
python podpub.py --dry-run
```
This prints the full rename plan, the RSS XML that would be written, and the commit message - no files are moved and nothing is pushed.
