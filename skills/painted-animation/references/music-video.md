# Long-form and music videos (lessons from the P(doom) video)

The P(doom) music video (156.6 s, 9 chapters, ~7,800 lines) was built by Claude Opus in Claude Code with only two instructions from the human: "use the Clawd character" and "give each lyric interesting visuals and transitions". This file captures how it was organised, so a video longer than ~20 s can be made the same way. Its storyboard is at [PDoomVideo/STORYBOARD.md](https://github.com/JohnHeibel/PDoomVideo/blob/main/STORYBOARD.md); a finished lyric video made with this skill is in [examples/xiaozhen](../examples/xiaozhen/).

## 1. Lock the song grid first

- **Ask for the song file and an LRC** (or the time each line starts). Never guess lyric timing from a slow "feel": pop songs are usually faster than they sound, and one line is often only 2–3 s.
- Measure the tempo and beat phase: `python3 scripts/beat_grid.py song.mp3 --from=100 --to=200 --lrc=song.lrc` (needs numpy). It prints the BPM, a beat time, and each lyric line's beat index. Lines usually start a fraction of a beat before a bar line (pickups).
  - P(doom): 88 BPM, beat 0.682 s. 小镇姑娘: 154 BPM (half-time 77), one line = 8 beats ≈ 3.1 s.
  - For a ballad, set `PROJECT.bpm` to the half-time pulse so idles and bounces don't look frantic.
- **Cut a clip on a bar line**: start one bar (or two) before the first line you use, end a bar or two after the last, and fade out:
  `ffmpeg -ss <start> -t <len> -i song.mp3 -af "afade=t=in:d=.25,afade=t=out:st=<len-2>:d=2" -c:a aac assets/clip.m4a`.
  Clip time = song time − start. Choose `start` so a beat lands on clip time 0, then `offset: 0`.
- Lyric table `LY` in `src/karaoke.js`: `[start, end, "text", opts]` in clip time, lines running into each other. `opts.sing` = seconds to sing the line (drives the fill); `opts.hold` keeps a line up (lifted, two-row KTV style) after the next starts; `opts.pun = { from, to, at }` strikes `from` and pops `to` above it.
- **Fast lines, one visual per line.** Budget every shot to its line's 8 beats; put the gag on the word it belongs to (find the word's beat), and when a gag needs more read time than the line gives, carry it into the next line (the koi keeps swimming and leaps out on the next line; the struck lyric is held above the next one) instead of slowing the song.
- Put the audio in `assets/` and set `PROJECT = { duration, bpm, offset, audio: 'assets/clip.m4a' }`.
- Write the beat times of each chapter at the top of its file as a comment: `const B = n => OFF + n * BEAT; // B(4) 2.94 · B(12) 8.39 …` so hits land exactly.

## 2. Storyboard shape for a song

- **One idea for the whole video**, stated in a paragraph (P(doom): "a stage show that goes off the rails… the whole apocalypse was a play").
- **Cast table**: each recurring character, their look and their arc (Clawd grows tiny → planet-sized → back to cute).
- **What ties it together**: a recurring set (all choruses return to the *same stage*, escalating each time), an escalating motif (the P(doom) thermometer climbs 8→34→61→86→99.9), a palette arc, a rule for transitions.
- **Chapters** = song sections (intro, verse, chorus, bridge…). Each gets one setting and a small palette, e.g. `The Lab · night indigo, lamp ochre, monitor teal`.
- **Shot table per chapter**: `| Time | Lyric | Shot | Out |`. One shot per lyric line (1.4–4 s). The *Out* column names how it leaves: "camera keeps pushing in", "CHOMP: mouth closes over the camera to black", "the cube drops through the floor".
- **Act the lyric, don't write it.** Every line is a visual gag that shows the meaning. The karaoke already carries the words, so no signs or captions that repeat them.
- A handful of big SFX across the whole video (FOOM, BOOM, CHOMP, SLAM) is fine; not more.
- End by rhyming with the opening (curtain opens at 0 s → curtain falls at the end).

## 3. Code layout for many chapters

- One file per chapter in `src/scenes/` (`c01_intro.js`, `c02_chorus1.js`, …), each an IIFE whose helpers stay private, ending with `shots([...])` for its shots. Add each `<script>` to `studio.html` in order.
- Shared, stable things (characters, recurring props like the stage/thermometer, palette, lyrics) live in shared files that chapter authors don't edit. Guest characters a later chapter reuses get exported on a shared object (`CAST.basilisk = (x, y, s, t, o) => {...}`).
- Chapter breaks get a transition (P(doom) used a brush wipe at each chapter boundary, with a different colour pair per chapter). Inside a chapter, the action carries across the cut.
- For karaoke, include `src/karaoke.js` after `timeline.js` in `studio.html` and fill `LY`. Keep key action above y ≈ 960 while a line shows. It loads a Chinese brush font (Ma Shan Zheng) for exactly the characters in `LY` through `window.EXTRA_FONTS`.

## 4. Parallel subagents (optional, for long videos)

P(doom)'s second generation was painted by parallel subagents, one per chapter, all briefed by the same guide. Do this only if the user asks for subagents or the video is long enough to justify it; otherwise build chapters yourself in order.

1. Write the storyboard, the shared files and at least one finished chapter yourself first, so there is a working reference for style and API.
2. Write a short `CHAPTER_BRIEF.md` in the project (the orchestrator's brief to subagents). It should contain:
   - "Read ANIMATION_GUIDE.md and STORYBOARD.md first."
   - Only edit your own chapter file. If a shared helper is missing, write it privately inside your IIFE. If you find a bug in a shared file, report it; don't edit it.
   - Frames are pure functions of `t` (they render in parallel, out of order): no state across frames, no `Math.random()`.
   - The canvas conventions (1920×1080, karaoke band), the beat grid, the chapter's palette, and the size rule (the lead character ≈ 40% of frame height in dance shots).
   - Performance budget: ≤ ~2.5 s/frame; hundreds of fills/strokes, not thousands.
   - The review loop: render sheets and strips of every shot, first/last frames, and the seams into and out of the chapter; open them and iterate until each shot is charming, readable and lively.
3. Spawn one subagent per chapter with that brief plus its chapter's storyboard rows and time range. Several can render at once.
4. When they finish, render sheets across every chapter seam yourself and fix continuity (palette, character scale, screen direction, the escalating motif).

## 5. Rendering a long video

Do not start the full-frame render merely because the storyboard or cloud previews were approved. At this final-render boundary, explicitly tell the user that a complete cloud render can be very slow and ask them to choose local or cloud rendering.

- **Local:** do not run the full render in the cloud. Confirm the user's operating system and local Node.js, Chrome and ffmpeg prerequisites, then provide the exact install, frame-render and encode commands for that platform.
- **Cloud:** state the video duration, total frame count and an estimated render time or cost, then wait for explicit confirmation before running the commands below.

Only after the user chooses cloud rendering, run:

```bash
node render.mjs --frames --workers=4          # parallel, resumable JPEG frames into out/frames
node render.mjs --encode --audio=assets/song.mp3 --out=out/video.mp4
```

`--frames` skips frames already on disk, so a crash or a fix to one chapter only needs `rm out/frames/f0XXXX.jpg` for that range and a re-run (or `--range=a:b`).
