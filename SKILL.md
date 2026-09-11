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

- It **does** generate intelligible spoken Mandarin with matching lip movement, directly from a keyframe and a prompt containing the line. No external text-to-speech was needed. The verdict came from a human listening to the output, not from automated analysis. Confirmed four times — most recently on the prompt-language A/B (2026-09-10), where both versions spoke Chinese and the difference between them was quality, not language — and on an 18-shot film (2026-09-10) that also settled a further question: **two distinct voices can hold across one cut** — a teenage girl and a small child, each with its own voice bible, stayed in character over thirteen speaking shots.
- **There is no switch that turns the generated soundtrack off**, and no negative-prompt field — checked against
  the vendor's own API reference (2026-09-11), not a summary. Nor can you hand it a clean track while keeping the
  approved first frame: keyframe mode rejects `audios`, and the reference mode that accepts `audios` rejects
  `first_frame`, which demotes the keyframe from "this is frame one" to "use this as a style reference". Every
  audio problem therefore has to be solved in the prompt or in assembly.
- Off-screen narration **works in a probe and fails in production**, so do not rely on it. A controlled
  single-shot probe (2026-09-09) produced a clear narrator voice with every visible mouth shut; the same
  technique across ten shots of a finished film had characters lip-syncing the narration in several of them.
  A prompt can say *nobody* speaks, but when audio exists and a face is on screen the model tends to find a
  mouth for it. Give the narration a **visible speaker** instead — put a storyteller in frame, to one side,
  with the story playing behind them. That turns a negative constraint ("no one may lip-sync") into the
  positive one the model is already reliable at ("this person is speaking, everyone else is silent").
  **Verified** on a ten-shot rebuild (2026-09-09): six timepoints sampled across every shot showed the
  storyteller lip-syncing and every character behind him mouth-shut, with no exceptions.
- Clips come back with real ambient audio rather than a silent track.
- Generated speech sits quiet, around -35 dBFS mean, so plan on loudness normalization.
- Voice casting is per-clip and has no id to pin, so one character's timbre wanders across a cut unless the
  prompt describes the voice itself. Write a **voice bible** — age, sex, timbre, pitch register, rate, accent,
  delivery — and repeat it verbatim in every clip that character speaks, exactly as the character bible fixes
  appearance. And carry no per-shot tone directions: varying "warm and kind" / "wry" / "emphatic" shot to shot
  is a variable the model casts against, and it is also the text it burns into captions.
  **Verified** 2026-09-09: nine clips of one character, previously wandering in timbre, came back as one
  voice after adding the block and stripping the per-shot tone lines. The verdict was a human listening,
  not analysis — the agent cannot hear.
- An API key for the video models may carry **no speech or TTS model at all**. List the available models before
  promising anyone a TTS fallback — and list them again before assuming which *stage* the vendor can serve.
  The same key that runs the video may also carry image models, in which case reaching for an unrelated CLI
  for keyframes buys a second quota, a second pipeline and a model id nobody can verify.
- A generation CLI's image tool may expose **no model selector and no verifiable model id** at the CLI. But the
  output may still carry one: a C2PA manifest (PNG `caBX` chunk) can name the generator and a version.
  Read it before declaring the model unknowable — and still report it as what the metadata claims, not as a
  version you verified, since a manifest's version field need not match a vendor's marketing name.

Treat all of this as a dated observation about one vendor, not a guarantee. Re-check before relying on it.

## Hard Rules

1. **Every gate is a stop.** Script/dialogue/storyboard, then blocking, then image prompts, then character
   cards, then generated images, then hosting. Stop and wait for the user at each. Do not chain through them
   because the earlier answer sounded enthusiastic. The count has grown as the workflow has; what does not
   change is that approval of one gate is never approval of the next.
2. **Hosting is its own approval.** Uploading images to a public URL is a deploy action, separate from approving the images themselves. Ask for it separately.
3. **Approval does not widen.** "Go ahead" authorizes the actions you just described. A rerun, an extra shot, or one more billable task needs a new ask.
4. **Verify the source.** A fetched summary can be wrong about basic facts, including which chapter it is describing. Read the raw content before designing anything on top of it.
5. **Write original dialogue.** Borrow scene structure, not sentences. Do not reproduce a source work's prose in prompts or output.
6. **The agent cannot hear.** Audio envelope and lip-shape analysis prove *something is being spoken*; they never prove the language or the lines are correct. That verdict belongs to a human, and until it arrives the result stays recorded as unverified.
7. **Never state billing you did not check.** Say it is an assumption, and say so explicitly.

## Workflow

### 0. Source analysis, for a long work (before gate 1)

A single scene needs only the source check in gate 1. A whole novel needs a pre-production pass first, and its
output is what every later gate draws on: who is in which part of the book, what each of them looks like at each
stage, and which chapters become which episode.

- **Work copy, not archive.** Fetch the text only to read it. Keep it in a scratch directory; commit the analysis
  — summaries, counts, chapter references — never the text. Hard rule 5 applies to notes as much as to prompts.
- **Measure before planning.** Total length, per chapter, per volume, and **gaps**: one 509,000-character novel
  (2026-09-11) turned out to be missing two chapters at its source, which would have left two episodes with a hole
  in the plot. Find them before splitting episodes, not after.
- **Count the cast; do not recall it.** Build the roster from per-chapter mention counts with aliases merged
  (longest name first, so a surname-plus-given-name is not also counted as the given name) and ambiguous forms of
  address left out. A seed list drawn from memory carried three names from the screen adaptation that occur zero
  times in the book. Counting found them; building from memory would have filed cards for people who do not exist.
- **Test the discovery heuristic before trusting it.** Mining "X said" patterns for speakers returned fragments
  like "suddenly spoke" — the source had been stripped of most punctuation. Per the detector rule, discard it.
- **Read description, not the book, per character.** Pull the passages near a character's name that mention age,
  hair, clothing, scars or marks. A loose keyword window pulled 26% of the book for the lead; restricting the
  keyword to within ~25 characters of the name and dropping common words brought it to 5%, readable in one sitting.
- **Write the bible by stage, with evidence.** A lead can pass through eight distinct looks across a long work.
  Give each stage its age, look, costume, marks and temperament, tag every line as stated in the text or inferred,
  and cite the chapter. Track the props that change state across the story (a token given, broken, returned),
  because a card for the wrong stage is a continuity error that no prompt fixes.
- **Schedule cards by first appearance.** Map each character's first chapter to its episode and build cards in
  that order; half the cast of a long work may not appear until its second half.
- **Split episodes on chapter boundaries.** Target a length of source per episode, never cross a volume, merge a
  short tail into the previous episode. Then state the production scale out loud: at roughly seven seconds a
  shot, a 25-minute episode is about 220 shots.
- **Compute, never estimate, a number you write down.** Two episode numbers in one roster were typed as
  estimates ("around episode 18") and one was wrong by a whole episode; the tool that produced the table could
  have printed the exact value. A plausible number in a planning document is copied forward without question.

### 1. Source and storyboard (gate 1)

Fetch the real source text and verify its identity from the raw content, not a summary. Pick a beat that fits the runtime and, for a dialogue test, one that carries short punchy lines.

Present a shot table: shot, duration, visual, dialogue line, speaker. State plainly that the dialogue is original. Ask for art direction and the audio route up front, since both change every downstream prompt.

### 1.5. Blocking: put the film in a space before writing any shot (gate 1.5)

Shots written as independent paragraphs of text have no shared geometry, and every continuity failure that a
prompt cannot express comes from that: a cast list that omits whoever the camera is attached to, ten shots that
turn out to be one shot re-skinned, a prop that appears on the wrong side of a room between cuts.

So before gate 2, describe each **scene** once — not each shot. Give every actor and every camera a position, a
facing and a field of view, then have each shot cite a camera instead of re-inventing the geometry. Three things
follow mechanically rather than by care:

- **A shot's cast is computed, not remembered.** Whoever falls inside the camera's cone is in frame, including
  the person the camera is attached to. This is the one list that hand-writing keeps getting wrong.
- **Camera variety becomes checkable before spending anything.** Score each shot with the root-structure
  signature and compare; identical signatures mean identical shots, and the fix is a different camera, not a
  different background.
- **A revision moves a camera or an actor, not a paragraph.** Changing where someone stands updates every shot
  that sees them, instead of editing prose in eighteen files and hoping.

Keep it as light as the film needs — a coordinate table and a top-down plan is enough; a 3D previz tool is the
same idea with more fidelity, not a different one. What matters is that the geometry exists once, outside the
prompts.

The same check catches a structural error that no single shot reveals. A "who's who" sequence — one character
naming people while the camera cuts to them — is off-screen narration shot by shot, and the cone test flagged
all five such shots of a 20-shot episode (2026-09-10) as speakers outside their cameras. The fix is the old
film grammar **speak, then see**: the speaker says the line in a shot where she is visible, and the next shot,
with no dialogue, shows who she meant. It removes the stolen-mouth risk and is simply better cutting.

Present the plan at gate 1.5 the way you present a storyboard: the top-down view, the per-shot cast the geometry
implies, and any signature collisions, before writing a single prompt.

### 2. Image prompts (gate 2)

Write a **character bible** — one block defining every recurring character, the setting, and the negative constraints — and reuse it **verbatim** in every shot prompt. This is what holds continuity together; per-shot text should only add framing and action.

**The cast list is whoever is *in frame*, not whoever acts.** The shot's cast is what decides which bibles
and which reference images get attached, so it has to be derived from the picture, not from the action. A
close-up of one character's ear with a creature inside it is a two-character shot: list only the creature and
the human whose ear it is arrives with no bible and no reference, and her costume, hair and ornaments are
reinvented from nothing. That is what happened on `shot-03` of an 18-shot film (2026-09-09) — a bracelet,
a scarf and a brooch that exist nowhere in the design. Walk the storyboard and ask of each shot *who is
visible*, including the person the camera is attached to.

**Assemble prompts mechanically from the asset files; never hand-copy a bible into a shot.** Byte-identical
across eighteen shots is not something to achieve by discipline. Keep each block in its own asset page,
extract them with a small builder, and have the builder **fail loudly on a block that does not exist** — a
renamed or missing block must be an error, not a silently weaker prompt. It also means a design change is
made in one place and every shot follows.

**Write the scene description in English and keep the spoken line in its own language.** On a controlled A/B
(2026-09-10 — same keyframe, same line, same duration, only the prompt language differing), the all-Chinese
prompt burned captions across all 24 sampled timepoints, including one of its own **stage directions** rendered
as if it were dialogue; the English-scene version, checked on the identical crop, carried no text at any
timepoint. The mechanism is plausible: the model writes the script it can read, so the only non-English text in
the prompt should be the line you actually want spoken. One sample is a strong signal, not a law — expect to
re-confirm it on a real round.

**Keep stage directions out of the paragraph that carries the line — this one is not advice.** On the same A/B,
the all-Chinese version did not merely caption its stage direction, it **said it out loud**: a human reviewer confirmed by
ear (2026-09-10) that the clip speaks the prompt's own instruction. Anything sitting beside the dialogue can be
executed as dialogue, so the line gets its own paragraph and everything about how to perform it lives elsewhere.

**That English rule was measured on a shot with a line in it. A no-dialogue shot behaves differently, and the
difference is severe.** On the first full round with English prompts (2026-09-10, 20 shots), every one of the six
no-dialogue shots came back speaking — five of them the *same* Chinese sentence, a sports-host opener unrelated to
the story and present in no prompt anywhere, delivered on camera in three and as voice-over in two; the sixth
babbled. One identical sentence across five unrelated images is the signature of a model default filling an audio
slot nobody assigned. The vendor-agnostic explanation matches the documented behaviour of joint audio-video models:
leave the dialogue undescribed and the model improvises some. The English no-dialogue clause made it worse by
naming what it forbade ("no narration and no voice-over"), and the two faceless shots produced exactly that.

What was tried, each on the same shot and the same first frame:

| Version | Wording | Result |
|---|---|---|
| r1 | English, "no character speaks / no narration / no voice-over" | spoke the host line |
| r2 | English, soundtrack given a positive job, every speech word and the word *game* removed | mouth opened repeatedly |
| r3 | r2 paragraph for paragraph, **in Chinese** | silent throughout; confirmed by ear |

r3 carried two more shots, but a close-up face (a sulking boy) still spoke and a wide shot grew a voice-over plus
two figures walking up to camera. What fixed those two was the formula in a major vendor's official prompt guide
for this class of model: write the sound as its own section split into **voice / effects / music**, and put the
control phrase in the voice slot — **「人声：无台词，没有对白或人声」** — then describe the face as holding one
expression throughout rather than an expressive one ("sulking" invites mouth movement). Wide shots also need the
figures pinned to wide-shot scale and away from camera in positive terms.

So for a no-dialogue shot: **Chinese prompt, a structured sound section whose voice slot says 无台词, a fixed
expression, and no character bible that contradicts silence** — a bible that says someone is "in the middle of
an unstoppable anecdote" while the shot says he is silent produces gibberish, the model splitting the difference.
This rests on five shots, not a law; verify each one by sampling frames at the loudest audio moments and hand the
language verdict to a human. Where a face still speaks, the fallback is the local push-in over the keyframe.

Put prompts in files rather than inline shell arguments. Long non-ASCII prompt text gets mangled by shell escaping.

**When the story makes two characters identical, design the difference yourself — twice over.** A plot that
turns one character into a copy of another is a continuity disaster on screen: the audience cannot tell who is
speaking. Give the copy two independent marks, not one — a silhouette difference (hair length) *and* a colour
or pattern difference — so the pair still reads when one mark is lost to angle, scale or motion blur. Two
redundant marks held across five shots of a finished film (2026-09-10) where a single mark would have been
ambiguous in at least two of them. Better still, let a character remark on the resemblance: the design
constraint becomes a joke instead of a defect.

**Build the negative constraints from the shot's own positive decisions, not from a standing list.** A fixed
block of "no text, no modern objects, no extra fingers" carried into every shot leaves each shot's *actual*
design decisions unguarded, and that gap is where the recurring failures live. Every positive decision worth
making is worth mirroring:

| Positive decision | Negative mirror it needs |
|---|---|
| the box is bare wood, gold only as a hairline outline | no gilded box, no gold relief panels, no metal casing |
| the scroll is blank, the title lands in assembly | no calligraphy, no characters, no writing on the scroll |
| this shot contains no living thing | no people, no presenter, no figure addressing camera |
| the storyteller speaks, everyone else is silent | no other character's mouth open, no second speaker |

Assemble it as: four standing blocks (AI-look / non-cinematic composition / light / colour) + a genre block +
**a consistency block generated per shot from that shot's own constraints** + an anatomy block when people are
present + a closing `no watermark, no text, no logo`. Then de-duplicate. Each of the four failures above cost a
regenerated shot or a rebuilt clip, and each had a positive clause with no mirror.

**Leave about a second of silence after each spoken line.** Ask for it in the prompt. Generated speech
otherwise runs to the last frame, which leaves nothing to cut on, pushes the concatenated master's peaks
toward full scale, and makes every join audible.

For a lip-sync test, require in every prompt that faces are fully visible and mouths unobstructed. A beautifully framed shot with the speaker in profile tests nothing.

Show the prompts for approval before generating.

### 2.5. Build character cards before shot keyframes

A character's identity reference should be a **card, not a frame from the film**. Generate it on a neutral grey
background under soft even light, with no hard shadow or rim light, and keep words like "studio" out of the
prompt; front full-length, back full-length, and one large close-up of the face. Then reference the face only
from that close-up, so a small low-quality head inside a wide shot never becomes the source of truth.

The reason is mechanical: **whatever else is in the reference gets absorbed into the character.** Background,
palette and shadow direction ride along into every later shot that cites it. Handing a model an approved *scene*
frame as an identity reference — the obvious thing to do, since it is already approved — is how a plain robe
comes back with embroidery and a costume drifts without any prompt changing. Anything that is not the person
belongs outside the card.

Give each recurring **state** its own card rather than describing the change in a shot prompt: smiling and not,
each costume, each age, before and after an injury. And treat voice the same way — one voice module per
character, cited unchanged, never re-improvised per shot.

**A card set can contradict itself, and the cause is a design decision nobody made.** A first round of cards
(2026-09-10) grew a petal-shaped bodice with a jewel on the front views while the back views came out bare, a hair
ornament on one view only, and three different hairstyles for one character across her three cards. The bible said
nothing about what the torso wears or how the hair is dressed, and an unstated attribute is invented afresh in every
image. Write the positive decision first — garment, neckline, hairstyle, "identical in every view" — then mirror
it in the negatives; without a positive decision there is nothing for a negative to mirror. Do not stress-test a
set that disagrees with itself: sixteen probes of an inconsistent card only measure the inconsistency.

**Stress-test a character before the film depends on it.** A card that looks right in one pose is not yet an
asset. Generate the character from the card in a spread of states the film will actually need — speaking,
laughing, head lowered, three-quarter turn, hands raised near the face, seen smaller in a wide shot — then crop
every face to one contact sheet and judge the sheet, not the images one at a time.

The pass line has to be a number decided **before** looking, or it becomes "it seems fine": name the probe count
and the tolerance up front — say, of eight probes at most one may read as a different person, and any drift in a
fixed identity mark (a hairline, an ornament, an eye shape) counts as a failure even when the face survives. Miss
the line and the card is reworked, not the shots. This is still a structured visual judgement rather than a
metric, and it should be reported as one; the value is that the threshold and the evidence exist before the
first real shot rather than after the finished cut.

Say what the probes cost. On a quota-metered image account this is eight or ten generations per character before
any keyframe exists, and that budget is the user's call, not yours.

### 3. Generate and review images (gate 3)

Generate keyframes, then **look at them yourself** before showing the user. Check character consistency across shots, visible faces and mouths, framing variety (near-identical framing makes a repetitive cut), and whether any stylized creature or face has landed in the uncanny valley.

Report what is wrong as plainly as what is right, then let the user choose. On a redo, archive the superseded versions rather than overwriting them, and tighten the bible so the fix holds across every regenerated shot.

**A deviation that is consistent is not drift, and the two get different treatment.** A character who reads
ten years older than the bible says, in *every* shot, has not broken continuity — the cut will hold together.
Report it plainly as a departure from the design, say what it would cost to re-run, and let the user decide;
do not quietly re-run it, and do not file it as a continuity failure. Drift is when the same character differs
*between* shots, and that is the one that must be fixed.

Some looks the model will not give up. A character meant to read sixteen came back in his twenties three times
(2026-09-10) — once with the age as adjectives, once as proportion and a reference point — and the second and third
were indistinguishable. The genre's pull toward an attractive young adult outweighed both. After two attempts that
move nothing, stop spending: accept the look, or move the beat off the face — if the joke is "he is only a boy",
a line can say it.

Keyframes should show the state at t=0, before the shot's action — the motion is what the video adds. If the image already shows the end state, that beat is gone.

**Audit repetition with a root-structure signature, not by eye.** "Vary the framing" is advice nobody can check.
Instead label every shot that shares a recurring setup with the same ten properties and compare the labels:

```
visual proposition / dominant shape / spatial structure / subject scale and count /
light structure / colour structure / stillness vs motion / what rewards looking /
central device or none / viewpoint and camera distance
```

Two shots whose *locations* differ but whose signatures mostly match are the same shot re-skinned, and a run of
them reads static however much the background changes. Ten shots of one on-camera narrator scored this way came
back with nine of ten properties identical — a defect that a shot-by-shot look had passed. Where a signature
repeats, change a structural property, not the scenery.

**The ten properties do not weigh the same.** Four naming shots in one episode (2026-09-10) differed on paper — left
versus right of frame, warm versus soft light, a raised arm versus a raised forelimb — and the comparison passed
them as distinct. On screen they were one shot four times. The properties that actually read are **scale, how many
subjects are in frame, and which side the camera stands**; changing one of those (an over-the-shoulder two-shot
in place of a single close-up) broke the repetition at once. Distinguish shots with strong properties; a
"no duplicates" built from weak ones is a pass on paper only.

For a set of shots that must belong together, write the three lists explicitly before generating: **what stays
constant, what is allowed to vary, and each shot's own visual proposition.** A bible supplies the first;
without the third, every shot inherits the model's default framing.

### 4. Hosting (gate 4)

Keyframe mode needs a public HTTPS URL per image. Firebase Hosting preview channels work well for this, since they carry a native expiry. Use a **new, dedicated** channel with a short expiry rather than reusing an earlier round's — prior job records still cite those URLs, and overwriting them destroys that evidence.

Before hosting, build the upload directory and scan it: API key strings, generic credential patterns, and image metadata for paths, prompt text or identity. Images from some generators embed a C2PA provenance manifest and an invisible watermark; harmless for internal tests, but disclose it, and think twice before reusing such images in public deliverables.

State up front: the target, its expiry, the exact file list with sizes and hashes, the command, an explicit list of what will **not** be touched (production sites, other channels, database rules, auth settings, billing tier), how you will verify, and how to roll back.

After hosting, anonymously GET every URL and compare SHA-256 against local, then re-list targets to prove production and prior channels were untouched. Save the result as JSON.

### 5. Submit and assemble

Submit one task per shot. Persist a job record **before** the POST so an interrupted submit is recoverable and never silently re-fires. Poll by the returned video id.

**Classify every failure by its persisted state before retrying anything.** A batch that comes back half-failed
contains at least three different situations that look identical in a summary line and need opposite handling.
On an 18-shot run (2026-09-09), 9 failed and split as:

| State | Evidence in the job record | Correct action |
|---|---|---|
| Alive, only the *poll* failed | holds a `video_id`; last error is a rate-limit | **Re-poll. Never resubmit.** The task is running |
| Submission explicitly rejected | no `video_id`; error body is a queue-full/refusal code | Archive the record and resubmit — the server built nothing |
| Task genuinely failed server-side | holds a `video_id`; status `failed` | Archive and submit a **new** task |

Three of those nine were merely rate-limited while their tasks ran to completion; a blanket resubmit would
have burned three billable tasks and left three orphans running. Encode the distinction as a **guard, not a
habit**: refuse to archive or resubmit any record that holds a `video_id` and has not failed.

A client that marks every thrown submit `submission_uncertain` is being properly conservative, but the error
body usually says which kind it is. An explicit refusal ("queue is full") means nothing was created and a
retry is not a duplicate POST; a timeout or a dropped connection genuinely is uncertain and must be
reconciled by querying, never by resubmitting.

**Backoff belongs in the orchestrator.** Polling every pending shot in a tight loop earns a rate limit, and a
rate-limited poll reads exactly like a failed task. Poll shots one at a time with a gap between them, and
back off exponentially on the vendor's limit codes rather than treating them as verdicts.

**A retry must clear the rejected record, or it can never fire.** The client persists the job record *before*
the POST, which is right: an interrupted submit stays recoverable and never silently re-fires. But that means a
submission refused with "queue full" leaves a record behind, and the orchestrator's backoff retry is then blocked
by the client's own duplicate-POST guard. The 503 backoff added after one round had therefore never once worked;
the round it seemed to rescue had been rescued by a hand-archived record. Before each retry, archive the stale
record — only when it holds no `video_id` **and** its error is an explicit refusal; a timeout or dropped
connection is still uncertain and must be reconciled by querying. If the record cannot be cleared, stop rather
than spin. An automatic path that has never run cleanly looks exactly like one that works until nobody is
covering for it by hand.

Assemble with a normalization pass (uniform scale, fps, pixel format; silent tracks padded; loudness normalized), then concatenate. Sample frames from the finished cut and look at them.

### 6. Record

Write down what was produced and, in its own section, record what remains **unverified** — language verdict, billing, any anomaly — rather than burying it in prose. When the user later gives a verdict, write it back and say who judged it.

## Round Isolation

A second run must not overwrite the first. Scripts that hardcode a single `output/` directory will clobber prior clips, job records and masters. Give every round its own subtree and its own shot definitions, and verify the no-argument path still behaves exactly as the first round did.

## Validation Checklist

- Source identity confirmed from raw content, not a summary.
- For a long work: length and chapter gaps measured, cast counted from the text, character bible written by stage
  with each line tagged stated or inferred and cited; no source text committed.
- Blocking exists before any shot prompt: every actor and camera placed, every shot citing a camera.
- Root-structure signatures compared; no two shots sharing a recurring setup have matching signatures.
- Character cards built on neutral grey, one per state; no scene frame used as an identity reference.
- Character stress test run, its pass line named before looking, and its result reported.
- Scene description in English, spoken line in its intended language, stage directions kept out of the line's
  paragraph.
- Character bible byte-identical across all shot prompts.
- Upload directory contains only intended files; credential and metadata scans clean.
- Every hosted image: HTTP 200, expected content type, SHA-256 equal to local.
- Production and prior hosting targets unchanged.
- Task count within what was authorized.
- Every shot's cast list matches who is *visible* in that shot, not just who acts.
- Every retry classified by persisted state first; nothing holding a live `video_id` was resubmitted.
- Before/after state comparisons taken with identical commands, and each call's success asserted before its
  body was read as state.
- Any automated detector used as evidence shown to stay quiet on known-clean input.
- Final cut: expected duration, resolution, fps, audio stream present; peak level checked for clipping.
- Sampled frames reviewed by eye, across time, on the whole frame.
- Superseded keyframes, clips and job records archived rather than overwritten.
- Every no-dialogue shot has a written sound section (voice slot: 无台词) and was checked for open mouths at its
  loudest moments; every shot's language verdict came from a human ear.
- Card sets internally consistent (same garment, hair and ornaments in every view) before any stress test.
- Unverified claims recorded as unverified.

## Regression Tests

These test the skill, not a film. Every rule below was bought with a wasted round, and each is the kind that
quietly stops being followed. Walk them after editing this document, and on any run that feels like it is
drifting back to prompt-stacking.

**T1 — A gate is a stop.** The user answers gate 1 enthusiastically ("这个方向太好了，快做").
*Fail:* proceeding into prompts, images or hosting on that momentum.
*Pass:* treating it as approval of gate 1 only, and asking again at the next.

**T2 — Cast comes from the frame.** A shot is a close-up of a creature sitting in a character's ear.
*Fail:* a one-character cast, so the human arrives with no bible and invented jewellery.
*Pass:* two characters, derived from who the camera sees — including whoever it is attached to.

**T3 — Negatives mirror this shot's decisions.** A shot establishes a bare-wood box and a blank scroll.
*Fail:* carrying only the standing "no text, no modern objects" block.
*Pass:* a per-shot block that names no gilding and no writing on the scroll, because this shot decided those.

**T4 — Some shots must not use the model's motion.** A title card on a blank scroll, or a pure landscape beat.
*Fail:* generating it again with a harder negative prompt after it was invaded by text or by a presenter.
*Pass:* keeping the audio, rebuilding the video locally as a push-in over the approved keyframe.

**T5 — Classify before retrying.** Nine of eighteen tasks come back failed.
*Fail:* resubmitting all nine.
*Pass:* reading each persisted record first, re-polling every one that holds a `video_id`, and resubmitting
only those the server explicitly refused.

**T6 — Sampling is not verification.** Asked whether a clip has burned-in captions or a closed mouth.
*Fail:* one or two frames, then stating what the clip does.
*Pass:* scanning across time and over the whole frame, and otherwise saying only what the sample showed.

**T7 — A detector needs a negative case.** An automated screen flags every frame.
*Fail:* reporting the hits with caveats.
*Pass:* running it on known-clean input first and discarding it when it cannot stay quiet.

**T8 — One remark is not a rule.** The user says a background felt too crowded on one shot.
*Fail:* writing a permanent constraint against background figures.
*Pass:* recording a provisional leaning, and promoting it only on a repeat or an explicit "从今以后".

**T9 — Never claim the ear's verdict.** Envelope analysis shows strong speech-like bursts.
*Fail:* reporting the dialogue as correct, or the language as confirmed.
*Pass:* reporting evidence of speech, and leaving language and content to the user.

**T10 — A failed call is not a state.** A verification command errors and returns no list.
*Fail:* reading the empty payload as "everything was deleted".
*Pass:* asserting success before interpreting the body, and re-taking the measurement before raising alarm.

**T11 — Continuity outranks convenience.** An image pipeline hits a quota mid-round.
*Fail:* finishing the round on a second pipeline because it is available now.
*Pass:* waiting out the reset, or telling the user plainly that the character will change mid-cut.

**T13 — A silent shot needs its sound written.** A shot has no dialogue and a face on screen.
*Fail:* telling the model nobody speaks, in English, and trusting it.
*Pass:* a structured sound section whose voice slot says 无台词, a fixed expression, and frames at the loudest
moments checked for open mouths before anyone listens.

**T14 — A retry clears what blocked it.** A submit comes back 503 queue-full and the backoff fires.
*Fail:* retrying into the client's own duplicate-POST guard, or archiving a record that holds a live `video_id`.
*Pass:* archiving only a record with no `video_id` and an explicit refusal, then retrying.

**T15 — Weak properties do not make shots different.** Four close-ups differ only in frame side and light warmth.
*Fail:* accepting the signature comparison's "no duplicates".
*Pass:* changing scale, subject count or camera side on at least one of them.

## Failure Modes

### Task completes but nothing downloads

The client looks for the asset under `metadata.url` while the API returns `url` at the top level, so the success path raises instead of saving. Accept both:

```python
url = result.get('url') or (result.get('metadata') or {}).get('url')
```

### A generation CLI refuses its own default model

Server-side default models move ahead of installed CLI versions, and the failure reads as a hard 400 telling you to upgrade. Pin an older model explicitly before reaching for a full CLI upgrade; most such CLIs cache the list of available model names locally.

### An image flag swallows the prompt

A CLI flag declared as taking multiple values (`-i, --image <FILE>...`) will greedily consume a trailing
positional prompt as one more filename. The call then finds no prompt, falls back to stdin, and — if you
deliberately closed stdin — exits in well under a second with a message about stdin rather than about the
flag. Terminate the list with `--` before the prompt. A whole batch failing instantly, while calls that omit
that one flag succeed, is the signature.

### A generation CLI hangs forever when driven in the background

Detached from a terminal with its stdout sent to `/dev/null` and its stdin closed, the CLI can block indefinitely —
the process stays alive at near-zero CPU and the batch never advances, which reads like a slow model rather than a
hang. Drive such tools from a script that gives each call an explicit timeout and one retry, passes
`stdin=DEVNULL` deliberately, and writes each call's output to its own log file rather than discarding it.

### Burned-in subtitles

Models add captions when dialogue is requested, and they arrive on some shots and not others. Whichever way the
deliverable goes, the model's own captions are the wrong source: they are inconsistent in style, placement and
wording, and they cannot be corrected without regenerating the shot.

So forbid text, subtitles and watermarks in every prompt regardless, and add subtitles yourself in the assembly
pass, from the script you already hold. Burn them for every shot, in one style, or for none.

Two traps when hunting for the model's own text. It does not only caption the **line** — it will also render
the prompt's **stage directions**, burning a tone instruction like "pleased with himself, warming up as he
goes" into the subtitle band as if it were dialogue. And the text does not only sit in that band: it can
appear as writing on props (a scroll, a hanging banner) mid-frame, and it can fade in partway through a
shot rather than being present at frame 0. So sample several timepoints across every clip and look at the
whole frame, not one frame of the lower third; a bottom-band blur cannot reach a prop in the middle of the
picture. Where the intrusion covers a title card, keep the clip's audio and rebuild its video as a slow
push-in on the approved keyframe — no new billable task, and the card comes out exactly as designed.

### An assembly script that assumes every shot is the same length

A cut whose shots all ran 5 seconds hides a pile of hardcoded 5s: the audio trim, the `-t` cap, the total-duration
assertion, the fade-out start, the review-frame offsets. The next round with mixed 6-12s shots then silently
truncates every longer clip — a 10-second narration loses its second half — and still produces a plausible file.
Derive every duration-dependent value from that clip's own probed length before a round with variable shots.

### Dialogue too quiet in the master

Generated speech can sit near -35 dBFS. Loudness normalization in the per-clip pass brings it up. Check the final peak level afterwards, since a concat re-encode can push peaks to full scale and clip.

### Loudness-normalizing an ambience-only shot

`loudnorm` to a dialogue target is right for a shot with a line in it and wrong for everything else. Applied
to a wind-only or water-only beat it makes ambience as loud as speech; applied to a locally rebuilt shot whose
bed was lifted from a quiet window (peak near -73 dBFS) it lifts the noise floor by fifty-odd dB and delivers
hiss. Drive the choice from the shot's own script entry: `loudnorm` where there is a line, a gentle limited
gain where there is not.

### Extra or missing utterances, and voices that collide

A shot's audio may contain more speech bursts than lines scripted. Count bursts in the envelope, flag the
discrepancy, and let a human judge on listening.

The worse form of this is **two voices overlapping** — half a second of speech landing on top of other speech,
which a listener hears as smeared and unintelligible rather than as an extra sentence. It was found by ear on
an otherwise reasonable-looking clip (2026-09-10), on the run-up to the model reciting a stage direction.

**No envelope marker for it survived testing.** The obvious candidate — an unusually long unbroken voiced run,
since overlapping speech never returns to the floor between phrases — was measured at 50 ms resolution against
clips the user had already judged good, and the good clips scored *worse* than the defective one (3.5 s of
unbroken voice against 2.05 s). Per the rule on detectors, it was discarded rather than reported with caveats.
Burst counts remain worth flagging; overlap does not appear to be detectable this way, and claiming otherwise
would be inventing confidence. This is the sharpest illustration of why the ear's verdict is not optional.

### A recurring on-camera narrator goes stiff

A storyteller who appears in ten shots is where the audience's eye lives, so if every keyframe puts him at the
same chest-up three-quarter angle with the same half-smile and the same one-hand-open gesture, the sequence
reads static no matter how much the story behind him changes. Vary shot scale and gesture deliberately across
his shots — closer and wider, one hand and two, pointing back into the scene and addressing camera — the same
way framing variety matters for any other recurring setup. Design it into the per-shot text; the bible fixes
who he is, not how each shot is framed.

### A blank surface is an invitation to write

A scroll, a banner, a sign left deliberately empty for a title you plan to add yourself is the one place the
model will put text, however firmly the prompt forbids it — and the characters it invents are often malformed.
It happened on the same title shot in two consecutive rounds. Either keep such surfaces out of frame, or plan
from the start to **not use the model's motion there**: keep the clip's audio and rebuild its video as a slow
push-in on the approved keyframe, then composite your own title. Cheap, exact, and it costs no extra task.

### An empty frame is an invitation to populate

The sibling of the blank-surface trap. A pure landscape shot — no characters, nothing to lip-sync — is
where the model will invent a **presenter**: a photoreal human, centre frame, gesturing at camera and
speaking. Observed twice in a row on the same shot (2026-09-09/10). The first attempt produced a modern
fitness influencer in a branded sports bra with a lavalier mic and a smartwatch; the retry, with the prompt
hardened to forbid people, modern clothing, logos, microphones and watches by name, produced the same
composition dressed in period costume instead. Forbidding the *attributes* does not work, because the
attraction is to the **composition**, not the wardrobe.

One contributing cause is worth fixing regardless: a no-dialogue clause that says "every character keeps
their mouth shut" mentions characters, and a shot that has none reads that as licence to supply one.
Build the negative clause from the shot's own cast — when the cast is empty, say the frame contains no
living thing at all, and never use the word "character".

But do not expect that to hold. The reliable remedy is the same one the blank-surface trap gets: **don't use
the model's motion for that shot**. Rebuild it locally as a slow push-in or pan over the approved keyframe
(`zoompan` over an upscaled still), which is exactly what a landscape beat wants anyway, costs no billable
task, and yields precisely the frame that was approved. Take the ambience from the discarded clip's quietest
window rather than its full track, which contains the invented presenter's voice.

### A no-dialogue shot invents its own line

The audio counterpart of the empty frame. Recognize it by the symptom a human listener caught (2026-09-11): shots with no
scripted line speaking, several of them the **same sentence**, unrelated to the story and present in no prompt.
Identical content across unrelated images cannot come from the images; it is the model filling an unassigned
audio slot with a default. Diagnosis and the wording that fixed it are in gate 2. Two things that did not help:
English "no one speaks / no narration / no voice-over" (it named what it forbade), and an English soundtrack given
a positive job with every speech word removed (the mouth still opened).

### A failed call read as a state change

The verification that proves nothing was damaged can itself manufacture a catastrophe. Listing hosting
channels after a deploy, run from a directory without the config file, returned `{"status": "error", ...}`
instead of a channel list; the comparison script read the missing list as *zero channels* and reported that
every channel including production had been deleted. Nothing had happened.

The shape generalizes past hosting: an empty result and a failed call are indistinguishable once you index
into the payload. **Assert the call succeeded before interpreting its body as state**, and take before/after
snapshots with byte-identical commands, flags and working directory — a comparison between two different
invocations is not a comparison. Treat an alarming diff as a suspect measurement until the measurement has
been re-taken cleanly; the sibling rule to "a fetched summary can be wrong about basic facts."

### A detector that fires on everything is not evidence

Running OCR over sampled frames to hunt burned-in captions produced a hit on 100% of them — hundreds of
garbage glyphs read out of rock texture, feathers and dry grass, at `--psm 11` on photographic frames. A
screen with no negative cases has no discriminating power, and reporting its output would have been a false
alarm dressed up as diligence.

Before trusting any automated detector, run it on frames **known to be clean** and confirm it stays quiet.
If it does not, discard it and say so rather than reporting its output with caveats. For burned-in text the
reliable method stays visual: crop the caption band from every sampled frame, tile the crops, and look —
tiling makes 72 frames one glance instead of seventy-two, and the whole frame still needs its own pass for
text on props.

The mirror-image mistake is just as easy: explaining the hits away. When all six no-dialogue shots of a round
flagged as speech-like, they happened to share a bell rack, and "a struck bell reads like a word in this metric"
was offered as the likely cause. It was wrong — five of the six were speech. A detector that fires on a whole class
licenses neither conclusion. Go and get the evidence it cannot give — frames at the loudest moments, checked for
open mouths — and leave the rest to the ear.

### Sampling a frame or two is not verification

Whether a mouth stays closed, whether text appears, whether a character drifts — these are properties of
*every* frame, and one or two sampled frames cannot establish them. A clip can start clean and change halfway.
Report what the sample showed ("the frames I checked show X"), never what the clip does, unless you scanned
across time. The failure is self-similar: having learned it for burned-in captions, it is easy to keep
sampling one frame per clip for mouths and state the conclusion with the same false confidence.

### Proving speech without hearing it

You cannot verify audio directly, but you can gather real evidence. Compute a per-100ms RMS envelope: discrete bursts against a low floor, with a dynamic range above roughly 20 dB, indicate speech, whereas flat ambience stays under about 12 dB. Then sample frames at the burst timestamps and check that mouth shapes actually change between them. Report this as evidence of speech, never as confirmation of language or content.

### The image account runs out of quota mid-batch

A generation CLI billed against a personal plan can exhaust its allowance partway through a batch: the calls
start failing in seconds with a usage-limit message naming a reset time, while earlier calls in the same run
succeeded. It is not an extra charge and not a prompt problem — do not retry in a loop or start rewriting
prompts. Report the reset time, and leave buying more credit to the account's owner.

### Falling back to a vendor's web UI for image generation

When a CLI's quota is exhausted, the same vendor's chat UI in a logged-in browser can still generate — but it is
a **different pipeline, and it shows**. Driving it taught three things worth keeping:

- **Never type a long prompt into the composer, and do not count on paste.** A browser-automation extension may
  deliver `type` but not `cmd+v`, `cmd+a` or `Delete`, leaving a half-typed draft that cannot be cleared. Worse,
  retyping a long non-ASCII prompt by hand corrupts it — single wrong characters slip in and the bible is no
  longer byte-identical, which is the one property it exists to have. **Attach the prompt file itself** and ask
  the model to read it; fidelity then costs nothing.
- **A portrait reference alone will not reproduce an established character.** Asked from the same photo, the web
  UI returned a different person (different hair, different face) from the one the CLI had been producing all
  round. Feeding it one of the already-approved frames as the appearance reference pulled it most of the way back
  — but the costume gained embroidery the bible calls plain, and the render stayed glossier and more dimensional
  than the flat storybook look of the rest.
- So treat a mid-round pipeline switch as a **continuity break, not a convenience**. If the round already has
  approved frames from one path, finish it on that path; waiting out a quota reset is cheaper than a character
  whose robe changes mid-cut.

### `timeout` is missing on macOS

macOS is not a coreutils box. Use your harness's own call timeout instead of wrapping the command.

### Stale `.git/index.lock`

A 0-byte lock file left behind by an interrupted git write blocks every later git operation, silently, for as long as it sits there. Confirm it is 0 bytes, has an old timestamp, and that no `git` process is running, then remove it.

### Video files will not stage into git

Many repositories deliberately exclude `*.mp4` and similar to stop repository bloat, and the exclusion is easy to mistake for an oversight. Check whether the rule is intentional and surface the conflict instead of forcing the add. Large media usually belongs in backup or object storage, not in git history, which cannot be trimmed later without a rewrite.
