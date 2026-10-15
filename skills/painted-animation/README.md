<div align="center">

# painted-animation

<a href="README.md"><img src="https://img.shields.io/badge/English-2B2233?style=for-the-badge" alt="English"></a>
<a href="README.zh-CN.md"><img src="https://img.shields.io/badge/简体中文-D97757?style=for-the-badge" alt="简体中文"></a>

A Claude Code skill for producing hand-painted animation and lyric videos with Claude Opus 5.5.
Part of [opus-video-skills](../../README.md).

</div>

![小镇姑娘](docs/xiaozhen-sheet.jpg)

## Overview

The skill enables Claude Opus 5.5 to produce complete animated videos, including storyboarding, character animation, camera work, transitions, music synchronisation and karaoke subtitles. All imagery is drawn procedurally in code; no image generation model is involved.

The method is derived from two projects by John Heibel: [PDoomVideo](https://github.com/JohnHeibel/PDoomVideo), a 156-second music video produced largely autonomously by Opus 5.5, and [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase), a general-purpose animation kit based on it. This skill packages both into a single workflow and extends it with a lyric-video pipeline that covers tempo detection, audio clipping and karaoke subtitles.

## How It Works

1. **Intake and storyboard.** For a music video, Claude first requests the audio and LRC lyrics, then presents an optional fill-in form for story, emotion, atmosphere / imagery, visual motif and content to avoid. Any creative field may be left blank. It then writes a shot list covering setting, palette, character arc and transitions. For each shot it lists the "reads", the timed sequence of what the viewer must understand.
2. **Drawing.** Each shot is a JavaScript function that paints the complete frame with p5.js and the [p5.brush](https://github.com/acamposuribe/p5.brush) watercolour library. Every frame is a pure function of time.
3. **Review.** Small contact sheets, frame strips and close-up crops may be rendered in the cloud. Claude inspects the images and revises the code until each shot meets the guidelines.
4. **Output.** Claude never starts a complete render without asking whether to render locally or in the cloud. Choosing local prohibits cloud full rendering and produces exact local commands instead. Frames are rendered in parallel and encoded to MP4 with ffmpeg, with audio muxed in if provided.

## Characteristics

- **Character consistency.** Each character is defined by a single drawing function, so its design is identical across all shots.
- **Frame-accurate timing.** Because frames are computed from time, events can be aligned precisely to beats or to individual sung syllables.
- **Localised revision.** A change to one element requires editing only the relevant code and re-rendering the affected range; the rest of the video is unaffected.
- **Built-in animation principles.** The engine provides anticipation, squash and stretch, follow-through and acted expression changes as reusable functions.
- **Automated self-review.** Rendered frames are checked against an explicit list covering legibility, timing, contacts, transitions and colour.

## Example: 小镇姑娘 (David Tao)

<p align="center"><img src="docs/koi-gag.gif" width="520" alt="大经理 → 大锦鲤"></p>

A 31-second lyric video for one verse of the song. Inputs: the lyrics, a request to take the song's wider context into account, a request to include the fan wordplay in which "大经理" (big manager) is heard as "大锦鲤" (big lucky koi), the audio file, and LRC timestamps.

- **Setting.** One small-town railway station throughout, with the ending mirroring the opening: her departure by train one year earlier, and his own departure at the end.
- **Visual motif.** A flower identifies the female character in each of her forms: at the train window, on the television, as a koi, and as a star.
- **Timing.** The song measures 154 BPM with 8 beats per line, and all action is cut to that grid.
- **Wordplay.** On the syllables "经理", the television image transforms into a koi while the subtitle is struck through and replaced with "锦鲤". The koi then leaves the screen and becomes the "shining star" of the following line.

The storyboard and scene code are available in [examples/xiaozhen](examples/xiaozhen/). The audio is not included for copyright reasons.

## Requirements

- Claude Code with Claude Opus 5.5
- Node.js, Google Chrome, ffmpeg
- Python 3 with numpy (for tempo detection only)

## Installation

See the [collection README](../../README.md#installation). As a plugin:

```
/plugin marketplace add tuzhechen2005/opus-video-skills
/plugin install painted-animation@opus-video-skills
```

## Usage

Describe the desired video in Claude Code, for example:

> Make a 15-second video of Clawd trying to catch a butterfly.

> Make a lyric video for this song. (with the audio file, LRC file, and a short description of the story and mood)

The skill can also be invoked directly with `/painted-animation`. It creates a project, presents a storyboard, builds and reviews each shot, and asks whether the full final render should run locally or in the cloud. Render times depend on the GPU; watercolour fills are considerably slower on integrated graphics.

## Repository Structure

| Path | Description |
|---|---|
| `SKILL.md` | Workflow and rules followed by Claude |
| `template/` | Animation engine: character, brushes, camera, transitions, karaoke, renderer |
| `scripts/new_project.sh` | Project scaffolding and toolchain check |
| `scripts/beat_grid.py` | Tempo and beat-phase detection with LRC line alignment |
| `references/music-video.md` | Guidelines for music videos and longer productions |
| `examples/xiaozhen/` | Storyboard and scene code for the example above |

## Acknowledgements

The animation engine and guide are adapted from [ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase) by John Heibel (MIT License; see [template/LICENSE](template/LICENSE)). The overall method follows his [PDoomVideo](https://github.com/JohnHeibel/PDoomVideo). The project uses p5.js, p5.brush, Puppeteer and ffmpeg. The skill and the example were produced with Claude Opus 5.5 in Claude Code.

## License

MIT. See the repository [LICENSE](../../LICENSE).
