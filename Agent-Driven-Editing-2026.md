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
| [Adobe Premiere Pro MCP](https://playbooks.com/mcp/adobe-premiere-pro) | Early stage | Many tools scaffolded but unimplemented. UXP scripting experimental |

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
