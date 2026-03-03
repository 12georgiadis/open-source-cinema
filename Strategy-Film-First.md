# The Film Comes First

**A strategy for auteur filmmaking with AI. March 2026.**

---

## The Trap

You can spend six months building the perfect pipeline and never cut a film.

The research is real. The tools are real. The stack in [Generative-Pipeline-March-2026.md](./Generative-Pipeline-March-2026.md) is correct. But a pipeline without a film to run through it is just infrastructure for infrastructure's sake.

The distinction that matters: tools in service of the film, not the film as pretext for building tools.

Fincher's obsessive process exists because he has films that require it. The obsession is subordinate to the vision. The risk for any filmmaker engaging seriously with AI tooling is becoming the person who builds the pipeline rather than the person who makes films that couldn't have existed otherwise.

These are not the same person.

---

## Three Phases

### Phase 1 — Cut the Film (Now)

Three decisions. Nothing else.

**1. DaVinci Resolve Studio.** $295 one-time. End of the FCP/Resolve debate. Jumper is now native inside Resolve (Workspace > Workflow Integrations). Three MCP servers are active. Python API is official. Text-based editing is built in. Fairlight handles audio. The question is answered.

**2. davinci-resolve-mcp.** This is the single wire that makes everything work: Claude Code → Resolve in natural language. Install it, point it at a running Resolve instance, and you have a direct channel from the CLI to the timeline. That's the foundation.

**3. StoryToolkitAI for rushes.** Free. Open source. Local Whisper transcription. Semantic search in transcripts. Export to EDL or directly to Resolve. If you have hours of interview footage with a subject, this is what it's built for.

With these three things: Claude Code reads the OTIO/EDL structure of the timeline, analyzes narrative architecture, proposes reconfigurations, you validate or reject, Resolve executes. This is not theoretical. It's operational today.

**What you don't touch in Phase 1:** ComfyUI, world models, ReCamMaster, generative video. All of that is post-structure work. Structure first.

---

### Phase 2 — Generative Inserts (During Post)

When the structure is locked and you know what's missing.

Not before.

Once you know a shot doesn't exist and can't be reshot:
- **fal.ai** for generation (direct Python call, no local setup overhead)
- **Chatterbox** if you need voice
- **MMAudio** for ambient sound on generated shots
- **SkyReels V4** for shots that need synchronized audio from the start

The generative layer only makes sense when you know the hole it needs to fill. Generating without a structural gap to fill is spending money on texture before you have a skeleton.

---

### Phase 3 — Design the Next Film Differently

The film after this one isn't shot the same way.

You shoot knowing you'll reshoot some angles via ReCamMaster in post. You think in terms of spatial material rather than discrete shots. You design narrative structure as a parameter space, not a linear script. The locations become 3D environments you can re-enter. The characters' voices are cloneable for retakes.

This is a different way of conceiving a film from the start — not retrofitting AI onto a traditional production. Phase 3 is only available once you've done Phases 1 and 2 and understand what AI can actually do inside your specific practice.

---

## Where This Is Going

Two things converging that matter.

**The CLI as structural intelligence.** Not "AI edits the film." That framing is wrong and won't produce cinema worth watching. The real shift: the CLI becomes the first collaborator capable of holding macro structure (90-minute arc) and micro detail (this cut, this silence, this sound) simultaneously, without losing the thread. A human editor holds this in memory with great effort and experience. The CLI holds it computationally and can test fifty structural variants in an hour. The editor's judgment is still the only thing that matters — but now it operates at the level of choosing between fifty tested structures rather than building one from scratch.

**The screenplay is disappearing as the primary document.** Not the story — the screenplay as the central structural object. In the emerging paradigm, the primary document becomes a navigable structure of narrative parameters: characters, arcs, tensions, registers, material textures. You don't write a script and shoot it. You define a space of narrative possibilities and navigate toward a film. The "cut" becomes a trajectory through that space, not a fixed sequence of decisions made once and executed.

This has been the implicit logic of documentary filmmaking forever — you don't know what film you have until you're in the edit. AI makes this logic available to fiction as well. The documentary filmmaker's relationship to material becomes the model for all serious filmmaking.

---

## What This Stack Is Not

It's not a shortcut to a film.

It's not a way to make films without a filmmaker.

It's not faster in the sense that matters — the thinking, the doubt, the time spent looking at a cut and knowing it's wrong before you can say why, the recognition of the moment that makes the film, the ethics of representation. None of that compresses.

What compresses: the mechanical labor. The assembly. The variant-testing. The research. The technical debt between an idea and its execution. The time between asking "what if this scene came first" and knowing whether it works.

That compression creates space. What you do with the space is the whole question.

---

## The Only Question That Matters Today

Do you have Resolve Studio or not?

Everything else follows from that.

---

*Part of the [open-source-cinema](https://github.com/12georgiadis/open-source-cinema) research project.*
