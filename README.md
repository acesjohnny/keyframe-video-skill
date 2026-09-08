# Keyframe Video Production — an Agent Skill

An [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) for producing short, multi-shot AI videos where every shot starts from a generated keyframe image.

The workflow applies to any keyframe-driven video model. It was written and verified against the Agnes video API (`agnes-video-2.5-flash`).

**English** · [中文](README.zh.md)

## What it is actually for

Most of the value is not "how to call a video API". It is the discipline around the call:

- **Four approval gates** — storyboard, image prompts, generated images, hosting — so the user approves the plan before the spend, instead of reviewing a finished artifact they did not want.
- **A character bible reused verbatim** across every shot prompt, which is what actually holds continuity together.
- **Honest verification.** An agent cannot hear audio. This skill shows how to gather real evidence that speech exists (RMS envelope bursts, changing mouth shapes at those timestamps) while never letting that masquerade as confirmation that the language or the lines are correct — that verdict stays with a human.
- **Failure modes that cost real time**, written down: a download path that reads the asset URL from the wrong nesting level, burned-in subtitles, dialogue mixed 35 dB too quiet, a second run silently overwriting the first, a stale git lock blocking every write.

## Install

Copy the skill into your agent's skills directory, for example:

```sh
mkdir -p ~/.claude/skills/keyframe-video-production
cp SKILL.md ~/.claude/skills/keyframe-video-production/
```

A Chinese edition is available as `SKILL.zh.md`.

Then ask your agent for a short video demo from a scene, and it should pick this up.

## Provenance

Extracted from two real production runs. The second verified that `agnes-video-2.5-flash` generates intelligible spoken Mandarin with matching lip movement directly from a keyframe, with no external text-to-speech — confirmed by a human listening, not by automated analysis.

## License

MIT — see [LICENSE](LICENSE).
