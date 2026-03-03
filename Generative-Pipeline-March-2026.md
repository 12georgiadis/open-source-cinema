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

### The Pareto Decision for Active Production

If you're already an FCP editor mid-production on a film: **don't switch**.

The relearning curve for Resolve is real — 2-3 weeks to be functional, 2-3 months to be fluent. Adding that friction to an active post-production is a risk to the film itself, which contradicts the whole point.

**For the current film (FCP):**
- VHS Analyzer → FCPXML export → import FCP ✅
- StoryToolkitAI → FCPXML export → FCP ✅
- Jumper native in FCP ✅
- fcpxml-mcp-server: 47 MCP tools for Claude Code → FCPXML automation ✅
- MacWhisper + Parakeet → SYNTHESIS.md ✅
- Color: DaVinci Resolve Free (color page only, no API needed) ✅

**For the next film:** Start in Resolve Studio from day one. The relearning is finite and planned, not imposed mid-production.

**What FCP cannot do that Resolve can:** live timeline state (Claude reads the running timeline directly). For 50 structural variants per hour, you eventually need Resolve. For the current film, the FCPXML roundtrip (1-click import) is not meaningful friction.

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

**VHS Analyzer** (custom tool, `goldberg/tools/vhs-analyzer/`) — Gemini Flash 3.1 as engine. Analyzes image and audio together with narrative context: "Joshua age 8 holds a toy gun, mother looks away." Understands cinematic significance, not just what's present. 4-phase pipeline: preprocessing → semantic analysis → synthesis → **FCPXML export**. Engine is swappable: Gemini Flash 3.1 (commercial, best quality) or local model (see below). Built specifically for The Goldberg Variations.

**Gemini Flash 3.1 ad-hoc** — same multimodal capability without the pipeline. For one-off questions on a specific clip. Used standalone: no timecode output. Used inside VHS Analyzer: full FCPXML output via Phase 4.

**[Twelve Labs](https://twelvelabs.io)** — cloud multimodal search at scale. $0.033/min. Useful for large rushes libraries you don't want to process locally.

### Local/Offline Alternatives to Gemini Flash 3.1 (March 2026)

The VHS Analyzer engine is swappable. Local models remove the API cost and keep footage private.

| Model | Type | Video+Audio | VRAM needed | Distill/Quant | Quality vs Gemini Flash 3.1 | License |
|---|---|---|---|---|---|---|
| **Gemini Flash 3.1** | Commercial API | ✅ native | — | — | Baseline | Proprietary |
| **Qwen3-Omni** (30B-A3B MoE) | Open | ✅ audio+video native | 31GB (Q4) | AWQ, GGUF | SOTA 22/36 audio-visual benchmarks | Apache 2.0 |
| **Qwen3.5** (397B-A17B MoE) | Open | ✅ video | 22GB active (AWQ) | AWQ 4-bit | 87.5 Video-MME, beats Gemini 3 Pro | Apache 2.0 |
| **Qwen3-VL** (235B-A22B MoE) | Open | ✅ video 2h native | 36GB+ (4-bit) | GGUF, AWQ | Best open source long-video, timestamp-precise | Apache 2.0 |
| **InternVL3.5** (Flash/241B) | Open | ✅ video | 6-24GB | GGUF | SOTA open-source, 4x faster than InternVL3, -50% visual tokens | Apache 2.0 |
| **Gemma 3** (4B/12B/27B) | Open, Google | ⚠️ images + short video | 4-32GB | GGUF | Good on images, weak on long-form | Apache 2.0 |
| **Qwen3-Omni-Captioner** | Open | ✅ audio description | ~12GB | GGUF | Best for ambient sound / SFX | Apache 2.0 |

### Hardware Mapping — Ton Setup

```
Windows 9950X + RTX 5090 (32GB VRAM) + 96GB DDR5
  → Qwen3-Omni 30B-A3B AWQ Q4: ~31GB VRAM, 52 tok/sec ✅ (meilleur choix)
  → Qwen3.5-35B-A3B AWQ 4-bit: ~22GB VRAM ✅
  → Qwen3-VL quantized (7B/32B): ✅ confortablement
  → InternVL3.5 Flash: ✅
  → Inference server vLLM-Omni → expose API locale → Mac Mini + MacBook s'y connectent
  → Sert aussi pour génération vidéo (ComfyUI, Wan 2.2)

Mac Mini M4 (16-32GB unified memory)
  → InternVL3.5 compact (7B GGUF) via MLX ✅
  → Qwen3-ASR Swift (ASR/TTS on-device, MLX) ✅
  → Gemma 3 12B ✅
  → Jumper, StoryToolkitAI (CPU) ✅

MacBook Air M3 (8-24GB unified memory)
  → Gemma 3 4B / InternVL3.5 compact via MLX ✅
  → Qwen3-ASR Swift ✅
  → Client léger → requêtes vers vLLM-Omni sur Windows ✅
```

**Architecture recommandée :** le Windows RTX 5090 tourne comme serveur d'inférence vLLM-Omni (OpenAI-compatible API locale). Mac Mini et MacBook le requêtent via réseau local ou Tailscale. Un seul modèle chargé en VRAM, toutes les machines en profitent.

### Distilled / Quantized — Formats Disponibles

| Format | Outil | Usage |
|---|---|---|
| **GGUF Q4_K_XL** | llama.cpp, Unsloth, LM Studio | Meilleur équilibre qualité/vitesse sur GPU |
| **AWQ 4-bit** | vLLM, Hugging Face Transformers | Production, meilleure compression que GGUF |
| **GGUF Q2_0** | llama.cpp | Si VRAM tight (80B modèles sur 32GB) |
| **MLX** | mlx-vlm, Qwen3-ASR Swift | Apple Silicon natif |

Sources distill pour Qwen3-Omni : `unsloth/Qwen3-Omni-GGUF`, `cyankiwi/Qwen3.5-35B-A3B-AWQ-4bit`.

### API Cloud si Pas Local

| Service | Modèles disponibles | Notes |
|---|---|---|
| **SiliconFlow** | Qwen3-Omni, Qwen3-VL, InternVL | Meilleure option pour Qwen3-Omni |
| **Together.ai** | Qwen3.5-397B-A17B | 8.6-19x plus rapide que Qwen3-Max |
| **Alibaba DashScope** | Qwen3-Omni natif | Latence la plus faible pour Qwen |
| **Gemini API** | Gemini Flash 3.1 | Commercial, meilleure qualité narrative |
| **Replicate** | InternVL3.5, LLaVA | Pay-per-use |

**Practical split:**
```
Rushes narratifs complexes (qualité max)    → Gemini Flash 3.1 (API, commercial)
Rushes confidentiels / volume important     → Qwen3-Omni AWQ sur RTX 5090 (local)
Audio description (son ambiant, musique)    → Qwen3-Omni-Captioner (local)
Analyse rapide / itération légère (Mac)     → InternVL3.5 compact via MLX
Long-form video 1-2h (Goldberg entiers)     → Qwen3-VL quantized sur RTX 5090
```

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

| Tool | Sees image | Sees speech | Understands meaning | NLE export | Offline | Cost |
|---|---|---|---|---|---|---|
| **Jumper** | ✅ 91.3% | ✅ | ❌ (pattern match) | ✅ FCP/Premiere/Resolve | ✅ | $249 one-time |
| **StoryToolkitAI** | ❌ | ✅ Whisper | ⚠️ embeddings (semantic) | ✅ EDL + Resolve native | ✅ | Free |
| **VHS Analyzer** (custom) | ✅ | ✅ | ✅ Gemini Flash | ✅ FCPXML + query.py | ❌ API | Pay-per-use |
| **FCP 12 Visual Search** | ✅ (12/92) | ✅ (no edit) | ❌ | Native | ✅ | Included |
| **Premiere Text-Based** | ❌ | ✅ | ❌ | ✅ best-in-class | ✅ | Subscription |
| **Gemini Flash 3.1 ad-hoc** | ✅ | ✅ | ✅ | ❌ standalone | ❌ | Pay-per-use |
| **Twelve Labs** | ✅ | ✅ | ✅ | EDL via Jockey | ❌ | $0.033/min |

### For Goldberg Specifically

Four non-redundant layers:

```
VHS family footage      → VHS Analyzer (Gemini Flash, FCPXML + query.py) ← already built
Joshua interview rushes → StoryToolkitAI (Whisper offline, keyword+semantic → Resolve)
Terrain visual rushes   → Jumper (visual search → Resolve)
Director voice memos    → MacWhisper + Parakeet v3 → SYNTHESIS.md
```

### Next Steps For The Existing Stack

Three directions to push `query.py` further:

**1. Embedding-based semantic search** — sentence-transformers (`all-MiniLM-L6-v2`, 80MB, offline). ~50 lines added to `query.py`. Enables "find moments where Joshua seems deceptive" without knowing the exact words the analysis used. Current query.py does keyword matching — embeddings unlock meaning-level search across all 360 segments.

**2. Automatic narrative clustering** — k-means on segment embeddings. Groups analyzed segments by thematic proximity without manual tagging. Output: "8 narrative clusters — cluster 3 = 42 moments around identity performance, cluster 7 = 18 moments around violence/weapons". Reveals structure before you start cutting.

**3. Story sequence builder** — from filtered results, generate FCPXML ordered by narrative logic. "mystery → revelation → denial → collapse" with best segments of each type in sequence. Already partially there with `--fort` filtering — needs the ordering layer on top.

**Voice memo pipeline** (MacWhisper + Parakeet → SYNTHESIS.md): add FCPXML export of strong moments with timecodes so director notes appear as markers on the working timeline alongside footage.

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

#### 5. ComfyUI MCP Servers (Claude Code ↔ Local Generation)

For complex generation (inpainting, ControlNet, style transfer) that fal.ai can't handle, ComfyUI on a local GPU machine is the answer. Several MCP servers bridge Claude Code to ComfyUI:

| Server | Stars | Best for |
|---|---|---|
| **[comfy-pilot](https://github.com/ConstantineB6/comfy-pilot)** | — | Most Claude Code-native: sees, edits, runs workflows. Embedded terminal inside ComfyUI. Describe "build me an inpainting workflow" → Claude creates nodes, connects them, runs. |
| **[mcp-comfyui](https://github.com/SamuraiBuddha/mcp-comfyui)** | — | Generic bridge: run saved workflows, upload images for img2img, check queue |
| **[mcp-comfyui-flux](https://github.com/dhofheinz/mcp-comfyui-flux)** | — | Containerized FLUX (schnell + dev fp8), background removal, 4x upscale |

**comfy-pilot** is installed as a ComfyUI extension on the machine running ComfyUI (the Windows RTX 5090), then exposes an MCP endpoint that Claude Code connects to via network (Tailscale).

#### 6. Script → Generation → Timeline Placement (The Full Chain)

The complete pipeline from a document describing a needed shot to a clip at the correct position in the timeline:

```
Script / document describes the narrative gap
  ↓
Claude Code reads → extracts: prompt + timecode + generation method
  ↓
GENERATION:
  Simple (txt2vid, img2vid)    → fal.ai Python SDK (cloud, ~30s, no local setup)
  Complex (inpaint, ControlNet) → comfy-pilot → ComfyUI on RTX 5090 (local, free)
  ↓
PLACEMENT:
  FCP   → fcp-xml MCP patches FCPXML at correct timecode → 1-click import
  Resolve → resolve-mcp imports + places directly in running timeline (live, no import)
```

**fal.ai for simple generation (works today in FCP, no Resolve needed):**

```python
import fal_client, urllib.request

# Generate a missing shot
result = fal_client.subscribe(
    "fal-ai/kling-video/v2.1/pro/text-to-video",
    arguments={
        "prompt": "Joshua age 8, holding toy gun, backyard, VHS grain, late afternoon",
        "duration": "5",
        "aspect_ratio": "16:9",
    }
)
urllib.request.urlretrieve(result["video"]["url"], "insert_001.mp4")
# → fcp-xml MCP inserts insert_001.mp4 into FCPXML at timecode 00:12:34:00
```

The key insight: Claude Code reads the script or document, identifies what's missing and where, generates, then patches the FCPXML. One human action: import the modified XML into FCP.

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

### Resolve ↔ FCP 12 — Roundtrip Reality (March 2026)

**The honest answer: it's broken by design.**

FCP 12 (10.6+) is at FCPXML **1.12** and exports a bundled `.fcpxmld` package (not even plain XML). Resolve 20 caps at **1.9–1.10**. Blackmagic hasn't moved on this since Resolve 17. The gap is not a bug — neither company has prioritized fixing it.

**Version gap:**
```
FCP 11  (Nov 2024) → FCPXML 1.13
FCP 12  (Jan 2026) → FCPXML 1.14 (.fcpxmld bundle)
Resolve 20         → reads max 1.9–1.10 (1.11 via CommandPost hack)
Gap                → 4 versions behind, no official fix
```

**Workarounds that partially work:**

1. **Force older export from FCP**: `File > Export XML > Previous Version` — gets you to 1.10, but Resolve reads it inconsistently. 1.9 is the safest, but FCP 12 doesn't always offer it.
2. **Unpack the .fcpxmld bundle**: Right-click > Show Package Contents > extract `info.fcpxml` — even then, often greyed out in Resolve's import dialog.
3. **CommandPost** — has a Sony Timecode Repair toolbox and can force FCPXML 1.11 output, which some older Resolve builds accept. Active dev as of 2025.
4. **Pre-populate Resolve Media Pool first** (drag original media files directly), then import XML with auto-import disabled — reduces relinking failures.
5. **Manual XML header edit** — change version number in a text editor (unreliable, last resort).

**What translates when it works at all:**

| Element | FCP → Resolve | Resolve → FCP |
|---|---|---|
| Cuts / edit points | ✅ | ✅ |
| Clip names | ✅ | ✅ |
| Basic transitions | ✅ mostly | ✅ mostly |
| Markers | ✅ | ✅ |
| Opacity, position, scale, rotation | ✅ | ✅ |
| FCP color corrections | ✅ ("Use Color Info") | ❌ |
| Resolve color nodes | — | ❌ |
| Speed ramps | ❌ breaks | ❌ |
| FCP-native effects / titles | ❌ | ❌ |
| FCP Keywords / Events | ❌ | ❌ |
| Timecode (MP4 Sony cameras) | ⚠️ breaks without CommandPost | — |

**What professional studios actually do (not roundtrip):**
```
Edit in FCP → lock cut → export XML + original media → Resolve (color only)
→ render graded ProRes → back to FCP as new media files (no XML)
```
No XML roundtrip on the return. The colorist renders, the editor replaces clips. Cleaner, no relinking hell.

**Direction that works: Resolve → FCP.** Resolve exports FCPXML in an older version that FCP accepts. The broken direction is FCP → Resolve (FCP exports 1.14, Resolve reads max 1.10).

**For Claude Code piloting via Resolve MCP:** Assemble structure in Resolve (rough cut, markers). Export FCPXML → FCP for fine cut. One-way. Color stays in Resolve. Return is graded media files, not XML.

---

### All Directions — Complete Roundtrip Table (March 2026)

**The universal rule that never changes:** Color grades and NLE-native effects never transfer between NLEs. Always render graded media as new files before handing off. The XML carries *structure* (cuts, markers, clip names). Never *rendering* (grades, effects, audio mix).

#### FCP ↔ Resolve

| Direction | Method | What transfers | What doesn't |
|---|---|---|---|
| FCP → Resolve | Export FCPXML 1.11 (force older version at export) | Cuts, markers, clip names, basic transitions, opacity/position/scale | Speed ramps, FCP-native titles/effects, keywords/events, Sony MP4 timecode (needs CommandPost) |
| Resolve → FCP | Resolve exports older FCPXML that FCP accepts natively | Cuts, markers, basic transforms | Resolve color nodes (never), Resolve-native effects |

**Key workaround — FCP → Resolve:** FCP 12 exports 1.14 by default (Resolve reads max 1.11). Use `File > Export XML > Previous Version` to force 1.11. Pre-populate Resolve's Media Pool with original files before importing XML to minimize relinking failures.

**Verdict:** Resolve → FCP works well. FCP → Resolve is fragile. Practical workflow: do structural work in Claude Code via Resolve MCP (rough assembly, marker placement), export FCPXML → FCP for fine cut. Return path = rendered ProRes files, not XML.

#### FCP ↔ Premiere Pro

| Direction | Method | What transfers | What doesn't |
|---|---|---|---|
| FCP → Premiere | **No native import.** Requires XtoCC ($49) or FCP→Resolve→AAF→Premiere | Cuts, basic clip structure (via workaround) | Almost everything else; workaround is lossy |
| Premiere → FCP | SendToX (third-party, limited) or .prproj → OTIO → FCPXML | Basic edit structure only | Audio mix, effects, color, complex timeline structure |

**Premiere CANNOT import FCPXML natively.** This is a common misconception. Adobe Premiere has no built-in FCPXML importer. The workarounds are: XtoCC ($49, converts FCPXML to .prproj), or FCP→Resolve (FCPXML 1.11)→AAF→Premiere.

**Verdict:** FCP ↔ Premiere roundtrip is the most painful of the three. Avoid unless necessary.

#### Resolve ↔ Premiere Pro

| Direction | Method | What transfers | What doesn't |
|---|---|---|---|
| Resolve → Premiere | AAF export from Resolve | Edit structure, clip references, basic audio | Color grades, Resolve Fusion effects, audio channel mapping (often broken) |
| Premiere → Resolve | AAF or XML from Premiere | Edit structure | Effects, color, Premiere-native audio tracks |

**AAF is the professional standard** for Resolve ↔ Premiere (and Resolve ↔ Avid). More reliable than XML for this pairing.

#### Summary matrix

| Route | Reliability | Recommended method |
|---|---|---|
| FCP → Resolve | ⚠️ fragile | FCPXML 1.11 forced export + CommandPost |
| Resolve → FCP | ✅ works | Standard FCPXML export from Resolve |
| FCP → Premiere | ❌ no native | XtoCC ($49) or via Resolve→AAF |
| Premiere → FCP | ❌ very limited | SendToX or OTIO bridge (experimental) |
| Resolve → Premiere | ⚠️ partial | AAF |
| Premiere → Resolve | ⚠️ partial | AAF |

**For this project (Claude Code + FCP as primary NLE):** The only roundtrip that matters is Resolve → FCP (when Claude Code assembles a rough cut in Resolve, you pull it into FCP for fine cut). That direction works. All other directions are one-way streets best handled by rendering new media rather than exchanging XML.

### OTIO — The Missing Layer

**[OpenTimelineIO](https://github.com/AcademySoftwareFoundation/OpenTimelineIO)** (Apple → ASWF, Apache 2.0) is the NLE-agnostic timeline interchange format that sits underneath FCP, Resolve, Premiere, and Avid. It's what you should use when Claude Code reasons about timeline structure — not FCPXML (FCP-specific) or EDL (limited).

**OTIO is NOT a better NLE interchange format.** Common misconception: using OTIO as a bridge FCP ↔ Resolve would be worse than direct FCPXML, because OTIO loses additional data at conversion (complex transitions, effects). And Resolve → FCP via OTIO adds zero value since Resolve's native FCPXML export already works.

**Where OTIO actually belongs — the manipulation layer:**

```
FCP exports FCPXML 1.14 (default)
→ Claude Code reads via otio-fcpx-xml-adapter (clean Python objects)
→ manipulates structure (insert generated clips, reorder sequences, place markers)
→ writes back as FCPXML 1.11 (for Resolve import) OR 1.14 (back to FCP)
```

OTIO's value is programmatic reasoning inside Claude Code — not as a transport format between NLEs. Use it to *think* about the timeline, not to *move* it between applications.

```python
import opentimelineio as otio

# Read any timeline — FCP, Resolve, Premiere all export OTIO
timeline = otio.adapters.read_from_file("goldberg_cut.fcpxml")  # or .otio, .edl

# Reason about structure
for clip in timeline.tracks[0].children:
    print(clip.name, clip.source_range)

# Modify — insert a generated clip at a specific timecode
new_clip = otio.schema.Clip(
    name="insert_joshua_childhood",
    source_range=otio.opentime.TimeRange(
        start_time=otio.opentime.RationalTime(0, 24),
        duration=otio.opentime.RationalTime(120, 24),  # 5 seconds
    )
)
timeline.tracks[0].insert(7, new_clip)

# Write back to any NLE format
otio.adapters.write_to_file(timeline, "goldberg_cut_v2.fcpxml")   # FCP
otio.adapters.write_to_file(timeline, "goldberg_cut_v2.resolve")  # Resolve
```

**Why OTIO over raw FCPXML:** FCPXML is FCP-specific and verbose. OTIO is the standard — works with every NLE, has a clean Python API, and is the correct "native language" for timeline manipulation at the Claude Code level. When the structural intelligence vision is fully realized, OTIO is the format in which it operates.

```
Claude Code ↔ OTIO (NLE-agnostic)
  → export .fcpxml  → FCP
  → export .resolve → Resolve Studio
  → export .edl     → any NLE
```

Install: `pip install opentimelineio`

**OTIO native NLE support status (March 2026) — corrected:**

| NLE | Native UI support | Via Python adapter |
|---|---|---|
| **FCP 12** | ❌ not in UI | ❌ **no FCP X/12 adapter exists** — only FCP 7 (legacy) |
| **Resolve 20** | ⚠️ export only (community plugin resolve-otio) | ✅ partial |
| **Premiere Pro** | ⚠️ in beta (basic cut structure, 2025) | ✅ otio-premiereproject (.prproj read) |
| **Blender VSE** | ✅ official addon (Blender Studio) | — |

**Correction (updated):** OTIO has TWO FCP adapters — do not confuse them:
- `otio-fcp-adapter` → **FCP 7** only (2011 legacy NLE)
- [`otio-fcpx-xml-adapter`](https://github.com/OpenTimelineIO/otio-fcpx-xml-adapter) → **FCP X / FCP 12** ✅ (official OTIO repo, PyPI, reads + writes FCPXML)
- [`otio-fcpx-xml-lite-adapter`](https://pypi.org/project/otio-fcpx-xml-lite-adapter/) → lightweight variant, tested on 1.9 and 1.13, markers + basic structure

OTIO **can** read and write FCP X FCPXML. Install: `pip install otio-fcpx-xml-adapter`

**OTIO native NLE support status (March 2026 — corrected):**

| NLE | Native UI support | Via Python adapter |
|---|---|---|
| **FCP 12** | ❌ not in UI | ✅ `otio-fcpx-xml-adapter` (official) |
| **Resolve 20** | ⚠️ export only (community plugin) | ✅ `resolve-otio` |
| **Premiere Pro** | ⚠️ beta (basic cut structure) | ✅ `otio-premiereproject` (.prproj read) |
| **Blender VSE** | ✅ official addon | — |

**For Claude Code:** OTIO is the right in-memory format for cross-NLE timeline manipulation. Read from FCP via `otio-fcpx-xml-adapter`, manipulate in Python, write to FCPXML for FCP or EDL for Resolve. Cleaner than raw FCPXML parsing.

### Pallaidium — Honest Assessment

**[Pallaidium](https://github.com/tin2tin/Pallaidium)** — Blender-based video editor with AI generation built in. Full Python API (Blender's `bpy`). Interesting in theory.

**Why it doesn't apply here:** Windows + Nvidia only. No serious audio engine. No color science. Community too small for production reliability. Not validated at feature-length doc level. The Blender VSE is not a professional editing environment — no proper multicam, no proxy workflow, no professional output pipeline.

**What it does well:** rapid prototyping of generative sequences on Windows. As an experimentation sandbox on the RTX 5090 machine, fine. As a primary editing environment for a documentary: no.

### ComfyUI Workflows — Pianelli Approach

**Alessandro Pianelli** builds production-grade ComfyUI workflows: multi-stage generation, style consistency, inpainting, ControlNet chains. His workflows are the current reference for quality generation on local GPU.

The right mental model for this stack:

```
GENERATION QUALITY  → ComfyUI (RTX 5090) + Pianelli-style workflows
                       (creative iteration, inpainting, ControlNet, style consistency)

GENERATION SPEED    → fal.ai Python SDK
                       (simple txt2vid or img2vid, cloud, ~30s)

NLE PLACEMENT       → OTIO → FCPXML → FCP (1-click import)
                       OR resolve-mcp → Resolve (live, when Studio acquired)
```

These are not competing options. ComfyUI and fal.ai handle different generation scenarios. The NLE placement layer is the same regardless.

### The Honest Assessment

FCP + this stack covers ~70% of what you'd do with Resolve Python API for narrative/structural work. The gap is live state access, color node control, and stable render automation. For a filmmaker who wants structural automation (rough cuts, markers, selects reels) and stays in FCP for the actual editing feel, this is a real option — not a workaround.

### MCP Servers — Installed Configuration (March 2026)

Active in `~/.claude.json` on MacBook Air M3:

```
fcp-xml          → fcpxml-mcp-server (47 FCP tools, FCPXML roundtrip)
resolve-mcp      → samuelgursky/davinci-resolve-mcp (primary, 562 stars)
resolve-mcp-apvlv → apvlv/davinci-resolve-mcp (Fusion + scene inspection)
applescript      → @peakmojo/applescript-mcp (UI-level FCP control)
```

All Resolve servers are pre-configured and waiting. They activate when DaVinci Resolve Studio is running. No reconfiguration needed when you buy Studio.

---

## Adobe Firefly Quick Cut — Competitive Context

Released in **beta, February 25, 2026**. The most relevant recent Adobe launch for documentary filmmakers.

### What It Does

Upload raw footage → describe the result in text → Quick Cut assembles a first cut automatically. Identifies key moments in interviews, builds initial story structure, handles B-roll organization. Aspect ratio, pacing, and duration are configurable.

### Who It's For

Explicitly designed for **content creators, marketers, and time-constrained editors**, not for professional narrative work. Best suited for: social videos, product demos, event recaps, podcast clips, interview summaries.

### What It Cannot Do vs Our Stack

| Capability | Quick Cut | Our Stack |
|---|---|---|
| Assembly from footage | ✅ | ✅ (Claude Code → FCPXML) |
| Natural language footage search | ✅ (Media Intelligence) | ✅ (Jumper 91.3%) |
| Understands narrative meaning | ❌ | ✅ (Gemini Flash 3.1) |
| Image + audio + meaning together | ❌ | ✅ (VHS Analyzer) |
| Private footage (local processing) | ❌ cloud only | ✅ (Qwen3-Omni on RTX 5090) |
| Multiple generation models | ❌ Firefly Video only | ✅ (Kling, Veo, Wan, fal.ai) |
| Requires Creative Cloud subscription | ✅ (~$60/month) | ❌ pay-per-use only |
| "Joshua seems deceptive here" | ❌ | ✅ |

**Verdict:** Quick Cut is a skilled intern for YouTube content. For a documentary about a morally complex character with hours of raw material and ethical layers, it's not the right tool and doesn't compete with this stack.

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
- **DaVinci Resolve Studio**: native (Workspace > Workflow Integrations, live March 2026) ✓

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

