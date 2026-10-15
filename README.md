<div align="center">

# opus-video-skills

<a href="README.md"><img src="https://img.shields.io/badge/English-0E0F0E?style=for-the-badge" alt="English"></a>
<a href="README.zh-CN.md"><img src="https://img.shields.io/badge/简体中文-DDF53D?style=for-the-badge" alt="简体中文"></a>

A Claude Code skill for producing hand-painted animated videos with Claude Opus 5.5.

</div>

## Overview

The skill in this repository teaches Claude Opus 5.5 to produce a complete hand-painted video, covering storyboarding, animation, review, lyric synchronization and final encoding. All imagery is generated procedurally in code; no image or video generation model is involved. Every frame is a pure function of time, rendered in headless Chrome and encoded with ffmpeg.

## Skill

### painted-animation

![painted-animation](skills/painted-animation/docs/xiaozhen-sheet.jpg)

Draws every shot with p5.js and the p5.brush watercolour library. Characters act through a library of expressions and motion principles. Music videos are cut to the measured beat, and lyric videos carry word-by-word karaoke subtitles. Example: a 31-second lyric video for 陶喆《小镇姑娘》. [Documentation](skills/painted-animation/README.md).

## Requirements

- Claude Code with Claude Opus 5.5
- Node.js, Google Chrome, ffmpeg
- Python 3 with numpy (painted-animation tempo detection only)

## Installation

**As plugins (recommended).** In Claude Code:

```
/plugin marketplace add tuzhechen2005/opus-video-skills
/plugin install painted-animation@opus-video-skills
```

`/plugin update` fetches new versions.

**As personal skills.** Clone the repository once and link the skills you want:

```bash
git clone https://github.com/tuzhechen2005/opus-video-skills ~/opus-video-skills
ln -s ~/opus-video-skills/skills/painted-animation ~/.claude/skills/painted-animation
```

Updating is then a `git pull` in `~/opus-video-skills`.

> **Upgrading from `painted-animation`.** This repository was previously the single `painted-animation` skill, cloned directly into `~/.claude/skills/painted-animation`. That layout no longer works, because the skill now lives in `skills/painted-animation/`. Remove the old clone and install with either method above.

## Usage

Describe the video in Claude Code. The matching skill is selected automatically, or it can be invoked by name:

> Make a 15-second video of Clawd trying to catch a butterfly.

For a music video, provide the song audio and an LRC lyric file, then optionally fill in the story, emotion, atmosphere / imagery, visual motif and content to avoid. The skill scaffolds a project, presents a storyboard, and reviews every shot. It never starts the complete final render without asking for an explicit local-or-cloud choice; choosing local prohibits cloud full rendering and returns exact local commands.

## Repository Structure

| Path | Description |
|---|---|
| `.claude-plugin/marketplace.json` | Plugin marketplace manifest |
| `skills/painted-animation/` | Watercolour animation skill: engine, guides, examples |

## Adding a Style

A new style is a new directory under `skills/` containing a `SKILL.md`, a runnable `template/` and at least one worked example, plus an entry in `.claude-plugin/marketplace.json`. The existing skill's contract is that frames are pure functions of time, `render.mjs` produces contact sheets and parallel frame renders, and every shot is reviewed from rendered images before the final encode.

## Acknowledgements

The painted-animation engine and guide are adapted from [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase) by John Heibel (MIT License), and the method follows his [PDoomVideo](https://github.com/JohnHeibel/PDoomVideo). The project uses p5.js, p5.brush, Puppeteer and ffmpeg. The skill and examples were produced with Claude Opus 5.5 in Claude Code.

## License

MIT. See [LICENSE](LICENSE).
