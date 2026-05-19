# courseskills

Course creation skills for **Codex** and **Claude Code**. This repository packages the same workflows as both clients' plugin formats and as standalone Claude Code skills:

| Skill | What it does |
|---|---|
| **ai-tutorials** | Design AI course syllabi, lectures, and hands-on projects |
| **codex2course** | Topic / outline -> handout -> Imagegen slide images -> PDF |
| **api2course** | Topic / outline -> handout -> OpenAI API slide images -> PDF |
| **pdf2video** | Slide deck -> per-slide narration -> TTS audio -> mp4 |
| **movecourse** | Copy generated lesson mp4 files into the aistudy101 website `course-assets` tree |

The repository is intentionally laid out around one shared `skills/` directory. Claude Code can install those skills directly, Claude Code can also install this repo as a plugin through `.claude-plugin/marketplace.json`, and Codex can install it from the Codex marketplace manifest at `.agents/plugins/marketplace.json`, which points back to the plugin manifest at `.codex-plugin/plugin.json`.

---

## Install

### Claude Code skills

This is the simplest install path. Requires [Node.js](https://nodejs.org/). Restart Claude Code after installing.

```bash
npx skills add likefallwind/courseskills
```

To install only one skill:

```bash
npx skills add likefallwind/courseskills --skill ai-tutorials
npx skills add likefallwind/courseskills --skill codex2course
npx skills add likefallwind/courseskills --skill api2course
npx skills add likefallwind/courseskills --skill pdf2video
npx skills add likefallwind/courseskills --skill movecourse
```

<details>
<summary>Manual skills install without Node.js</summary>

```bash
mkdir -p ~/.claude/skills

curl -fsSL https://raw.githubusercontent.com/likefallwind/courseskills/main/skills/ai-tutorials/SKILL.md \
  -o ~/.claude/skills/ai-tutorials.md

curl -fsSL https://raw.githubusercontent.com/likefallwind/courseskills/main/skills/codex2course/SKILL.md \
  -o ~/.claude/skills/codex2course.md

curl -fsSL https://raw.githubusercontent.com/likefallwind/courseskills/main/skills/api2course/SKILL.md \
  -o ~/.claude/skills/api2course.md

curl -fsSL https://raw.githubusercontent.com/likefallwind/courseskills/main/skills/pdf2video/SKILL.md \
  -o ~/.claude/skills/pdf2video.md

curl -fsSL https://raw.githubusercontent.com/likefallwind/courseskills/main/skills/movecourse/SKILL.md \
  -o ~/.claude/skills/movecourse.md
```

</details>

### Claude Code plugin

Use this if you want plugin-style namespacing and marketplace updates:

```text
/plugin marketplace add likefallwind/courseskills
/plugin install courseskills@courseskills
```

This works because the repo includes `.claude-plugin/marketplace.json`, which points to the plugin in this repo. Installed plugin skills are namespaced:

```text
/courseskills:ai-tutorials
/courseskills:codex2course
/courseskills:api2course
/courseskills:pdf2video
/courseskills:movecourse
```

For local plugin development or testing, point Claude Code directly at this checkout:

```bash
claude --plugin-dir /path/to/courseskills
```

### Codex plugin

Codex installs this repository through its marketplace system. The marketplace entry lives at `.agents/plugins/marketplace.json`; the plugin itself lives at `.codex-plugin/plugin.json` and loads the shared `skills/` directory.

Use the install path that matches the Codex environment you actually run. Windows Codex App / Windows CLI and WSL Codex CLI use different home directories and do not automatically share marketplace state.

#### Codex App on Windows

Open the Codex App plugin page, choose **Add marketplace**, and enter:

```text
Source: https://github.com/likefallwind/courseskills.git
Git ref: main
Sparse paths: .agents/plugins
```

After the marketplace appears, select the `courseskills` marketplace source, search for `courseskills`, and install the plugin.

Do not use `.codex-plugin/` as the marketplace sparse path. `.codex-plugin/plugin.json` is the plugin manifest, while **Add marketplace** discovers marketplace manifests from `.agents/plugins/marketplace.json`.

If the App keeps showing `Failed to add marketplace` after a previous failed attempt, remove the stale marketplace from the **Windows** Codex CLI and then add it again:

```powershell
codex.cmd plugin marketplace remove courseskills
codex.cmd plugin marketplace add "https://github.com/likefallwind/courseskills.git" --ref main --sparse ".agents/plugins"
```

Restart the Codex App after a successful CLI add.

#### Codex CLI on Windows PowerShell

Install the Windows Codex CLI if needed:

```powershell
npm.cmd install -g @openai/codex
```

Use `npm.cmd` / `codex.cmd` if PowerShell blocks `npm.ps1` or `codex.ps1` with an execution-policy error.

Then add and install the plugin:

```powershell
codex.cmd plugin marketplace remove courseskills
codex.cmd plugin marketplace add "https://github.com/likefallwind/courseskills.git" --ref main --sparse ".agents/plugins"
codex.cmd plugin list --source courseskills
codex.cmd plugin install courseskills --source courseskills
```

This configures the Windows Codex home, normally under `C:\Users\<you>\.codex`, and is the right place to fix Windows Codex App marketplace state.

#### Codex CLI on WSL, Linux, or macOS

Use this when you run Codex from WSL/Linux/macOS shells:

```bash
codex plugin marketplace remove courseskills || true
codex plugin marketplace add \
  'https://github.com/likefallwind/courseskills.git' \
  --ref main \
  --sparse .agents/plugins

codex plugin list --source courseskills
codex plugin install courseskills --source courseskills
```

On WSL, this configures the Linux home directory, usually `~/.codex` inside WSL. It will not fix or update the Windows Codex App unless that App is explicitly running against the same WSL-side configuration.

#### Codex plugin layout

The plugin root loaded by Codex contains:

```text
courseskills/
├── .codex-plugin/plugin.json
└── skills/
```

If you prefer a purely local/private marketplace, clone this repo and register a local source path such as `./plugins/courseskills` in your own `~/.agents/plugins/marketplace.json`.

---

## Prerequisites

### codex2course

- Codex or Claude Code with the `imagegen` skill/tool available for slide image generation.
- Python 3 + [Pillow](https://pillow.readthedocs.io/) for PDF assembly:

  ```bash
  pip install Pillow
  ```

### api2course

- `OPENAI_API_KEY` set in the environment. The image script uses `gpt-image-2`.
- Python 3. The OpenAI image script uses only the standard library.
- Python 3 + [Pillow](https://pillow.readthedocs.io/) for PDF assembly:

  ```bash
  export OPENAI_API_KEY=your_key_here
  pip install Pillow
  ```

### pdf2video

- A completed slide deck from `codex2course` or `api2course`.
- [ffmpeg](https://ffmpeg.org/) on `PATH` for video assembly.
- TTS provider:
  - `edge`: free Microsoft Edge read-aloud voices via `edge-tts`.
  - `minimax`: paid API route. Requires `MINIMAX_API_KEY` and Python `requests`.

  ```bash
  pip install edge-tts requests
  export MINIMAX_API_KEY=your_key_here
  ```

### movecourse

- An AI-generated course directory containing `lessonN/*.mp4` files.
- The aistudy101 website checkout at `/home/likefallwind/code/aistudy101-website`.

---

## Shared workflows

Both Codex and Claude Code use the same skills and output layout. The skills are incremental: they inspect what already exists and continue from the next missing stage instead of redoing approved work.

```text
# ai-tutorials
Design a 10-lesson LLM application development course for CS undergrads.

# codex2course
Create a 6-hour Python async course for backend engineers.

# api2course
Use api2course to create a 6-hour Python async course with OpenAI API generated slides.

# pdf2video
Turn the course in ./course/ into a narrated lecture video, voice: zh-CN-YunxiNeural.

# Full course asset flow
Design a Vibe Coding course, build slides, then produce a narrated mp4.

# Publish generated videos only
Move only this generated course's videos to aistudy101 course-assets as ai-enlightenment.
```

---

## Output layout

```text
course/
├── outline.md          # Course info + image-generation settings
├── handout.md          # Full teaching notes
├── slide-units/        # Per-slide content derived from handout.md
├── slides/             # Rendered .png pages: 000-cover ... zzz-ending
├── course-deck.pdf     # Assembled slide deck
├── audio.md            # TTS voice / padding settings
├── narration/          # Per-slide spoken text
├── audio/              # Per-slide .mp3 files
└── course-video.mp4    # Final narrated lecture video
```

---

## Scripts

Several skills ship helper scripts used internally. You can also run them directly:

| Script | Purpose |
|---|---|
| `skills/codex2course/scripts/split_handout.py` | Slice `handout.md` into `slide-units/` |
| `skills/codex2course/scripts/images2pdf.py` | Combine `slides/*.png` into a PDF |
| `skills/api2course/scripts/generate_openai_image.py` | Generate one slide image with the OpenAI Image API |
| `skills/api2course/scripts/split_handout.py` | Slice `handout.md` into `slide-units/` |
| `skills/api2course/scripts/images2pdf.py` | Combine `slides/*.png` into a PDF |
| `skills/pdf2video/scripts/synth_audio.py` | Synthesize narration audio |
| `skills/pdf2video/scripts/assemble_video.py` | Pair slides + audio into `course-video.mp4` |
| `skills/movecourse/scripts/movecourse.py` | Copy or move `lessonN/*.mp4` into website course-assets |

```bash
python skills/codex2course/scripts/split_handout.py course/handout.md
python skills/codex2course/scripts/images2pdf.py course/slides course/course-deck.pdf
python skills/api2course/scripts/generate_openai_image.py --prompt-file course/prompt.txt --out course/slides/001.png
python skills/pdf2video/scripts/synth_audio.py course/
python skills/pdf2video/scripts/assemble_video.py course/
python skills/movecourse/scripts/movecourse.py --course ai-enlightenment --source course/ --dry-run
```
