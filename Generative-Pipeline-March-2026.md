# Generative Pipeline for Auteur Documentary — March 2026

**State of the art, March 2026. Research session: NLE selection, semantic search, generative stack, world models, audio AI, reverse-engineering Martini.**

> This document extends [Agent-Driven-Editing-2026.md](./Agent-Driven-Editing-2026.md) with updated findings from March 2026. Focus: choosing the right NLE for AI-driven post-doc, semantic footage analysis, generative video/audio pipeline, and what it means to build a Martini-equivalent open source stack.

---

## Table of Contents

- [NLE Selection — The Real Comparison](#nle-selection--the-real-comparison)
- [Semantic Footage Analysis](#semantic-footage-analysis)
- [Generative Video Pipeline (Reverse Martini)](#generative-video-pipeline-reverse-martini)
- [World Models — What's Actually Accessible](#world-models--whats-actually-accessible)
- [Audio AI — SOTA March 2026](#audio-ai--sota-march-2026)
- [FCP Scripting Ecosystem](#fcp-scripting-ecosystem)
- [Open Source Pépites](#open-source-pépites)
- [Complete Stack](#complete-stack)

---

## NLE Selection — The Real Comparison

### The Question

Which NLE is best for a filmmaker who wants to pilot post-production from Claude Code — and who does narrative documentary, not content creation?

### Benchmark: Agent Control Capability

| NLE | Native API | MCP Server | Text-Based Editing | Visual Search | One-Time Cost |
|---|---|---|---|---|---|
| **DaVinci Resolve 20 Studio** | Python + Lua (official Blackmagic) | [samuelgursky/davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp) | ✅ Native (transcript visible + exportable) | ❌ None native | $295 |
| **Final Cut Pro 12** | FCPXML + AppleScript limited | [DareDev256/fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server) (47 tools, Jan 2026) | ⚠️ Transcript Search only (no edit via text) | ❌ 12/92 on benchmark | $299 |
| **Premiere Pro** | ExtendScript (official) | [hetpatel-11/Adobe_Premiere_Pro_MCP](https://github.com/hetpatel-11/Adobe_Premiere_Pro_MCP) (97 tools) | ✅ Best workspace | Jumper plugin | ~$60/month subscription |
| **Pallaidium / Blender VSE** | Python complete | No dedicated MCP | ❌ | ❌ | Free (Windows + Nvidia only) |

### The Fundamental Difference: API vs FCPXML Roundtrip

DaVinci Resolve's Python API accesses the application's internal state in live memory: MediaPool objects, Timeline clips, Color nodes, timecodes. You read and write to the running application directly.

FCP has no equivalent. Everything goes through FCPXML — export the timeline to XML, modify it with Claude, re-import. Functional for batch operations (markers, rough cuts, reordering), but no live state access, no color node control, no automated render.

```
Resolve: Claude → Python API → running Resolve instance (live)
FCP:     Claude → FCPXML file → import into FCP (roundtrip)
```

### FCP Scripting Ecosystem (Not Nothing)

FCP has a real scripting ecosystem that wasn't well documented until recently:

- **[CommandPost](https://commandpost.fcp.cafe/)** — macOS app (Hammerspoon + Lua). Exposes `cp.finalcutpro` Lua API. Controls FCP at UI level: clip selection, lane navigation, effects, batch rename, CSV export of timeline/browser. CLI: `cmdpost -c 'lua code'`. Requires one paid LateNite app (min ~$10) for v2.0 with FCP 12 subscription support.

- **[fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server)** — 47 MCP tools (Jan 2026, built by a music video director). Markers, rough cuts, A/B rolls from markers, EDL/SRT/shot list export, FCPXML → Resolve v1.9 conversion. Serious batch automation.

- **[applescript-mcp](https://github.com/peakmojo/applescript-mcp)** — executes any AppleScript from Claude Code. Combined with CommandPost's AppleScript bridge, gives Claude Code live UI control of FCP.

Chain: `Claude Code → applescript-mcp → AppleScript → CommandPost Lua → FCP`

**What FCP can do with this stack vs what it can't:**

| Operation | FCP + CommandPost + FCPXML-MCP | Resolve Python API |
|---|---|---|
| Batch markers | ✅ | ✅ |
| Rough cut assembly | ✅ (FCPXML roundtrip) | ✅ (live) |
| Live state read | ❌ | ✅ |
| Color node control | ❌ | ✅ |
| Automated render/export | ⚠️ Fragile AppleScript | ✅ |
| Stability | UI scripting = fragile | Official API = stable |

**Verdict for narrative documentary:** Resolve Studio is objectively better for Claude Code automation. FCP covers ~70% of structural/narrative operations (markers, selects, rough cuts) but collapses when you need color, render automation, or live state. If you're committed to FCP for editing feel, use FCP + Jumper + fcpxml-mcp-server + CommandPost. If you want the full pipeline, Resolve Studio.

---

## Semantic Footage Analysis

For documentary filmmakers, finding the right moment in hours of rushes is the core problem. Several tools solve different parts of it.

### Visual Semantic Search

**[Jumper](https://getjumper.io)** — the most important tool here. Native plugin for FCP, Premiere Pro, and **DaVinci Resolve Studio** (Workspace > Workflow Integrations, live as of March 2026). Natural language visual search: "guy in green t-shirt", "wide shot of empty street", "Find Similar" from a selected clip.

Benchmark (46 searches, nouns/verbs/locations/shot types):
- Jumper: **91.3%**
- Premiere Pro Media Intelligence: 68/92
- FCP 12 Visual Search: **12/92**
- Resolve 20 native: ❌ (no visual search — Jumper fills this gap)

Price: **$249 lifetime** (or $149/year).

### The Three Axes of Footage Search

These tools operate on three orthogonal axes. None replaces the others.

**Axis 1 — Visual content (what the image shows)**

**[Jumper](https://getjumper.io)** — finds moments by what appears on screen. "Man sitting alone", "wide shot empty street", "gun in frame". No transcription involved. 91.3% accuracy. Native in FCP, Premiere, Resolve Studio.

**FCP 12 Visual Search** — same axis, built-in, on-device Apple Silicon. Benchmark: 12/92. Weak on specific subjects, decent for broad categories.

**Axis 2 — Speech content (what was said)**

**[StoryToolkitAI](https://github.com/octimot/StoryToolkitAI)** — offline Whisper transcription (local, free) + semantic search in transcripts + story assembly + EDL/FCPXML export. Deep Resolve Studio integration: bidirectional markers, real-time timeline navigation. Engine: **Whisper** (local, no cloud). Built by a documentary editor for his own cutting room. For Goldberg: "find every moment Joshua talks about Mike", "find every time he laughs", "find his denials". Outputs timecodes directly into Resolve.

**Premiere Pro Text-Based Editing** — best text-based editing workflow of any NLE. Cut the film by editing the transcript as a text document; cuts appear in the timeline. Transcript is local via Adobe Sensei.

**Axis 3 — Multimodal contextual analysis (image + sound + meaning simultaneously)**

**VHS Analyzer** (custom tool, `goldberg/tools/vhs-analyzer/`) — Gemini Flash 3.1 as engine. Analyzes image and audio together with narrative context: "Joshua age 8 holds a toy gun, mother looks away." Understands cinematic significance, not just what's present. 4-phase pipeline: preprocessing → semantic analysis → synthesis → **FCPXML export**. Cost: ~$X/hour of footage via Gemini API. Built specifically for The Goldberg Variations.

**Gemini Flash 3.1 ad-hoc** — same multimodal capability without the pipeline. For one-off questions on a specific clip. No timecode output, no NLE export.

**[Twelve Labs](https://twelvelabs.io)** — cloud multimodal search at scale. $0.033/min. Useful for large rushes libraries you don't want to process locally.

### FCP 12 AI Features (January 2026)

- **Visual Search** — natural language, on-device Apple Silicon, local. Benchmark: 12/92 (vs Jumper 91.3%)
- **Transcript Search** — local Whisper, ~9.5h transcribed in 5 min on M3 Max. US English primarily. Search finds timecode but doesn't allow cutting via text (that's Premiere's edge)
- **Beat Detection** — analyzes music for beat grid, snap cuts to rhythm

### Premiere Pro AI Features

- **Text-Based Editing** — best-in-class. Edit the transcript like a Word doc, cuts execute in timeline
- **Media Intelligence** — scene/object/face detection. Faster than Jumper but 68/92 accuracy
- **Scene Edit Detection** — finds cuts in consolidated footage automatically
- **Jumper plugin** — native, 91.3% visual search

### Parakeet v3 (NVIDIA, CoreML)

Parakeet RNNT-1.1B runs locally on Apple Silicon via CoreML. Faster than Whisper at equivalent quality on **English**. MacWhisper supports it as backend. For Joshua Ryne Goldberg's recordings (English speaker): Parakeet v3 > Whisper on speed and accuracy. For French or mixed-language content: Whisper large-v3 remains superior.

### Comparison

| Tool | Sees image | Sees speech | NLE export | Offline | Cost |
|---|---|---|---|---|---|
| **Jumper** | ✅ 91.3% | ✅ (speech search) | ✅ FCP/Premiere/Resolve | ✅ | $249 one-time |
| **StoryToolkitAI** | ❌ | ✅ Whisper local | ✅ EDL + Resolve native | ✅ | Free |
| **VHS Analyzer** (custom) | ✅ (+ context) | ✅ (+ meaning) | ✅ FCPXML | ❌ Gemini API | Pay-per-use |
| **FCP 12 Visual Search** | ✅ (12/92) | ✅ (transcript only) | Native | ✅ | Included |
| **Premiere Text-Based** | ❌ | ✅ (cut via text) | ✅ best-in-class | ✅ | Subscription |
| **Gemini Flash 3.1** | ✅ | ✅ | ❌ | ❌ | Pay-per-use |
| **Twelve Labs** | ✅ | ✅ | EDL via Jockey | ❌ | $0.033/min |

### For Goldberg Specifically

Four non-redundant layers:

```
VHS family footage      → VHS Analyzer (Gemini Flash, FCPXML) ← already built
Joshua interview rushes → StoryToolkitAI (Whisper offline, find what he said → Resolve)
Terrain visual rushes   → Jumper (find what appears on screen → Resolve)
Director voice memos    → MacWhisper + Parakeet v3 → Voice Memo Pipeline → SYNTHESIS.md
```

---

## Generative Video Pipeline (Reverse Martini)

### What Martini Actually Is

[Martini.film](https://www.martini.film/) (YC W26, founded by a cinematographer) is a model-agnostic orchestrator with cinema UX: multi-model generation (Veo 3.1, Kling 3.0, Sora 2, Seedance 1.5, Nano Banana Pro, Flux 2 Max), camera control, web-based collaborative timeline, XML export to Resolve/Premiere/FCP. 200+ films produced.

It's not a novel technology — it's an interface built for how filmmakers think. The tech underneath is all available open source or via API.

### Component-by-Component Replacement

#### 1. Video Generation

**Local (ComfyUI):**

| Model | Hardware | Best for |
|---|---|---|
| Wan 2.2 14B | 16GB VRAM | Quality generation, camera control native |
| Wan 2.2 1.3B | 8GB VRAM | Fast iteration |
| LTX-2 (Lightricks, Apache 2.0) | 12GB VRAM | Video + audio sync native, 4K/50fps |
| SkyReels V2/V3 | 16GB VRAM | Long-duration, cinematic (trained on 10M film/TV clips) |

**Cloud (fal.ai, pay-per-use, no subscription):**

| Model | Cost | Best for |
|---|---|---|
| Wan 2.6 | ~$0.05/sec | General generation |
| Kling 2.6 Pro | ~$0.07/sec | Camera motion precision |
| Kling 3.0 | ~$0.10/sec | Highest quality |
| Veo 3.1 (+ audio) | ~$0.20/sec | Best overall + audio sync |

```python
import fal_client, urllib.request

# Wan 2.6 img2vid
result = fal_client.subscribe(
    "fal-ai/wan/v2.2-5b/image-to-video",
    arguments={
        "image_url": fal_client.upload_file("/path/to/frame.jpg"),
        "prompt": "slow dolly backward, 16mm grain, shallow DOF",
        "resolution": "720p",
        "num_frames": 81,
        "fps": 24,
    }
)
urllib.request.urlretrieve(result["video"]["url"], "shot_001.mp4")
```

#### 2. Camera Control

**In ComfyUI — Wan 2.2 Fun Camera** (easiest entry point):

```
LoadImage → Load Diffusion Model (wan2.2_fun_camera_HIGH_noise)
          → WanCameraEmbedding [camera_motion: "Zoom In" | "Pan Left" | "Zoom Out" | ...]
          → WanVideoSampler (HIGH noise, 35-40 steps)
          → WanVideoSampler (LOW noise, 10-15 steps)
          → VAEDecode → SaveVideo
```

Official workflow JSON (drag into ComfyUI): https://docs.comfy.org/tutorials/video/wan/wan2-2-fun-camera

**ReCamMaster** — reshoot an existing video from a new camera trajectory:
- Install via [kijai/ComfyUI-WanVideoWrapper](https://github.com/kijai/ComfyUI-WanVideoWrapper)
- Model: `Wan2_1_kwai_recammaster_1_3B_step20000_bf16.safetensors` from HuggingFace
- Workflow: [wanvideo_1_3B_ReCamMaster_example_01.json](https://github.com/kijai/ComfyUI-WanVideoWrapper/blob/main/example_workflows/wanvideo_1_3B_ReCamMaster_example_01.json)
- Note: uses Wan2.1 (Kuaishou's internal model is better but closed)

**[TrajectoryCrafter](https://github.com/TrajectoryCrafter/TrajectoryCrafter)** — 6DoF redirection on monocular video (ICCV 2025 Oral). Requires 28GB VRAM.

**[GEN3C](https://github.com/nv-tlabs/GEN3C)** — NVIDIA, 3D cache-informed camera control. Most geometrically precise.

#### 3. Bridge to NLE

**Falafel** — Electron bridge from fal.ai directly to Resolve Media Pool: https://heyyanshuman.com/posts/falafel_launch (beta, check https://github.com/getfalafel)

**Python direct import:**

```python
import DaVinciResolveScript as dvr_script

resolve = dvr_script.scriptapp("Resolve")
media_pool = resolve.GetProjectManager().GetCurrentProject().GetMediaPool()
media_pool.ImportMedia(["/path/to/shot_001.mp4"])
```

**Via MCP (natural language):**
```
"import /Users/.../shot_001.mp4 into the media pool"
"create timeline Goldberg_seq_01 from imported clips"
"add a red marker at 00:01:23:12 labeled 'check pacing'"
```

#### 4. davinci-resolve-mcp Setup

```bash
git clone https://github.com/samuelgursky/davinci-resolve-mcp.git
cd davinci-resolve-mcp
./install.sh  # generates config with absolute paths
```

Add to `~/.claude.json`:
```json
{
  "mcpServers": {
    "davinci-resolve": {
      "command": "/absolute/path/venv/bin/python",
      "args": ["/absolute/path/resolve_mcp_server.py"]
    }
  }
}
```

Requires DaVinci Resolve **Studio** running. Free version does not expose the scripting API.

### The Architecture

```
Claude Code
  ├── davinci-resolve-mcp → DaVinci Resolve Studio
  │     ├── text-based editing
  │     ├── Fairlight (audio)
  │     └── color + deliver
  │
  ├── ComfyUI REST API → Wan 2.2 / LTX-2 / ReCamMaster (local)
  │
  ├── fal.ai Python SDK → Kling 3.0 / Veo 3.1 (cloud, pay-per-use)
  │
  └── World Labs Marble API → 3D environments (~$1.20/world)
```

### What Martini Does That This Doesn't

The real USP of Martini is the interface — a Figma-like collaborative canvas built by a cinematographer, where you think in shots not prompts. The tech underneath this stack covers ~80% of Martini's capabilities. What's missing: the collaborative canvas, the polished UX, and the unified interface. For a solo filmmaker, this is irrelevant. The gap is real but not blocking.

---

## World Models — What's Actually Accessible

### March 2026 Landscape

| Tool | Access | Price | Footage | Filmmaker use |
|---|---|---|---|---|
| **Genie 3** (DeepMind) | US only, Google AI Ultra | $250/month | Not tested | Inaccessible from Paris |
| **World Labs Marble API** | Open | ~$1.20/world | Video → 3D | Spatial previs, installations |
| **SkyReels V3** | Open source | Free (self-host) | Yes (films/TV) | Long-duration, talking avatar |
| **SkyReels V4** | Preview | API mid-March | Yes | Video + audio 1080p/32fps |
| **MAGI-1** (Sand AI) | Open source | Free | Yes | Streaming autoregressive |
| **ReCamMaster** | Open source | Free | Yes | Camera reframe on existing footage |
| **TrajectoryCrafter** | Open source | Free (28GB VRAM) | Yes | 6DoF camera redirect |
| **GEN3C** (NVIDIA) | Open source | Free | Yes | 3D-informed camera control |

### World Labs Marble API

Image/video/text → navigable 3D environment. Export: Gaussian splats, mesh, video. SDK: Python + JavaScript.

```python
# docs.worldlabs.ai/api
import worldlabs

client = worldlabs.Client(api_key="your_key")
world = client.worlds.generate(
    prompt="abandoned factory, grey morning light, debris on floor",
    quality="standard"  # 1500 credits = ~$1.20
)
world.download(format="gaussian_splat", path="./world_001.ply")
```

Render the splat with the open-source **Spark** library (Three.js). Tested by Escape.ai for turning 2D films into navigable 3D spaces.

---

## Audio AI — SOTA March 2026

### Voice Cloning

**[Chatterbox](https://github.com/resemble-ai/chatterbox)** (Resemble AI, MIT) — current open-source SOTA for voice cloning. 63.75% preference over ElevenLabs in blind tests. Zero-shot from 5-10 seconds of reference audio. Emotion exaggeration control (first open-source model with this). 23 languages.

**[Higgs Audio V2.5](https://www.boson.ai/blog/higgs-audio-v2.5)** (BosonAI, Apache 2.0) — pretrained on 10M hours of audio. Multi-speaker native, simultaneous speech + background music generation. Beats GPT-4o-mini-TTS on EmergentTTS-Eval (75.7% win rate).

F5-TTS is no longer SOTA in quality but remains unbeatable in speed (33x realtime).

### TTS Expressive

| Model | Params | License | Strength |
|---|---|---|---|
| **Sesame CSM** | 1B | Apache 2.0 | Conversational context, consistent voice over time |
| **Orpheus 3B** | 3B | Open | Prosody, emotional expression |
| **Dia** (Nari Labs) | 1.6B | Open (EN only) | Multi-speaker, nonverbal tags `(laughs)` `(gasps)` |
| **Kokoro 82M** | 82M | Apache 2.0 | 96x realtime, edge computing |

### Video → Audio (Sync)

**[MMAudio](https://github.com/hkchengrex/MMAudio)** (UIUC + Sony AI, MIT, CVPR 2025) — generates synchronized audio from video. 157M params, 8 seconds in 1.23 seconds. Three model sizes. SOTA video-to-audio among public models.

```python
# ~$0.006/run on Replicate
import replicate
output = replicate.run(
    "hkchengrex/mmaudio:latest",
    input={"video": open("shot_001.mp4", "rb")}
)
```

### Sound Design

| Tool | Use | Price |
|---|---|---|
| **AudioCraft AudioGen** (Meta, OSS) | SFX + ambiance from text | Free (local GPU) |
| **ElevenLabs SFX** | Quick precise effects | Freemium |
| **MusicGen** (Meta, OSS) | Music from text/melody reference | Free (local GPU) |
| **Suno v5** | Full tracks up to 8 min, stems, sample-to-song | Pro/Premier subscription |

### Dialogue Cleanup

**Adobe Enhance Speech V2** — free, best for field recordings (exterior noise, phone audio). Browser: podcast.adobe.com/enhance (larger model than Premiere plugin).

**iZotope RX 11** ($400 standard) — professional standard. Dialogue De-Reverb, Spectral Repair, Dialogue Isolate V11. For heavy reverb or complex artifacts.

Workflow: Enhance Speech V2 as first pass (fast, free, excellent on exteriors). RX 11 for problem cases.

---

## FCP Scripting Ecosystem

Documented here because it's frequently underestimated. FCP has a serious scripting ecosystem centered on FCPXML and CommandPost.

### Key Projects

**[fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server)** (Jan 2026, MIT) — 47 MCP tools via FCPXML roundtrip. Timeline analysis, QC, markers, rough cuts, A/B rolls, EDL/SRT export, FCPXML → Resolve v1.9 conversion. Built by a music video director (350+ projects). 7000 lines of Python, 501 passing tests.

**[CommandPost](https://commandpost.fcp.cafe/)** — macOS app (Hammerspoon + Lua). `cp.finalcutpro` API: clip selection, lane navigation, effects, batch rename, CSV export. CLI: `cmdpost -c 'lua code'`. v2.0.1 supports FCP 12 subscription.

**[applescript-mcp](https://github.com/peakmojo/applescript-mcp)** — executes any AppleScript from Claude Code.

```json
// ~/.claude.json
{
  "mcpServers": {
    "applescript": {
      "command": "npx",
      "args": ["@peakmojo/applescript-mcp"]
    }
  }
}
```

**FCPXML Python libraries:**
- [fcpxml-subtitle-generator](https://github.com/taejun0622/fcpxml-subtitle-generator) — Whisper → FCPXML subtitles
- [pyFcpxmlCreator](https://github.com/zeibou/pyFcpxmlCreator) — generate FCPXML programmatically
- [Text_To_Video_Edits-FCP-Python](https://github.com/DmytroNorth/Text_To_Video_Edits-FCP-Python) — cuts from text instructions

### The Honest Assessment

FCP + this stack covers ~70% of what you'd do with Resolve Python API for narrative/structural work. The gap is live state access, color node control, and stable render automation. For a filmmaker who wants structural automation (rough cuts, markers, selects reels) and stays in FCP for the actual editing feel, this is a real option — not a workaround.

---

## Open Source Pépites

Projects that aren't widely discussed but matter.

### [ViMax](https://github.com/HKUDS/ViMax) — Multi-Agent Filmmaking Pipeline

2361 stars, MIT, last updated Feb 2026. Multi-agent Python system: Script Agent, Storyboard Agent, Consistency Agent (RAG for character visual reference across scenes), Video Generator Agent. Full pipeline from idea to multi-shot clip. Most concrete agentic filmmaking project available.

### [Narrative Context Protocol](https://github.com/narrative-first/narrative-context-protocol) — The MCP for Storytelling

MIT license, backed by USC Entertainment Technology Center and Write Brothers (Dramatica, Movie Magic Screenwriter). JSON schema for transporting narrative intent between agents without semantic drift. Encodes Storyform: plot, genre, theme, character arcs. If you build a multi-agent pipeline that touches story structure, this is the standard to use.

[arXiv paper](https://arxiv.org/abs/2503.04844)

### [StoryToolkitAI](https://github.com/octimot/StoryToolkitAI) — The Documentary Editor's Tool

Built by a documentary editor for his own cutting room. Not a product — a working tool. Whisper transcription + semantic search + story assembly + NLE export. Most immediately useful for documentary work with hours of interview rushes.

### [Open-Sora 2.0](https://github.com/hpcaitech/Open-Sora) — Fine-Tunable Video Foundation

11B params, full open source (weights + training code). Supports 2.39:1 (CinemaScope) aspect ratio. VBench: reduces gap with Sora from 4.52% to 0.69%. If you want to fine-tune a video model on your own footage style, this is the base.

### [FilMaster](https://filmaster-ai.github.io/) — Cinema Language from 440k Clips

ICLR 2026 (HKU + Kuaishou + Microsoft Research + Tsinghua). Learns cinematic language from 440,000 real film clips via multi-shot RAG, generates editing logic based on simulated audience feedback loops. No public code yet, but the most serious academic work on cinematographic AI reasoning.

### Flawless AI DeepEditor

Post-production performance modification: 4K, ACES color, non-destructive retiming and re-takes. Used by Hollywood studios to modify delivered films without reshoots. Almost unknown outside studio post circles.

---

## Complete Stack

### For Auteur Documentary + AI Generative

```
NLE:        DaVinci Resolve Studio ($295 one-time)
            ├── Fairlight DAW (audio, integrated)
            ├── Text-based editing (transcript visible + exportable)
            └── Python API → Claude Code via davinci-resolve-mcp

Footage     Jumper ($249 one-time) — visual search, native in Resolve/FCP/Premiere
analysis:   StoryToolkitAI (free) — transcript search (Whisper local) → EDL/Resolve
            VHS Analyzer (custom) — Gemini Flash multimodal → FCPXML (Goldberg VHS)
            MacWhisper + Parakeet v3 (CoreML) — director voice memos → SYNTHESIS.md

Generative  ComfyUI + Wan 2.2 / LTX-2 / SkyReels V3 (local)
video:      fal.ai Python SDK (cloud, pay-per-use, no subscription)
            ReCamMaster via ComfyUI-WanVideoWrapper (reshoot existing footage)
            Wan 2.2 Fun Camera (camera control in ComfyUI)

World       World Labs Marble API (~$1.20/world)
models:     SkyReels V3 (open source, self-host)

Audio:      Chatterbox (voice cloning, MIT, free local)
            MMAudio (video→audio sync, MIT, free local)
            AudioCraft (SFX + music, Meta, free local)
            Adobe Enhance Speech V2 (dialogue cleanup, free)
            Reaper ($60 one-time) for complex sound sessions

Claude      davinci-resolve-mcp (Resolve control)
Code:       ComfyUI REST API (generative video)
            fal.ai Python SDK (cloud models)
            World Labs API (3D environments)
            applescript-mcp (if FCP)
            fcpxml-mcp-server (if FCP)
```

### Budget (One-Time Only)

| Tool | Cost |
|---|---|
| DaVinci Resolve Studio | $295 |
| Jumper | $249 |
| Reaper | $60 |
| ComfyUI, Wan 2.2, LTX-2, SkyReels V3, ReCamMaster, StoryToolkitAI, Chatterbox, MMAudio, AudioCraft | Free |
| fal.ai, World Labs | Pay-per-use |
| **Total fixed** | **~$600** |

No subscriptions.

---

## Key Links

- [davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp)
- [fcpxml-mcp-server](https://github.com/DareDev256/fcpxml-mcp-server)
- [CommandPost](https://commandpost.fcp.cafe/)
- [StoryToolkitAI](https://github.com/octimot/StoryToolkitAI)
- [Jumper](https://getjumper.io)
- [Twelve Labs](https://twelvelabs.io)
- [ComfyUI Wan 2.2 camera workflow](https://docs.comfy.org/tutorials/video/wan/wan2-2-fun-camera)
- [ReCamMaster](https://github.com/KwaiVGI/ReCamMaster)
- [ComfyUI-WanVideoWrapper (kijai)](https://github.com/kijai/ComfyUI-WanVideoWrapper)
- [fal.ai Wan 2.6 guide](https://fal.ai/learn/devs/wan-26-developer-guide-mastering-next-generation-video-generation)
- [World Labs API docs](https://docs.worldlabs.ai/api/pricing)
- [Chatterbox](https://github.com/resemble-ai/chatterbox)
- [MMAudio](https://github.com/hkchengrex/MMAudio)
- [ViMax](https://github.com/HKUDS/ViMax)
- [Narrative Context Protocol](https://github.com/narrative-first/narrative-context-protocol)
- [Open-Sora 2.0](https://github.com/hpcaitech/Open-Sora)
- [FilMaster](https://filmaster-ai.github.io/)
- [SkyReels V3](https://github.com/SkyworkAI/SkyReels-V3)
- [TrajectoryCrafter](https://github.com/TrajectoryCrafter/TrajectoryCrafter)
- [GEN3C](https://github.com/nv-tlabs/GEN3C)
- [Falafel (fal.ai → Resolve bridge)](https://heyyanshuman.com/posts/falafel_launch)

---

## Where This Is Going — Anticipation 6–18 Months

> The CLI is becoming the new timeline. The NLE is becoming a render window, not a decision-making tool.

### The Fundamental Shift: World Models As Film Space

Right now, generative tools produce *clips*. Within 12–18 months, world models will produce *persistent 3D spaces* you navigate and film from. The workflow becomes:

1. **Shoot once** → world model reconstructs the scene in 3D
2. **Navigate freely** → refilm any angle, any light, in post
3. **Export as clip sequence** → Resolve for grade and render

This is not speculative. Genie 3 (Google DeepMind, $250/mo, accessible via VPN) already does this for interactive scenes. SkyReels V3 and MAGI-1 (open source) do streaming autoregressive world generation — not fixed clips, but continuous navigable space.

### Jumper — Cross-NLE Status (March 2026)

- **FCP 12**: native plugin, sidebar integration ✓
- **Premiere Pro**: native plugin ✓
- **DaVinci Resolve**: "in development" (confirmed roadmap, no date)

For Resolve: export OTIO/EDL from Jumper web app, import manually. Not fluent yet.

### The Porous Stack (Full Vision)

```
LAYER 1 — Semantic Capture
  Whisper v3 large (transcript) → StoryToolkitAI (EDL export)
  Jumper (visual semantic search, offline, $249 lifetime) → FCP/Premiere
  Gemini Flash 3.1 (multimodal ad-hoc queries, already running)

LAYER 2 — Narrative Structure
  Claude Code → FCPXML / OTIO / EDL (CLI as editor)
  davinci-resolve-mcp or fcpxml-mcp-server
  The CLI decides structure. The NLE renders it.

LAYER 3 — Generation
  fal.ai (cloud, pay-per-use) → Wan 2.6, Kling 3.0, Veo 3.1
  ComfyUI (local, zero variable cost) → Wan 2.2 Fun Camera
  ReCamMaster → reshoot existing rushes from new angles

LAYER 4 — 3D World Space
  World Labs Marble API → $1.20/world, Python+JS SDK
  Genie 3 via VPN → interactive navigable worlds (no API yet)
  SkyReels V3 → open source, Apache 2.0, streaming autoregressive
  MAGI-1 (Sand AI) → 24B params, open weights, cinematic long-form

LAYER 5 — Audio
  Adobe Enhance Speech V2 (dialogue cleanup, free)
  Chatterbox MIT (voice cloning SOTA, beats ElevenLabs 63.75%)
  MMAudio (video → ambient sound, MIT, CVPR 2025)
  Suno v5 (music, 8 min, stems, sample-to-song)

LAYER 6 — Final NLE
  DaVinci Resolve Studio $295 (Python API + MCP + color + render)
  OR FCP 12 (fcpxml-mcp-server + CommandPost) if staying Mac-native
```

### What Changes For Hybrid Documentary Practice

- **Your rushes** → ReCamMaster puts them in a reshootable space
- **Cutaway shots** → generate by navigating the world model of your locations
- **Editing** → define narrative trajectories, not manual cuts
- **StoryToolkitAI + Jumper** → become selection tools inside a navigable narrative space, not a linear timeline

### Open Source World Models to Watch

| Model | Status | VRAM | License | Notes |
|-------|--------|------|---------|-------|
| **SkyReels V3** | Released | 80GB | Apache 2.0 | Streaming continuous worlds |
| **MAGI-1** (Sand AI) | Open weights | 80GB | Apache 2.0 | 24B params, cinematic |
| **Open-Sora 2.0** | Released | 48GB | Apache 2.0 | CinemaScope support |
| **TrajectoryCrafter** | Released | 28GB | Apache 2.0 | 6DoF from single image |
| **GEN3C** (NVIDIA) | Released | 24GB | Research | 3D-informed generation |

### The 18-Month Horizon

The distinction between "shooting" and "generating" collapses. A documentary becomes:
- Real footage (your presence, your eye, your ethics)
- World-modeled spaces (navigated in post)
- Generated transitions and texture (Wan 2.x, Kling)
- AI-reframed narrative structure (Claude Code → OTIO)

The filmmaker's irreplaceable role: **the question, the presence, the cut that matters**. Everything else becomes configurable.

