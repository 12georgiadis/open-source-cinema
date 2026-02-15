# Magic Lantern RAW (CinemaDNG) -- Complete Professional Workflows
## DaVinci Resolve & Final Cut Pro

---

# PART 1: DaVinci Resolve with CinemaDNG from ML RAW

---

## 1. PRE-PROCESSING: MLV to CinemaDNG Conversion

Before any NLE work, Magic Lantern `.MLV` files must be converted to CinemaDNG sequences.

### Recommended Tool: MLV App
- Open source, cross-platform (macOS/Windows/Linux)
- Download: https://mlv.app/ | GitHub: https://github.com/ilia3101/MLV-App

### MLV App Export Settings for DaVinci Resolve

| Setting | Value | Notes |
|---|---|---|
| Export Codec | CinemaDNG (Uncompressed or Lossless) | Lossless saves ~50% space with no quality loss |
| Naming Convention | **DaVinci Resolve naming** (checkbox) | Uses BMD file naming convention so Resolve treats the DNG sequence as a single clip |
| Bit Depth | 14-bit (native) or 16-bit (Dual ISO) | Match your camera's recording bit depth |

### Critical RAW Corrections in MLV App (apply BEFORE export)

These corrections are baked into exported CinemaDNG files. All other MLV App processing parameters (exposure, contrast, color) are NOT exported with CinemaDNG -- they must be applied later in Resolve.

| Correction | When to Enable | Notes |
|---|---|---|
| **Vertical Stripes** | Always (if your camera produces them) | Removes column noise from Canon sensors. Known issue: severe stripes can cause horizontal/grid artifacts -- test first |
| **Fix Focus Pixels** | Always (especially EOS M, 650D, 700D, 100D) | Removes pink dots that appear in RAW video. Uses camera-specific pixel maps. Autodetection available |
| **Pattern Noise** | Recommended | a1ex's pattern noise reduction algorithm. Removes Fixed Pattern Noise (FPN) |
| **Chroma Smoothing** | Recommended (2x2 or 3x3) | Removes color noise and false color artifacts. 3x3 also fixes residual focus dot issues visible with highlight reconstruction |
| **Darkframe Subtraction** | Optional (for static noise) | Load a darkframe (shot with lens cap on, same ISO/exposure) via the darkframe folder icon. Compensates FPN that varies with ISO and exposure time |

### Alternative Conversion Tools
- **RAWMagic** (macOS, $30 App Store) -- easiest tool, produces CinemaDNG compatible with Resolve, handles audio sync well
- **MLVFS** (macOS) -- virtual filesystem, mounts MLV as CinemaDNG folders. Use `--resolve-naming` flag for BMD naming convention
- **raw2cdng** (Windows) -- command-line converter
- **SlimRAW** -- lossless compression of DNG files (shrinks to ~50% of original size, works in both Resolve and Premiere)
- **Fast CinemaDNG Processor** (Windows) -- GPU-accelerated (NVIDIA), supports MLV natively

---

## 2. IMPORT INTO DaVinci Resolve

### Project Settings (File > Project Settings)

#### Master Settings Tab

| Setting | Recommended Value | Notes |
|---|---|---|
| Timeline Resolution | Match your footage (e.g., 1920x1080, 1728x972, 3584x1320) | ML RAW resolutions vary by camera and mode. 5D3: up to 3584x1320 (crop mode), EOS M: 1736x976, etc. |
| Timeline Frame Rate | Match your recording FPS | Common: 23.976, 24, 25, 29.97. CRITICAL: Cannot be changed after editing begins |
| Playback Frame Rate | Same as Timeline | |
| Video Format | Match (e.g., 1080p24) | |

> **Important**: Set the frame rate FIRST before doing anything else. You cannot change it after media is on the timeline.

#### Color Management Tab (File > Project Settings > Color Management)

Three approaches, from simplest to most flexible:

**Option A: DaVinci YRGB (Non-Managed) -- RECOMMENDED for ML RAW**
- Color Science: DaVinci YRGB
- No automatic transforms. Full manual control via Camera RAW tab + CST nodes
- Best for ML RAW because there is no official IDT for Canon ML cameras

**Option B: DaVinci YRGB Color Managed**
- Color Science: DaVinci YRGB Color Managed
- Color Processing Mode: Custom
- Timeline Color Space: DaVinci Wide Gamut / DaVinci Intermediate
- Timeline Working Luminance: Custom 10000 nits (preserves full DR)
- Output Color Space: Rec.709 Gamma 2.4 (for SDR monitoring)
- Input Color Space: Leave on Auto (Resolve will attempt to detect; for ML CinemaDNG you may need to manually assign)

**Option C: ACES (ACEScct)**
- Color Science: ACEScct
- ACES Output Transform: Rec.709
- **WARNING**: There is NO official ACES IDT (Input Device Transform) for Magic Lantern Canon cameras. Choosing any IDT for ML CinemaDNG reveals an identical image -- the IDT appears to do nothing. Resolve debayers the RAW data directly into ACES. Results are usable but not specifically tailored to Canon sensor color science. Consider Option A or B instead.

### Camera RAW Tab (File > Project Settings > Camera RAW)

| Setting | Value | Notes |
|---|---|---|
| RAW Profile | CinemaDNG | |
| Decode Using | **Project** (for uniform settings) or **Clip** (for per-clip control) | "Project" applies to all CinemaDNG clips; "Clip" allows individual override in the Color page Camera RAW palette |

#### Color Space & Gamma Options for ML CinemaDNG

| Color Space | Gamma | Best For | Notes |
|---|---|---|---|
| **Rec.709** | Rec.709 | Quick results, SDR finish | Clean, natural-looking. "One way street" -- data is compressed. Fine if working start-to-finish with CDNGs |
| **P3 D60** | 2.6 | Wide gamut, DCI delivery | Described as "the most neutral colorspace option" for ML footage. Less magenta shift on skin tones than Rec.709 |
| **Blackmagic Design** | Blackmagic Design Film | Maximum DR preservation, CST workflow | Log-encoded. Initial image appears very flat. Requires CST nodes to convert for grading/viewing. Recommended by experts: "decode to the widest space available and keep it wide until the end" |

#### Additional Camera RAW Settings

| Setting | Recommendation | Notes |
|---|---|---|
| Decode Quality | Full Res (for final); Half Res or Quarter Res (for editing) | Play Quality can be reduced without affecting render |
| White Balance | Manual (set Temperature/Tint) | Avoid Auto WB; set manually in camera or adjust here |
| Tint | Adjust as needed | |
| Exposure | Boost +1 to +2 EV | ML RAW tends to be underexposed in Resolve's decode. Boost to properly position mid-tones |
| Highlight Recovery | Enable with caution | Recovers additional highlight sensor data. Can cause artifacts in extreme cases. Combine with MLV App's chroma smoothing to avoid focus dot visibility |
| Apply Pre Tone Curve | **ON** for ML CinemaDNG | Default is OFF for new projects, but turning it ON "may provide better results for CinemaDNG raw files coming from other sources" (i.e., non-BMD cameras like Canon ML) |
| Color Temp (Kelvin) | Set to match scene | |
| Sharpness | 0 (apply later in grading) | |
| Luminance Smoothing | 0 (handle in grading) | |
| Color Smoothing | 0 (handle in grading) | |

### Importing CinemaDNG Sequences

1. Open **Media** page
2. Navigate to the folder containing your CinemaDNG sequence (exported from MLV App with DaVinci Resolve naming)
3. The folder should appear as a **single clip** in the Media Pool (thanks to BMD naming convention)
4. If it appears as individual frames: right-click > **Show Individual Frames** (toggle off), or check that MLV App exported with "DaVinci Resolve naming" enabled
5. Drag to the Media Pool

> **Tip**: Keep all CinemaDNG folders within a single parent directory for easier relinking later.

---

## 3. COLOR GRADING PIPELINE FOR ML RAW

### Approach A: CST Node-Based Workflow (Recommended)

This is the most flexible approach for ML RAW, giving full manual control.

#### Project Settings
- Color Science: DaVinci YRGB (non-managed)
- Camera RAW: Decode to **Blackmagic Design / BMD Film** (widest available space)

#### Complete Node Tree Structure

```
[Node 1: CST IN] --> [Node 2: NR] --> [Node 3: WB] --> [Node 4: Exposure] -->
[Node 5-7: Secondaries (Parallel)] --> [Node 8: Creative Look/LUT] -->
[Node 9: CST OUT] --> [Node 10: Halation/Glow] --> [Node 11: Film Grain] --> [OUT]
```

**Node 1 -- CST IN (Color Space Transform)**
- Effect: Color Space Transform (drag from Effects library)
- Input Color Space: Blackmagic Design (matches Camera RAW decode)
- Input Gamma: Blackmagic Design Film
- Output Color Space: DaVinci Wide Gamut
- Output Gamma: DaVinci Intermediate
- Purpose: Converts camera-native color to a wide, device-independent working space

**Node 2 -- Noise Reduction**
- Apply Temporal NR first (analyzes differences between frames)
- Temporal NR: Frames = 2-3, Luma = 5-15, Chroma = 10-25
- Spatial NR: Use sparingly (Luma = 3-8, Chroma = 5-15)
- For ML RAW artifacts: If residual pattern noise or vertical stripe artifacts remain after MLV App processing, increase Temporal NR Chroma. Consider Neat Video OFX plugin for severe cases
- Rule: Always denoise BEFORE adjusting tone/color so corrections don't amplify grain

**Node 3 -- White Balance**
- Use Offset wheel (or Temperature/Tint in Primaries)
- Optional: Set gamma to Linear, Luma Mix to 0, then adjust Gain or Offset wheel for neutral WB
- Use vectorscope to verify neutral point

**Node 4 -- Exposure & Contrast**
- Lift/Gamma/Gain or Curves
- Contrast Pivot: 0.336 (maintains 18% gray in DWG/Intermediate)
- Stop pushing contrast when waveform starts to compress
- This is technical correction only -- no creative adjustments here

**Nodes 5-7 -- Secondaries (Parallel Nodes)**
- Use **parallel nodes** branching from Node 4 output
- Each node addresses one element:
  - Node 5: Skin tone isolation (HSL Qualifier + Power Window + Tracking)
  - Node 6: Sky/environment (Qualifier + Window)
  - Node 7: Specific element correction
- Parallel nodes apply corrections simultaneously without affecting each other

**Node 8 -- Creative Look / Film Emulation LUT**
- Apply AFTER secondaries but BEFORE CST OUT
- This is where film emulation LUTs go (Kodak 2383, Fuji 3513, etc.)
- Right-click node > 3D LUT > select your LUT
- LUTs designed for DWG/Intermediate work best here
- Can reduce LUT intensity via Key Output Gain
- Alternative: Use PowerGrades (Juan Melara's PFE, CinePrint35, Jamie Fenn's DWG LUTs)
- Or use Dehancer plugin for comprehensive film emulation

**Node 9 -- CST OUT (Color Space Transform)**
- Input Color Space: DaVinci Wide Gamut
- Input Gamma: DaVinci Intermediate
- Output Color Space: Rec.709
- Output Gamma: Gamma 2.4
- Purpose: Converts from working space to display/delivery space

**Node 10 -- Halation / Glow (optional)**
- Apply AFTER CST OUT (in display space)
- Halation simulates light bleeding through film base
- Glow adds bloom to highlights
- Place before film grain

**Node 11 -- Film Grain (optional)**
- ALWAYS the last node in the tree (serial node)
- Grain responds to luminosity, so it must see the final image

#### One Node, One Job Principle
- Label each node clearly (right-click > Label)
- Each node performs exactly one task
- This allows bypassing, inserting, or reordering without breaking the grade

### Approach B: DaVinci YRGB Color Managed (RCM)

If using RCM with DaVinci Wide Gamut / Intermediate:

1. Set Camera RAW to Decode Using: **Project**
2. The system auto-transforms footage into the Timeline Color Space
3. No CST nodes needed -- Resolve handles transforms automatically
4. Grade directly using Primaries, Curves, Qualifiers, Power Windows
5. Output Color Space setting handles the final conversion

Advantages: Simpler, fewer nodes. Disadvantages: Less manual control, may not handle ML footage optimally without proper input detection.

### Matching ML RAW with Other Cameras (BMD, RED, ARRI)

Using DaVinci Wide Gamut / Intermediate as the common working space:

| Camera | Input CST Settings |
|---|---|
| Magic Lantern CinemaDNG | BMD / BMD Film (or Rec.709 / Rec.709 if decoded that way) |
| Blackmagic (BRAW) | Camera RAW: Decode to DWG/DI directly (no CST needed) |
| ARRI (LogC3) | ARRI Wide Gamut 3 / ARRI LogC3 |
| ARRI (LogC4) | ARRI Wide Gamut 4 / ARRI LogC4 |
| RED (R3D) | REDWideGamutRGB / Log3G10 (auto-handled by Resolve) |
| Sony (S-Log3) | S-Gamut3.Cine / S-Log3 |
| Canon (C-Log) | Cinema Gamut / Canon Log 3 |

All footage converges into DWG/Intermediate, where grading is consistent and device-independent.

### Film Emulation LUTs

#### Free Options
- **DaVinci Resolve built-in**: Film Looks folder includes some Kodak Print Film Emulations
- **Jamie Fenn's DWG LUTs**: Kodak 2383 & Fuji 3510 matched for DWG workflows (https://www.jamiefenn.com/p/free-dwg-film-emulation-luts/)
- **Juan Melara PFE**: Kodak 2383, 2393, Fuji 3510, 3513DI, 3521XD as PowerGrades (https://juanmelara.com.au/blog/print-film-emulation-luts-for-download)

#### Premium Options
- **CinePrint35**: 500T, 250D, 160T stocks + Kodak 2383 / Fuji 3513 print film (https://www.tombolles.net/cineprint35)
- **Dehancer** (OFX plugin): Comprehensive film emulation with grain, halation, gate weave
- **FilmConvert Nitrate**: Camera-profiled film stock emulation (Kodak, Fuji stocks)
- **Colourlab.Ai Look Designer 2.0**: Deep collection of film emulation LUTs

#### Where to Apply in the Node Tree
- In DWG workflow: Apply film emulation LUT on a node BETWEEN your grading nodes and the Output CST (Node 8 in the tree above)
- In RCM: Apply before the automatic Output Transform
- PFE LUTs specifically expect log film scans in Rec.709 as input -- read each LUT's documentation

### Power Windows, Qualifiers & Tracking for ML RAW

ML RAW (14-bit) provides excellent color data for qualifier selections:

**Qualifiers (HSL):**
- Click eyedropper in toolbar > click/drag on viewer to select color range
- Toggle highlight view: Shift + H
- For skin tones: Start with Hue qualifier, then refine with Saturation and Luminance
- 3D Qualifier: Draw lines over the target area for more precise isolation (excellent for faces in mixed lighting)

**Power Windows:**
- Combine with qualifiers for spatial isolation
- Draw circular/bezier window around subject's face
- Adjust softness for gradual falloff
- Track the window to follow movement

**Tracking:**
- Open tracker palette (crosshair icon)
- Select Pan, Tilt, Zoom, Rotation as needed
- Track forward/backward from playhead
- Resolve's tracker handles ML RAW well due to full-res data

### Dealing with ML RAW Specific Issues in Resolve

#### Residual Vertical Stripes
- Primary fix: MLV App preprocessing (enable Vertical Stripes correction)
- In Resolve: Temporal NR (increase Chroma slider) can reduce remaining banding
- Severe cases: Use Neat Video OFX plugin with a noise profile built from a banding area
- Last resort: Export frames, use Topaz DeNoise 5 (has specific band noise removal), or GIMP + G'MIC "Banding denoise" filter

#### Pattern Noise
- Primary fix: MLV App preprocessing (enable Pattern Noise)
- In Resolve: Spatial NR (Chroma slider) for residual artifacts
- Darkframe subtraction in MLV App for static noise

#### Focus Dots (Pink Pixels)
- Primary fix: MLV App (enable Fix Focus Pixels with correct pixel map for your camera)
- Enable Chroma Smoothing 3x3 in MLV App if dots persist with highlight reconstruction
- In Resolve: If residual dots remain, slight chroma blur or Spatial NR Chroma can help
- DaVinci Resolve 10+ fixed the pink fringing issue that was specific to ML RAW black level metadata

#### Noise Reduction Settings for ML RAW

| Setting | Luma Value | Chroma Value | Notes |
|---|---|---|---|
| Temporal NR (Motion Estimation) | 5-15 | 10-25 | Most effective. Analyzes inter-frame differences. Higher frames = better but slower |
| Temporal NR Frames | 2-3 | 2-3 | Number of frames analyzed |
| Spatial NR (Radius) | 3-8 | 5-15 | Use sparingly to avoid softening. Chroma can be pushed higher |
| Spatial NR Mode | Better or Enhanced | Better or Enhanced | "Enhanced" in Studio version uses DaVinci Neural Engine |

#### Sharpening Recommendations
- Apply on a separate node AFTER noise reduction
- Resolve Sharpening: Amount = 0.2-0.5, Radius = 0.5-0.8
- Use a Power Window for selective sharpening (e.g., on face only)
- For out-of-focus ML footage: Circular window on subject, Sharpening ~0.03, Scaling ~0.44 (subtle perceived sharpness increase)
- Consider Midtone Detail (in Primary Wheels) for texture enhancement: +10 to +30
- Do not over-sharpen ML RAW -- the sensor resolution is already modest (5D3: ~1800 lines)

---

## 4. CONFORM WORKFLOW (Offline/Online)

### Overview
Edit with lightweight proxies (ProRes) > Export FCPXML > Import into Resolve > Relink to full-res CinemaDNG > Grade > Deliver

### Step 1: Generate Proxies from CinemaDNG
1. Import CinemaDNG sequences into Resolve's Media Pool
2. Create a timeline with all clips
3. Go to **Deliver** page
4. Export as **Apple ProRes 422 LT** (or ProRes Proxy for maximum speed)
5. **Critical**: Maintain matching filenames between proxies and CinemaDNG originals
6. Export renders

### Step 2: Edit Offline in FCP (or Premiere)
- Import ProRes proxies into FCP
- Edit your film
- Lock the edit

### Step 3: Export FCPXML from FCP
- File > Export XML
- **Choose XML version 1.9** (Resolve supports v1.3-1.9)
- The file must have `.fcpxml` extension (NOT `.fcpxmld`)
- Strip audio if possible and export a clean, video-only FCPXML
- Ensure no file renaming has occurred

### Step 4: Import FCPXML into DaVinci Resolve (Conform)
1. In Resolve, import all CinemaDNG source clips into the Media Pool first
2. File > Import > Timeline (Shift + Cmd + I)
3. Open your `.fcpxml` file
4. **Deselect "Automatically import source clips into media pool"** -- this forces relinking to your pre-imported CinemaDNG files
5. Name the timeline

### Step 5: Relink to Full-Res CinemaDNG
- If clips are offline (flagged red): select clips in Media Pool > right-click > **Change Source Folder** > navigate to CinemaDNG directory
- For manual relink: right-click timeline clip > **Conform Lock with Media Pool Clip**
- Verification tip: Put reference ProRes on top track at 50% opacity to confirm alignment

### Step 6: Verify
- Scrub through timeline checking every edit point
- Verify all clips are linked to CinemaDNG (not proxies)
- Check Camera RAW palette is accessible on Color page (confirms RAW link)

### Common Pitfalls
- **Speed effects**: Optical flow retimed clips from FCP import as composite clips. Open each composite, apply CinemaDNG settings, return to main timeline
- **File name duplication**: The #1 cause of conform failures. Never rename files
- **AIFF audio**: Resolve does not import AIFF files. Convert to WAV before round-tripping
- **Handles**: Add 1+ second of extra media at in/out points before export, or handles cannot be extended after conform
- **Mixed frame rates**: Use the "Mixed frame rate format" option during FCPXML import

---

## 5. DELIVERY FROM DaVinci Resolve

### DCP (Digital Cinema Package) -- Festival Delivery

**Requirements**: DaVinci Resolve **Studio** (paid version)

| Setting | Value | Notes |
|---|---|---|
| Preset | DCP (on Deliver page) | |
| Codec | Kakadu JPEG 2000 | Included with Resolve Studio |
| Bitrate | 200 Mbps (recommended) | Max is 250 Mbps (overkill). Min is 50 Mbps |
| Resolution | DCI 2K Flat: 1998x1080 or DCI 4K Flat: 3996x2160 | For 1.85:1 Flat. For 2.39:1 Scope: 2048x858 (2K) or 4096x1716 (4K) |
| Frame Rate | 24 fps (Academy standard) | If shot at 23.976, DCP-o-matic can conform to 24fps |
| Color Space | DCI-P3 | Grade in DCI-P3 for full range (requires calibrated monitor), or grade in Rec.709 (colors map correctly to P3) |
| Audio Codec | Linear PCM | |
| Audio | 48 kHz, 24-bit | |
| Audio Channels | 5.1 surround (SMPTE order) preferred | Stereo doesn't translate well to theater systems |
| DCP Standard | **Interop** (for 24fps) | Use **SMPTE** if non-24fps |
| Encryption | Unencrypted | Resolve creates perfect unencrypted DCPs. Encryption requires DCP-o-matic. Sundance/SXSW recommend indies skip encryption |

**Composition Settings (ISDCF Naming)**:
Click Edit to generate the standardized name: `Title_FTR-1_F_EN-XX_US-NR_51_2K_ST_YYYYMMDD_FAC_IOP_OV`
- Title, Content Type (FTR=Feature), Aspect Ratio (F=Flat), Language, Territory, Audio, Resolution, Standard, Date, Facility, Package Type

**Delivery**:
- Physical drive: Format as **Linux EXT2 or EXT3** with **128-inode size**. NOT NTFS, NOT exFAT, NOT Mac formatted
- Digital: ZIP the DCP folder, upload to festival platform
- Never rename the DCP folder or alter its internal structure
- Always verify by importing the DCP back into your Resolve timeline for playback check

### ProRes 422 HQ (Broadcast Delivery)

| Setting | Value |
|---|---|
| Format | QuickTime (.MOV) |
| Codec | Apple ProRes 422 HQ |
| Resolution | Match timeline (1920x1080 or 3840x2160) |
| Frame Rate | Match project |
| Data Levels | Auto (do NOT force Full) |
| Color Space Tag | Auto / Same as Project |
| Audio | Linear PCM, 48 kHz, 24-bit |

> Note: ProRes export from Resolve on Windows requires the Studio version. Mac: both free and Studio can export ProRes.

### H.265/HEVC (Streaming Delivery)

| Setting | Value |
|---|---|
| Format | MP4 or QuickTime |
| Codec | H.265 (HEVC) |
| Encoding Profile | Main (or Main 10 for 10-bit) |
| Bitrate | 15,000 kbps (1080p) / 40,000 kbps (4K) |
| Multi-pass | Enabled (better quality) |
| Audio | AAC, 320 kbps, 48 kHz |

For maximum quality: Encoding Profile: Main 4:4:4:10, Constant QP 0 (lossless equivalent).

### IMF (Interoperable Master Format)

IMF (SMPTE ST.2067) is the mastering and delivery standard used by Netflix, Amazon, Disney+, and major distributors. It supports multiple video/audio/subtitle tracks, Dolby Atmos, HDR, and supplemental packages (re-deliver only changed sections instead of the entire film).

#### DaVinci Resolve Studio Settings

| Setting | Value |
|---|---|
| Format | IMF |
| Codec | Kakadu JPEG 2000 |
| Package Type | App2 Extended (App2e) |
| Standard | SMPTE ST.2067 |
| Encoding Profile | IMF or Broadcast |
| Bit Depth | 12-bit or 16-bit |
| Lossless Compression | Optional |
| Audio | Linear PCM, 48 kHz, 24-bit |

Resolve Studio ($295, one-time) is the most accessible commercial option for creating basic IMF packages with Kakadu GPU-accelerated JPEG2000 encoding.

#### Open Source IMF Tools

A complete open source IMF ecosystem exists, though fragmented:

| Tool | Role | Repo | Status |
|---|---|---|---|
| **Photon** (Netflix) | IMF validation (the reference tool) | [github.com/Netflix/photon](https://github.com/Netflix/photon) | Active, v5.0.0 (March 2025) |
| **IMFTool** | CPL editing, asset management | [github.com/IMFTool/IMFTool](https://github.com/IMFTool/IMFTool) | Active, v1.9.7 (Nov 2024) |
| **OpenJPEG** | JPEG 2000 encoding/decoding (IMF profiles) | [github.com/uclouvain/openjpeg](https://github.com/uclouvain/openjpeg) | Active, v2.5.4, ~1,100 stars |
| **asdcplib** | MXF wrapping (AS-02 Track Files) | [github.com/cinecert/asdcplib](https://github.com/cinecert/asdcplib) | Active |
| **imflib** | Complete IMP creation (video + audio) | [github.com/MarkusPfundstein/imflib](https://github.com/MarkusPfundstein/imflib) | WIP, passes Photon validation |
| **FFmpeg** | IMF demuxer (reading only) | Merged upstream (Feb 2022) | `ffmpeg -f imf -i CPL.xml` |
| **OpenJPH** | HTJ2K (10x faster JPEG2000) | [github.com/aous72/OpenJPH](https://github.com/aous72/OpenJPH) | Active, decode only for now |

Full list maintained at: [imfug.com/open-source](https://www.imfug.com/open-source/)

#### Can You Create a Full IMF Package in Open Source?

Theoretically yes. Practically, it's fragile:

```
Source video (ProRes, DPX, EXR)
  --> FFmpeg (extract frames)
  --> OpenJPEG (encode JPEG2000 with IMF profile)
  --> asdcplib (wrap as MXF AS-02 Track Files)
  --> imflib or manual XML (generate CPL, PKL, Asset Map)
  --> Photon (validate the package)
  --> IMFTool (inspect, add tracks)
```

**What's missing in open source:** No Dolby Vision/Atmos support (requires Dolby licenses). No automated supplemental packages. OpenJPEG is extremely slow for feature-length 4K encoding (days vs hours with Kakadu). No encryption support (SMPTE ST 2067-10).

#### For Indie Filmmakers: Is IMF Relevant?

- **Delivering to Netflix/Disney+:** Yes, IMF is mandatory. But in practice, the post-production facility or distributor creates the IMF package, not the filmmaker.
- **Festival distribution:** No. Festivals want DCPs (DCP-o-matic for open source, Resolve Studio for commercial).
- **Self-distribution:** No. Platforms accept ProRes or H.264/H.265.
- **Archiving:** IMF is excellent for versioned archives (component-based), but DPX/EXR + WAV is simpler for an indie.

**Bottom line:** Know that IMF exists. Know that open source tools can validate and inspect IMF packages. But for indie filmmaking, DCP (via DCP-o-matic or Resolve Studio) and ProRes deliverables are what you'll actually create yourself. If a platform requires IMF, they'll either create it from your ProRes master or work with a certified facility.

#### Netflix Test Content

Netflix published [Meridian](https://sites.google.com/netflix.com/opencontent) (requires Google account), a Creative Commons short film available as a complete IMF package (UHD, 4K 59.94 HDR, Dolby Atmos, ~88.5 GiB). Use it to test and learn IMF workflows with the open source tools.

### Color Space Summary per Delivery Format

| Delivery | Output Color Space | Gamma |
|---|---|---|
| DCP (Cinema) | DCI-P3 | Gamma 2.6 |
| Broadcast (SDR) | Rec.709 | Gamma 2.4 |
| Web/Streaming (SDR) | Rec.709 | Gamma 2.4 (or sRGB/2.2 for web) |
| HDR10 | Rec.2020 | PQ (ST.2084) |
| Dolby Vision | Rec.2020 | PQ + RPU metadata |
| HLG (Broadcast HDR) | Rec.2020 | HLG |

---

# PART 2: Final Cut Pro with ML RAW

---

## 1. IMPORT WORKFLOW

### The Problem
Final Cut Pro does **not** natively support CinemaDNG or MLV files. You need an intermediary step.

### Method 1: ProRes Proxy Workflow (Most Common)

1. **Convert MLV to CinemaDNG** using MLV App (with RAW corrections applied)
2. **Import CinemaDNG into DaVinci Resolve**
3. In Resolve, create a timeline and export all clips as **ProRes 422 LT** (or ProRes Proxy) via the Deliver page
4. **Import ProRes files into FCP**
5. Edit in FCP using these proxies
6. For final grading: export FCPXML from FCP > conform in Resolve with original CinemaDNG (see Conform section)

### Method 2: MLV App Direct Export to ProRes

1. Open MLV in MLV App
2. Apply all RAW corrections (vertical stripes, focus pixels, pattern noise, chroma smoothing)
3. Set processing parameters (exposure, WB, contrast) to taste
4. Export as **Apple ProRes 4444** (preserves maximum quality including processing parameters)
5. Import ProRes files directly into FCP
6. **Limitation**: You lose RAW flexibility -- the processing is baked in

### Method 3: Color Finale Transcoder 2 (Direct CinemaDNG in FCP)

1. Install Color Finale Transcoder 2 (paid plugin from Color Trix)
2. In FCP: Window > Extensions > Color Finale Transcoder
3. Browse to CinemaDNG sequences
4. The extension can:
   - **Transcode** CinemaDNG to ProRes (Proxy through 4444 XQ), with RAW adjustments (color temp, tint, ISO, sharpness, highlight recovery, color space)
   - **Direct timeline editing** via FCP plugin (v2.0) -- place CinemaDNG on timeline without transcoding
5. Transcoded files import directly into FCP's library
6. Camera metadata is preserved and visible in FCP

**Color Finale Transcoder 2 RAW Adjustment Controls:**
- Color Temperature / Tint
- ISO override
- Sharpness / Detail
- Highlight Recovery
- Debayering Mode
- Color Space selection
- LUT Preview
- RGB Parade, Waveform, Histogram scopes

### Method 4: RAWMagic (macOS)

1. Drag MLV files onto RAWMagic
2. Converts to CinemaDNG sequences that "just work" as if from a Blackmagic camera
3. Audio is trimmed to match video length (easier sync)
4. Import resulting CinemaDNG into Resolve for proxy creation or direct use

### Library & Event Organization

| Element | Recommendation |
|---|---|
| Library | Create one per project (e.g., `MyFilm.fcpbundle`) |
| Event | One per shooting day or scene |
| Keyword Collections | Use for characters, locations, shot types |
| Smart Collections | Set up for aspect ratios, frame rates |
| Storage Location | External SSD for media; keep Library on internal for performance |

---

## 2. EDITING WORKFLOW IN FCP

### Optimized vs. Proxy Media

| Setting | Use Case | How |
|---|---|---|
| **Proxy Media** | Offline editing with low-power machine | FCP > Preferences > Playback > Proxy. Or: File > Transcode Media > Create Proxy Media |
| **Optimized Media** | Better quality editing (ProRes 422) | FCP > Preferences > Playback > Original/Optimized |
| **Original Media** | Final quality (if machine can handle it) | Default |
| Toggle in Viewer | Switch between Proxy/Optimized/Original | View > Proxy & Optimized Media |

For ML RAW proxy workflow: You already have ProRes proxies from Resolve or MLV App. Import those directly -- no need to create additional proxies in FCP.

### Multicam Editing with ML RAW + Other Cameras

1. Create a **Multicam Clip**: File > New > Multicam Clip
2. Add all camera angles (ProRes from ML, native files from other cameras)
3. Sync method: **Audio** (if guide audio available) or **Timecode** (if jam-synced)
4. Enable **Angle Viewer**: View > Show in Viewer > Angles
5. Edit in real time by clicking angles during playback

> Frame rate matching: If ML RAW is 24p and another camera is 25p, FCP will conform the 25p to match the multicam clip's base frame rate.

### Color Correction Tools in FCP

#### Color Board (legacy, simple)
- Window > Go To > Color Board (or Cmd + 6 after selecting clip)
- Four panes: Color, Saturation, Exposure, (per Shadows/Midtones/Highlights/Global)
- Good for quick adjustments
- **Transfers to DaVinci Resolve via FCPXML**

#### Color Wheels (preferred)
- Inspector > Color section > Color Wheels
- Lift/Gamma/Gain wheels + Temperature/Tint
- More intuitive than Color Board
- **Does NOT transfer to Resolve via FCPXML**

#### Color Curves
- Inspector > Color section > Color Curves
- Luma curve + individual R/G/B curves
- Hue vs. Hue, Hue vs. Sat, Hue vs. Luma, Luma vs. Sat
- Powerful for targeted adjustments
- **Does NOT transfer to Resolve via FCPXML**

> **Key point for round-tripping**: Only Color Board adjustments transfer via FCPXML. If planning to grade in Resolve, do NOT apply Color Wheels or Curves corrections in FCP -- they will be lost.

---

## 3. FINISHING IN FCP

### Applying LUTs

#### Native (FCP 10.4+ / FCP 12)
1. Select clip in timeline
2. Inspector (Cmd + 4) > Effects > Add Effect
3. Search for **"Custom LUT"** effect
4. Drag onto clip (or double-click to add)
5. In Inspector: click dropdown > Choose Custom LUT > navigate to your `.cube` file
6. Select your LUT (e.g., "Kodak 2383 Rec709", "S-Log3 to Rec709")

#### Via Color Finale 2 Pro
1. Install Color Finale 2 Pro plugin
2. Apply Color Finale effect to clip
3. Open LUT Gallery (visual browser)
4. Add LUT folders to whitelist (Color Finale > Preferences > LUTs tab)
5. Click "Insert LUT or Preset" > browse and select
6. Adjust LUT intensity
7. Stack multiple LUTs with blend modes
8. Export custom LUTs as `.cube` files for reuse

#### Via LUT Utility (Color Grading Central)
- Dedicated LUT loading plugin for FCP
- Apply to individual clips or adjustment layers

#### LUT Application Order
1. **First**: Apply noise reduction
2. **Second**: Primary color correction (exposure, WB)
3. **Third**: Camera LUT (log-to-display conversion)
4. **Fourth**: Creative/Film emulation LUT
5. **Fifth**: Final adjustments

### Third-Party Grading Plugins

#### Color Finale 2 Pro
- Log Wheels, Six Vectors, Shuffle
- Halation, bloom, film grain, vignette simulation
- Non-destructive layer-based grading
- ACES working color space support
- LUT creation and export
- Area tracker and masks
- Camera Matrix

#### FilmConvert Nitrate
- Camera-specific film stock emulation (Canon 5D3 profile available)
- Grain simulation from real film scans
- Kodak and Fuji stocks
- Adjustable film curve, saturation, grain amount
- Export grades as `.cube` LUTs for use on set or in other apps

### Noise Reduction in FCP

#### Built-in Noise Reduction
1. Select clip in timeline
2. Effects Browser (Cmd + 5) > Basics category > **Noise Reduction**
3. Drag to clip
4. **Apply as FIRST effect** (top of effects list) for best performance
5. Inspector adjustments:
   - Amount: Low / Medium / High
   - Sharpness: adjustable

#### Third-Party: Neat Video (Recommended for ML RAW)
- Neat Video v6 for FCP: professional-grade denoising
- Builds custom noise profile from your footage
- Can detect and remove structured noise (banding)
- Preserves detail without plastic/unnatural look
- Significantly better than built-in NR for severe noise

#### Third-Party: BorisFX Continuum
- Multiple denoise algorithms
- Can target specific frequency ranges

### Sharpening in FCP
- Effects > Basics > **Sharpen**
- Or use Color Finale's Details/Sharpness tool
- Apply AFTER noise reduction
- Use sparingly on ML RAW -- the sensor resolution is already the limiting factor

---

## 4. ROUND-TRIP FCP <-> DaVinci Resolve

### Step 1: Export FCPXML from FCP

1. Select project in Browser (or open in timeline)
2. **File > Export XML**
3. Choose **XML version 1.9** (Resolve supports v1.3-1.9)
4. Verify file has `.fcpxml` extension (NOT `.fcpxmld` -- v1.10+ creates these, won't work)
5. Save

### Step 2: Import into DaVinci Resolve

1. Create new project in Resolve
2. File > Import > Timeline (Shift + Cmd + I)
3. Open `.fcpxml` file
4. Configure:
   - Name the timeline
   - Deselect "Automatically import source clips into media pool" (if relinking to CinemaDNG)
   - Check "Use color information" to bring FCP color adjustments
   - "Mixed frame rate format" handles footage with different FPS
5. Resolve auto-relinks media if found in expected directories

### Step 3: Relink to CinemaDNG (if editing was done with proxies)

1. Pre-import CinemaDNG sources into Media Pool
2. Select offline clips > right-click > **Change Source Folder**
3. Navigate to CinemaDNG parent directory
4. Resolve resolution-independently relinks -- all crops, resizes, tracking, power windows are preserved

### Step 4: Grade in Resolve
- Apply full CST workflow as described in Part 1
- Use Camera RAW palette for per-clip RAW adjustments
- Apply creative grades, LUTs, film emulation

### Step 5: Export from Resolve back to FCP

1. Go to **Deliver** page
2. Select **"Final Cut Pro"** preset
3. Choose codec: ProRes 422 HQ (standard) or ProRes 4444 (HDR/animation)
4. Set destination folder
5. Click **Add to Render Queue** > **Render All**
6. Resolve exports: graded video files + XML file

### Step 6: Import back into FCP

1. In FCP: File > Import > XML
2. Navigate to the XML file Resolve created
3. A new Event is created with all graded clips
4. Titles and timing are preserved

### What Transfers (FCP -> Resolve)

| Element | Transfers? |
|---|---|
| Timeline structure | Yes |
| Clip positions, durations, edits | Yes |
| Opacity, Position, Scale, Rotation | Yes |
| Keyframed transform animations | Yes |
| Color Board adjustments | Yes |
| Crossfades/dissolves | Yes |
| Time remaps / speed changes | Yes (some caveats with Optical Flow) |
| Compound clips | Yes (decompose first for best results) |
| Multicam clips | Yes |
| Audio levels | Yes |

### What Does NOT Transfer

| Element | Transfers? |
|---|---|
| Color Wheels corrections | No |
| Color Curves corrections | No |
| Secondary corrections (Color/Shape Masks) | No |
| Titles (design/formatting) | No (appear as gray boxes; text content and timing preserved) |
| .gif files or freeze frames | No |
| Generators | No |
| FCP-specific effects/plugins | No |
| Tracking data / masks from Resolve | No (baked into rendered files) |
| AIFF audio | No (convert to WAV) |

### Using Compressor for Final Delivery from FCP

After grading round-trip is complete and graded clips are back in FCP:

1. **File > Share > Send to Compressor** (or use Compressor Presets destination)
2. In Compressor, select format:
   - **Broadcast**: ProRes 422 HQ in MOV container, apply Broadcast Safe filter
   - **Streaming**: H.264 or HEVC with custom bitrate control
   - **Web**: HEVC for up to 40% smaller files than H.264
3. Compressor offers control over bitrate, encoding profile, frame rate conforming
4. Can create MXF files (AVC-Intra, D-10/IMX, ProRes, XDCAM HD)
5. Export HLS-ready files for adaptive streaming
6. Add captions/subtitles

---

## 5. SPECIFIC FCP TIPS FOR ML RAW

### Frame Rate Handling
- ML RAW frame rates depend on camera settings and resolution mode
- Common: 23.976 (NTSC), 25 (PAL), 29.97, 24
- 5D3 NTSC: 23.976fps, 29.97fps; PAL: 25fps
- EOS M NTSC: 23.976fps, 29.97fps, 59.94fps; PAL: 25fps
- **Set FCP project frame rate to match BEFORE editing** -- it cannot be changed after media is on the timeline
- If first clip sets wrong frame rate: Modify > Edit Project Properties (only works before editing)
- For GyroFlow stabilization: exact frame rate is critical to avoid "Framerate mismatch" errors

### Aspect Ratio Management

ML RAW records at various non-standard resolutions depending on the mode:

| Camera | Mode | Resolution | Aspect Ratio |
|---|---|---|---|
| 5D3 | Full-frame 1080p | 1920x1080 | 16:9 |
| 5D3 | Crop mode 3.5K | 3584x1320 | 2.71:1 |
| 5D3 | 1x3 binning | 1920x818 | 2.35:1 |
| 5D3 | 5K (burst) | 5208x3480 | 3:2 (sensor native) |
| EOS M | Standard | 1736x976 | ~16:9 |
| EOS M | 5K anamorphic | 1264x2134 | Vertical (needs rotation/stretch) |

**In FCP:**
- Set project to your desired output aspect ratio (e.g., 1920x1080 for 16:9, 2048x858 for 2.39:1 Scope)
- Use **Spatial Conform** in Inspector: Fit, Fill, or None
- For 3:2 sensor crops in a 16:9 project: use "Fit" and add letterboxing, or "Fill" and allow crop
- For anamorphic ML modes: apply desqueeze in Inspector (Scale X to stretch)
- Custom aspect ratios: modify Project Properties to any resolution

### Audio Sync

**The Challenge**: ML RAW video and audio are handled differently depending on the recording mode:
- `mlv_snd` module records audio within the MLV file
- Audio files are always slightly longer than video
- Some conversion tools don't handle audio perfectly

**Solutions:**
1. **RAWMagic**: Creates CinemaDNG files with properly trimmed audio -- best automatic sync
2. **MLV App**: Exports separate audio file alongside video. Import both into FCP, use Synchronize Clips
3. **Manual sync in FCP**:
   - Select video + audio clips
   - Clip > Synchronize Clips
   - Sync by: Audio (waveform matching) or First Marker
4. **External audio recorder**: If using separate audio (Zoom, Tascam), use FCP's Synchronize Clips with audio waveform analysis
5. **H.264 proxy reference**: Record ML RAW + H.264 proxy simultaneously. H.264 has embedded audio. Sync CinemaDNG to H.264 audio, then replace video

### Stabilization on ML RAW Footage

**FCP Built-in:**
1. Select clip in timeline
2. Inspector > Video tab > Stabilization section
3. Enable Stabilization
4. Method: Automatic, InertiaCam, or SmoothCam
5. Translation Smooth / Rotation Smooth / Scale Smooth: adjust to taste
6. Works well for minor shake. Can produce wobble/jello on handheld ML RAW

**GyroFlow (DaVinci Resolve OFX)**:
- For ML cameras with gyroscope data or separate gyro logger
- GyroFlow OFX plugin in Resolve's Fusion page
- Handles 1x1 and 1x3 anamorphic clips
- Must apply exact frame rate to avoid mismatch errors
- Export GyroFlow project files linked back in Fusion

### Speed Ramping

1. Select clip in timeline
2. **Retime Menu** (Cmd + R to show Retime controls)
3. Click the speed indicator > choose desired speed (50%, 200%, etc.)
4. For variable speed: Retime Menu > Blade Speed (Shift + B) to create speed segments
5. Drag the speed segment boundaries to ramp
6. Retime quality: Video Inspector > Retime > **Optical Flow** (best quality, slowest render) or **Frame Blending**

> **Caution**: Optical Flow speed changes on ML RAW may cause artifacts at edit points when conforming back to Resolve (imports as composite clips).

---

# APPENDIX: Quick Reference Cheat Sheet

## Recommended Workflow Summary

### For Maximum Quality (Feature Film / Festival)

```
MLV --> MLV App (RAW corrections + CinemaDNG export with Resolve naming)
    --> DaVinci Resolve (import CinemaDNG, export ProRes proxies)
    --> FCP (edit with proxies, export FCPXML v1.9)
    --> DaVinci Resolve (conform FCPXML, relink CinemaDNG, full color grade)
    --> Deliver (DCP for festivals, ProRes 422 HQ for broadcast, H.265 for streaming)
```

### For Speed (Short Form / Web Content)

```
MLV --> MLV App (RAW corrections + ProRes 4444 export with processing baked in)
    --> FCP (edit + basic grade)
    --> Compressor (deliver H.265 for web)
```

### For Color Finale Users

```
MLV --> MLV App (RAW corrections + CinemaDNG export)
    --> Color Finale Transcoder 2 (transcode to ProRes or edit CinemaDNG directly in FCP)
    --> FCP (edit + grade with Color Finale 2 Pro)
    --> Compressor or direct share
```

---

# SOURCES

## Forums & Communities
- [Blackmagic Forum: Cinema DNG Workflow](https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=219255)
- [Magic Lantern Forum: Good Project Settings for Resolve](https://www.magiclantern.fm/forum/index.php?topic=17324.0)
- [Magic Lantern Forum: Workflow for DaVinci Resolve](https://www.magiclantern.fm/forum/index.php?topic=26669.0)
- [Magic Lantern Forum: DaVinci Resolve and ML Raw](https://www.magiclantern.fm/forum/index.php?topic=15801.0)
- [Magic Lantern Forum: CinemaDNG Decode Settings](https://www.magiclantern.fm/forum/index.php?topic=26707.0)
- [Magic Lantern Forum: Vertical Stripes Revisited (5D3)](https://www.magiclantern.fm/forum/index.php?topic=17795.0)
- [Magic Lantern Forum: Heavy Vertical Banding Noise Removal](https://www.magiclantern.fm/forum/index.php?topic=15372.0)
- [ACESCentral: ACES Workflow for CinemaDNG](https://community.acescentral.com/t/aces-workflow-for-color-grading-cinemadng/1801)

## Professional Guides & Tutorials
- [Frame.io: How to Manage RAW Footage Using Nodes in DaVinci Resolve](https://blog.frame.io/2024/06/03/how-to-manage-raw-footage-using-nodes-in-davinci-resolve/)
- [Frame.io: A New Approach to Grading Color in DaVinci Resolve](https://blog.frame.io/2024/04/01/new-resolve-template-node-graphs-color-grading/)
- [Frame.io: Color Management Cheat Sheet for DaVinci Resolve](https://blog.frame.io/2023/12/04/color-management-cheat-sheet-davinci-resolve/)
- [Frame.io: How to Color Manage Using Nodes in DaVinci Resolve](https://blog.frame.io/2024/01/08/color-management-nodes-davinci-resolve/)
- [Frame.io: Turbo Charge Your FCP X Workflow with DaVinci Resolve](https://blog.frame.io/2017/10/04/turbo-charge-fcpx-workflow-davinci-resolve/)
- [CROMO Studio: The Perfect Node Tree for Color Grading](https://cromostudio.it/cromo-tips/understanding-nodes-and-node-order-in-davinci-resolve)
- [Juan Melara: Basic Resolve Node Structure and Order of Operations](https://juanmelara.com.au/blog/basic-resolve-node-structure-and-order-of-operations)
- [Fstoppers: Shooting 3.5K Raw with Magic Lantern](https://fstoppers.com/education/shooting-35k-raw-magic-lantern-heres-how-color-davinci-and-edit-premiere-249719)
- [CineD: Guide RAW on 5D Mark III with Magic Lantern](https://www.cined.com/guide-raw-on-a-5d-mark-iii-magic-lantern/)
- [Franck Rondot: Workflow sans perte ML RAW Resolve et FCP](https://www.franck-rondot.com/blog-photographe/262-workflow-sans-perte-magic-lantern-raw-davinci-resolve-final-cut-pro-x.html)
- [No Film School: Magic Lantern DNG RAW in Resolve](https://nofilmschool.com/2013/07/magic-lantern-dng-raw-files-supported-davinci-resolve-9-1-5)
- [No Film School: Resolve 10 Fixes ML RAW Fringing](https://nofilmschool.com/2013/09/davinci-resolve-10-fixes-magic-lantern-raw-fringing)

## Color Grading & LUTs
- [Jamie Fenn: Free DWG Film Emulation LUTs](https://www.jamiefenn.com/p/free-dwg-film-emulation-luts/)
- [Juan Melara: Print Film Emulation LUTs](https://juanmelara.com.au/blog/print-film-emulation-luts-for-download)
- [CinePrint35 Film Emulation PowerGrades](https://www.tombolles.net/cineprint35)
- [Mononodes: Color Management in DaVinci Resolve](https://mononodes.com/color-management-in-davinci-resolve/)
- [Miracamp: Best Color Space for DaVinci Resolve](https://www.miracamp.com/learn/davinci-resolve/best-color-space)

## DCP & Delivery
- [Mixing Light: Create a DCP in DaVinci Resolve](https://mixinglight.com/color-grading-tutorials/create-digital-cinema-package-dcp-davinci-resolve/)
- [Color Culture: Making DCP in DaVinci Resolve](https://colorculture.org/making-dcp-in-davinci-resolve/)
- [Jonny Elwyn: How to Make a DCP for Film Festival Projection](https://jonnyelwyn.co.uk/film-and-video-editing/how-to-make-a-dcp-for-film-festival-projection/)
- [Mixing Light: What is an IMF and How to Create One](https://mixinglight.com/tutorial-category/imf/)
- [DaVinci Resolve Manual: IMF Encoding](https://www.steakunderwater.com/VFXPedia/__man/Resolve18-6/DaVinciResolve18_Manual_files/part3964.htm)

## FCP Round-Tripping
- [Larry Jordan: Round-Tripping Between FCP and DaVinci Resolve](https://larryjordan.com/articles/round-tripping-projects-between-final-cut-pro-and-davinci-resolve/)
- [Ripple Training: Round-Tripping with FCP in DaVinci Resolve](https://www.rippletraining.com/blog/davinci-resolve/round-tripping-with-final-cut-pro-x-in-davinci-resolve/)
- [Partners in Post: Roundtripping FCP 10.4.5 and Resolve 15](https://www.partnersinpost.com/blog/2019/2/1/roundtripping-between-final-cut-104-and-davinci-resolve-15-part-1)

## Tools
- [MLV App](https://mlv.app/) | [GitHub](https://github.com/ilia3101/MLV-App)
- [Color Finale Transcoder 2](https://colorfinale.com/transcoder)
- [Color Finale 2 Pro](https://colorfinale.com/color-finale-2)
- [FilmConvert Nitrate for FCP](https://www.filmconvert.com/plugin/final-cut-pro)
- [Neat Video for FCP](https://www.neatvideo.com/blog/post/reduce-noise-fc)
- [FastCinemaDNG: Focus Pixel Suppression](https://www.fastcinemadng.com/info/mlv/mlv-focus-pixels.html)
- [MLVFS (Virtual Filesystem)](https://github.com/davidmilligan/MLVFS)
- [SlimRAW (Lossless DNG Compression)](https://www.slimraw.com/article-proxies.html)
- [Netflix: Color Managed Workflow in Resolve ACES](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360002088888-Color-Managed-Workflow-in-Resolve-ACES)

## FCP Noise Reduction & Plugins
- [Apple Support: Reduce Video Noise in FCP](https://support.apple.com/guide/final-cut-pro/reduce-video-noise-ver7d031487b/mac)
- [Neat Video Quick Start for FCP](https://www.neatvideo.com/support/quick-start-guides/nv5/fc)
- [Larry Jordan: FCP Export Best Quality ProRes](https://larryjordan.com/articles/how-to-get-the-best-image-quality-exporting-prores-master-files/)
- [Apple Support: Share from FCP Using Compressor](https://support.apple.com/guide/final-cut-pro/share-using-compressor-ver1ff89071/mac)

## Stabilization
- [Magic Lantern Central: Stabilizing CinemaDNG with GyroFlow in Resolve](https://www.patreon.com/posts/stabilizing-raw-136840021)
