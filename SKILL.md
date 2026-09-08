---
name: keyframe-video-production
description: Use when producing a short keyframe-driven AI video — designing shots, generating keyframe images, hosting them on a temporary public URL, submitting video generation tasks, and assembling the cut. Covers spoken dialogue, character continuity across shots, and stopping for human approval before each irreversible or billable step.
---

# Keyframe Video Production

Take a source scene and turn it into a short multi-shot video, where each shot starts from a generated keyframe image.

The workflow applies to any keyframe-driven video model. It was written and verified against the Agnes video API (`agnes-video-2.5-flash`), which takes a `first_frame` image URL plus a prompt and returns an MP4.

## Fit

Use this when:

- The target is a short demo (typically 3 shots x 5s), not a finished film.
- Shots must start from a specific image, so images need to be publicly fetchable over HTTPS.
- Character or scene continuity across shots matters.
- The user wants to approve the plan before spend, not review a finished artifact.

Do **not** use this for pure text-to-video with no keyframe, long-form editing, or anything published outward without a fresh confirmation.

## Verified Capability Notes

Measured on `agnes-video-2.5-flash` in keyframe mode, 2026-09:

- It **does** generate intelligible spoken Mandarin with matching lip movement, directly from a keyframe and a prompt containing the line. No external text-to-speech was needed. The verdict came from a human listening to the output, not from automated analysis.
- Clips come back with real ambient audio rather than a silent track.
- Generated speech sits quiet, around -35 dBFS mean, so plan on loudness normalization.
- An API key for the video models may carry **no speech or TTS model at all**. List the available models before promising anyone a TTS fallback.

Treat all of this as a dated observation about one vendor, not a guarantee. Re-check before relying on it.

## Hard Rules

1. **Four gates, four stops.** Script/dialogue/storyboard, then image prompts, then generated images, then hosting. Stop and wait for the user at each. Do not chain through them because the earlier answer sounded enthusiastic.
2. **Hosting is its own approval.** Uploading images to a public URL is a deploy action, separate from approving the images themselves. Ask for it separately.
3. **Approval does not widen.** "Go ahead" authorizes the actions you just described. A rerun, an extra shot, or one more billable task needs a new ask.
4. **Verify the source.** A fetched summary can be wrong about basic facts, including which chapter it is describing. Read the raw content before designing anything on top of it.
5. **Write original dialogue.** Borrow scene structure, not sentences. Do not reproduce a source work's prose in prompts or output.
6. **The agent cannot hear.** Audio envelope and lip-shape analysis prove *something is being spoken*; they never prove the language or the lines are correct. That verdict belongs to a human, and until it arrives the result stays recorded as unverified.
7. **Never state billing you did not check.** Say it is an assumption, and say so explicitly.

## Workflow

### 1. Source and storyboard (gate 1)

Fetch the real source text and verify its identity from the raw content, not a summary. Pick a beat that fits the runtime and, for a dialogue test, one that carries short punchy lines.

Present a shot table: shot, duration, visual, dialogue line, speaker. State plainly that the dialogue is original. Ask for art direction and the audio route up front, since both change every downstream prompt.

### 2. Image prompts (gate 2)

Write a **character bible** — one block defining every recurring character, the setting, and the negative constraints — and reuse it **verbatim** in every shot prompt. This is what holds continuity together; per-shot text should only add framing and action.

Put prompts in files rather than inline shell arguments. Long non-ASCII prompt text gets mangled by shell escaping.

For a lip-sync test, require in every prompt that faces are fully visible and mouths unobstructed. A beautifully framed shot with the speaker in profile tests nothing.

Show the prompts for approval before generating.

### 3. Generate and review images (gate 3)

Generate keyframes, then **look at them yourself** before showing the user. Check character consistency across shots, visible faces and mouths, framing variety (near-identical framing makes a repetitive cut), and whether any stylized creature or face has landed in the uncanny valley.

Report what is wrong as plainly as what is right, then let the user choose. On a redo, archive the superseded versions rather than overwriting them, and tighten the bible so the fix holds across every regenerated shot.

Keyframes should show the state at t=0, before the shot's action — the motion is what the video adds. If the image already shows the end state, that beat is gone.

### 4. Hosting (gate 4)

Keyframe mode needs a public HTTPS URL per image. Firebase Hosting preview channels work well for this, since they carry a native expiry. Use a **new, dedicated** channel with a short expiry rather than reusing an earlier round's — prior job records still cite those URLs, and overwriting them destroys that evidence.

Before hosting, build the upload directory and scan it: API key strings, generic credential patterns, and image metadata for paths, prompt text or identity. Images from some generators embed a C2PA provenance manifest and an invisible watermark; harmless for internal tests, but disclose it, and think twice before reusing such images in public deliverables.

State up front: the target, its expiry, the exact file list with sizes and hashes, the command, an explicit list of what will **not** be touched (production sites, other channels, database rules, auth settings, billing tier), how you will verify, and how to roll back.

After hosting, anonymously GET every URL and compare SHA-256 against local, then re-list targets to prove production and prior channels were untouched. Save the result as JSON.

### 5. Submit and assemble

Submit one task per shot. Persist a job record **before** the POST so an interrupted submit is recoverable and never silently re-fires. Poll by the returned video id.

Assemble with a normalization pass (uniform scale, fps, pixel format; silent tracks padded; loudness normalized), then concatenate. Sample frames from the finished cut and look at them.

### 6. Record

Write down what was produced and, in its own section, what remains **unverified** — language verdict, billing, any anomaly — rather than burying it in prose. When the user later gives a verdict, write it back and say who judged it.

## Round Isolation

A second run must not overwrite the first. Scripts that hardcode a single `output/` directory will clobber prior clips, job records and masters. Give every round its own subtree and its own shot definitions, and verify the no-argument path still behaves exactly as the first round did.

## Validation Checklist

- Source identity confirmed from raw content, not a summary.
- Character bible byte-identical across all shot prompts.
- Upload directory contains only intended files; credential and metadata scans clean.
- Every hosted image: HTTP 200, expected content type, SHA-256 equal to local.
- Production and prior hosting targets unchanged.
- Task count within what was authorized.
- Final cut: expected duration, resolution, fps, audio stream present.
- Sampled frames reviewed by eye.
- Unverified claims recorded as unverified.

## Failure Modes

### Task completes but nothing downloads

The client looks for the asset under `metadata.url` while the API returns `url` at the top level, so the success path raises instead of saving. Accept both:

```python
url = result.get('url') or (result.get('metadata') or {}).get('url')
```

### A generation CLI refuses its own default model

Server-side default models move ahead of installed CLI versions, and the failure reads as a hard 400 telling you to upgrade. Pin an older model explicitly before reaching for a full CLI upgrade; most such CLIs cache the list of available model names locally.

### Burned-in subtitles

Models add captions when dialogue is requested. Every prompt must forbid text, subtitles and watermarks explicitly.

### Dialogue too quiet in the master

Generated speech can sit near -35 dBFS. Loudness normalization in the per-clip pass brings it up. Check the final peak level afterwards, since a concat re-encode can push peaks to full scale and clip.

### Extra or missing utterances

A shot's audio may contain more speech bursts than lines scripted. Count bursts in the envelope, flag the discrepancy, and let a human judge on listening.

### Proving speech without hearing it

You cannot verify audio directly, but you can gather real evidence. Compute a per-100ms RMS envelope: discrete bursts against a low floor, with a dynamic range above roughly 20 dB, indicate speech, whereas flat ambience stays under about 12 dB. Then sample frames at the burst timestamps and check that mouth shapes actually change between them. Report this as evidence of speech, never as confirmation of language or content.

### `timeout` is missing on macOS

macOS is not a coreutils box. Use your harness's own call timeout instead of wrapping the command.

### Stale `.git/index.lock`

A 0-byte lock file left behind by an interrupted git write blocks every later git operation, silently, for as long as it sits there. Confirm it is 0 bytes, has an old timestamp, and that no `git` process is running, then remove it.

### Video files will not stage into git

Many repositories deliberately exclude `*.mp4` and similar to stop repository bloat, and the exclusion is easy to mistake for an oversight. Check whether the rule is intentional and surface the conflict instead of forcing the add. Large media usually belongs in backup or object storage, not in git history, which cannot be trimmed later without a rewrite.
