# Tangbao Hatches · 《糖宝出世》 · 《糖寶出世》

A 15-second, three-shot sample with spoken Mandarin: the second run of the keyframe video workflow in [`SKILL.md`](../../SKILL.md), and the one that established the model can speak Chinese with matching lip movement straight from a keyframe.

| File | What it is |
|---|---|
| `shots.json` | The three prompts and their original lines exactly as submitted |
| `keyframes/shot-01.png` … `shot-03.png` | The approved t = 0 images — each clip starts from its keyframe |
| `tangbao-hatches.mp4` | The finished cut: 15.0 s, 1280×720, H.264 + AAC; no subtitles. The README's inline player plays the same file |

**Pipeline.** Scene and lines approved → image prompts approved → keyframes generated, the first set rejected and redone → keyframes hosted on a temporary preview channel → one `agnes-video-2.5-flash` task per shot (`mode: keyframe`, `720P`, `16:9`, 5 s) → clips normalized to one frame rate and pixel format, loudness-normalized, and concatenated with ffmpeg.

**Verdict.** A human listened and confirmed all three lines were Mandarin and matched the script. No automated analysis was taken as that verdict.

All media here are AI-generated; the keyframe PNGs carry the image generator's C2PA provenance manifest. The scene is inspired by the novel *Hua Qiangu* (《花千骨》) by Fresh果果; the three lines were written for this run, and no text from the novel is used.

中文说明见上一级的 [简体中文 README](../../README.md)；中文說明見上一層的 [繁體中文 README](../../README.zh-Hant.md)。
