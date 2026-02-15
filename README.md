# Open Source Cinema

**RAW video filmmaking with hacked cameras and open hardware.**

I'm a filmmaker. I shoot auteur cinema with tools that weren't designed for it, or rather, tools that the manufacturers *deliberately prevented* from doing what the hardware could already do. This is a documentation of my workflow, my tools, and why open source matters in cinema.

---

## Table of Contents

- [Philosophy](#philosophy)
- [Magic Lantern RAW on Canon 5D Mark III](#magic-lantern-raw-on-canon-5d-mark-iii)
  - [Why RAW on a DSLR](#why-raw-on-a-dslr)
  - [The Hack](#the-hack)
  - [Resolution Modes I Use](#resolution-modes-i-use)
  - [Anti-Aliasing Strategy](#anti-aliasing-strategy)
  - [Dual ISO: 14 Stops from a $1500 Body](#dual-iso-14-stops-from-a-1500-body)
- [My Post-Production Workflow](#my-post-production-workflow)
  - [On-Set: Proxy + RAW Simultaneous Recording](#on-set-proxy--raw-simultaneous-recording)
  - [Offline Edit](#offline-edit)
  - [Online Conform + Color Grading](#online-conform--color-grading)
  - [Software Pipeline](#software-pipeline)
- [Aspect Ratios & Framing](#aspect-ratios--framing)
- [Equipment](#equipment)
- [AXIOM: The Open Source Cinema Camera](#axiom-the-open-source-cinema-camera)
  - [What Is It](#what-is-it)
  - [Technical Specs](#technical-specs)
  - [What I Find Compelling](#what-i-find-compelling)
  - [The RAW Pipeline Connection](#the-raw-pipeline-connection)
  - [Honest Assessment](#honest-assessment)
  - [How I Could Use It](#how-i-could-use-it)
- [Why This Matters](#why-this-matters)
- [Forum Activity & References](#forum-activity--references)

---

## Philosophy

Cinema cameras are black boxes. You pay $20,000-$80,000 for an ARRI or a RED, and you get a sealed pipeline: their sensor, their debayering, their color science, their codec. You trust their engineers made the right choices. Most of the time they did.

But what if you could open the box? What if the $1,500 DSLR sitting in your bag had a sensor capable of 14-bit RAW, 12-13 stops of dynamic range, and resolutions up to 3.5K, and the manufacturer just... turned it off? Left it locked behind firmware that only outputs compressed 8-bit H.264?

That's what Magic Lantern proved. And that's what the AXIOM camera is building from scratch.

I've been working with these tools since 2013. This repo documents what I've learned.

---

## Magic Lantern RAW on Canon 5D Mark III

### Why RAW on a DSLR

The difference between Canon's native H.264 and Magic Lantern RAW is not subtle.

| | Canon H.264 | ML RAW (14-bit) | ML RAW + Dual ISO |
|---|---|---|---|
| Bit depth | 8-bit (256 levels) | 14-bit (16,384 levels) | 14-bit (interpolated) |
| Dynamic range | ~10-11 stops | ~12-13 stops | ~14 stops |
| Codec | H.264 IPB, lossy | Uncompressed or lossless | Interlaced dual gain |
| Post latitude | Very limited (banding at +1.5 EV) | Massive | Comparable to ARRI ALEXA |
| Color depth | 4:2:0 | Full RGB Bayer | Full RGB Bayer |

Push an H.264 file 1.5 stops in post and you get banding, halos around highlights, crushed blacks. Push a 14-bit RAW file 3 stops and it holds. The image has *body*. Transitions between shadow and light are smooth, not quantized. It looks like film stock, not video.

For the kind of cinema I make, where available light, long takes, and post-production latitude are essential, this is not a luxury. It's the difference between working and not working.

### The Hack

Magic Lantern is not a modified firmware. It's an independent program that runs alongside Canon's firmware, loaded from the memory card at each boot. The only permanent change is enabling boot from card (a reversible flag).

What the community achieved through reverse engineering:

- **RAW video recording** (2013): discovered that the sensor can write its raw data directly to card. Canon deliberately disabled this.
- **Dual ISO** (2013): found undocumented registers on the CMOS controller chip that allow programming two different ISO levels simultaneously. This is analog amplification *before* A/D conversion.
- **Full sensor readout** (2017): proved the 5D3 sensor can do 4K and even 5.7K. Canon limited the camera to 1080p H.264.
- **Lossless compression**: implemented JPEG lossless compression of RAW data, making continuous 3.5K recording possible.
- **30-minute limit bypass**: removed the recording limitation imposed for EU customs taxation reasons.

The project was started by [Trammell Hudson](https://trmm.net/Magic_Lantern_firmware/) in 2009. It has been used on all seven continents and aboard the International Space Station. It remains active in 2025 with support expanding to new Canon bodies.

### Resolution Modes I Use

| Mode | Resolution | Bit Depth | Crop | FPS | Continuous | Use Case |
|---|---|---|---|---|---|---|
| **1080p full frame** | 1920x1080 | 14-bit lossless | 1x (full frame, 3x3 binning) | 23.976/25 | Yes, unlimited | Primary shooting mode. Reliable, superior to H.264 |
| **3.5K Super 35** | 3584x1320 | 10-bit lossless | ~1.5x (Super 35 equivalent) | 24/25 | Yes (~90 MB/s) | Cinema mode. Resolution above 2K DCI. ~48% lossless compression |
| **5.7K anamorphic (1x3)** | 5796x1934 | 10-bit | 1x3 binning | 24 | With SD overclock | Anamorphic stretch for ultra-wide. No aliasing from hardware binning |
| **1080p crop 60fps** | 1920x1080 | 10/12-bit | 1:1 (~2.6x) | 50/60 | Yes | Slow motion. Pixel-for-pixel readout, photo-quality |
| **3:2 full frame** | 1920x1280 | 14-bit | 1x (full frame) | 24 | Yes | Native sensor ratio. With anamorphic lens: desqueeze to ~3K scope |

The **3.5K 10-bit lossless** is the practical cinema mode. The crop factor is equivalent to Super 35mm, the resolution exceeds 2K DCI, and it records continuously on a fast CF card (Komputerbay 1050x or Lexar 1066x).

The **5.7K mode** is not a standard shooting mode (it's 7.4 fps at full sensor readout), but the 1x3 binning anamorphic variant is usable at 24fps. I've been exploring whether real-time live view is possible during 5.7K 1x3 recording with SD card overclocking at 160MHz, and comparing the output to AI upscaling (Topaz Video Enhance) from 1:1 crop to 5.7K.

### Anti-Aliasing Strategy

Aliasing is the enemy. On Canon DSLRs, the line-skipping readout in video mode creates moire and aliasing artifacts that can ruin footage. My approach:

**In-camera:**
- **1x3 binning modes**: hardware-level binning before readout. No aliasing by design.
- **Crop mode (1:1 pixel readout)**: reads the sensor pixel-for-pixel. No line skipping, no aliasing. The trade-off is a tighter crop (~2.6x).
- **MV1080 with crop mode on EOS M**: the best regular shooting mode without aliasing on that body.

**Optical:**
- **Mosaic Engineering VAF filter** on the Canon 7D: optical low-pass filter designed specifically for video. Essential for the 7D which has severe aliasing.

**In post:**
- Explored **enfuse/hugin align_image_stack** technique for reducing aliasing on pre-graded TIFF files.
- MLV App demosaicing: **AMaZE** for final export (best overall quality), **IGV** for high ISO footage, **LMMSE** for low ISO. Both LMMSE and IGV can reduce aliasing artifacts compared to AMaZE.

### Dual ISO: 14 Stops from a $1500 Body

This is the most impressive hack Magic Lantern achieved. The technical explanation:

1. The 5D3 sensor has an **8-channel readout** (most Canon bodies have 4).
2. ML discovered it could program **2 independent amplifier circuits** at different ISO levels.
3. Half the sensor lines are read at **ISO 100** (preserving highlights), the other half at **ISO 1600** (recovering shadows).
4. The amplification is **analog**, before A/D conversion. This is fundamentally different from (and superior to) digital push in post.
5. Lines are interlaced in pairs (0-1 at one ISO, 2-3 at the other) to maintain the RGGB Bayer pattern.
6. The result is fused in post-production to create a single image with ~14 stops of dynamic range.

The principle is similar to how the ARRI ALEXA sensor works: dual readout combining shadows and highlights early in the pipeline. Except the ALEXA costs $40,000+.

Trade-offs: halved vertical resolution, increased moire in over/underexposed zones. But for scenes with extreme contrast (available light interiors with windows, night exteriors), it's transformative.

I process Dual ISO footage through **Switch** (formerly cr2hdr.app) on macOS.

---

## My Post-Production Workflow

### On-Set: Proxy + RAW Simultaneous Recording

The 5D Mark III can record H.264 proxy and MLV RAW simultaneously. This is a critical feature for me.

RAW playback on-camera is unstable and causes frequent crashes. I stopped using it because I had to reset the camera too many times. The H.264 proxy playback is stable and lets me review takes immediately on set, which is essential when working with actors.

The proxy file also serves as the offline editing medium. No transcoding step needed before editing.

### Offline Edit

```
[Card] --> Copy .MLV + .H264 files
                |
                +--> H.264 proxy --> Import into NLE
                |                         |
                |                    [OFFLINE EDIT]
                |                         |
                |                    Export FCPXML / XML / AAF
                |
                +--> .MLV files --> [ARCHIVE / Online storage]
```

I edit offline with the H.264 proxies in **Premiere Pro CC**, **Avid Media Composer**, or **Final Cut Pro X**. The timeline is light, responsive, and I can work on any machine.

The key is proper file naming from the shoot. Reel numbers and consistent naming conventions make the conform work.

### Online Conform + Color Grading

```
[MLV files]
     |
     v
[MLV App] --> CinemaDNG export (fast pass: raw data preserved)
     |
     v
[DaVinci Resolve]
     |
     +--> Import FCPXML from offline edit
     |
     +--> Conform: relink to CinemaDNG sequences
     |         (deselect "Automatically import source clips")
     |
     +--> Color grade on RAW
     |         Camera RAW --> Rec.709 decode
     |         CST node --> DaVinci Wide Gamut / Intermediate
     |         Grade in DWGI
     |         CST out --> delivery color space
     |
     v
[DELIVERABLES] --> ProRes 4444 / DPX / final format
```

**Critical detail**: CinemaDNG export from MLV App only applies RAW corrections (bad pixel fix, vertical stripes, pattern noise, deflicker). Exposure, white balance, contrast are NOT baked in. The raw data is preserved for grading in Resolve.

**On color science**: there is no official ACES IDT for Canon cameras with Magic Lantern. The DaVinci Wide Gamut / DaVinci Intermediate workflow is more reliable than attempting ACES with an improvised input transform.

This is the same offline/online methodology used on ARRI ALEXA productions. You can do offline/online from 720p ALEXA proxy and reconnect to 2K RAW with proper naming. Same principle here, different price point.

### Software Pipeline

| Stage | Tool | Role |
|---|---|---|
| RAW processing | **MLV App** | MLV to CinemaDNG, RAW corrections, debayering |
| Dual ISO | **Switch** (cr2hdr) | Dual ISO frame fusion |
| Quick preview | **MlRawViewer** | GPU-accelerated MLV preview |
| Filesystem access | **MLVFS** | Mount MLV as DNG folder (slow but useful) |
| DNG compression | **SlimRAW** | Lossless DNG compression for archival |
| Fast conversion | **fastcinemadng** | Batch DNG processing |
| Offline edit | **Premiere Pro / Avid / FCP X** | Proxy-based editing |
| Conform + grade | **DaVinci Resolve** | FCPXML import, CinemaDNG grading |
| Upscaling (test) | **Topaz Video Enhance AI** | Comparing AI upscale vs native resolution |

All core tools (MLV App, MlRawViewer, MLVFS, Switch, raw2dng) are **open source**. The entire pipeline from camera to conform can run on free software.

---

## Aspect Ratios & Framing

I frame for specific ratios that align with the cinema I reference:

| Ratio | Usage | Reference |
|---|---|---|
| **1.33:1** (4:3) | Academy ratio. Intimate, portrait-friendly. | Philippe Garrel, Xavier Dolan (*Mommy*) |
| **1.66:1** | European widescreen. Less aggressive than 1.85. | Standard European theatrical |
| **1.16:1** | Near-square. Experimental, claustrophobic. | Auteur / experimental cinema |
| **3:2** (1.5:1) | Native sensor ratio of the 5D3. With 1.33x anamorphic desqueeze: scope. | |
| **2.39:1** | Anamorphic scope via 5.7K 1x3 binning + desqueeze. | |

Custom cropmarks for these ratios are loaded into Magic Lantern for on-set framing.

---

## Equipment

| Item | Role | Notes |
|---|---|---|
| **Canon 5D Mark III** | Primary cinema body | Full frame, 14-bit RAW, Dual ISO, best ML support |
| **Canon 7D** | Secondary / APS-C body | With Mosaic Engineering VAF filter for anti-aliasing |
| **Canon EOS M** | Compact / B-cam | MV1080 crop mode for alias-free shooting, C-mount compatible |
| **Komputerbay 1050x CF** | Primary recording media | Required for continuous 3.5K recording (~90 MB/s write) |
| **Mosaic Engineering VAF** | Optical low-pass filter | Essential for 7D video (severe aliasing without it) |
| **PL-mount modified body** | Cinema lens compatibility | Modified Canon bodies available ~$600 on eBay |

---

## AXIOM: The Open Source Cinema Camera

### What Is It

The [AXIOM](https://www.apertus.org/) is the world's first **fully open source cinema camera**: open hardware (CERN OHL), open software (GPL v3), open documentation (CC BY-SA). Built by the [apertus](https://apertus.org/) community since 2006.

It won the **Ars Electronica** prize in 2012. The crowdfunding campaign in 2014 raised 204,568 EUR (175% of goal). It's certified by the Open Source Hardware Association and supported by the Free Software Foundation Europe.

### Technical Specs

| Spec | Details |
|---|---|
| **Sensor** | ams CMV12000, Super 35mm / APS-C |
| **Resolution** | 4096 x 3072 (4K, native 4:3) |
| **Bit depth** | 12-bit per pixel |
| **Shutter** | **Global shutter** (no rolling shutter artifacts) |
| **Max framerate** | 300 fps (10-bit), 150 fps (4K full res) |
| **Dynamic range** | ~10 stops native, **up to 15 stops in PLR HDR mode** |
| **Sensor swap** | Yes, physically interchangeable |
| **FPGA** | Xilinx Zynq Z-7020 (reprogrammable image pipeline) |
| **Recording** | External via USB 3.0 (~400 MB/s per module) or HDMI |
| **Output** | 3x independent HDMI (Full HD 4:4:4 from 4K downscale) |
| **Lens mount** | Sony E-mount native, Canon EF / Nikon F / MFT adapters |
| **OS** | Linux (Arch Linux), SSH + WebUI control |
| **Price** | Developer Kit: 3,990 EUR / Compact: 5,990 EUR |

### What I Find Compelling

**The FPGA changes everything.** Every other camera (ARRI, RED, Blackmagic) uses a fixed ASIC for image processing. The AXIOM's Xilinx FPGA is fully reprogrammable. You can modify the debayering algorithm, the LUT pipeline, the noise correction, the color space conversion. You're not locked into someone else's image processing choices.

**Global shutter at this price point.** No jello, no banding, no rolling shutter artifacts. Most cinema cameras under $10,000 have rolling shutters. The CMV12000 has a true global shutter.

**The sensor is interchangeable.** Not the lens. The *sensor*. This is a fundamentally different approach to camera design: you upgrade modules, not the entire body.

**150 fps at 4K.** Most cinema cameras limit to 60 fps in 4K. The AXIOM can do 150 fps at full 4K resolution.

**The PLR HDR mode** creates a piecewise linear response curve (like film negative) from a single exposure, extending dynamic range to ~15 stops without the resolution penalties of Dual ISO.

**The Magic Lantern connection.** A1ex, the lead developer of Magic Lantern, worked directly on the AXIOM Beta's color science and sensor characterization. The same people who reverse-engineered Canon's sensor are now building a camera from scratch. The CinemaDNG workflow is directly transferable.

### The RAW Pipeline Connection

For someone coming from Magic Lantern RAW, the transition is natural:

```
ML Workflow:              AXIOM Workflow:
.MLV --> MLV App          raw12 --> raw2dng
     --> CinemaDNG             --> CinemaDNG
     --> DaVinci Resolve       --> DaVinci Resolve
     --> Grade on RAW          --> Grade on RAW
```

Same philosophy: raw sensor data, DNG sequences, full latitude in post. The tools are similar (raw2dng is analogous to mlv_dump). The grading environment is identical.

The difference: the AXIOM was *designed* for RAW from day one. No hack, no bandwidth limitations from a CF card bus, no firmware workarounds. The pipeline is native.

**Key GitHub repositories:**

| Repository | Description | Language |
|---|---|---|
| [axiom-firmware](https://github.com/apertus-open-source-cinema/axiom-firmware) | Complete firmware (Linux, VHDL gateware, tools) | VHDL, Python, C |
| [nctrl](https://github.com/apertus-open-source-cinema/nctrl) | Hardware abstraction layer, sensor params as filesystem | Rust |
| [axiom-recorder](https://github.com/apertus-open-source-cinema/axiom-recorder) | GPU pipeline capture (USB3/Ethernet) | Rust, GLSL |
| [dng-rs](https://github.com/apertus-open-source-cinema/dng-rs) | Zero-copy DNG read/write library | Rust |
| [AXIOM-Remote](https://github.com/apertus-open-source-cinema/AXIOM-Remote) | Hardware remote + emulator | C++ |
| [webui](https://github.com/apertus-open-source-cinema/webui) | Web interface for camera control | JavaScript |

82 repositories total under [apertus-open-source-cinema](https://github.com/apertus-open-source-cinema).

### Honest Assessment

The AXIOM Beta is not yet a production cinema camera. As of early 2026:

**What works:**
- 4K RAW capture via USB 3.0 to external device
- Global shutter, no artifacts
- PLR HDR for extended dynamic range
- Full Linux access, SSH control
- CinemaDNG output for standard post-production

**What doesn't (yet):**
- No production-ready enclosure (the Developer Kit is exposed PCBs)
- No internal recording (requires external computer/NUC)
- No integrated codec (no ProRes/BRAW internal like Blackmagic)
- No autofocus
- No built-in audio
- No camera profile in DaVinci Resolve or ACES
- Significant technical learning curve (Linux, terminal, SSH)

The Compact version (with CNC aluminum body, WiFi, WebUI) is in development at 5,990 EUR but doesn't have a confirmed shipping date.

### How I Could Use It

**For installation work and experimental cinema**: the programmable FPGA opens creative possibilities that no other camera offers. Custom debayering, real-time image processing, sensor-level experimentation. The process becomes part of the work.

**As a development platform**: contribute to the firmware, test builds, document workflows. The same testing role I've been doing on Magic Lantern for 9 years, but on purpose-built hardware.

**For specific production scenarios**: the global shutter alone justifies it for any shoot with fast movement, strobing lights, or LED walls. 150 fps at 4K is exceptional for slow motion work.

**Combined with ML workflow knowledge**: the CinemaDNG pipeline is identical. DaVinci Resolve grading is identical. The offline/online methodology transfers directly. All the workflow infrastructure I've built over years of ML RAW work applies without modification.

**The long game**: if the Compact ships with a stable workflow, this becomes a legitimate open source alternative to a Blackmagic Pocket 6K. Worse autofocus (none), worse codec integration, but: global shutter, interchangeable sensor, reprogrammable pipeline, 150fps 4K, and you own the entire stack.

---

## Why This Matters

The entire cinema industry runs on proprietary black boxes. Canon locks out RAW capabilities that the hardware supports. RED patents their compressed RAW codec. ARRI's color science is a trade secret. Even Blackmagic, which uses an open codec (CinemaDNG), ships closed firmware.

Magic Lantern proved that a community of reverse engineers could unlock capabilities worth $20,000+ from a $1,500 camera body. The AXIOM project is building the logical next step: a cinema camera where everything, from the PCB schematics to the FPGA gateware to the Linux firmware, is open.

This is not about saving money (though it does). It's about **control**. Understanding every step of the image pipeline from photon to pixel. Being able to modify any part of the chain. Not depending on a manufacturer's decision about what you're allowed to do with hardware you own.

For auteur cinema, where the tools shape the work and the process is inseparable from the result, this matters.

---

## Forum Activity & References

I've been active on the [Magic Lantern Forum](https://www.magiclantern.fm/forum/) since May 2013 under the username [12georgiadis](https://www.magiclantern.fm/forum/index.php?action=profile;u=24232). 213+ posts across 25 threads over 9 years.

### Key Thread Contributions

**Experimental Builds & Testing:**
- [crop_rec on steroids: 3K, 4K, 1080p48](https://www.magiclantern.fm/forum/index.php?topic=19300.msg206111#msg206111) -- Testing 5.7K modes, live view during recording
- [Danne's crop_rec_4k, 5DIII](https://www.magiclantern.fm/forum/index.php?topic=23041.msg240658#msg240658) -- 5.7K anamorphic, SD overclock exploration
- [Danne's crop_rec_4k for EOS M](https://www.magiclantern.fm/forum/index.php?topic=25781.msg210495#msg210495) -- MV1080 crop mode workflow
- [ML Cinema Camera: Dual ISO without aliasing](https://www.magiclantern.fm/forum/index.php?topic=22818.msg207408#msg207408) -- Dual ISO quality testing
- [12-bit and 10-bit RAW video development](https://www.magiclantern.fm/forum/index.php?topic=5601.msg195759#msg195759) -- Bit depth testing

**Workflow & Post-Production:**
- [MLV App development](https://www.magiclantern.fm/forum/index.php?topic=20025.msg205582#msg205582) -- FCPXML workflow testing, demosaicing comparison
- [Relink JPEG proxy to regular DNG](https://www.magiclantern.fm/forum/index.php?topic=20873.msg195640#msg195640) -- Offline/online workflow
- [Switch (cr2hdr) for macOS](https://www.magiclantern.fm/forum/index.php?topic=15108.msg206893#msg206893) -- Dual ISO processing
- [fastcinemadng](https://www.magiclantern.fm/forum/index.php?topic=19021.msg205684#msg205684) -- CinemaDNG conversion
- [Reducing aliasing in post with enfuse/hugin](https://www.magiclantern.fm/forum/index.php?topic=21089.msg193655#msg193655) -- Anti-aliasing techniques

**Framing & Cinema Aesthetics:**
- [New film ratio cropmarks](https://www.magiclantern.fm/forum/index.php?topic=21439.msg195613#msg195613) -- 1.33:1, 1.66:1, 1.16:1 ratios (Garrel, Dolan references)

**Camera Testing & Comparison:**
- [5.7K anamorphic 5D Mark III footage](https://www.magiclantern.fm/forum/index.php?topic=25180.msg231599#msg231599)
- [GH5 vs 5D Mark III](https://www.magiclantern.fm/forum/index.php?topic=22895.msg206982#msg206982)
- [ArcziPL's crop_rec_4k for 70D](https://www.magiclantern.fm/forum/index.php?topic=25786.msg224059#msg224059) -- DPAF evaluation
- [dfort's experiments for 7D](https://www.magiclantern.fm/forum/index.php?topic=25880.msg195500#msg195500)

**Other:**
- [DNG silent picture for film scanning](https://www.magiclantern.fm/forum/index.php?topic=9973.msg96881#msg96881) -- Full sensor resolution capture for analog film digitization
- [raw2dng.app macOS](https://www.magiclantern.fm/forum/index.php?topic=5508.msg38713#msg38713) -- Early CinemaDNG workflow (2013)

---

## License

This documentation is released under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

The tools referenced are open source under their respective licenses:
- [Magic Lantern](https://www.magiclantern.fm/) -- GPL v2
- [MLV App](https://github.com/ilia3101/MLV-App) -- GPL v3
- [AXIOM / apertus](https://github.com/apertus-open-source-cinema) -- GPL v3 (software), CERN OHL (hardware)
