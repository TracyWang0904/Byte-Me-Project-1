# Specification Phase Exercise

A little exercise to get started with the specification phase of the software development lifecycle. In this exercise, your team specifies a set of improvements and new features for [The Slide Machine](https://theslidemachine.com) — see the [instructions](instructions.md) for detail, and the [background](background.md) for an introduction to the software product you are tasked with extending.

## Team members

- [Tracy Wang](https://github.com/TracyWang0904)
- [Emma Ao](https://github.com/emma6594)
- [Jingjing Wang](https://github.com/JingjingWang129)
- [Uuriintuya Ganzorig](https://github.com/Uuriii1003)

## Review of the Current Application

Findings from the team's independent use of [theslidemachine.com](https://theslidemachine.com).

### Strengths

- Speech recognition is clear and converts spoken language into consistent, natural-sounding text.
- A dedicated discovery/browse interface lets users search for colleagues' and other relevant finished slide decks to use as references.
- Supports uploading a previously created presentation and adding new, explanatory slides in the same format based on further voice input — an instructor can reuse a deck someone else made and personalize it while teaching their first class, saving prep time.
- The AI read-aloud (narration) feature gives coherent explanations rather than simply reading the slide text verbatim, which is friendly to self-study — a student who finds a useful deck online can upload it and listen to the AI's "lecture," which benefits auditory learners.

### Weaknesses

- Cannot generate an introduction/opening slide without voice input.
- When spoken input is short, transcription is unreliable and drops parts of the speech.
- Some slide themes make text hard to read because text and visual elements compete for the same space.
- Text editing is limited: existing text can be changed, but no new text box can be added, which limits how much information can be inserted.
- Text formatting is very limited: the editor lets users change text content but provides no controls for basic formatting such as font style or size.
- Changing the layout/style of one slide changes the whole presentation's style, rather than only that slide.
- If an instructor later adds more to a topic mentioned earlier, the new slide cannot be placed near the other slides on that topic — slides are ordered strictly by when they were spoken.
- Quiz answers tend to be the most specific/longest option, because quiz questions are generated directly from slide text and pull answers verbatim from it.
- The distinction between "add slide" and "add whiteboard" is unclear from the button labels alone; a first-time user has to experiment with both to learn when to use each.
- The delete-slide action is hard to discover during editing — the functionality exists but is not visible without extra searching.

### Gaps

- No way to provide a URL/website as seed material.
- Refinement changes apply immediately, with no save/confirm step before they take effect.
- No undo button.
- Limited ability to add custom visual elements such as shapes, icons, graphics, or photos.
- No support for understanding/transcribing other languages.
- No apparent mechanism for an instructor to mark or revise a generated slide after verbally correcting or updating what they said (needs re-testing to confirm).
- No live draft control: no mechanism to temporarily hold, discard, or revise AI-generated slide content created from a spoken idea that wasn't yet finished.
- No mechanism to mark certain spoken content as non-lecture material so it doesn't influence generated slides (needs re-testing to confirm).
- No mechanism for an instructor to signal they are returning to or continuing an earlier topic during live generation.

## Prior Art & Originality

Our proposal is **Live Structural Control for Slides**. The Slide Machine currently treats a lecture as a linear stream: as the instructor speaks, the system decides whether to update the current slide or create a new one, strictly in speaking order. Real lectures are not linear — instructors introduce side examples, answer student questions, return to earlier concepts, or notice that the AI has organized their content incorrectly — and today they have no way to correct any of that while the lecture is still running. Our proposal gives instructors control over where generated content belongs in the lecture's conceptual structure, without pausing transcription or generation to do it. The guiding principle is that **speaking order does not always equal presentation structure**. This is not a general-purpose slide editor (fonts, colors, themes, and image placement are explicitly out of scope) — it is a coherent set of interactions for correcting and directing *structure* live: topic branching, redirecting the AI's current generation target, merging/splitting/moving/reattaching slides, reclassifying already-generated content, explicit topic-transition cues, and quick undo/recovery for all of the above. Several of these directions trace directly to gaps and weaknesses our own team found while using the live app (see [Review of the Current Application](#review-of-the-current-application) above) — most directly, "no undo button," "no live draft control," "no mechanism to signal returning to an earlier topic," and "a new slide can't be placed near earlier slides on the same topic."

**What we checked:** the SDD's Future Work (§18) and Open Questions (§19), the Delivery Roadmap (including its risks & cut-line section and Phase 3 outstanding list), and the repository's open GitHub issues — none address live topic/deck structure, and none propose branching, redirection, retroactive reclassification, structural undo, or topic-transition cues during capture.

- **SDD Future Work (§18)** — the deferred items (local AI models, real-time translation, extracting the STT→generation pipeline into its own service, an MCP agent server, multi-user collaborative editing, seat-based billing, richer analytics/recommendation, a faculty setup guide) are all unrelated to live structural control.
- **SDD Open Questions (§19)** — none of the still-open questions (plan pricing, pilot exemptions, student roster source, latency targets, image licensing enforcement, image disambiguation depth, Slides export fidelity, coverage-gate scope, preflight concept-set limits, MCP auth/scope, AI-imagery accuracy) touch on deck structure or live correction.
- **Delivery Roadmap** — the risks & cut-line section and the Phase 3 outstanding list (`CAP-5` live captions, `EDIT-8` duplicate slide, `PLAY-4`/`PLAY-5`, `PREP-1..4` preflight, `IMG-4` AI imagery, hardening) include nothing about structural control, branching, or live correction.
- **Open GitHub issues** — none propose structural control during capture. The closest related item, issue #27 ("Required Transcript Viewing"), proposes forcing linear, unskippable playback — the opposite concern from ours — and remains unresolved as of this check.

**Adjacent existing/shipped features we checked each proposed capability against, to avoid re-proposing them:**

- **Topic branching** — live generation only ever decides, per spoken phrase, "update the current slide" or "start a new slide" (`GEN-8`), a strictly linear choice with no concept of a tangent to set aside and rejoin. Nothing like a branch exists anywhere in the app.
- **Active generation target / redirect** — live generation shows only a generic "something is generating" cue (`GEN-5`'s activity indicator); there is no display of *which topic* is the AI's current target, and no way to redirect new content to a different one. The post-lecture Refine job does narrate which slide it is working on, but that is a background batch job's progress readout, not a live, redirectable target.
- **Structural correction (merge / split / move / reattach)** — merging and splitting slides already exist, but only inside `GEN-4`'s post-lecture "Reformat with AI" pass, where the **AI** decides whether and how to merge or split as part of a holistic, opt-in, post-lecture rewrite — the instructor cannot trigger either action live or control the result. Moving a slide already exists (`EDIT-1` includes manual slide reordering), but only **after** the lecture ends, by hand, with no live or automatic grouping. "Reattach" (moving a branch to a different parent) has no counterpart, since branching itself doesn't exist.
- **Retroactive adjustment** — no mechanism anywhere lets an instructor reclassify already-generated content (e.g., turn an existing slide into a branch after realizing the discussion became a tangent); this depends on the branching concept above, which is new.
- **Topic transitions** — `CAP-4` already ships a fixed voice-command vocabulary (start/stop/pause/resume/rewind/fast-forward), and `GEN-8`'s opt-in manual new-slide mode already supports an explicit "next slide" cue to force a slide boundary. None of these carry topic-level meaning, though — there is no "continue previous topic" or "return to main topic" cue; the existing commands only mark *that* a boundary should occur, never *why*.
- **Undo / quick recovery** — an undo/redo control already exists, but it is scoped entirely to whiteboard pen strokes (`EDIT-5`, "Undo / redo, per slide"); there is no undo for slide content edits, merges, splits, moves, or any other structural change, which matches our own team's review finding of "no undo button" for the editing experience generally.

**What is new:** live, instructor-driven control over the deck's conceptual structure while capture is still running — topic branching with a one-action return to the main thread; a visible, redirectable generation target; manual merge/split/move/reattach of slides during (not only after) the lecture; retroactive reclassification of already-generated slides; explicit topic-transition cues beyond a bare slide boundary; and undo/recovery that covers structural changes, not just whiteboard strokes. None of this appears in the current application, the SDD, the Roadmap, or any open issue or pull request we reviewed.

## Stakeholders

See instructions. Delete this line and replace with the name(s) of the stakeholder(s) you interviewed and lists showing their goals/needs, and problems/frustrations. Note which type of user each stakeholder represents. You may use pseudonyms or partial names to maintain their privacy, but you must privately share their full names and contact information as part of your submission of this exercise

## Product Vision Statement

See instructions. Delete this line and place your Product Vision Statement here — one sentence describing the improvements and new features your team is proposing for The Slide Machine.

## User Requirements

See instructions. Delete this line and place a list of your User Stories here, grouped by type of user. These should describe functionality that is new or changed, not functionality the app already has.

## Activity Diagrams

See instructions. Delete this line and place images of your UML Activity diagrams here, each with the text of the user story it illustrates.

## Wireframes

See instructions. Delete this line and place your wireframe diagrams here, covering every new screen and every existing screen your proposal changes, for every type of user.

## Clickable Prototype

See instructions. Delete this line and place a publicly-accessible link to your clickable prototype here.

## Stakeholder Demo

See instructions. Delete this line and place a link to the deck The Slide Machine generated during your presentation here, after you have presented.

## Exit Ticket

See instructions. Delete this line and place a link to the exit-ticket quiz you generated from your demo deck and distributed to the class, along with a short note on what — if anything — you had to correct in the generated questions before publishing.
