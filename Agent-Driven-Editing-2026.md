# Agent-Driven Editing: Where Cinema Meets Code

**State of the art, February 2026. For auteur filmmakers, not content creators.**

---

## The Question

Can an AI agent edit a feature film?

Not assemble clips. Not auto-cut a vlog. Not apply trendy transitions to a 30-second reel. *Edit a film.* Make structural decisions about narrative, pacing, emotional weight. Control an NLE at the level a professional editor does, or at least handle the mechanical work so the filmmaker can focus on the decisions that matter.

This is a real question now. Not because the technology is ready (it isn't), but because the building blocks exist for the first time. And the difference between the building blocks and the finished tool is smaller than most people think.

---

## Table of Contents

- [What Actually Exists (Not Hype)](#what-actually-exists-not-hype)
  - [MCP Servers for NLEs](#mcp-servers-for-nles)
  - [OpenTimelineIO (OTIO)](#opentimelineio-otio)
  - [NLE Scripting APIs](#nle-scripting-apis)
- [NLE Comparison for Agent Control](#nle-comparison-for-agent-control)
  - [DaVinci Resolve](#davinci-resolve)
  - [Blender VSE](#blender-vse)
  - [Final Cut Pro](#final-cut-pro)
  - [Premiere Pro](#premiere-pro)
  - [FFmpeg and MLT: The Headless Path](#ffmpeg-and-mlt-the-headless-path)
- [The Protocol Layer](#the-protocol-layer)
  - [MCP (Model Context Protocol)](#mcp-model-context-protocol)
  - [OpenTimelineIO as the Backbone](#opentimelineio-as-the-backbone)
  - [The Missing Piece: A Headless NLE](#the-missing-piece-a-headless-nle)
- [CLI Editing vs GUI: Is the Interface Still Necessary?](#cli-editing-vs-gui-is-the-interface-still-necessary)
  - [What Can Already Be Done Without a GUI](#what-can-already-be-done-without-a-gui)
  - [What Still Requires a GUI (And Why)](#what-still-requires-a-gui-and-why)
  - [The Cursor Analogy (And Where It Breaks)](#the-cursor-analogy-and-where-it-breaks)
  - [The Actual Architecture](#the-actual-architecture)
  - [But Wait: The Multimodal Loop Changes Everything](#but-wait-the-multimodal-loop-changes-everything)
  - [The Claude Code Editing Workflow](#the-claude-code-editing-workflow)
  - [Where This Is Going](#where-this-is-going)
- [What Editing Actually Is (And What It Isn't)](#what-editing-actually-is-and-what-it-isnt)
  - [80% Off the Timeline](#80-off-the-timeline)
  - [The Timeline Part](#the-timeline-part)
  - [What Agents Should Actually Automate](#what-agents-should-actually-automate)
- [AI Tools for FCP: Keywords, Smart Collections, Logging](#ai-tools-for-fcp-keywords-smart-collections-logging)
  - [FCP 12 Built-in AI](#fcp-12-built-in-ai-january-2026)
  - [Third-Party Tools](#third-party-tools-ranked-by-relevance-for-features)
  - [Smart Collections via FCPXML](#smart-collections-via-fcpxml)
  - [The Complete Assistant Editor Agent Pipeline](#the-complete-assistant-editor-agent-pipeline)
  - [Sound Department Automation](#sound-department-automation)
  - [What's Still Missing](#whats-still-missing)
- [My Assessment](#my-assessment)
  - [What Works Today](#what-works-today)
  - [What's Coming (2026-2027)](#whats-coming-2026-2027)
  - [What Will Never Work (And Shouldn't)](#what-will-never-work-and-shouldnt)
- [For Auteur Cinema](#for-auteur-cinema)
- [Key Resources](#key-resources)

---

## What Actually Exists (Not Hype)

### MCP Servers for NLEs

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/) is Anthropic's open protocol that lets AI models interact with external tools via standardized servers. As of February 2026, three NLEs have MCP servers:

**DaVinci Resolve** -- the most developed:

| Project | Stars | Coverage | Notes |
|---------|-------|----------|-------|
| [samuelgursky/davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp) | ~485 | Claims 202 features (100% of API) | 8% verified macOS, 82% needs verification |
| [apvlv/davinci-resolve-mcp](https://github.com/apvlv/davinci-resolve-mcp) | -- | Project + timeline + media + Fusion | Includes Fusion node creation, Lua execution |
| MCP Video Editing Assistant | -- | Behavior learning | Analyzes editing patterns and cut rhythms |

**Final Cut Pro:**

| Project | Tools | Notes |
|---------|-------|-------|
| [DareDev256/fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) | 34 tools | Analysis, QC, editing, generation via FCPXML roundtrip |

**Premiere Pro:**

| Project | Status | Notes |
|---------|--------|-------|
| [Adobe Premiere Pro MCP](https://github.com/hetpatel-11/Adobe_Premiere_Pro_MCP) | Early stage | Many tools scaffolded but unimplemented. UXP scripting experimental |

**Blender VSE:** No dedicated MCP server exists for video editing. [3D-Agent](https://3d-agent.com/blender-ai) controls Blender for 3D modeling via MCP, but nothing targets the VSE specifically.

### OpenTimelineIO (OTIO)

[OpenTimelineIO](https://github.com/AcademySoftwareFoundation/OpenTimelineIO) is the most important tool in this entire discussion. Created by Pixar, now an Academy Software Foundation project. Version 0.18.1 (November 2025), ~1,800 GitHub stars.

OTIO is an open source API and interchange format for editorial timeline information. Think of it as a modern, programmable EDL. The native format (`.otio`) is JSON, which means an LLM can read, understand, and generate timelines directly.

**What OTIO can represent:**

| Element | Support |
|---------|---------|
| Clips with source ranges | Full |
| Gaps | Full |
| Transitions (crossfades, etc.) | Full |
| Multi-track stacks | Full |
| Markers with metadata | Full |
| Linear time warps (speed changes) | Full |
| Multiple media references per clip | Full (since v0.15) |
| Effects (with enabled flag) | Partial (since v0.18) |
| Color grading / CDL | In discussion |
| Keyframes | Not yet |

**NLE support:**

| NLE | OTIO Support |
|-----|-------------|
| DaVinci Resolve | Native (since v18.5): compound clips, markers |
| Final Cut Pro | Via FCPXML adapter (bidirectional conversion) |
| Premiere Pro | Beta import/export |
| Blender VSE | Community addon ([VSE_OTIO_Export](https://github.com/tin2tin/VSE_OTIO_Export)) |
| Kdenlive | Official adapter ([KDE GitLab](https://invent.kde.org/multimedia/kdenlive-opentimelineio)) |
| Nuke Studio / Hiero | Beta (since v13.2) |
| RV / Open RV | Native (first commercial adopter) |

**Why this matters for agents:** OTIO is the only format where an AI can programmatically create a complete timeline in Python, convert it to FCPXML or AAF, and import it into any NLE. The agent doesn't need to control the NLE directly. It generates the timeline as data, the filmmaker imports it.

```python
import opentimelineio as otio

# An agent can build a timeline from scratch
timeline = otio.schema.Timeline(name="Agent Rough Cut")
track = otio.schema.Track(name="V1", kind=otio.schema.TrackKind.Video)

clip = otio.schema.Clip(
    name="Interview_A_01",
    media_reference=otio.schema.ExternalReference(
        target_url="/media/interview_a_01.mov"
    ),
    source_range=otio.opentime.TimeRange(
        start_time=otio.opentime.RationalTime(1440, 24),  # TC 00:01:00:00
        duration=otio.opentime.RationalTime(360, 24)       # 15 seconds
    )
)
track.append(clip)
timeline.tracks.append(track)

# Export to any format
otio.adapters.write_to_file(timeline, "rough_cut.fcpxml")  # For FCP
otio.adapters.write_to_file(timeline, "rough_cut.otio")     # Native
```

**Critical development:** OTIO v0.16 introduced **experimental editing commands** in C++ (insert, overwrite, roll). This is the seed of a headless NLE data model. Not there yet, but the direction is clear.

CLI tools: `otioconvert` (format conversion), `otiotool` (inspect, trim, flatten, concatenate timelines).

### NLE Scripting APIs

The scripting landscape in February 2026:

| NLE | Language | API Coverage | Headless | Real-time Control |
|-----|----------|-------------|----------|-------------------|
| **DaVinci Resolve** | Python, Lua | ~25-35% of app | `-nogui` (needs daemon) | Yes (live API) |
| **Blender** | Python (bpy) | ~95% of app | `--background` (true headless) | Yes |
| **Final Cut Pro** | None (FCPXML roundtrip) | N/A | Not possible | No |
| **Premiere Pro** | ExtendScript (dying) / UXP (experimental) | ~45-55% | No | Partial |

---

## NLE Comparison for Agent Control

### DaVinci Resolve

**The best-positioned traditional NLE for AI agent control.**

The API hierarchy: `Resolve` > `ProjectManager` > `Project` > `MediaPool` / `Timeline` > `TimelineItem`. Documentation is notoriously poor (a single README.txt). Community docs fill the gap:
- [Unofficial API Docs](https://deric.github.io/DaVinciResolve-API-Docs/)
- [X-Raym's Formatted Docs](https://extremraym.com/cloud/resolve-scripting-doc/)
- [pydavinci](https://github.com/pedrolabonia/pydavinci) -- Pythonic wrapper with type hints

**What an agent can do:**

| Operation | Possible | Notes |
|-----------|----------|-------|
| Create/manage projects | Yes | Full CRUD |
| Import media | Yes | Folders, clips, audio |
| Create timelines | Yes | From clip lists or empty |
| Place clips on timeline | Append only | Cannot insert at arbitrary positions |
| Markers | Yes | Full CRUD with metadata |
| Set render settings | Yes | All delivery parameters |
| Start/monitor renders | Yes | Queue management |
| Apply LUTs/CDLs per node | Yes | Via DRX files |
| Control color wheels/curves | No | Major limitation |
| Fusion scripting | Yes | Node creation, Lua execution |
| Import/export EDL, AAF, FCPXML | Yes | Full |
| Switch pages | Yes | Media/Edit/Color/Deliver |

**The Fusion angle:** Resolve's integrated compositing engine (Fusion) has a complete scripting API via Python and Lua. An agent can create node trees, connect them, and execute scripts. This opens VFX automation that no other NLE scripting API matches.

**Headless:** Resolve supports `-nogui` but the application daemon must be running. Not a pure CLI tool. Pop-up dialogs can block automation on headless servers.

**Verdict:** Best for serious filmmaking because it's already the industry standard for finishing. The API covers the pre-production and delivery ends well (import, organize, render), but the creative middle (editing decisions, color grading) still requires human hands. For an auteur workflow, this is actually correct: the agent handles the mechanical pipeline, the filmmaker makes the creative decisions.

### Blender VSE

**The most complete API, but not a cinema-grade NLE.**

Blender's Python API (`bpy`) covers ~95% of the entire application, including the Video Sequence Editor. It runs truly headless (`blender --background`). It can be imported as a Python module (`import bpy`). The VSE 2026 roadmap includes storyboarding tools and camera editing from the VSE.

**The problem:** Blender VSE is not designed for feature film editing. No multi-cam, no professional conform workflow, no advanced audio mixing, no industry-standard color pipeline. Its strength is as a **headless pipeline tool** -- assembling sequences programmatically, rendering, batch processing -- not as the editorial environment for a 90-minute auteur film.

**For agents:** The combination of true headless mode + complete Python API + open source makes Blender VSE the best tool for building custom automated video pipelines. But the filmmaker would never sit in front of Blender VSE to make editorial decisions.

### Final Cut Pro

**Elegant programmatic interface, no real-time control.**

FCP cannot be scripted in real time. There is no live API. The only programmatic path is **FCPXML roundtripping**: generate XML, import into FCP, edit, export XML, modify, reimport. [CommandPost](https://commandpost.fcp.cafe/) (used at Netflix, Pixar, BBC) bridges some gaps with Lua scripting and hardware panel control, but it's not designed for agent automation.

The [FCPXML MCP server](https://github.com/DareDev256/fcpxml-mcp-server) is actually impressive: 34 tools including rough cut generation from keywords, beat sync editing, flash frame detection, and gap closure. But everything works through file roundtripping, not live control.

**For agents:** FCP is the NLE where the agent prepares FCPXML files that the filmmaker imports. The interaction model is **batch, not interactive**. Generate a rough cut as FCPXML, import it, the filmmaker refines it in FCP, exports XML, the agent processes the changes. This roundtrip model works, but it's slow.

**For auteur cinema:** FCP is an excellent editorial tool. The magnetic timeline is uniquely suited to fluid editorial work. But the agent story is limited to FCPXML generation and analysis, which is still useful.

### Premiere Pro

**Stuck in an API transition. Not recommended for agent workflows.**

Premiere is caught between three API generations: ExtendScript (legacy, dying September 2026), CEP (being phased out), and UXP (experimental, incomplete). As of February 2026, there is no stable, comprehensive scripting API. The MCP server is early stage with many unimplemented tools.

Adobe is investing in built-in AI features (Firefly-powered Generative Extend, Media Intelligence, auto-captioning) but these are closed, not scriptable. Adobe builds AI into their tools; they don't build tools for AI.

**For agents:** Wait. The UXP transition may eventually provide a modern API, but it's not there yet. If you're building an agent workflow today, Premiere is the wrong choice.

### FFmpeg and MLT: The Headless Path

**FFmpeg** is a render engine, not an NLE. It has no timeline model, no project concept, no markers, no undo. But it is the most scriptable media tool in existence. [editly](https://github.com/mifi/editly) provides a declarative JSON-to-video pipeline on top of FFmpeg. [AI-FFmpeg](https://github.com/woniu9524/ai-ffmpeg) translates natural language to FFmpeg commands.

**MLT Framework** ([mltframework.org](https://www.mltframework.org/)) is more interesting. It's the engine behind Kdenlive and Shotcut. It has:
- An XML-based timeline description (multi-track, effects, transitions)
- A CLI tool (`melt`) that renders timelines without any GUI
- Python bindings
- Active development (v7.36.0, December 2025)

MLT is the closest thing to a "headless NLE" that exists today. An agent could generate MLT XML, preview with `melt`, and render the final output. No GUI required at any point. The quality is production-grade (Kdenlive is used for professional work). The limitation is the ecosystem: no color science comparable to Resolve, no industry standard finishing tools.

---

## The Protocol Layer

### MCP (Model Context Protocol)

MCP is the protocol that makes agent-NLE communication possible. An MCP server exposes tools (functions the agent can call), resources (data the agent can read), and prompts (templates for common operations). The agent doesn't need to know the NLE's internal API -- it talks to the MCP server, which translates to NLE-specific commands.

The three Resolve MCP servers already demonstrate this: an AI agent using Claude can create timelines, import media, set render parameters, and execute Fusion scripts by calling MCP tools. The agent never touches the Resolve API directly.

**What MCP enables that scripting alone doesn't:**
- Natural language interaction with the NLE
- Contextual awareness (the agent can read the current timeline state before making changes)
- Error handling through conversation (if an operation fails, the agent can reason about why and try an alternative)
- Multi-NLE workflows (an agent with MCP servers for both Resolve and FCP can orchestrate a roundtrip)

### OpenTimelineIO as the Backbone

OTIO is the **interchange layer** that connects everything. The workflow:

```
Agent (reasoning, decisions)
  |
  v
OTIO Python API (timeline as data structure)
  |
  +---> .fcpxml (import into FCP)
  +---> .aaf (import into Avid)
  +---> .otio (native, inspect, modify)
  +---> .edl (legacy systems)
  +---> Resolve native (via OTIO support in Resolve 18.5+)
  |
  v
NLE (human filmmaker reviews, refines, grades)
  |
  v
Export (OTIO / XML / AAF)
  |
  v
Agent (reads the changes, understands the filmmaker's decisions)
```

OTIO is **not** an NLE. It's a data model. But it's the data model that an agent can reason about. An agent that understands OTIO can:
- Generate rough cuts from metadata and transcriptions
- Analyze an existing edit (duration of shots, pacing, structure)
- Propose restructurings ("move the climax 10 minutes earlier, adjust surrounding scenes")
- Convert between NLE formats transparently
- Track versions and changes across editorial iterations

The fact that OTIO is JSON makes it particularly LLM-friendly. A timeline is just structured text. An LLM can read it, reason about it, and generate valid modifications.

### The Missing Piece: A Headless NLE

What doesn't exist yet but is clearly needed: a **purpose-built headless NLE** designed for AI agent control.

Not a GUI editor with a scripting API bolted on. A cinema-grade rendering engine with:
- OTIO as its native timeline model
- MCP as its control interface
- FFmpeg/MLT as its rendering backend
- Full color pipeline (OCIO-based, DaVinci Wide Gamut level)
- Headless by default, GUI optional
- Every operation available via API

OTIO's experimental editing commands (insert, overwrite, roll) are the embryo of this. MLT provides the rendering engine. OCIO provides the color science. The pieces exist. Nobody has assembled them.

This would be the **"Cursor for video editing"** that the industry is waiting for. Not a SaaS that dies in 24 hours. Not a CapCut clone with a chatbot. A serious tool for serious filmmakers who happen to also write code or work with AI agents.

---

## CLI Editing vs GUI: Is the Interface Still Necessary?

This is the real question underneath everything else. Not "which NLE is best for agents" but: **do we still need a graphical timeline to edit a film?**

Cursor changed coding. You write intent in natural language, the agent writes code, you review the diff. The GUI (the code editor) is still there, but its role shifted: from being the place where you *write* code to being the place where you *review* what the agent wrote. The creative act moved from typing to directing.

Can the same thing happen to film editing?

### What Can Already Be Done Without a GUI

Today, in February 2026, the following operations are possible entirely from the command line or via scripting, with **zero graphical interface**:

| Operation | Tool | How |
|-----------|------|-----|
| **Build a timeline from scratch** | OTIO Python API | Define clips, gaps, transitions as code. Export to any format. |
| **Assemble a rough cut from metadata** | OTIO + Python | Read a CSV/JSON of shot descriptions, match to media files, generate timeline |
| **Convert between edit formats** | `otioconvert` CLI | EDL to FCPXML to AAF to OTIO in one command |
| **Trim, reorder, concatenate edits** | `otiotool` CLI | `otiotool edit.otio --trim 00:01:00:00 00:05:00:00 -o trimmed.otio` |
| **Detect cuts in a video** | FFprobe + OTIO | Automated scene detection, output as OTIO timeline |
| **Render a multi-track timeline** | MLT `melt` CLI | Read XML timeline, render to ProRes/H.265/DPX without any GUI |
| **Render from JSON spec** | [editly](https://github.com/mifi/editly) | Declare clips, layers, transitions in JSON5. FFmpeg renders. |
| **Import media, create bins, set up project** | Resolve Python API | Headless via `-nogui` |
| **Set render settings, start render queue** | Resolve Python API | Full delivery pipeline |
| **Create DCP** | DCP-o-matic CLI | `dcpomatic2_cli --dcp-path /output film.xml` |
| **Transcode, conform, reformat** | FFmpeg | The universal CLI media tool |
| **Transcribe dialogue** | Whisper CLI | `whisper audio.wav --model large --output_format srt` |

That's not nothing. That's the entire pipeline *except* the creative editorial decisions.

### What Still Requires a GUI (And Why)

Here's the honest list of what you cannot do from a CLI, and the reason is not technical but **perceptual**:

**1. Watching the cut.**

This is the fundamental one. Editing is a temporal art. You need to *see* the image and *hear* the sound at the right speed, in sequence, to judge whether a cut works. A timeline is data; a film is experience. No amount of metadata about a shot tells you what it *feels like* to watch it at that moment in the sequence. You can describe a face in JSON. You can't feel the weight of holding on that face for three seconds from a timecode range.

There are CLI playback tools (`mpv`, `melt`, Open RV) but the point isn't playback per se. It's **playback in the context of editorial decision-making**: scrubbing, marking in/out points while watching, feeling the rhythm. This requires a responsive interface.

**2. Spatial composition.**

Picture-in-picture, split screens, repositioning, scale, rotation. These are visual operations that need visual feedback. You can define coordinates in code, but you can't *see* whether the composition works without seeing it.

**3. Color grading.**

Color is haptic. A colorist works with panels, wheels, curves, responds to what they see on a calibrated monitor. The Resolve API deliberately does not expose primary color controls. This is correct. Color grading by API call is like painting by coordinates.

**4. Sound mixing.**

Audio levels, spatial positioning, dynamics -- these need to be heard in context. A CLI can set parameters, but mixing is an aural activity.

**5. The non-verbal part of editorial rhythm.**

The decision to cut 4 frames earlier. The instinct that a reaction shot needs to breathe. The feeling that the music should enter 2 seconds before the image change. These are not articulable in advance. They emerge from watching the material. No prompt engineering replaces this.

### The Cursor Analogy (And Where It Breaks)

Cursor works for coding because:
- Code is text. An LLM can read, understand, and generate text natively.
- Code correctness is verifiable (tests, type checking, compilation).
- The "review" step (reading a diff) uses the same medium as the "creation" step (writing code).

Film editing breaks this analogy because:
- A timeline is data, but the *output* (the film) is audiovisual. The agent works in one medium (data), the filmmaker evaluates in another (experience).
- Film "correctness" is not verifiable. There's no test suite for whether a cut is good.
- The review step (watching the film) is fundamentally different from the creation step (manipulating timeline data).

**So: Cursor-for-editing is not "you type intent, the agent edits, you watch the result."** That feedback loop is too slow and too lossy. By the time you render and watch, you've lost the micro-decisions that make editing an art.

**The real analogy is different.** Cursor doesn't replace the IDE. It transforms the IDE into a conversation space where the programmer directs and the agent executes. The equivalent for film editing is not removing the GUI. It's **embedding the agent into the GUI** so the filmmaker can direct the agent while watching the material.

"Move this clip 12 frames earlier." "Try the wide shot instead of the close-up here." "Make this section 20% tighter." "Find me a reaction shot of the woman from scene 4 where she looks surprised." Spoken or typed into the NLE, executed by the agent, reviewed by the filmmaker in real time. The timeline stays on screen. The filmmaker stays in the flow of watching.

### The Actual Architecture

The answer is neither pure CLI nor pure GUI. It's **hybridization**, and the protocol stack looks like this:

```
Layer 4: FILMMAKER (watches, directs, decides)
           |
           | natural language / voice / gestures
           v
Layer 3: AGENT (reasons, plans, executes)
           |
           | MCP protocol (tool calls)
           v
Layer 2: NLE MCP SERVER (translates to NLE-specific API)
           |
           | Python/Lua scripting API
           v
Layer 1: NLE ENGINE (Resolve / MLT / Blender)
           |
           | renders frames
           v
Layer 0: DISPLAY (calibrated monitor, speakers)
           |
           | photons, sound waves
           v
         FILMMAKER (perceives, judges, loops back to Layer 4)
```

The GUI doesn't disappear. But its role changes:

| Before agents | With agents |
|--------------|-------------|
| Filmmaker drags clips on timeline | Agent places clips, filmmaker reviews |
| Filmmaker sets in/out points manually | Filmmaker says "tighter" or "longer", agent adjusts |
| Filmmaker searches bins by scrolling | Agent searches by description ("find the close-up where he hesitates") |
| Filmmaker sets up render presets | Agent handles entire delivery pipeline |
| Filmmaker does QC pass manually | Agent detects flash frames, gaps, sync issues automatically |

The GUI becomes a **review and direction interface**, not a manipulation interface. Like how a director on set doesn't operate the camera but watches the monitor and gives instructions.

**Which protocol makes this possible?** MCP. It's the only protocol designed for real-time, conversational, tool-using AI agents. The NLE exposes an MCP server. The agent talks to it. The filmmaker talks to the agent. The timeline is the shared canvas.

OTIO handles the data model underneath. FCPXML handles FCP-specific roundtripping. FFmpeg/MLT handle the rendering. But MCP is the conversational bridge. It's the protocol that lets the filmmaker say "try it without the music" and have the agent mute the audio track, instantly, while the timeline keeps playing.

**This is the Cursor model adapted for film.** Not CLI editing. Not replacing the GUI. Embedding the agent into the GUI and turning the filmmaker into a director of the edit, not an operator of the edit.

### But Wait: The Multimodal Loop Changes Everything

The analysis above has a blind spot. I said the GUI is necessary because the agent can't *see* the film. **That's no longer true.**

As of early 2026, multimodal LLMs can process video:

| Model | Video Capability |
|-------|-----------------|
| **Gemini 2.0 Flash** | Processes 1+ hour of video natively, understands temporal sequence, identifies shots, actions, emotions, dialogue |
| **Gemini 2.5 Pro** | Extended context for long-form video analysis |
| **GPT-4o** | Image analysis, video frame extraction |
| **Claude** | Image/screenshot analysis (not native video yet, but frame-by-frame works) |

This closes the perception gap. Consider this workflow:

```
CLAUDE CODE CLI (or any coding agent)
  |
  | reads FCPXML as text (it's XML)
  | reads OTIO as text (it's JSON)
  v
AGENT MANIPULATES TIMELINE
  |
  | modifies XML/JSON structure
  | (insert clip, trim, reorder, add transition)
  v
FFMPEG / MELT RENDERS LOW-RES PREVIEW
  |
  | outputs .mp4 preview
  v
MULTIMODAL LLM WATCHES THE PREVIEW
  |
  | Gemini analyzes: pacing, continuity, emotional arc,
  | shot composition, audio-visual sync
  v
AGENT EVALUATES AND ADJUSTS
  |
  | "the cut at 02:15 is too abrupt"
  | "the wide shot at 03:40 breaks the intimacy"
  | "the rhythm slows down in act 2"
  v
LOOP (modify FCPXML --> render --> watch --> evaluate)
  |
  v
FILMMAKER REVIEWS FINAL RESULT
  |
  | imports FCPXML into FCP or Resolve
  | watches on calibrated monitor
  | makes final creative adjustments
  v
DONE
```

This is **exactly the Cursor loop**:
1. Agent reads code (= reads FCPXML/OTIO)
2. Agent modifies code (= modifies timeline data)
3. Tests run (= FFmpeg renders preview)
4. Agent evaluates results (= multimodal model watches the render)
5. Loop until satisfactory
6. Human reviews and approves (= filmmaker watches in NLE)

The key insight: **FCPXML is the best format for this** because:
- It's the most expressive XML timeline format (FCP's magnetic timeline model is richer than EDL or basic AAF)
- An LLM can read and write XML natively
- FCP is the most filmmaker-friendly NLE for the final review step
- The FCPXML spec is [publicly documented by Apple](https://developer.apple.com/documentation/professional-video-applications/fcpxml-reference)
- OTIO has a bidirectional FCPXML adapter, so OTIO and FCPXML are interchangeable

And **OTIO is the universal layer** because:
- OTIO is JSON (even more LLM-native than XML)
- OTIO converts to/from FCPXML, AAF, EDL
- OTIO has a Python API that a Claude Code agent can call directly
- OTIO's experimental editing commands (insert, overwrite, roll) mean the agent can use semantic editing operations, not raw XML surgery

### The Claude Code Editing Workflow

Concretely, what this looks like with Claude Code (or any CLI coding agent):

```bash
# Agent reads the current edit
cat project.fcpxml

# Agent modifies the timeline (via OTIO Python, or direct XML manipulation)
python3 -c "
import opentimelineio as otio
tl = otio.adapters.read_from_file('project.fcpxml')
# ... manipulate timeline ...
otio.adapters.write_to_file(tl, 'project_v2.fcpxml')
"

# Agent renders a low-res preview
ffmpeg -i project_v2.fcpxml -vf scale=960:540 -c:v libx264 -preset ultrafast preview.mp4
# (in practice: melt or Resolve headless for FCPXML rendering)

# Multimodal model watches the preview and evaluates
# (via API call to Gemini, or Claude vision on extracted frames)

# Agent adjusts based on evaluation, loops

# Filmmaker imports final FCPXML into FCP for review
```

The NLE becomes the **last mile**. The filmmaker opens the FCPXML in FCP, watches it on a proper monitor, makes the final 10-20% of adjustments that require human perception. The agent handled the other 80-90% iteratively.

### Where This Is Going

Three converging vectors:

**1. CLI agents get better at structured data.** Claude Code already manipulates JSON, YAML, XML, TOML. FCPXML and OTIO are just more of the same. No new capability needed -- just prompting and tool use.

**2. Multimodal context windows grow.** Gemini already handles 1+ hour of video. As context windows expand, agents will be able to "watch" entire rough cuts and reason about narrative structure across the full film. This is not hypothetical. It works today for short-form. Feature length is a scaling problem, not a capability problem.

**3. OTIO matures as a headless editing model.** The experimental editing commands (v0.16+) are becoming real. When OTIO has full insert/overwrite/ripple/roll operations with a stable API, it becomes a complete headless editing data model. Add MLT or FFmpeg as the render engine, and you have a CLI NLE.

The endpoint: **a filmmaker working in Claude Code (or its successor) the way a programmer works in Cursor.** Not staring at a timeline GUI for 12 hours a day. Directing an agent that builds, modifies, and renders edits. Reviewing the output. Giving notes. The agent executes. The NLE is the final delivery tool, not the creative workspace.

This won't replace the monteur/monteuse on a Jacques Audiard film. But it will let a filmmaker who edits their own work move 5x faster through the mechanical parts, and spend their creative energy on the decisions that actually make the film.

---

## What Editing Actually Is (And What It Isn't)

Most of the AI-editing discourse gets this wrong because it imagines editing as *operating a timeline*. Dragging clips, trimming, applying transitions. That's not editing. That's execution.

Here's what a feature film edit actually looks like in practice:

### 80% Off the Timeline

The real work of a monteur happens away from the NLE:

- **Structural discussions with the filmmaker.** Standing at a board covered in post-its, color-coded by scene/character/theme. Moving them around. Saying "what if we open with the ending?" That's the edit. The timeline is just where you execute that decision afterwards.
- **Writing.** Scene descriptions, structural notes, editing notes for the filmmaker, notes for the sound designer, VFX briefs, continuity reports. A monteur on a feature writes more than they trim.
- **Spreadsheets.** Shot lists, dailies reports, VFX tracking, delivery specs, music cue sheets. Excel is an editing tool.
- **Talking.** The relationship between filmmaker and editor is a conversation. The edit emerges from that dialogue. The best cuts come from a monteur saying "I tried something, watch this" and the filmmaker responding with their body language before they say a word.

### The Timeline Part

When you're actually on the NLE, a surprising amount is mechanical:

- Trimming. Extending or shortening a clip by a few frames.
- Reordering. Moving a scene earlier or later.
- Trying versions. "Let's try it without the music." "What if we cut the flashback?"
- Conforming. Matching proxies to originals.
- Exports. AAF for sound, XML for color, EDL for VFX, DCP for festivals.

This mechanical work is exactly what agents can handle.

### What Agents Should Actually Automate

The insight is: **the agent doesn't replace the monteur. The agent replaces the assistant editor.**

On a feature film, the assistant editor handles:

| Task | Current Method | Agent Method |
|------|---------------|-------------|
| Ingest & organize dailies | Manual folder structure, metadata entry | Script-based: read card structure, create bins by day/scene/camera, apply naming conventions |
| Proxy generation | FCP/Resolve native or FFmpeg batch | Automated: detect source format, generate matching proxies, verify integrity |
| Sync audio | Tentacle Sync, PluralEyes, manual | Automated: detect TC or waveform match, sync, verify drift |
| Transcription | Manual or Simon Says | Whisper/WhisperX: transcribe all dialogue, identify speakers, generate FCPXML with keyword ranges |
| Logging | Manually watching every take and noting content | AI vision: tag shot type, framing, who's in frame, actions, emotions. Generate FCPXML keywords and smart collections |
| Script integration | Manually matching takes to script lines | Cross-reference transcription with script, auto-link by dialogue match |
| LUT application | Manual per-clip in Resolve or batch | Automated: read camera metadata, apply correct show LUT per source |
| Build dailies | Render with burn-in (TC, clip name, date) | Automated: FFmpeg or Resolve render queue with metadata overlay |
| Roundtrips | Export AAF/FCPXML, manage versions manually | Agent tracks versions, generates correct format for each department, manages handles |
| QC | Watch everything, check for flash frames/gaps/sync | Automated scan: flash frames, gaps, audio sync drift, format compliance |
| Delivery | Manual render presets per deliverable | Agent manages delivery spec sheet, renders all formats, validates each |

**Every single one of these tasks is already automatable today** with existing tools. The problem isn't capability. It's that nobody has assembled the pipeline.

---

## AI Tools for FCP: Keywords, Smart Collections, Logging

As of February 2026, here's what exists for automated logging and annotation in Final Cut Pro:

### FCP 12 Built-in AI (January 2026)

| Feature | What It Does | Limitation |
|---------|-------------|-----------|
| **Visual Search** | Natural language search in footage: "clouds", "someone running", "a car". Local ML on Apple Silicon. | Doesn't create persistent keywords. Search-on-demand only. |
| **Transcript Search** | Transcribes source clips (not just timelines). Search by exact words or natural language. | Requires Apple Silicon + macOS 15.6. |
| **Find People** | Detects face count, infers shot type (wide/medium/close-up). Creates analysis keywords. | No individual face recognition (doesn't name people). |
| **Beat Detection** | Analyzes music for rhythm alignment. | For music videos, not general logging. |

Apple's approach: search on demand, not structured annotation. Useful for finding shots, but doesn't build a structured metadata layer.

### Third-Party Tools (Ranked by Relevance for Features)

**Eddie AI** ([heyeddie.ai](https://www.heyeddie.ai/)) -- The most advanced AI logging tool.
- Analyzes rushes locally, creates summaries, identifies soundbites, classifies shot types
- Groups clips by content, generates hierarchical bins
- Exports FCPXML with transcript ranges searchable in FCP
- Rough cut generation from natural language prompts
- Supports Premiere, Resolve, and FCP
- Limitation: max 10 A-roll or 50 B-roll clips per project

**Lumberjack System** ([lumberjacksystem.com](https://www.lumberjacksystem.com/)) -- The professional reference for doc logging.
- Real-time logging on set (iOS app), keywords by category (Locations, People, Activities)
- Free transcription in 16 languages
- Builder NLE: text-based editing, export to FCP as timeline
- Multi-logger support (several people logging simultaneously)

**FCP Video Tag** (Ulti.Media) -- Generates FCPXML with keywords from vision AI.
- MobileNet neural network analyzes footage frame by frame
- Object recognition + OCR + audio transcription (Whisper, 40+ languages)
- Tag Manager with correspondence rules ("child" detected → also add "person")
- Exports FCPXML with keywords attached to clips
- Fully local processing

**Jumper** (Witchcraft Software) -- AI search engine as FCP Workflow Extension.
- Natural language search inside FCP, reads library files directly
- Better search results than FCP 12's Visual Search in comparative tests
- No keywords generated, search-only

**Peakto** (CYME) -- DAM with local AI, FCP integration.
- Indexes FCP libraries, generates transcriptions, detects people
- Suggests keywords (accept/reject), aesthetic analysis
- Exports Video Bins as XML to FCP, Resolve, Premiere
- NAB Show 2025 Product of the Year

**Simon Says** ([simonsaysai.com](https://www.simonsaysai.com/)) -- Transcription with native FCP integration.
- FCP Workflow Extension, 100 languages, ~95% accuracy
- Returns speaker-labeled ranges with favorites and notes into FCP clips
- On-premise option for security-sensitive productions (SOC2, TPN)
- Assembly mode: build sequences by selecting text passages

### Smart Collections via FCPXML

Apple's FCPXML spec (v1.12, FCP 12) fully supports programmatic smart collection creation:

```xml
<smart-collection name="Close-ups Scene 4" match="all">
  <match-keywords enabled="1" rule="includesAll" value="close-up, scene-4"/>
</smart-collection>

<smart-collection name="All Dialogue Takes" match="any">
  <match-keywords enabled="1" rule="includesAny" value="dialogue, interview"/>
</smart-collection>
```

An agent can:
1. Analyze rushes (via Whisper for dialogue, via vision model for shot type/content)
2. Generate FCPXML with keywords per clip and keyword ranges per segment
3. Define smart collections based on the keywords
4. Import into FCP → the entire logging structure appears automatically

The FCPXML DTD for `smart-collection` supports rules: `includesAny`, `includesAll`, `doesNotIncludeAny`, `doesNotIncludeAll`. Combined with `match-text` (search in notes), `match-ratings`, `match-media` (format, framerate), and `match-time` (date ranges), smart collections can be as precise as needed.

**Tools for generating FCPXML programmatically:**
- Python `xml.etree.ElementTree` + [Apple FCPXML DTD](https://developer.apple.com/documentation/professional-video-applications/fcpxml-reference) (the most reliable approach)
- [Pipeline Neo](https://github.com/TheAcharya/pipeline-neo) (Swift, professional framework)
- [bmjs-fcpxml](https://github.com/brent258/bmjs-fcpxml) (Node.js)
- [fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) -- 34 tools, can be used by Claude directly

### The Complete Assistant Editor Agent Pipeline

Assembling all these tools into a single automated pipeline:

```
CAMERA CARDS ARRIVE
  |
  v
INGEST (Silverstack v9 or Hedge)
  - Checksum-verified copy
  - Automatic folder structure by camera/day/scene
  - Metadata preservation
  |
  v
PROXY GENERATION (FFmpeg batch or FCP native)
  - Match source format
  - ProRes Proxy or H.264 at 50%
  - Preserve timecode and reel names
  |
  v
AUDIO SYNC (Tentacle Sync Studio or Syncaila)
  - TC-based sync from Tentacle devices
  - Waveform match as fallback
  - Verify sync with sample check
  |
  v
AI ANALYSIS (parallel agents)
  |
  +-- WHISPER/WHISPERX: transcribe all dialogue
  |   - Speaker diarization
  |   - Word-level timestamps
  |   - Output: SRT + keyword ranges
  |
  +-- VISION MODEL (Gemini / GPT-4o / local model):
  |   - Shot type (CU/MCU/MS/WS/EWS)
  |   - Who's in frame (face detection → character names)
  |   - Actions and emotions
  |   - Location/setting tags
  |   - Output: keyword list per clip
  |
  +-- SCRIPT MATCHER: cross-reference transcription with script
      - Match takes to scene/line numbers
      - Note deviations from scripted dialogue
      - Output: script-scene keywords
  |
  v
FCPXML GENERATION (Python + pyFcpxmlCreator)
  - Create Event with proper structure
  - Attach keywords per clip (shot type, characters, location, scene)
  - Attach keyword ranges for dialogue segments
  - Define Smart Collections:
    - By character (all clips with X)
    - By scene number
    - By shot type
    - By emotion/action tags
  - Attach transcription as clip notes
  |
  v
FCP IMPORT
  - Filmmaker opens FCP, imports FCPXML
  - All clips are organized, keyworded, searchable
  - Smart Collections ready to use
  - Transcripts searchable via Transcript Search
  |
  v
DAILIES (Resolve headless or FFmpeg)
  - Show LUT applied per camera
  - Burn-in: source TC, clip name, date, scene/take
  - Render for filmmaker review
  |
  v
FILMMAKER EDITS
  - Structured material, everything findable
  - The assistant editor work is done
  - Focus on creative decisions
```

### Sound Department Automation

Sound is often the most time-consuming roundtrip in post-production:

| Task | Tool | Notes |
|------|------|-------|
| **Roles in FCP** | iXML from Sound Devices/Zoom | Professional recorders transmit channel labels that FCP reads as Roles/SubRoles automatically |
| **Export for mix** | [X2Pro5](https://x2pro.net/) (Marquis) | FCPXML → AAF/MXF for Pro Tools. Preserves L-cuts, J-cuts, transitions, levels, fades |
| **Reconform after recut** | [Matchbox](https://www.thecargocult.nz/products/matchbox/) (Cargo Cult) | The dominant tool. Compares old/new AAF, sends only changes to Pro Tools. Emmy Award-winning. |
| **Stems export** | FCP Roles → separate files | D/M/E stems via role-based export for international delivery |

### What's Still Missing

| Gap | Status |
|-----|--------|
| Automatic audio classification (dialogue/ambience/SFX) | Research exists, no commercial tool |
| True script lining (matching takes to script with continuity notes) | Still human work |
| Single unified tool for the complete pipeline | Doesn't exist; it's an assembly of parts |
| AI that understands narrative structure of rushes | Eddie AI is first step, not feature-film grade yet |

---

## My Assessment

### What Works Today

1. **Agent-assisted dailies and organization.** An agent with the Resolve MCP server can import footage, create bins, add markers, and generate string-outs. This is assistant editor work. It works now.

2. **OTIO-based rough cut generation.** An agent can read transcriptions, metadata, or descriptions, build an OTIO timeline, and export it to any NLE. The filmmaker imports it as a starting point.

3. **Automated delivery.** Setting up render presets, creating DCPs, managing multiple deliverables -- this is entirely scriptable in Resolve. An agent can handle the entire delivery pipeline.

4. **FCPXML analysis and QC.** The FCP MCP server can detect flash frames, find gaps, check durations, generate EDLs. Quality control at speed.

5. **Format conversion.** OTIO as a hub: read an EDL, convert to FCPXML, import into Resolve, export as AAF for Avid. An agent can orchestrate multi-format workflows.

### What's Coming (2026-2027)

1. **MCP becomes standard** for AI-NLE communication. Blackmagic will eventually acknowledge it. Apple might integrate it into FCP's Workflow Extensions.

2. **OTIO editing commands mature** into a viable headless NLE data model. The experimental API stabilizes. Color management (CDL, LUT references) lands in the schema.

3. **Resolve extends its scripting API.** Community pressure is mounting. Color page parameters and timeline manipulation are the most requested additions.

4. **The "assistant editor" agent becomes a standard production role.** Handles logging, selects, string-outs, rough assembly. Human editor makes all final decisions.

5. **Multi-agent systems** move from academic research ([FilmAgent](https://github.com/HITsz-TMG/FilmAgent), SIGGRAPH Asia 2024) into production tools. A director agent, an editor agent, a colorist agent, each with different expertise and tools.

### What Will Never Work (And Shouldn't)

1. **An agent making creative editorial decisions for an auteur film.** The cut is the film. The timing of a cut, the decision to hold on a face for three seconds longer, the choice to cut away at the moment of violence -- these are directorial decisions. They encode the filmmaker's vision. An agent that makes these decisions is not editing. It's generating content.

2. **Replacing the colorist with an API call.** Color grading at the level of a theatrical feature is haptic, intuitive, responsive to the material in real time. The Resolve scripting API deliberately does not expose color wheels and curves. This is correct.

3. **Full automation of the edit.** The films that a16z imagines when they write "agentic video editing" are not auteur cinema. They're YouTube videos, corporate content, social media. Different universe. At the level of a Cesar-winning film, the agent is a tool, never the editor.

4. **One protocol to rule them all.** MCP, OTIO, REST APIs, FCPXML -- they'll coexist. The stack will be hybrid. Anyone promising a single solution is selling something.

---

## For Auteur Cinema

Here's the workflow I see emerging for auteur filmmakers who want to integrate AI agents:

### The Hybrid Model

```
SHOOT (iPhone / R6 III / BMPCC 6K Pro / 5D3 ML RAW)
  |
  v
INGEST + ORGANIZE (Agent via Resolve MCP)
  - Import media, create bins by day/scene/camera
  - Generate proxies
  - Sync audio
  - Add technical markers (exposure, focus issues)
  |
  v
TRANSCRIBE + INDEX (Agent via Whisper / external tools)
  - Transcribe all dialogue
  - Index footage by content, emotion, visual description
  - Build a searchable database of every shot
  |
  v
ROUGH CUT GENERATION (Agent via OTIO)
  - Agent reads the script and transcriptions
  - Proposes a rough assembly based on script structure
  - Generates OTIO timeline, exports to FCPXML
  |
  v
EDITORIAL (Filmmaker in FCP or Resolve)
  - Import the agent's rough cut as a starting point
  - Edit the film: make every creative decision
  - The agent didn't make the film. It prepared the material.
  |
  v
CONFORM + GRADE (Resolve, human colorist or filmmaker)
  - Online conform with CinemaDNG / BRAW / Canon RAW
  - CST sandwich workflow (see main README)
  - Color grading: human hands on the controls
  |
  v
DELIVERY (Agent via Resolve MCP)
  - DCP creation
  - IMF packaging (if required)
  - Multiple format renders
  - QC checks
```

### What This Gives the Filmmaker

Not replacement. **Acceleration of the mechanical parts** so you spend more time on the creative decisions. An auteur director who also edits (which is most of us) loses weeks on organization, transcoding, syncing, proxy generation, delivery formatting. Those weeks are the agent's job.

The creative work -- the cut, the rhythm, the emotional architecture of the film -- stays with the filmmaker.

### Which Protocol

For this hybrid model, the stack is:
- **MCP** for real-time NLE control (Resolve, eventually FCP)
- **OTIO** for timeline generation, analysis, and interchange
- **FCPXML** for FCP roundtripping
- **Python** as the glue

The protocol isn't one thing. It's a stack. MCP provides the agent-to-NLE bridge. OTIO provides the timeline data model. FCPXML provides FCP-specific interop. Python connects everything.

### The Real Question

The question isn't "can an AI edit a film?" The question is: **what editorial operations are mechanical enough to automate, and what operations require human vision?**

Organization, syncing, transcoding, format conversion, QC, delivery: mechanical. Automate them.

Shot selection, pacing, structure, rhythm, emotional weight, when to cut and when to hold: vision. Keep them human.

The best AI editing workflow for auteur cinema is the one where you forget the AI is there. It handled the infrastructure. You made the film.

---

## Key Resources

### MCP Servers
- [samuelgursky/davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp) -- Most complete Resolve MCP
- [apvlv/davinci-resolve-mcp](https://github.com/apvlv/davinci-resolve-mcp) -- Fusion integration
- [DareDev256/fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) -- 34 tools for FCP

### OpenTimelineIO
- [GitHub](https://github.com/AcademySoftwareFoundation/OpenTimelineIO) -- ~1,800 stars, ASWF project
- [Documentation](https://opentimelineio.readthedocs.io/en/latest/)
- [PyPI](https://pypi.org/project/OpenTimelineIO/) -- v0.18.1
- [Raven viewer](https://github.com/OpenTimelineIO/raven) -- OTIO timeline viewer

### Scripting and Wrappers
- [pydavinci](https://github.com/pedrolabonia/pydavinci) -- Pythonic Resolve API wrapper
- [CommandPost](https://commandpost.fcp.cafe/) -- FCP automation (Lua)
- [editly](https://github.com/mifi/editly) -- Declarative JSON-to-video via FFmpeg
- [MLT Framework](https://www.mltframework.org/) -- Headless multimedia engine

### Research
- [FilmAgent](https://github.com/HITsz-TMG/FilmAgent) (SIGGRAPH Asia 2024) -- Multi-agent film production
- [Prompt-Driven Agentic Video Editing](https://arxiv.org/abs/2509.16811) (Memories.ai) -- Semantic narrative editing
- [a16z: It's Time for Agentic Video Editing](https://a16z.com/its-time-for-agentic-video-editing/)
- [Hi! PARIS: AI Cinema -- "World + Observer"](https://hi-paris.fr/2026/02/04/ai-cinema-are-we-ready-for-world-observer/)

### NLE Documentation
- [DaVinci Resolve API (unofficial)](https://deric.github.io/DaVinciResolve-API-Docs/)
- [Blender Python API](https://docs.blender.org/api/current/index.html)
- [Apple FCPXML Reference](https://developer.apple.com/documentation/professional-video-applications/fcpxml-reference)
- [Adobe Premiere Pro Scripting Guide](https://ppro-scripting.docsforadobe.dev/)

---

## License

This document is released under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
