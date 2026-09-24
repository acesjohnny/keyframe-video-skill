---
name: keyframe-video-production
description: Use when producing a short keyframe-driven AI video — designing shots, generating keyframe images, hosting them on a temporary public URL, submitting video generation tasks, and assembling the cut. Covers spoken dialogue, character continuity across shots, and stopping for human approval before each irreversible or billable step.
---

# Keyframe Video Production

Take a source scene and turn it into a short multi-shot video, where each shot starts from a generated keyframe image.

The workflow applies to any keyframe-driven video model. It was written and verified against the Agnes video API (`agnes-video-2.5-flash`), which takes a `first_frame` image URL plus a prompt and returns an MP4.

## Fit

Use this when:

- The target is a multi-shot video — a short demo, an episode, or a whole adapted work — in which every shot
  starts from a specific keyframe image.
- The video API needs that image as a publicly fetchable HTTPS URL, or accepts it inline (see gate 4).
- Character or scene continuity across shots matters.
- The user wants to approve the plan before spend, not review a finished artifact.

Do **not** use this for pure text-to-video with no keyframe, editing existing footage, or anything published outward
without a fresh confirmation.

**Any agent, any keyframe-driven model.** Nothing here depends on one agent or one vendor. The agent needs to read
files, run shell commands and make HTTP calls — Claude Code, Codex and Hermes Agent all qualify. The video model needs
a keyframe (first-frame) mode. It was written and verified against `agnes-video-2.5-flash`, with keyframes from an
OpenAI image model and from `agnes-image-2.5-flash`; other image and video models (MiniMax's among them) fit the same
gates, with their own limits to check.

## Verified Capability Notes

Measured on `agnes-video-2.5-flash` in keyframe mode, 2026-09:

- It **does** generate intelligible spoken Mandarin with matching lip movement, directly from a keyframe and a prompt containing the line. No external text-to-speech was needed. The verdict came from a human listening to the output, not from automated analysis. Confirmed four times — most recently on the prompt-language A/B (2026-09-10), where both versions spoke Chinese and the difference between them was quality, not language — and on an 18-shot film (2026-09-10) that also settled a further question: **two distinct voices can hold across one cut** — a teenage girl and a small child, each with its own voice bible, stayed in character over thirteen speaking shots.
- **You can attach a voice sample, and Agnes accepts it** (2026-09-21, `agnes-video-2.5-flash`): pass a short
  clip of the character's voice in `audios` in `reference` mode and name it in the prompt as `<Audio 1>`. The task is
  accepted and completes; in the one A/B pair run so far the listener judged the voice closer to the sample than the
  same prompt without it, and none of the sample's words leaked into the new line. The price is the first frame —
  reference mode will not take one. Recipe under "A voice sample, or a voice description" below.
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
- **Price, checked on the vendor's price page on 2026-09-11:** `agnes-video-2.5-flash` and the `agnes-image-*-flash`
  models were listed at $0 as a limited-time offer, while `agnes-video-2.5` (without Flash) was billed per second.
  Offers end; check the page again before quoting it.
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
- **A clip can cut inside itself.** Asked for a shot, the model sometimes returns two or three shots in one clip —
  a reverse angle, a costume change, a stranger walking in, a different building — so a 5-second request yields
  2 usable seconds. On one 18-clip round (2026-09-14) seven clips did it. Writing **"one single unbroken take from
  one locked camera position; the location, angle and framing stay exactly as in the first frame from the first
  frame to the last"** at the top of the video prompt stopped it: **verified** on the next fifteen tasks
  (2026-09-15), zero internal cuts, including a moving point-of-view walk and a slow push-in. Detect it rather than
  trusting it — see *The model cuts inside a clip* below.
- **But a cut you script is a capability, not a defect.** The unbroken-take rule cures *unscripted* cuts. Asked
  explicitly for three timed shots in one clip, the same model delivers them on time: a 3 s + 6 s + 3 s street
  scene (2026-09-21, `agnes-video-2.5-flash`, **keyframe** mode) cut at 3.08 s and 9.92 s — exactly two cuts, no
  extras — with each line landing inside its own shot. A vendor web portal built on the same models does
  this routinely: three of its clips cut at 3.04/9.00, 3.00/6.21/8.04 and proportionally on a third. Keep the
  unbroken-take sentence for single shots; for a scripted sequence replace it, never combine the two. See
  *Scripted multi-shot sequences* below.
- An API key for the video models may carry **no speech or TTS model at all**. List the available models before
  promising anyone a TTS fallback — and list them again before assuming which *stage* the vendor can serve.
  The same key that runs the video may also carry image models, in which case reaching for an unrelated CLI
  for keyframes buys a second quota, a second pipeline and a model id nobody can verify.
- A generation CLI's image tool may expose **no model selector and no verifiable model id** at the CLI. But the
  output may still carry one: a C2PA manifest (PNG `caBX` chunk) can name the generator and a version.
  Read it before declaring the model unknowable — and still report it as what the metadata claims, not as a
  version you verified, since a manifest's version field need not match a vendor's marketing name.

Treat all of this as a dated observation about one vendor, not a guarantee. Re-check before relying on it.

**A second image vendor is not a drop-in.** Swapping the keyframe generator to MiniMax `image-01` (2026-09-16,
one paid image) surfaced three limits in one attempt, all of which apply to any vendor swap:

- **Keys are regional.** The same key is valid on `api.minimaxi.com` and rejected on `api.minimax.io`. Probe with a
  request that cannot produce output — send an invalid parameter and read the error — rather than burning a real
  generation to find out which host a key belongs to.
- **Prompt length is a hard cap, and truncation is silent.** 1500 characters against a 4136-character keyframe
  prompt; the harness cut it and generated anyway. Keep a short variant per task when a model has a cap, and warn
  in the plan when no short variant exists.
- **One reference image, not a set.** Our keyframes ride on a character card plus a scene card plus a prop card;
  `subject_reference` takes a single image. The result matched neither the approved face nor the scene, and the
  pearl in the prompt never appeared. Character consistency across shots is exactly what the extra references buy,
  so a one-reference vendor is for standalone images, not for a continuity-bound film.

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

### Before anything: agree where the run is driven

Ask once, at the start, and record the answer on the project page. There are three ways to run the same pipeline,
and the person may switch between them mid-project:

| Mode | Who does the work | What the other side does |
|---|---|---|
| **Agent** | The agent writes the storyboard, blocking, prompts and task files, and runs generation, hosting and submission through its own scripts. | The control surface is not touched; it can open the project later and see everything. |
| **Control surface** | The person drives the canvas: writes shots, generates cards and keyframes, approves, submits, cuts. | The agent advises, reviews what came back, and fixes files when asked. |
| **Both** | The agent authors and checks (storyboard, blocking, prompts, cards, self-review); the person reviews, approves and watches progress in the canvas. Each stage names which side dispatches it. | — |

They interoperate because **the files are the product** (see *Driving this from a local control surface*): both
sides read and write the same task files, generated images, approvals and job records, so "syncing" means the other
side re-reading, never copying. What keeps them from colliding:

- **One dispatcher per lane at a time**, across every project on the account — the free tier rate-limits the
  account, not the project. Before an agent-side submission, check the control surface has nothing queued or running
  in that lane in *any* project.
- **A hand-off waits out the spacing.** A probe that submits one task and hands the rest to the other side caused an
  HTTP 429 four seconds later. Wait the full inter-submission gap first, or let one side send everything.
- **The control surface's selected project is shared.** Switching it to inspect another project changed what the
  person was looking at while a video round ran there. Read through files, or ask before switching.
- **Agent-side runners call providers exactly the way the control surface does** — same command, stdin closed, same
  output paths, same result fields in the task file — so either side can pick up the other's work.
- **Before handing a project to the control surface, mark superseded versions** rejected, with the reason, so its
  review queue shows only what is live.
- **Never restart the control surface while any job runs in any project**; a restart marks running jobs
  interrupted.
- **The person can change modes by saying so.** "Don't touch the canvas for now" and "sync it now" are both normal
  instructions; the files make either one cheap.

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

#### Story shape and shot mix, for a film meant to be watched (from one reference, 2026-09-23)

A capability test only needs a beat. A film meant for an audience needs a shape, and the shape can be planned to
fit what the models do well. These points come from taking apart one widely shared AI short (13.5 minutes, about
147 detected cuts, median shot 4 s) — one data point, not a measured rule; treat the ratios as a starting point.

Before the shot table, write down two things and show them at gate 1:

- **One thread-through object or idea** that every sequence returns to (in the reference: rice — a well that
  gives rice, a grain picked up and called theft, a leaking jar, the last line "come and eat"). A shot that does
  not touch it needs a reason to exist.
- **One real-world landing** for the ending — a custom, a saying, a thing the viewer still sees today — so the
  film ends with "so that's where it comes from". That is the reason people pass it on.

Then check the shot table against these, which also steer around the models' weak spots:

- **Front-load the spectacle.** Crowds, creatures and fire sat in the first ~17% of the runtime; after that,
  almost none. The opening buys attention; the rest is carried by lines and faces.
- **Dialogue as single-person close shots, cut every 1–3 s, one short line each.** The steadiest shot a keyframe
  model makes, and short takes keep lip-sync short. This is the same grammar as *speak, then see* in gate 1.5.
- **Many inserts and empty frames** — hands, a sack of grain, feet in snow, a reflection, fruit falling into snow.
  They carry the emotion and hide continuity drift between character shots.
- **Almost no action.** The one attack in the reference is two brief shots of the creature, with mist covering
  the contact.
- **Three leads, each with one mark you can see at thumbnail size** (white beard and robe; mud-spotted face;
  white-ribboned hair). Put the mark on the character card (gate 2.5).
- **Colour as meaning.** One cold base palette; the warm colour reserved for one idea (fire, food), and the ending
  allowed to turn warm. Write the rule into every keyframe prompt's style block.
- **Chapter title cards** in large calligraphy over an empty frame split a long film into parts, and make a jump
  in look between parts read as intended.
- **An emotional beat every one to two minutes.** Length is not the problem; a long stretch with nothing new is.

To study a reference yourself: detect cuts with ffmpeg's `select='gt(scene,0.3)',showinfo` (at the default log
level — `-v error` hides showinfo's lines), take one frame from the middle of each shot into a numbered contact
sheet, and read burned-in subtitles by cropping the subtitle band and running OCR on it. For a film that has
subtitles burned in, OCR is more accurate than speech recognition and needs no ASR model.

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

### 2.4. Previs a camera move before generating it (reported, not measured here)

A walkthrough of an agent driving Blender through an MCP connector (watched 2026-09-16) makes the case for blocking a
**camera route** in 3D before any generation, for shots whose geometry is the hard part: a 10 cm protagonist running
across a carpet, sucked up a vacuum tube, the camera pulling back to reveal the product, all in one unbroken take. The
route is checked and fixed in the grey-box stage — where a turn is awkward, when the product appears — and only then
does the previs render plus character and product sheets go to a video model. Three details worth copying:

- **Deliberately crude figures.** Boxes and spheres; dancers were plain blocks. The stage answers where people stand,
  which way they face, when the camera passes them — nothing else. But keep the action relationships that the shot
  actually needs: a handshake, climbing an obstacle, a hand reaching a specific object.
- **Colour-code facing.** Green face = front, red face = back, so a camera swinging around the group tells you whether
  it lands on faces or backs.
- **Say what each reference governs, and what must not be inherited.** Previs video: camera path, positions, timing.
  Character sheet: face and wardrobe. Scene image: environment and style. Prompt: action, performance, sound. Then
  state explicitly that the grey mannequins, the blocking colours, the sheet's studio background and its neutral
  standing pose are **not** to appear in the film. This is the same failure as *A continuity reference that is too
  strong gets copied* above, handled by division of labour instead of by dropping the reference.

Unverified here: this workflow needs a video model that accepts a **video** reference (the previs) rather than only a
first frame, so it does not apply to a keyframe-only route without checking what the model takes.

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

**A continuity reference that is too strong gets copied, not followed.** Attaching an approved keyframe so a new
shot stays "in the same room" can hand back that keyframe almost unchanged. A point-of-view opening meant to start
wide at the doorway, with the character small across the room, came back (2026-09-15) as the approved medium back
view it was given for continuity — the character filling most of the frame height, leaving the camera nowhere to
walk. When the new shot must differ in **scale or distance** from an approved frame, drop that frame from the
references, keep the character and scene cards, and state the scale in the prompt as a proportion of frame height
("she occupies about half of the frame height").

**A reference-film still brings its set and its faces with it.** A still attached "for composition and lighting
only" was reproduced with its whole set dressing — display stands, banners, lanterns — and a tight profile still
came back with the reference actor's face rather than the approved character's. When replicating a reference is
the goal, that is the fastest route to it; when it is not, it is a leak. Either way, check the result's face against
the character card, and expect a room built from a still to break continuity with earlier shots built from the scene
card: rebuild the scene card from the new look before continuing, rather than letting two rooms alternate.

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

Keyframe mode needs each image where the video API can reach it. Pick the route before generating, because it
decides whether this gate exists at all:

| Route | Hosting needed? | Status |
|---|---|---|
| The image model returns a hosted URL (`agnes-image-2.5-flash`, `response_format: url`) and the same URL goes to `agnes-video-2.5-flash` | No | **Verified** 2026-09-11 |
| The video API accepts the image inline as a Base64 data URL (MiniMax's first-frame parameter, per its docs) | No | Vendor documentation only |
| A temporary static host with an expiry — a Firebase Hosting preview channel, or an object-storage bucket with a signed URL | Yes | Firebase **verified** across six rounds; object storage not tested |
| A local HTTP server behind a tunnel (Cloudflare quick tunnel) | Yes | **Verified** 2026-09-11 |

What the two verified 2026-09-11 routes taught:

- **A provider's hosted URL skips this gate but not the evidence.** The Agnes image URL carried no signature or
  expiry parameter and was still served after the video finished, but its retention is undocumented. Download a
  copy the moment it is returned and hash it; the copy, not the URL, is the archive.
- **A local server alone is unreachable.** `localhost` and LAN addresses mean nothing to the vendor's servers, and
  most home connections have no public address. A tunnel supplies one. Bind the server to `127.0.0.1`, serve a
  directory containing only the keyframes, and tear both down when the task completes.
- **Verify reachability from outside, not through the local resolver.** The first attempt failed its own pre-check
  because the machine's resolver would not resolve the freshly created tunnel hostname; it looked like a dead
  tunnel. Resolve through a public DNS server, fetch through that address, compare the hash, and only then submit.
- **The vendor fetched the image once.** The server log showed a single request about fifteen seconds after
  submission. Keep the URL alive until the task completes anyway — that is what the vendor's documentation asks.
- **Check that the route works where the user is.** A route that works from one network can fail from another:
  some DNS servers, UDP, or a provider's storage domain may be unreachable. Test it from the user's network before a
  round depends on it.

For a temporary static host, Firebase Hosting preview channels work well, since they carry a native expiry. Use a **new, dedicated** channel with a short expiry rather than reusing an earlier round's — prior job records still cite those URLs, and overwriting them destroys that evidence.

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

## Driving this from a local control surface

The workflow above was built as a chat-driven sequence, then wrapped in a local web canvas so a non-programmer can
run it. What the wrapping taught, in the order it bites:

- **The files are the product; the page is a view.** Every piece of state already on disk — one task yaml per asset
  *version*, one folder per generation round, one edit list per cut — stays the source of truth. The page reads and
  writes those files and owns nothing itself, so the same project is equally workable from a terminal, from an
  agent, or from the page, and a half-finished session leaves no state trapped in a browser.
- **Plan, confirm, run — with the plan spelled out.** Every provider call goes through a written plan the human
  confirms: how many calls, how many seconds, what it costs, what becomes public, and what will be skipped because
  it already exists. Re-validate at confirm time and refuse if the underlying files changed since the plan was
  drawn; a plan is a claim about the world, and the world moves.
- **Persist the job record before the call, not after.** Restarting the service marks anything that was running
  *interrupted* rather than lost; resuming re-plans and re-confirms. A record that already holds a provider task id
  is polled, never resubmitted — the same guard as gate 5, now enforced by the thing that does the submitting.
- **Bill what was produced, not what was planned.** Three failed attempts against a rejected key were recorded as
  spend because the ledger trusted the plan (2026-09-16). Write the amount from the run's own result, once per job,
  and let a failed run record zero.
- **Detect credentials; never hold them.** The page checks whether a CLI is logged in, whether an environment
  variable is set, whether a credentials file exists — and shows only that, plus the command to fix it. No key is
  read, echoed, stored in the repo, or accepted through a form. Say plainly that an environment variable needs the
  service restarted before it is visible.
- **Offer only what is actually wired.** List every model a project might use, but mark which ones the tool can
  really drive and which only record a choice, and refuse the unwired ones at plan time with that sentence. A
  greyed button with a reason beats a run that dies halfway.
- **Stamp ownership on generated artefacts.** An edit list produced by the generic renderer carried no mark, and
  ownership was inferred from an "edited in" field — which the editor rewrites on every save, so an edited copy was
  handed to the wrong renderer and crashed (2026-09-17). Write an explicit `renderer:` (or equivalent) field that
  survives editing, and fall back to structure, never to authorship strings, for older files.
- **Relative paths belong to where the file lands.** The first auto-generated edit list computed its clip paths
  against the default folder while being written somewhere else, so the renderer found nothing. Compute against the
  real destination.
- **Cut from the speech, and hand the cut onward.** The first assembly is generated from each clip's own
  word-timestamped speech span (see *A silence detector is not a speech window*), and the same edit list exports to
  OpenTimelineIO and Final Cut XML so the cut can be finished in a real NLE instead of being re-matched by hand.
- **Order the surface the way the pipeline depends.** Cards before keyframes, keyframes approved before video,
  video before the cut. Show, per shot, exactly which referenced card is still unapproved and make that line the
  button that fixes it; offer a default reference-sheet template per asset kind (multi-angle character turnaround,
  same-light scene pair, three-view prop) so a first card is one click rather than a blank prompt box.

## Lessons from a narrated teaching series (2026-09-16/17)

A three-episode classroom series — one recurring on-camera teacher, a flat watercolour picture-book look, Mandarin
narration, burned-in subtitles — was produced end to end through the canvas: 3 episodes, 44 model tasks across
11 rounds, 7 shots reused from an existing slide deck. What it added to this document:

### The keyframe's style does not carry into the video

Every keyframe prompt asked for flat watercolour, and every keyframe came out flat watercolour. The **video** prompts
said nothing about style, and the clips drifted: skin gained shading and highlights, painted backdrops turned glossy,
and on two shots the model cut away after one or two frames into a **photographic** version of the same room. A
reviewer caught the milder drift by eye on a finished cut; per-second contact sheets (keyframe, 1 s, 4 s, 7 s) made it
unmistakable. Adding one sentence to the video prompt, straight after the unbroken-take line, fixed it:

> The whole clip stays a flat watercolour children's-book illustration exactly like the first frame — soft paper
> texture and painted colours; it never turns into a photograph or live-action footage.

**Verified** on 14 teacher shots across three episodes (2026-09-17): every regenerated clip held the look, including
the ones that had turned photographic. Treat the style lock as a standing block, byte-identical in every video prompt
of a stylised film, exactly like the character bible — the image prompt's style clause does not transfer.

### Lines the model mispronounces, and when to stop re-rolling

A human listened to every dialogue clip. Three failure shapes recurred, each on more than one attempt:

| Line feature | What happened | Fix that held |
|---|---|---|
| Reduplication (「很久很久以前」) | a syllable swallowed, **twice in a row** on the same line | rephrase without the repeat (「好久以前的時代」) |
| Question word at the very end (「……甚麼？」) | sounded foreign-accented, **three rounds running** — a stronger voice block helped the rest of the line but not the last word | rephrase the question so it does not end on that word (「人和狗是怎樣互相幫忙的？」) |
| Polyphonic character (「種子」) | read with the wrong tone | swap to an unambiguous word (「穀粒」) |

After **one** repeat of the same fault, change the line rather than the dice. Screen new lines mechanically before
gate 2: no reduplicated characters, no line-final question word, no polyphonic character where another word will do.
A local speech recogniser helps as a tripwire (it transcribed the swallowed line as 「很久久久」 both times) but never as
the verdict — it also turned correct lines into near-homophones (「穀粒」→「古力」, 「馴服」→「巡浮」).

### A voice sample, or a voice description

The video API has no voice id, and keyframe mode will not take an approved recording — only reference mode takes a
voice sample (below). In keyframe mode what can be held constant is the voice block, so audit it: extract the `VOICE:` paragraph from every submitted prompt across every round and compare
hashes. The audit found three shots of the first episode still carrying the pre-fix block after the rest had moved on.
And say plainly that **every regeneration is a new recording**: lines already approved by ear must be listened to
again, even when the only change was a style sentence.

A stronger voice block did help: naming the speaker as a native Mandarin speaker "like a primary-school Chinese
teacher on national television … no foreign or non-native accent" removed most of an accent a listener had flagged.

**Checked against the vendor's own model page (2026-09-21):** `agnes-video-2.5-flash` has **no** voice, speaker,
timbre, `voice_id` or TTS parameter. The only audio input is `audios` — up to three reference clips, addressed in the
prompt as `<Audio N>` — and it exists **only in `reference` mode, which rejects `first_frame`**. So a pinned voice
costs the keyframe as frame one; that trade is the whole decision. A vendor portal built on the same models offers
"upload a 5-second sample" and "generate a timbre from a description" as reusable voice ids. The vendor's only
documented example uses `audios` for rhythm and ambience ("以 `<Audio 1>` 的节奏和环境氛围作为参考"), never for a voice.

**Tested once (2026-09-21, flash, one A/B pair):** a 5 s clip of the lead's own voice (cut from a finished shot,
normalised to -18 LUFS, hosted on its own 24h channel) went in as `<Audio 1>` with a new line the sample never says,
and the prompt told the model to use it "only as the sound of his voice: do not play it, and do not say any of its
words". Control B was identical minus the paragraph and `audios`. Flash accepted it; ASR heard the new line in both
and none of the sample's words (tripwire passed); **the listener judged A closer to the sample than B**. A rough
pitch measure did *not* show it (A 120 Hz, B 119 Hz, sample 131 Hz) — the ear caught what the number missed, so do
not use pitch as the verdict. Costs seen in the same pair: reference mode re-framed the keyframe (close-up became
a medium shot) and drifted the face younger and more retouched. One pair is a lead, not a rule: before a whole film
leans on it, repeat it on a second character and a second line, and weigh the lost first frame shot by shot — a
voice sample suits a talking head whose framing can drift, not a shot whose composition the blocking fixes.

**How to attach a voice sample:**

1. Cut 4–6 s of the character speaking alone — from an approved clip, or a recording the user supplies — with no
   music or second voice. Normalise it (`loudnorm=I=-18`) and export mp3.
2. Host it like a keyframe: its own Firebase preview channel, 24h expiry, **its own confirmation** (it is a deploy),
   then check HTTP 200, `audio/mpeg` and SHA-256 against the local file.
3. Submit in reference mode — no `first_frame`; the approved keyframe goes in `images` as a style/identity reference:

   ```json
   {"model": "agnes-video-2.5-flash", "mode": "reference", "seconds": "5", "size": "720P",
    "aspect_ratio": "16:9", "n": 1,
    "images": ["https://<channel>/S15.png"],
    "audios": ["https://<voice-channel>/voice_C1.mp3"],
    "prompt": "... VOICE: he speaks in exactly the voice of the man speaking in <Audio 1> — the same timbre, pitch, speaking rate and accent. Use <Audio 1> only as the sound of his voice: do not play it, and do not say any of its words. ..."}
   ```

   Keep the written voice sheet in the prompt as well; the sample adds to it, it does not replace it.
4. Check: ASR must hear the new line and none of the sample's words (tripwire); frame the output against the
   keyframe for framing and face drift; then **the user listens** — the agent cannot judge a voice, and a pitch
   number is not a verdict. Record the verdict and who gave it.
5. Up to three `audios` per task, so a two-speaker shot can carry `<Audio 1>` and `<Audio 2>` — untested; A/B it
   before relying on it. The same page lists a `seed` parameter this
pipeline has never used, and confirms **4–12 s** per clip: a portal clip planned at 15 s came back at 12.26 s.

**Keep the voice sheet to the voice.** A sheet that ends "only he speaks in this shot; everyone else stays silent"
carries a per-shot staging rule inside a per-character block, and copied verbatim into a two-speaker clip it
contradicts itself. Put timbre, pitch, rate, accent and delivery in the sheet; put *who speaks in this shot* in the
shot.

### Card gate and card shape

- **Do not skip gate 2.5.** In this run the cards were generated and immediately used as references for shot
  keyframes without being shown; the user caught it. Cards are approved first, then stress-tested, then used.
- **Define every visible attribute before the card.** The bible said nothing about trousers or shoes, so the model
  invented them; adding them after the cards left the cards out of step with the bible and forced a v2.
- **A mark the model never draws is not an identity mark.** Dimples were in the bible; neither the card nor any of
  eight probes showed them. Record that, keep or drop it by the user's call, and do not count it in the pass line.
- **Colour drifts between views.** The back view came out with a pinker cardigan than the front; regenerating it with
  the front view attached as a reference and the colour pinned in words fixed it.
- **A canvas character is one card per version**, not one per view: merge front, back and face into a single
  turnaround image, keep the per-view tasks aside, and approve through the canvas's own review call.

### Reusing already-approved art

Scene shots whose picture already existed in an approved slide deck used those slides as first frames directly —
no new images, and the film matched the classroom material. It worked for all seven such shots. A slide drawn for a
different purpose may carry what a film shot must not: an eighth candidate had two cartoon children and pseudo-writing on a
tablet, so it was redrawn as an object-only still with abstract marks.

### Rate limits across projects

Two canvas projects submitting at once drew **HTTP 429 `rate_limit_exceeded`** ("free users") on the second one's first
task. The body is an explicit refusal and the record held no task id, so nothing was built; the canvas nonetheless
marked it `submission_uncertain` and stopped the batch. Archive such a record, then resubmit the same authorised
count. Better: submit one project's batch at a time. The error text invites a paid upgrade — that is the account
owner's decision, never the agent's.

### Hosting reuse across rounds

Later rounds reused the first round's preview channel instead of redeploying: before each submit, fetch every image
anonymously again and compare hashes, write the result into the new round, and check the channel's remaining life.
A round that must finish before a channel expires says so when asking for approval.

### Post-production details that bit

- **Speed variants**: speed the picture with `setpts=(PTS-STARTPTS)/k` and the sound with `atempo=k` (pitch kept),
  scale the subtitle times by the same factor, and leave title cards, still push-ins and the closing hold at normal
  speed. Keep each speed as its own edit-list version and master name. **Check the renderer really uses `atempo`**:
  one applied `setpts` to the picture but cut the audio to the new length with `atrim`, so a 1.25× master silently
  dropped the last fifth of every line (2026-09-21). And in ffmpeg put `-t` **before** `-i` when cutting the body off
  a master to speed it — after `-i` it limits the *output*, which a sped-up body never reaches, so the whole master
  including its title card got sped and the card then appeared twice.
- **`alimiter` normalises by default.** Its `level` option defaults to on and lifts the whole mix to the limit;
  pass `level=false` wherever a limiter is meant only as a ceiling, then re-measure every master.
- **Wrap subtitles at punctuation.** A character-count wrap split 「市集」 across two lines; break after the last comma or
  enumeration mark in the line when one exists past the first third.
- **Traditional-Chinese checkers over-correct.** OpenCC `s2hk` rewrote 「群」 as 「羣」; keep a short allow-list of the
  house forms a project has decided on, and fail on everything else.
- **A brand mark belongs in the edit list, not in the prompts.** Generators are told to produce no logos, so the
  mark is overlaid in assembly: put `watermark` in the EDL (overlay PNG, whether to skip the title card) and fail
  loudly if the file is missing. Overlay a *still* PNG with `-loop 1` on its input — without it the overlay is a
  single frame and vanishes after the first frame. Gate it with `enable='gte(t,<title_end>)'` so it does not sit on
  the title card, and render the watermarked cut under its own master name rather than over the clean one.
- **Never write over a versioned file.** A new edit list was saved under a tag that already existed and destroyed a
  1.5× speed variant (not in git). It was rebuilt from its 1.25× sibling and proved identical against the rendered
  timeline, but the rule is simpler: check the name is free and fail if it is not.

## Lessons from driving the whole pipeline through a console (2026-09-18/19)

The earlier runs had an agent writing prompts and task files in the repo while a canvas displayed them. Moving
authoring into the console surfaced a different class of problem: not "did the model do a good job" but "does
the person driving it ever know what is going on".

### Compose prompts from the project instead of asking for them

A shot's image prompt is not creative work that has to start from a blank box: the project already holds the
staging, the sheets of everyone in frame, the props, the line and the house style. Assemble it and let the user
edit. The assembly is deterministic — no model call, no cost, no waiting — so a task dialog can always open
filled in. Shape that held up:

    image  【frame】staging + camera move
           【<name> · copy verbatim】the character sheet, **unedited**
           【<scene> · copy verbatim】the scene sheet
           【<prop>】each prop in frame
           【style】the project's one style sentence
           【never】no text, letters, numbers, labels, subtitles, watermark, signature, logo; no extra fingers
           【not in this shot】only this character, or nobody at all

    video  the unbroken-take sentence
           the style lock, **repeated here** — the keyframe's style does not carry over by itself
           the scene in prose, then what happens, then the camera
           VOICE: the voice sheet, **unedited**, and who is the only speaker
           the line, marked "say exactly this, do not change or add words"
           no subtitles, no on-screen text, no watermark, no logo

Two fields are quoted **verbatim and never paraphrased**: the character sheet and the voice sheet. Paraphrasing
either is how a face or a voice drifts between shots, and the drift is invisible until the clips are side by side.

### A voiceover shot needs the opposite instruction

The composer's first version saw "this shot has a line and a character" and helpfully added *"…is about to speak;
face and mouth completely unobstructed."* The shot it did that to read: *"voiceover narration, the old man has his
back to camera, no lip sync."* Exactly backwards, and it would have produced the single most common defect in this
pipeline: the video model lip-syncs whatever face it can find whenever there is speech on the track.

So classify the shot before composing. When the staging says voiceover, narration, back to camera or no lip sync,
invert every instruction: the image prompt says the mouth does not move, the video prompt says the person on
screen does not speak while the line is narrated over them. Keyword matching on the staging text is enough to
catch it, and it is worth doing because the failure is silent — the clip looks fine until someone notices the
mouth is moving to words the character is not saying.

### Generated line hygiene can be checked, not just hoped for

The mispronunciation table earlier in this file was learned one re-roll at a time. Most of it is mechanical, so
check a draft before anyone spends a video call on it: reduplication in both shapes (`AA` like 「看看」 and `ABAB`
like 「很久很久」), a line-final question word, a line long enough that the reading drifts, a character sheet too
thin to hold a face. Report these as advice, not as a gate — the person may have a reason — but report them at
the moment the draft appears, not after the clips come back.

### A model's draft must never write the project directly

When a text model proposes a whole shot list, the tempting shortcut is to write it straight into the project
files. Don't. Write it to a disposable draft, show it per shot with a checkbox, and on approval push it through
**the same validated save a hand edit uses** — same id rules, same "this scene does not exist" refusal, same
"you cannot delete a shot that already has a keyframe". A draft is then always discardable, applying is never a
special path, and a bad generation costs nothing but the click that throws it away.

### The console has to say what it is doing

Three separate reports of "it is stuck" this week were all the console failing to narrate work that was in fact
proceeding normally:

- **A long job with no progress line reads as a dead button.** A two-minute storyboard call showed nothing at
  all, because the page read job state from an endpoint that does not carry jobs. Show the state, how long it
  has been running, and what happens when it finishes; refresh the page yourself when it does.
- **"Queued" explains nothing.** Say what it is waiting for: "one image job at a time; waiting for «…» to
  finish". Otherwise a correct queue looks like a hang, and the user starts clicking.
- **A batch action must count the work that does not exist yet.** "Generate all keyframes" looked only at tasks
  already created, so on a project with one hand-made task out of twelve shots it reported *everything is
  already generated* — while eleven shots had no task at all. A batch over a pipeline stage covers items not yet
  started, items started but incomplete, and items done; say how many fall in each bucket before acting.

### Small things that make a console honest

- **Multi-select lists need select-all and a live count.** A column of checkboxes with no count and no select-all
  is a manual chore, and the primary button should state what it will do with the selection
  ("Create task (with 3 reference images)"), so it is not a leap of faith.
- **Confirm that an approval landed.** Approve re-rendered the whole dialog, the page jumped, and it read as if
  nothing happened. A state line at the top and a brief confirmation on the click are enough.
- **Match the script the project is written in.** A composer with hard-coded Traditional labels emitted half
  simplified, half traditional prompts on a simplified project. Detect from the project's own text.
- **Never store a secret where the code lives.** Keys belong outside the repo, mode 600, with the shell's own
  environment variable winning over the stored copy, only the last four characters ever echoed back, and nothing
  written to a log. And whatever detects "is this configured" must read the same place the provider reads, or
  the panel says ready while every call fails.

### Detection that reads a path owned by another tool

A blanket rename across a module (`credentials` → `keystore`) rewrote a *data string* along with the code, so
the check for the Claude CLI's login looked for `~/.claude/.keystore.json`. The user logged in repeatedly and
the panel kept saying "not logged in". Any string naming a file another tool owns is data, not code: keep those
paths in one table and pin them with a test that asserts the exact value.

Related: detecting an OAuth CLI by "the command exists and its credential file exists" cannot see an **expired**
session. A logged-out-by-timeout CLI passes the check and fails at call time. Say so in the UI rather than
implying a green check means the next call will work.

## Scripted multi-shot sequences (2026-09-21)

A 28-shot comedy was generated one shot per clip, the way this document describes. A vendor web portal built on the
same models packs a whole beat — three or four shots — into one clip instead, each with its seconds, shot
size, camera and landing frame. One controlled test on this pipeline showed the capability holds here too, and that
the portal's writing style and this document's blocking fix each other's weak point.

### What the model does when asked

Same street beat, same first frame, same lines, one variable changed:

| | One clip per shot (S14/S15/S16) | One scripted clip, 3 s + 6 s + 3 s |
|---|---|---|
| Generations | 3 | 1 |
| Cuts | none, by design | 3.08 s and 9.92 s — exactly two |
| Lines | one per clip | all three, each inside its own shot |
| Reverse angles | both close-ups show the same skyline — a cheat | shot 1 shows Pudong behind them, the reverses show the Bund opposite — geographically right |
| Sound across the cut | joins built in assembly | continuous; her line started 0.5 s before the cut to her, over his close-up, and his mouth stayed shut — a clean reaction shot |

The economics matter as much as the craft. Packed by location, 28 shots are about **nine** generations, and on a free
tier where one submission can wait twenty minutes for a queue slot, that is most of a day.

### What it gets wrong, and why blocking fixes it

**A person appears twice.** "Her shoulder soft in the foreground" without a side produced *two* identical women, one
either side of him. **Screen direction flips.** Shot 1 had her left and him right; the reverse over his shoulder put
him left and her right — the camera crossed the line. The portal's own clips show the third failure this document
already guards against: a line spoken while the speaker's face is cropped out of frame.

All three are geometry, and the blocking pass already has it. So a scripted sequence is written *from* the blocking:

- **Fix screen direction once, above the shots**, from the actors' positions: "THE WOMAN is always on the left of
  frame, THE MAN always on the right; the camera never crosses to the other side of them."
- **Name the shoulder and the edge** in every over-the-shoulder: "behind her; only the back of her head and her
  right shoulder, at the LEFT edge of the frame; nobody at the right edge."
- **Count people**: "each person appears exactly once in every shot", mirrored in the negatives ("no second woman,
  no duplicated person, never the same person on both sides of the frame").
- **Run the cone and mouth-orientation checks per sub-shot**, not per clip. A speaker off-frame is a failure whether
  it happens in shot 1 or shot 3.

### Shape of the prompt

    This clip is N shots cut together, exactly as timed below. Cut only at <t1> s, <t2> s; no other cuts.
    Each shot picks up exactly where the previous one landed.            ← replaces the unbroken-take sentence
    <style lock>
    SCREEN DIRECTION — fixed for all shots: …                          ← from the blocking
    [SCENE] … [CHARACTER · COPY VERBATIM] …
    SHOT 1 (0–t1 s) — size, height, camera, who is where, what happens. Lands on: …
    SHOT 2 (t1–t2 s) — … picks up from that landing. Lands on: …
    VOICE (<name>): <voice sheet, voice only>                           ← one per speaker
    DIALOGUE — each line spoken only inside its own shot, by the person named. Say exactly these words.
    Shot 1 — <NAME>:
    <line>
    <no-text clause>

The **landing → pick-up** chain is the portal's real craft: each shot says where it ends, the next says it starts
there. The lines stay in their own block, one per shot, as this document already requires — the portal inlines them
among the stage directions and a speech recogniser found no stage direction read aloud across its three clips, which
is weak counter-evidence to that rule, not a reason to drop it.

### When to use which

Use a scripted sequence for **one continuous beat in one place** — a conversation, a reveal, a walk. Keep one clip per
shot where a single shot must be redone alone (a sequence is regenerated whole), where the keyframe must govern
every shot (only shot 1 has a first frame; later shots carry identity by text), and for every shot that needs the
unbroken take. Sequences are capped by the 12-second clip limit; a beat that needs more is two sequences.

**Status — verified 2026-09-21 on two clips of the same beat.** The first, written without the geometry, showed both
failures above. The second changed only the wording in this section — screen direction stated once, the shoulder
and frame edge named, one person per shot, the negatives mirrored — and both failures were gone: one woman, at the
left edge only; her left and him right in all three shots; cuts at 3.12 s and 9.42 s against a plan of 3 and 9; every
line inside its own shot. The listener judged the two voices acceptable. Two clips of one beat is a strong signal,
not a law — re-confirm on the first real round.

### Running a queue you do not control (2026-09-20/21)

- **The free tier's queue has hours.** One afternoon: four hours, 48 probes, all `video_queue_full`. 03:00 the next
  morning the first probe got in; between 06:15 and 07:33, twenty clips completed. Schedule unattended rounds for the
  early morning and probe with one real submission rather than hammering a full queue with the whole batch.
- **A probe and a dispatcher are two senders.** A probe submitted shot 1 and handed the rest to the canvas, which sent
  shot 2 four seconds later: HTTP 429. The dispatcher's 61-second spacing only counts what the dispatcher sent. Wait
  out the spacing before handing over, or let the dispatcher send the probe too.
- **A record on disk is not a submission.** Job records are written *before* the POST, which is what makes an
  interrupted submit recoverable — so counting record files counts attempts. Count records that hold a video id.
- **A dispatcher with a deadline needs feeding.** The canvas stops a video job after two hours. At twenty minutes per
  queue slot that is three clips, then `failed`. Re-confirm when it stops; records that hold a video id are polled,
  never resubmitted, so re-confirming is safe.
- **Changing API keys does not change the queue.** A second key authenticated fine and got the same
  `video_queue_full`. Its billing endpoint reported a payment method and an open limit, yet a non-flash submission
  was refused with `insufficient_user_quota, remaining $0.000000`. Only a submission tells you the balance; the
  OpenAI-compatible billing endpoints return boilerplate.

### A detector that never fires is not evidence either

The shot-change scan found zero internal cuts in 28 clips. Before believing it, it was run on two clips joined end to
end — one hit, at the join — and on a single clip — none. A subtitle screen that also reported zero was given a frame
with text drawn onto it and **missed it**: night footage puts more bright pixels in the lower third than a caption
does. It was discarded and the frames were judged by eye. A silent detector needs a positive control exactly as a
noisy one needs a negative control.

## A narrated film with no presenter, and TTS narration (2026-09-21/23)

A 12-shot mooncake-history short was first built around a real person as on-camera presenter, then rebuilt with **no presenter at all**: watercolour illustrations animated as silent clips, narration generated separately by a text-to-speech API and laid under them locally. The rebuild is now the default shape for this kind of explainer, and it removed four defects at once.

### Why the on-camera presenter was dropped

The presenter version passed every gate — identity card, stress test, face probe — and the owner still judged the finished cut incoherent. Four causes, and none of them is fixed by a better prompt:

- **One voice per clip.** A video model with no voice id casts a slightly different speaker in every clip; six speaking shots were six people.
- **The face drifts exactly where the shot is hard.** Ordinary shots held; a single clip asked to orbit *and* change era *and* change costume came back as a different man, and reference mode drifted the face again.
- **A costume change mid-episode** makes the audience re-identify the presenter, on top of jumping between five locations.
- **The presenter interrupts the story.** Legend and evidence already have pictures; a person stepping in every few seconds to say one line pauses them.

Default to no presenter for explainers. Keep the identity assets — they cost nothing to keep and the decision may reverse.

### Narration as its own track

Generate the picture and the voice separately, and mix locally:

- **Picture**: every clip is generated as a *no-dialogue* shot (Chinese prompt, structured sound section, fixed expressions). Nothing on screen ever has to lip-sync, so the stolen-mouth failure cannot occur.
- **Voice**: a TTS API returns one file per line, with a chosen voice id, fixed across the whole film. Line length is known before the shot list is finalised, so shot durations are planned from the audio rather than negotiated with the video model.
- **Cost of a rewrite** drops to a few hundred characters of TTS instead of a regenerated clip, and subtitles are burned from the script itself, with speech recognition kept only as a tripwire.

Measured on one vendor (MiniMax `t2a_v2`, 2026-09-23): 303 system voices (45 Mandarin, 6 Cantonese, the rest across a dozen languages), six models, eight emotion values, speed/volume/pitch, mp3 or wav. Two vendor limits worth writing down: **48 kHz was rejected** (32 kHz accepted), and **voice cloning was forbidden on the account** while "design a voice from a description" worked. Treat these as one dated observation, not a guarantee.

### Two audio-editing rules that cost a re-render each

- **Speech-recognition word times are a tripwire, not an edit point.** Cutting narration at the reported end of the last word clipped 「…今天这样的」; the reported end ran early. Take the head from the first word minus a margin and keep everything to the end of the file.
- **Never silence-trim the tail of a narration line.** An unstressed final particle sits below a -45 dB threshold and gets eaten. The same lesson in reverse: trimming the head by the recogniser's first-word time ate 「所以」 at the start of a line; trim the head by silence detection instead.
- **Leave the last word clear of the next transition.** A crossfade over the tail fades the final syllable out even when the audio is intact; pad each segment so speech ends at least ~0.9 s before the join.

### The model can return a partly filled frame

Several clips came back 1280×720 with the picture occupying only 1178×666 or 1172×660, black to the right and below, each clip different. On hard cuts the mismatched borders flash like a slide transition. **Run a black-border detection over every source clip before assembly** (intersection of `cropdetect` across the whole clip), crop to the content and scale to cover. Checking the finished master is not enough — measure each source.

### Transitions: local, and fewer than you think

Thirteen designed transitions (portals through a mooncake, a yolk becoming the moon, a torn-paper wipe, a moon-locked montage, an iris through a magnifying glass) were all built locally with `xfade`, custom `expr` wipes and still-frame zooms — no model calls, exact control, no style drift. The owner then cut **all** of them and kept a single closing push-in and freeze. Build transitions locally so they are cheap to make *and* cheap to throw away, and expect the restrained cut to win.

### Music and fonts are licensing decisions, not asset hunts

- The platform's own "free music" library (Douyin/CapCut here) typically licenses use **inside that platform**, which does not cover a film published elsewhere. Prefer a library whose licence is unconditional (Pixabay Content License was used), download with `curl`, and record track, author, page and licence in a `LICENSE.md` next to the audio.
- Mix the bed at about -15 dB with a **sidechain compressor keyed by the narration**, fade in and out, then re-normalise the master.
- A free font in a given calligraphic style may simply not exist: for clerical script, the GPL-licensed candidate carries a public infringement claim and was dropped by Debian, and the publicly-licensed foundry alternative was an old TTC that FreeType refuses. Say so and fall back to a clean OFL face (LXGW WenKai here) rather than shipping a risky one.

## Lessons from a 31-shot replication pilot (2026-09-23/24)

A two-minute mythological opening, 31 shots with seven recurring characters, three mounts and two props, was
taken from cards to a mixed cut. Two image routes, one video model, one TTS route. What held and what did not:

### Which image route for which shot

| Shot content | Single-reference model (one `subject_reference`) | Multi-reference model (every card attached) |
|---|---|---|
| One person, close or medium | usually right first time when the face close-up is the reference | fine |
| Empty frame, prop, lone creature | fine | fine |
| Two or more people in frame | **faces merge**: the reference face bleeds into the others, ages and costumes swap | right first time |
| Rider on a mount | the mount turns into a horse; the rider's forehead mark lands on the animal | right first time |
| Hands only, no face | an unrelated person appears anyway, twice, despite the negative | right first time |

On this run the single-reference model took 64 calls for 15 cards and 18 usable keyframes; the multi-reference
route took 14 calls for 13 keyframes, every one usable on the first try. The user's verdict: **the
single-reference model's character images are not good enough for a film meant to be watched.** Use it for empty
frames and props at most; route every card and every keyframe with a person in it through a model that accepts
all the cards in frame, and say in the prompt what not to inherit from them (grey background, side-by-side layout,
neutral pose).

### Cards

- **One call per scale.** Asked for full-length views and a face close-up in one image, the single-reference model
  returned three full-length figures twice. Generate the close-up, then generate the full-length turnaround
  *with the close-up attached*, and compose the card locally. The attachment is what fixed a turnaround whose hair
  had come back black while the close-up was white.
- **A reference that is too strong is copied.** A turnaround generated with the close-up attached came back as
  another close-up. When that happens, drop the reference and lean on the text.
- **Age and sex drift is not fixed by adjectives.** A fourteen-year-old boy came back as a small girl twice. Stop
  after two, and either accept the look (a younger traditional depiction was fine here) or change route.

### Video

- **Queue.** 31 clips submitted in the evening finished the next morning after 315 explicit queue-full refusals,
  which cost nothing. A 48-hour hosting expiry, not 24, was the right call.
- **Hard transformations worked first time**: a figure emerging from a column of fire, a body bursting into flame,
  a bamboo scroll folding into a rod, a tilt down onto a prop.
- **The 4-second floor changes the edit.** The reference cut every one to three seconds. Trimming 4–5 s clips down to
  those lengths read as choppy to the user, who preferred every clip whole. Plan each shot at four seconds or more,
  or put fast cutting *inside* a clip with a scripted multi-shot prompt.
- **A spoken line often starts late**, around three seconds into a five-second clip. Set the in-point from the
  speech segment, not from zero.

### Voice

- **Stock TTS voices pitched and slowed to play an old sage, a king and a child were rejected as poor.** For character
  dialogue, generate one line as a sample and get a yes before generating the rest. On this account the vendor's
  voice-design and music endpoints were unavailable (plan not supported; closed to new users).
- **The video model's own voice is the fallback that keeps lip-sync**: keep its audio on on-camera lines, and treat
  off-screen lines as a separate problem.

### Post

- **Unify the grade in post** when two image models disagree on a palette; do not regenerate to match.
- **Subtitles and title cards as transparent PNG overlays** work where the local encoder has no text or subtitle
  filter.
- **Licensed stock music** needs its source page, author and licence recorded next to the file.

## Round Isolation

A second run must not overwrite the first. Scripts that hardcode a single `output/` directory will clobber prior clips, job records and masters. Give every round its own subtree and its own shot definitions, and verify the no-argument path still behaves exactly as the first round did.

## Validation Checklist

- Source identity confirmed from raw content, not a summary.
- Character sheet and voice sheet quoted verbatim in every prompt that uses them — never paraphrased.
- Voiceover shots forbid lip movement instead of demanding a visible mouth.
- A model-proposed shot list was reviewed as a draft and applied through the ordinary validated save.
- Every long-running step shows progress; no step leaves the operator guessing whether it is working.
- For a long work: length and chapter gaps measured, cast counted from the text, character bible written by stage
  with each line tagged stated or inferred and cited; no source text committed.
- Blocking exists before any shot prompt: every actor and camera placed, every shot citing a camera.
- Root-structure signatures compared; no two shots sharing a recurring setup have matching signatures.
- Character cards built on neutral grey, one per state; no scene frame used as an identity reference.
- Character stress test run, its pass line named before looking, and its result reported.
- Scene description in English, spoken line in its intended language, stage directions kept out of the line's
  paragraph.
- Character bible byte-identical across all shot prompts.
- Hosting route chosen before generating, and shown to work from the user's own network.
- Upload directory contains only intended files; credential and metadata scans clean.
- Every hosted image: HTTP 200, expected content type, SHA-256 equal to local.
- Production and prior hosting targets unchanged.
- Task count within what was authorized.
- Every shot's cast list matches who is *visible* in that shot, not just who acts.
- Every retry classified by persisted state first; nothing holding a live `video_id` was resubmitted.
- Before/after state comparisons taken with identical commands, and each call's success asserted before its
  body was read as state.
- Any automated detector used as evidence shown to stay quiet on known-clean input.
- Every video prompt opens with the unbroken-take wording; every source clip scanned for internal cuts.
- In and out points chosen per clip from its speech window or action, recorded in an edit decision file.
- Final cut: expected duration, resolution, fps, audio stream present; peak level checked for clipping; no
  letterbox bars; shot changes only at planned boundaries; every dialogue window transcribed and whole.
- Spend recorded from what a run produced, not from its plan; failed runs recorded zero.
- No credential value read, echoed or written into the repository — only presence checked.
- Every generated artefact says which tool owns it, in a field that survives later edits.
- Loudness compared with the reference **per shot**, not only integrated; any boosted quiet recording checked for
  band balance; the normalization mode actually used read from its summary.
- Sampled frames reviewed by eye, across time, on the whole frame.
- Superseded keyframes, clips and job records archived rather than overwritten.
- Every no-dialogue shot has a written sound section (voice slot: 无台词) and was checked for open mouths at its
  loudest moments; every shot's language verdict came from a human ear.
- Card sets internally consistent (same garment, hair and ornaments in every view) before any stress test.
- Unverified claims recorded as unverified.
- Every video prompt of a stylised film carries the style-lock sentence, byte-identical; per-second contact sheets
  compared against the keyframe.
- The `VOICE:` block hashed across every submitted prompt of the project; any stale copy found and replaced.
- Every regenerated dialogue clip re-listened, including lines approved in an earlier round.
- New lines screened for reduplication, line-final question words and avoidable polyphonic characters.
- A versioned output's name checked free before writing.
- A scripted sequence: cut count and timing match the plan; screen direction written once from the blocking; every
  over-the-shoulder names its shoulder and frame edge; each person appears once per sub-shot; cone and mouth checks
  run per sub-shot.
- Voice sheets hold only the voice; who speaks is stated per shot.
- A speed variant's audio uses `atempo`; the master's line endings checked, not just its total length.
- Every automated screen shown to fire on a positive control and stay quiet on a negative one before its result is
  reported.
- Progress counted from records holding a video id, not from record files.
- Where the run is driven (agent, control surface, or both) asked at the start and recorded; one dispatcher per lane
  at a time across every project; no restart of the control surface while any job runs.

- Every source clip measured for black borders (cropdetect intersection) and cropped to content before assembly.
- Narration trimmed by silence detection, never by speech-recognition word times; no tail trim at all; the last word clear of the next transition by ~0.9 s.
- Narration voice id, model and speed recorded; the same voice across the whole film.
- Every downloaded music or font asset carries a licence note naming track, author, page and licence terms; platform-internal licences refused for films published elsewhere.

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

**T16 — Trim by content, not from zero.** A 4.5-second dialogue clip goes into a 3-second slot.
*Fail:* `-t 3` from the first frame, then trusting the probe.
*Pass:* locating the speech window, cutting around it, and transcribing that window of the finished master.

**T17 — A continuity reference can override the shot.** A new wide opening must happen in an approved room.
*Fail:* attaching the approved close back view "for continuity" and accepting a near-copy of it.
*Pass:* keeping the scene card, dropping the frame whose scale conflicts, and stating the scale as a share of frame
height.

**T18 — Loud enough overall is not matched.** The master reads -14.4 LUFS; the reference's opening is 10 dB louder.
*Fail:* reporting the integrated figure, or boosting the clip ambience by the difference and trusting the arithmetic.
*Pass:* comparing per shot, muting layers to find the one that owns the level, and measuring each rebuilt mix.

**T19 — Loudness matched, tone not.** After a large boost the opening matches on loudness but sounds hissy.
*Fail:* calling it done because the loudness table is green, or low-passing every shot hard.
*Pass:* comparing band balance against the reference, low-passing the hissy shots, keeping a higher cutoff where a
transient sound lives in the top end, and confirming loudness did not move.

**T20 — The image prompt's style does not reach the video.** Keyframes are flat watercolour; the video prompt says
nothing about style.
*Fail:* trusting the keyframe to hold the look.
*Pass:* a style-lock sentence in every video prompt, checked on keyframe-versus-seconds contact sheets.

**T21 — Same fault twice means change the line.** A listener hears the same word mangled in two rounds.
*Fail:* a third re-roll of the identical line.
*Pass:* rephrasing around the word, then asking again.

**T22 — Regenerated audio is new audio.** Only a style sentence changed; the line was approved last round.
*Fail:* carrying the old approval forward.
*Pass:* asking the listener to hear it again.

**T23 — 429 is a refusal, not uncertainty.** A submit returns `rate_limit_exceeded` and no task id.
*Fail:* leaving the batch stopped, or upgrading the plan.
*Pass:* archiving the record, resubmitting the authorised count, and running one project at a time.

**T24 — A tag that exists is taken.** A new cut is ready and `edl-v3.json` already exists.
*Fail:* writing it anyway.
*Pass:* choosing the next free tag.

**T25 — The deliverable ends the wait, not the process.** A CLI writes its output file and then sleeps for a
minute. The driver returns as soon as the file is complete and stable, well before the process exits, and leaves
no child running. A file that is still growing must not be mistaken for a finished one.

**T26 — A voiceover shot inverts the mouth instruction.** A shot whose staging says narration / back to camera
produces an image prompt that forbids lip movement and a video prompt that says the person on screen does not
speak — while the line itself is still narrated.

**T27 — A draft cannot corrupt the project.** A generated shot list that references a scene which does not
exist is refused at apply time with the offending shot named, and the project files are left byte-identical.

**T28 — A batch covers work not yet started.** With one task created out of twelve shots, "generate all"
reports eleven to create plus one already done — never "everything is already generated".

**T29 — External paths are pinned.** The recorded auth-file path for each OAuth CLI equals the path that tool
actually writes. A rename that changes one of these strings fails the suite.

**T30 — A scripted cut is not an intrusion.** A clip was asked for three timed shots and the detector finds two cuts.
*Fail:* flagging them as internal cuts and regenerating with the unbroken-take sentence.
*Pass:* checking the cuts against the planned boundaries — two planned, two found, each within a second — and passing it.

**T31 — Over-the-shoulder names a side.** A sub-shot reads "her shoulder soft in the foreground".
*Fail:* sending it; the model may put her on both sides.
*Pass:* naming the shoulder and the frame edge from the blocking, and stating she appears once.

**T32 — A two-speaker clip does not copy a one-speaker rule.** Both voice sheets end "only he/she speaks in this shot".
*Fail:* pasting them verbatim into a clip where both speak.
*Pass:* the sheets carry the voice only; the dialogue block says who speaks in which shot.

**T33 — A silent screen gets a positive control.** A subtitle detector reports zero hits across 112 frames.
*Fail:* reporting "no burned-in subtitles".
*Pass:* drawing text onto one of those frames, finding the detector misses it, discarding it and judging by eye.

**T34 — Two senders share nothing.** The agent probes the queue with one submission, it gets in, and the rest of the
batch is handed to the control surface.
*Fail:* confirming the batch immediately — the next submission lands seconds after the probe and draws a 429.
*Pass:* waiting out the inter-submission gap before handing over, or letting one side send the whole batch.

**T35 — Switching the shared view is an action.** The person is running a video round in project A; the agent wants
to inspect project B.
*Fail:* selecting B in the control surface to read its jobs.
*Pass:* reading B's files directly, or asking before switching.

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

**A timeout does not mean nothing was produced.** The CLI writes the image before it finishes composing its reply,
so the timeout can fire after the file has landed. A driver that treats every timeout as failure then retries and
overwrites a good image with a second attempt (2026-09-15, caught only because the file's timestamp predated the
timeout). After a timeout, check whether the output exists at a plausible size and keep it if it does; retry only
when it does not. And never let a retry overwrite an existing output — write each attempt to its own name or skip.

**Better: stop waiting when the file is done, not when the process is.** Waiting for exit is not just slow, it
looks broken. One character card landed on disk about a minute in; the CLI then spent **nine more minutes**
writing its closing summary (a 405 KB log) while the console sat on "generating". The user concluded it was
stuck and pressed generate again, which queued a duplicate behind it. The deliverable is the contract, so poll
for it: once the file is *complete* and has stopped growing for a couple of seconds, end the process. Complete
means structurally complete, not merely non-empty — a PNG carries its `IEND` chunk, a JSON parses. Keep the
timeout as the backstop, and make sure the child is killed on every exit path so a leaked CLI cannot burn quota
later.

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

### Trimming every clip from its first frame

Generation length is whole seconds and the edit wants 3.5 or 6.5, so the obvious assembly trims each clip to its
edit length from frame zero. It silently cuts dialogue: a model that starts speaking at 2.56 seconds into a 4.5-second
clip loses its whole line to a 3-second trim, and the master ships with one word of it (2026-09-14). Nothing flags
it — the file probes clean, and a pre-production TTS timing gate says nothing about when the *video model* chooses
to speak.

**Choose each clip's in and out points from its content.** For a dialogue clip, find the speech window (a silence
detector gives the bursts; word timestamps from a local speech recognizer give the order), start a beat before it and
end a beat after it. For an action clip, look at frames across time and keep the stretch where the action reads and
no internal cut intrudes. Record the chosen points per shot in an edit decision file with a one-line reason, so a
re-edit is a data change. Let dialogue audio lead or trail the picture (J- and L-cuts) when the picture must cut
before the line ends. Then verify the master, not the clips: transcribe each dialogue window of the finished cut and
check the line is whole, and burn subtitles from the script, timed to the measured speech.

A silence detector is not a speech window. A line with a natural pause — 「这么漂亮的匣子，谁看了不心动？」 has 0.8 s of
silence after 匣子 — comes back as two sound windows, and cutting at the end of the first one ships half the line
(2026-09-16). Take the span from the first to the last word of a word-timestamped transcript of the clip, or merge
the sound windows that the transcript covers, and transcribe the finished master: that pass is what caught it.

### The model cuts inside a clip

A shot-change detector (`select='gt(scene,0.25)'` over a downscaled copy) run on each **source clip** finds the cuts
the model added; run it on the **master** and every hit should land on a planned shot boundary, so anything else is
an intrusion. On the clips that had internal cuts, the segments after the cut carried the continuity damage — changed
costume, an extra person, another location — so the usable part was almost always before the first cut. Trim to it,
and regenerate the shot with the unbroken-take wording if what remains is too short. A dark night scene can hide real
cuts from the detector at that threshold; lower it for dark footage and confirm by eye.

For a **scripted** sequence the same detector becomes the acceptance test: the hits must match the planned boundaries
in number and land within about a second of them. A missing cut means two shots merged; an extra one is the old
intrusion.

### Dialogue too quiet in the master

Generated speech can sit near -35 dBFS. Loudness normalization in the per-clip pass brings it up. Check the final peak level afterwards, since a concat re-encode can push peaks to full scale and clip.

### Loudness-normalizing an ambience-only shot

`loudnorm` to a dialogue target is right for a shot with a line in it and wrong for everything else. Applied
to a wind-only or water-only beat it makes ambience as loud as speech; applied to a locally rebuilt shot whose
bed was lifted from a quiet window (peak near -73 dBFS) it lifts the noise floor by fifty-odd dB and delivers
hiss. Drive the choice from the shot's own script entry: `loudnorm` where there is a line, a gentle limited
gain where there is not.

### Matching a reference's loudness shot by shot

A master can hit its integrated target and still be wrong where it matters. A cut at -14.4 LUFS, close to its
reference overall, had an opening 10 dB quieter than the reference's for three shots running (2026-09-15). Only
a **per-shot** comparison shows it: momentary loudness binned at 0.25 s, averaged per shot **in energy, not in
decibels**, set against the same shot's window in the reference.

Four things decide whether a fix lands:

- **Find which layer owns the level before boosting anything.** Mute one bus at a time, rebuild the mix and
  measure. In that opening a synthesized wind bed sat about 6 dB above the clips' own ambience, so lifting the
  ambience by 8 dB moved the shots by about 1 dB. Lift the owner, or both.
- **Measure the rebuilt mix; do not compute the gain.** `loudnorm` with `linear=true` silently falls back to
  dynamic mode when the linear gain would break the true-peak ceiling — it did on every build of that cut — and
  dynamic mode lifts quiet passages non-linearly. The opening came up 12 dB for an 8–12 dB boost, and a further
  2 dB changed nothing. Read `Normalization Type` in the summary, and iterate with an audio-only rebuild that
  takes seconds instead of a full render.
- **A boosted quiet recording brings its hiss.** Ambience recorded low and raised 18 dB matched on loudness and
  came out bright: energy above 5 kHz stood 12.7 dB under the mid band against 20.7 dB in the reference. A
  per-shot low-pass fixed most of it without moving loudness (to 18.1 dB). Set the cutoff by content: 2 kHz on
  hiss-only room tone, 4 kHz where a transient such as porcelain clinking lives in the top end.
- **Prove a mixing refactor is sound-neutral by comparison, not by checksum.** Noise generators without a seed
  make every build differ. Build twice with the new code; if run-to-run difference equals old-to-new difference
  (here -31.3 dB against -31.2 dB), the change is neutral. Better still, seed every noise generator (`anoisesrc=…:seed=N`) so a rebuild
  of the same edit is bit-identical and a checksum becomes enough.
- **Choose the normalization mode on purpose.** Plain gain plus a peak limiter makes a +N dB change in the mix
  +N dB in the master; `loudnorm`'s dynamic fallback does not. Switching an existing cut between them moves its
  quiet passages — the same settings came out 3–6 dB quieter in the opening under linear gain — so keep old
  versions on the mode they were tuned with and record the mode actually used after each render.

Keep mix settings (bus and item gain, mutes, ducking, target) as data in each cut version and save every change
as a new version. A browser preview of the mix is an approximation — it cannot reproduce sidechain ducking or
the final normalization — so check the rendered master against the reference before calling it done.

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

### A local push-in that sticks to the top-left corner

Building a push-in locally by scaling up frame by frame and cropping a fixed 1280×720 window, with the crop offset
written as a fraction of `in_w - 1280`, looks correct and is not: FFmpeg's `crop` takes `in_w`/`in_h` from the first
frame, so as the scaled frame grows the offset stays near zero and every push drifts toward the top-left corner. Two
test renders with different horizontal centres came out identical (2026-09-15), which is the tell. Compute the
overflow from the zoom expression itself — the same expression the scale uses — and multiply that by the centre
fraction. While scaling to cover, also cover the frame rather than padding it: a 1280×704 source padded to 720 ships
with 8-pixel black bars top and bottom.

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

### A speech recogniser hallucinates on near-silence

Run on a no-dialogue clip, `whisper-large-v3-turbo` produced 「请不吝点赞 订阅 转发…」, 「字幕志愿者 …」 and long runs of one
repeated character (「明明明…」, 「好好好…」) — known artefacts of subtitle-heavy training data, not speech. Treat any
transcript of a clip whose loudness sits around -40 LUFS or lower as noise; decide by looking at mouths at the
loudest moments.

### Sampling a frame or two is not verification

Whether a mouth stays closed, whether text appears, whether a character drifts — these are properties of
*every* frame, and one or two sampled frames cannot establish them. A clip can start clean and change halfway.
Report what the sample showed ("the frames I checked show X"), never what the clip does, unless you scanned
across time. The failure is self-similar: having learned it for burned-in captions, it is easy to keep
sampling one frame per clip for mouths and state the conclusion with the same false confidence.

### Proving speech without hearing it

You cannot verify audio directly, but you can gather real evidence. Compute a per-100ms RMS envelope: discrete bursts against a low floor, with a dynamic range above roughly 20 dB, indicate speech, whereas flat ambience stays under about 12 dB. Then sample frames at the burst timestamps and check that mouth shapes actually change between them. Report this as evidence of speech, never as confirmation of language or content.

### Authorship strings are not provenance

An artefact said who last edited it, and the pipeline used that to decide which renderer owned it. Saving an edit
rewrote the field, and the next render handed a generic edit list to a film-specific script, which died on a key it
expected (2026-09-17). Authorship, "edited in", "created by" and timestamps all describe the last touch, not the
contract. When a downstream step must know which tool owns a file, write that as its own field, keep it through
edits, and let the fallback be structural (does the file carry the fields that tool requires) rather than textual.

### A relative path computed against the wrong base

A generated edit list stored clip paths relative to the folder it was *usually* written to, then got written
somewhere else, and the renderer reported missing sources. The same shape appears with a script path resolved from
the project root when the tool runs from a subfolder. Compute a relative path from the directory the file will
actually live in, and resolve a configured path against the working directory the tool will actually use — then
assert the target exists before the run rather than inside it.

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
