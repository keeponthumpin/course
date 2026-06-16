# DMX & TouchDesigner — Full Masterclass Notes

**Audience**: TD users with some experience, no lighting background  
**Format**: ~3 hours, theory blocks + hands-on exercises  
**Structure**: 7 blocks + 3 exercises

-----

## Before you start

**Room setup checklist**

- Everyone has TD open (2023.x minimum)
- At least one Art-Net visualizer running (Depence, QLC+, or TD’s built-in fixture visualizer)
- Ideally: one physical fixture on the network (PAR, LED bar, or pixel tube) — not required but makes the point land harder
- Slides on screen, TD window ready to share

**Opening frame (say this)**

> “Today is not a TD tutorial. You already know TD. Today is about understanding the protocol underneath the tool — because the moment something breaks on a real show, the node UI disappears and you’re staring at raw data. The people who can debug that are the ones who understood what was actually going on.”

-----

## Block 1 — Where lighting control comes from (20 min)

### Goal

Give students a mental model of *why* DMX exists. Not history for history’s sake — this explains why the protocol is shaped the way it is, which explains why TD works the way it does.

### Talking points

**Start with the flashlight exercise**

- Ask everyone to get their phone out, flashlight ready
- Split the room into rows
- You signal left hand up → left side turns on, right hand up → right side turns on, both down → all off
- Run it a few times, build a chase pattern
- Land the point: “I just programmed a chase effect. Using unpaid labor. Like Apple.”
- Then: “Everything we do today is this. Turning lights on and off with different colors. That’s it. The software is just a way to do this faster and with more precision.”

**Ancient Greece → theater automation**

- Deus ex machina: rope + lever + actor lowered from above = first recorded lighting/staging automation
- The mechanism matters: someone had to decide *when* to pull the rope. That’s a cue. Cues still exist in every lighting console today.
- Medieval: torches → chandeliers → gas lamps → electricity
- Key moment: someone put multiple bulbs at different positions on stage and realized you could *accent* different areas. Birth of stage lighting design.

**The problem of the 1970s–80s**

- Each fixture manufacturer had their own proprietary control protocol
- A console from one country couldn’t talk to fixtures from another
- If a dimmer block broke, you had to fly in a specialist from the manufacturer’s country
- Explain dimmer block: it’s the interface between the control signal and the mains power going to the lamp
  - Analogy: your brain (TD) sends a signal, your hand (dimmer block) controls how much power flows, the phone flashlight (fixture) is what actually lights up
  - A dimmer can also do gradual intensity — 0% to 100% — not just on/off
- Result: the industry had too many incompatible standards and everyone was suffering

**DMX512 — the solution**

- A group of engineers and manufacturers standardized in 1986: USITT DMX512
- Digital Multiplex, 512 channels
- One-way serial protocol: the console broadcasts, fixtures listen
- Every channel is a number from 0 to 255 (8-bit)
- Fixtures have an address — they listen to their slice of the 512 channels
- This is still the backbone of professional lighting today, 40 years later

**Key insight to land**

> “DMX is just a list of 512 numbers, refreshed up to 44 times per second. That’s it. Every lighting effect you’ve ever seen at a concert, in a theater, at Eurovision — is ultimately a list of numbers changing over time. Understanding that changes how you think about everything in TD.”

-----

## Block 2 — DMX protocol deep dive (25 min)

### Goal

Students should be able to mentally model what a DMX signal looks like, understand universes and addressing, and be able to calculate channel requirements for any fixture.

### Talking points

**The DMX frame**

- 512 channels = one universe
- Each channel: integer 0–255
- Refresh rate: up to 44Hz (some controllers go higher, but 44Hz is the standard)
- Unidirectional: the console talks, fixtures listen — no acknowledgment, no error correction
- Physical: originally RS-485 balanced signal, 3-pin or 5-pin XLR cable
- Today: mostly transmitted over IP via Art-Net or sACN (more on this shortly)

**Fixture addressing**

- Every fixture has a start address: the first DMX channel it listens to
- It then consumes as many channels as it needs (its “footprint”)
- Example: a simple RGB PAR has a 3-channel footprint
  - Address 1 → listens to channels 1, 2, 3 (R, G, B)
  - Address 4 → listens to channels 4, 5, 6
- If you set two fixtures to the same address, they both respond to the same data — useful for duplicating effects, dangerous if unintentional

**Channel math — work through this together**

- Simple RGB PAR: 3 channels (R, G, B)
- RGBW fixture: 4 channels (R, G, B, W)
- RGBWUV fixture: 6 channels (R, G, B, W, Amber, UV)
- Pixel tube with 8 RGB pixels: 8 × 3 = **24 channels**
- How many pixel tubes fit in one universe?
  - 512 ÷ 24 = 21.3 → **21 tubes** (you can’t split a fixture across a universe boundary)
  - This is a real constraint you hit constantly on large installs

**The universe boundary caveat**

- A fixture’s channels must all live within one universe
- If your last tube would start at channel 505, it only has 8 channels left (505–512) but needs 24 → you can’t fit it, it goes in the next universe
- In large installs (think 200+ fixtures) you’re managing dozens of universes
- TD handles multiple universes natively — one Art-Net Out CHOP per universe, or one node with universe offset parameter

**Art-Net and sACN**

- DMX over IP — same 512-channel frame, packaged in a UDP packet
- Art-Net: most common, slightly older, simpler to configure, works over standard ethernet/wifi
- sACN (E1.31): newer, multicast-native, slightly more efficient for large systems
- In TD: Art-Net Out CHOP, sACN Out CHOP — both work the same way from a patching perspective
- For this class we’ll use Art-Net

**Practical gotcha: network configuration**

- Art-Net uses the 2.x.x.x or 10.x.x.x IP range by default
- Your machine and your fixtures/visualizer need to be on the same subnet
- In real shows: dedicated lighting network, separate from internet/production network
- IGMP snooping on managed switches matters for multicast — if you see missing universes on a show, check this first

-----

## EXERCISE 1 — Address calculation (10 min)

**Scenario**: You have a rig with the following:

- 12 × RGBW PARs
- 6 × pixel tubes (12 pixels each, RGB)
- 4 × moving heads (16-channel mode)

**Tasks**:

1. Calculate total channel count
1. Assign addresses sequentially, identify where universe 1 ends and universe 2 begins
1. Identify which fixture type crosses the universe boundary (if any) and how you’d handle it

**Answer**:

- 12 × RGBW = 12 × 4 = 48 channels
- 6 × pixel tubes = 6 × 36 = 216 channels
- 4 × moving heads = 4 × 16 = 64 channels
- Total = 328 channels → fits in one universe (just)
- If you add 4 more pixel tubes: 328 + 144 = 472 → still fits, but barely
- 3 more pixel tubes would overflow (472 + 108 = 580 → universe 2 needed)

**Discussion point**: in a real show you often leave gaps between fixture types for patching flexibility. A lighting programmer will typically address fixtures in groups with some headroom between them.

-----

## Block 3 — Color theory for lighting (20 min)

### Goal

Students understand RGB vs CMY, why LEDs use additive mixing, what RGBW adds, and how white conversion works in TD (since they’ll need it for pixel pipelines).

### Talking points

**Additive vs subtractive**

- CMY (Cyan, Magenta, Yellow): subtractive — used in print, paint, physical pigments. Start with white, subtract colors by absorbing wavelengths.
- RGB: additive — used in screens and LEDs. Start with darkness, add light.
- Why can’t LEDs use CMY? A diode emits light — it can’t absorb it. You can only add, not subtract. So RGB is the only model that makes physical sense for a light source.
- Interesting edge case: you *can* simulate subtraction after addition (mix to white, then filter with a gel or a physical CMY color wheel in a moving head). Moving heads often have CMY wheels precisely for this reason.

**The history of RGB LEDs**

- Green LED: invented first (relatively easy — gallium phosphide)
- Red LED: soon after
- Blue LED: not until 1994 — Shuji Nakamura, won the Nobel Prize in Physics for it in 2014
- Without blue, you can’t make white. Before blue LEDs, “white” in stage lighting meant incandescent or halogen bulbs.
- Multi-chip LED (single die, multiple colors): became possible after blue was invented. Before that, fixtures used separate R, G, B diodes — which means you needed different quantities of each (different wavelengths, different perceived brightness) to balance to white.

**RGBW — why add a white channel?**

- RGB mixing to white works but produces a slightly “cold” or “dirty” white because the three additive primaries don’t perfectly reconstruct a broad-spectrum white source
- Adding a dedicated white phosphor diode gives you a much cleaner, warmer white
- Also more efficient: one white diode at full is brighter and uses less power than R+G+B all at full
- Fixture modes: some RGBW fixtures let you control W separately (4-channel mode) or mix it in automatically based on RGB values (3-channel mode with onboard processor)

**White conversion in TD**

- In pixel pipelines you often have RGB texture data but RGBW fixtures
- You need to extract a white component and redistribute the RGB
- Simple formula: `W = min(R, G, B)` then `R' = R - W`, `G' = G - W`, `B' = B - W`
- This is a one-liner in GLSL — do it in a pixel shader, not in Python
- TD has a built-in Color Space TOP but for DMX pixel pipelines you usually want a custom GLSL TOP for precision and speed

**Fixture types overview**

|Type                     |Typical channels              |Common use                         |
|-------------------------|------------------------------|-----------------------------------|
|PAR (simple)             |3–4 (RGB or RGBW)             |Wash, floor lighting, blinders     |
|PAR (full)               |6–8 (RGBWUV + dimmer + strobe)|Feature lighting, effects          |
|Pixel tube               |3 × pixel count               |Linear pixel effects, installations|
|LED bar                  |3 × pixel count               |Pixel mapping, strip lighting      |
|Moving head (wash)       |16–25                         |Pan, tilt, color, gobo, zoom       |
|Moving head (spot)       |20–30+                        |Gobo, prism, iris, animation wheel |
|Laser (via ILDA/Pangolin)|Not DMX — separate protocol   |Aerial effects                     |

**Moving heads — brief overview**

- More channels = more parameters: pan (coarse + fine = 16-bit), tilt, color wheel, gobo wheel, prism, zoom, iris, strobe, dimmer
- 16-bit channels: two DMX channels for one parameter (MSB + LSB) for smooth motion — this is exactly what you implement in TD for winch/motor control too
- In TD you’ll typically control moving heads via a CHOP network that maps your data to the fixture’s channel map (defined in its manual)

-----

## Block 4 — TouchDesigner architecture for DMX (30 min)

### Goal

Students understand where DMX output fits in the TD data flow, how the operator families relate to the DMX pipeline, and specifically what the GPU/CPU boundary means for performance.

### Talking points

**Quick recap: TD operator families**

- Students know this, but anchor it to DMX context:
  - **CHOP**: channel data — this is where DMX values live. A CHOP with 512 channels, values 0–1, is a DMX universe.
  - **TOP**: texture data — pixel color. When you have 500 RGB fixtures, you represent them as a texture (500px wide, 1px tall, RGB). Much faster than 1500 individual CHOP channels.
  - **DAT**: tables and scripts — useful for fixture patch data, address maps, configuration
  - **SOP/COMP**: less relevant for DMX output directly, but SOPs are useful for 3D fixture positioning

**The TD DMX pipeline — two approaches**

*CHOP-based (small rigs, simple effects)*

```
Generator CHOP → Math/Logic CHOPs → DMX Out CHOP
```

- Good for: up to ~100 channels, simple on/off/intensity control, easy to understand
- Limitation: each channel is a separate CHOP channel — managing 512 individual values gets unwieldy fast

*TOP-based (pixel rigs, large installs)*

```
Generator TOP → GLSL/Processing TOPs → DMX Out CHOP (via pixel-to-channel conversion)
```

- Each pixel = one fixture (or one pixel of a fixture)
- A 100px × 1px texture = 100 RGB values = 300 DMX channels
- Processing happens on GPU — massively faster
- The conversion step (TOP → CHOP) is where the GPU/CPU boundary matters

**GPU vs CPU — the real explanation**

*CPU analogy*: imagine a building with 16 specialist workshops. Each workshop can make anything — a custom PCB, an ornate wardrobe, a circuit board. They’re incredibly capable, but expensive, slow, and they can only work on one job at a time per workshop. TD (unlike most modern software) runs primarily on a single CPU thread — so effectively you have one workshop available, not 16.

*GPU analogy*: imagine a warehouse with 10,000 simple robots, each capable of exactly one operation: “take this number, multiply it by that number, put the result here.” They can’t do anything complex, but they can all do it simultaneously. That’s what makes textures fast — every pixel’s color calculation happens in parallel.

*The transfer problem*: moving data between CPU and GPU has a cost. Every time you read a pixel value in Python (TOP.numpyArray()), you’re forcing a transfer from GPU memory to CPU memory — that’s expensive. The DMX POP was significant precisely because it keeps the data on the GPU all the way to the output, avoiding this transfer.

**The DMX POP (Pixel to DMX)**

- Introduced in TD 2022 builds
- Takes a TOP input (your pixel data) and outputs DMX frames directly
- No Python, no CHOP conversion, no GPU→CPU transfer
- Handles universe splitting automatically based on fixture count and channel width
- This is what you should use for any pixel-based fixture rig

**Python’s role in DMX pipelines**

- Not for per-frame DMX value generation (too slow)
- Good for: configuration, address calculation, patch table generation, fixture management systems
- Use DATs + Python extensions for your fixture database, then feed values via CHOPs/TOPs

**The software stack (simplified)**

```
Your TD patch (Python/CHOP/TOP)
         ↓
TouchDesigner C++ core
         ↓
Art-Net UDP packets
         ↓
Network → Fixtures
```

Each layer adds latency. CHOP→DMX is fast. Python→DMX is slower. If you’re computing pixel values in Python every frame, you will see frame drops on any rig above ~100 channels.

**Optimization rules of thumb**

1. If it touches every pixel every frame → GLSL shader
1. If it runs once at startup or on config change → Python is fine
1. If it’s per-fixture logic (not per-pixel) → CHOP is fine
1. Never use `numpy` in a perform-mode callback unless you profile it first

-----

## EXERCISE 2 — Basic DMX output in TD (25 min)

### Goal

Students get something lighting up. Understand the full signal chain from generator to fixture.

### Steps

**Part A: CHOP-based output (understand the mechanism)**

1. Create a **Constant CHOP**
- Set number of channels to 3
- Name channels `r`, `g`, `b`
- Values: 0.5, 0.2, 0.8
1. Add a **Math CHOP** after it
- Range: 0–1 → 0–255 (multiply by 255)
- This is your raw DMX value
1. Add an **Art-Net Out CHOP**
- Network address: your visualizer’s IP (or 255.255.255.255 for broadcast)
- Universe: 0
- Check that channels r, g, b map to DMX channels 1, 2, 3
1. Open your visualizer — you should see your PAR with the color you set
1. **Now experiment**:
- Change the values — does the color update?
- Add a **LFO CHOP** and merge it with the Constant — you have animation
- Add a **Pattern CHOP** (ramp) across multiple channels — you have a chase

**Part B: Multi-fixture with a texture (scale it)**

1. Create a **Constant TOP** — set resolution to 8×1 (8 fixtures)
1. Set a color in the TOP parameters
1. Add a **GLSL TOP** with this pixel shader:
   
   ```glsl
   uniform float uTime;
   out vec4 fragColor;
   void main() {
     float x = vUV.s;
     float brightness = step(mod(x * 8.0 + uTime * 2.0, 1.0), 0.5);
     fragColor = vec4(brightness, 0.2, 0.8, 1.0);
   }
   ```
1. Wire `absTime.seconds` to `uTime` via a CHOP→parameter link
1. Add a **DMX Out CHOP** with “Convert from TOP” mode
1. You now have an animated chase on 8 fixtures, running entirely on the GPU

**Discussion after exercise**

- How would you change the speed? The color? The direction?
- What’s the maximum number of fixtures before you’d feel a performance hit in approach A vs approach B?
- Where would you add a master dimmer control?

-----

## Block 5 — Building a real patch (30 min)

### Goal

Students understand how to structure a production-quality DMX patch — not just “make it work” but “make it maintainable.”

### Talking points

**The patch as a system**

Before touching TD, ask: what does this rig need to do?

- How many fixtures, what types?
- What effects are needed?
- What needs to be controllable in real time vs pre-programmed?
- Does it need to respond to external input (audio, OSC, MIDI, timecode)?

Think in layers:

1. **Data generation layer**: where values come from (noise, LFO, audio analysis, pre-programmed cues)
1. **Mapping layer**: translating your creative data to fixture channels (RGB conversion, RGBW conversion, address mapping)
1. **Output layer**: Art-Net Out, universe management, network routing

**Fixture patch table (DAT-based)**

Build a patch table in a Table DAT:

```
fixture_id | name      | address | universe | channels | type
1          | PAR_L1    | 1       | 0        | 4        | RGBW
2          | PAR_L2    | 5       | 0        | 4        | RGBW
3          | TUBE_01   | 9       | 0        | 24       | RGB_PIXEL_8
...
```

- Generate Art-Net routing from this table using a Script DAT
- When you repatch a fixture (change address), you change one cell, not rewire half your network
- Python extension can expose methods like `getFixtureChannels('PAR_L1')` for other parts of the patch to use

**Master dimmer**

- Always have one: a single float value (0–1) that multiplies your entire output
- Implement as a Math CHOP at the end of your chain, or as a uniform in your GLSL shader
- Essential for: fade-to-black, emergency blackout, smooth transitions between scenes

**Blackout vs fade**

- Hard blackout: set all channels to 0 instantly — useful for cue transitions, safety
- Fade to black: lerp all values to 0 over N seconds — use a **Filter CHOP** on your master dimmer
- In TD: a **Lag CHOP** or **Filter CHOP** on the master value gives you smooth fades without touching your content layer

**Cue system basics**

- A cue = a snapshot of channel states + a transition time
- Simple implementation in TD:
  - Store cues as rows in a Table DAT
  - On cue trigger, lerp current values to target values over the cue’s fade time
  - Use a **Timer CHOP** to drive the lerp progress
- More advanced: use TD’s built-in Animation COMP as a timeline
- Production systems (GrandMA, ETC Eos) do this with dedicated hardware/software — TD is unusual in that it can both generate content *and* run cues

**Timecode sync**

- On shows with a fixed timeline (concerts, broadcast, exhibitions), lighting cues run on SMPTE timecode
- TD receives timecode via **LTC In CHOP** (audio timecode) or **Art-Net Timecode** (network)
- Build your cue triggers off timecode position, not wall clock — your content stays locked to the show even if TD pauses or restarts

**Feedback and monitoring**

- Art-Net is one-way — you can’t ask a fixture “what’s your current value”
- For monitoring: RDM (Remote Device Management) is a bidirectional extension of DMX, but not widely used in creative installs
- In TD: build a monitoring panel that shows your expected output values — what you *think* the fixtures are receiving
- On large rigs: log fixture states to a DAT every N seconds for debugging

-----

## Block 6 — Audio-reactive and generative (25 min)

### Goal

Students can distinguish between audio-reactive and generative approaches, know how to get audio into TD, and understand how to map audio data to DMX output.

### Talking points

**The terminology distinction (important)**

This is a common confusion in the creative coding world:

- **Audio-reactive**: the visuals/lights *respond* to audio input. Kick drum → flash. Volume → brightness. Frequency bands → colors. The audio is the driver, the lights are the output. The system has no internal state — take away the audio and everything freezes.
- **Generative**: the system has its own internal logic and evolves itself over time. It might be *influenced* by audio, but it doesn’t depend on it. A generative system has state, rules, feedback loops. Take away the audio and it keeps running.

> “Most things people call generative are actually reactive. If it’s just noise plugged into noise, that’s a patch, not a system.”

Both are valid — just be clear about which one you’re building.

**Getting audio into TD**

Three sources:

1. **Audio Device In CHOP**: real-time microphone or line input. Set sample rate, buffer size. Use for live performance.
1. **Audio File In CHOP**: plays back a pre-recorded file. Use for fixed-timeline shows or testing.
1. **DAW integration**: Ableton Live or Bitwig via Link protocol or Ableton Link CHOP. Gives you beat-locked sync, individual track levels, clip triggers.

**Audio Analysis CHOP**

- Takes audio and outputs frequency band levels
- Parameters: number of bands, smoothing, attack/release
- Output channels: one per frequency band (0–1 values)
- Use these directly to drive DMX channels, or map them to shader uniforms

**Mapping audio to lights — practical approaches**

*Simple brightness control*:

- Take low-frequency band (kick/bass) → master dimmer
- Effect: lights pulse with the beat
- Add a **Lag CHOP** for attack/release shaping (fast attack, slow release = strobe feel; slow attack, fast release = reversed feel)

*Color mapping*:

- Low freq → red channel
- Mid freq → green channel
- High freq → blue channel
- Result: color shifts with the spectral content of the music

*Chase from beat detection*:

- **Beat CHOP**: detects beat phase from audio
- Output: 0–1 ramp per beat
- Use ramp to index into a fixture sequence for beat-locked chases

**Ableton Link / DAW sync**

- Ableton Link CHOP gives you: BPM, beat phase, bar phase
- More reliable than audio beat detection for live performance (no false triggers)
- Use beat phase to drive LFOs that are locked to musical tempo
- Enables: pre-programmed effects that stay in time even as the DJ changes tempo

**True generative approaches**

For a generative system that *also* responds to audio, think of it as two layers:

1. Autonomous system: noise fields, cellular automata, reaction-diffusion, particle systems — these run independently and produce interesting motion on their own
1. Audio influence: audio parameters *modulate* properties of the autonomous system (speed, scale, chaos amount) rather than directly controlling output values

Example: a reaction-diffusion system where the feed rate is influenced by kick drum energy. The system is always running and interesting; the audio makes it react visibly without turning it into a simple flash.

-----

## EXERCISE 3 — Audio-reactive chase (20 min)

### Goal

Connect a live audio source to a fixture chase and tune the mapping.

### Steps

1. Create an **Audio Device In CHOP** — set to your mic or system audio
1. Add an **Audio Spectrum CHOP** — 8 bands, set smoothing to 0.3
1. Observe the 8 output channels — they should respond to audio
1. Create a **Select CHOP** to grab just the low-frequency band (channel 0)
1. Add a **Lag CHOP** — attack 0.01, release 0.3 (fast on, slow off)
1. This is your “kick detector”
1. Create a **Pattern CHOP** — Ramp pattern, 8 samples (for 8 fixtures)
1. Add a **Math CHOP** to offset the ramp position using a counter driven by your beat signal
1. Multiply the ramp output by your kick detector value — brightness pulses with the beat, position advances on each kick
1. Wire to Art-Net Out — watch your 8 fixtures chase in time with the audio

**Tuning discussion**

- What does changing the Lag release time do to the feel?
- What happens if you use a high-frequency band instead of a low-frequency band?
- How would you add color (not just brightness) to the chase?

-----

## Block 7 — Production workflow & real-world considerations (15 min)

### Goal

Bridge the gap between “working in studio” and “working on a show.” Students leave knowing what they don’t know and what to prepare for.

### Talking points

**Network architecture on shows**

- Dedicated lighting network: never share with internet, video, or production network
- Managed switches only: unmanaged switches drop Art-Net packets under load
- Static IPs for everything: DHCP causes address changes on reboot — catastrophic during a show
- IGMP snooping: if you’re using sACN (multicast) and fixtures are disappearing, check this on your switch config
- Fiber for long runs: copper ethernet max 100m; fiber goes further and eliminates ground loops

**Redundancy**

- On important shows: two TD machines, one hot standby
- Art-Net merge: some fixtures/nodes accept input from two sources and merge them — useful for redundancy
- Always have a backup scene that can be triggered instantly (a simple, stable state)
- Network loop protection: spanning tree protocol — make sure it’s enabled on your switches

**Fixture library and documentation**

- Every fixture has a DMX personality chart in its manual — read it before patching
- Different modes have different channel counts — the fixture must be set to the same mode you’ve patched in TD
- Keep a patch sheet: fixture ID, address, universe, physical location, mode. Update it when anything changes.

**On working with lighting designers**

- LDs think in cues, scenes, and groups — learn this language
- If they’re running a console (MA3, Eos) and TD is a node in their system, you’ll receive OSC or Art-Net triggers from them
- If TD is the primary control, export your patch data in a format they can import (CSV, MVR, or show files)
- Be explicit about what TD controls vs what the console controls — shared universes cause conflicts

**Performance and stability**

- Target: 44fps or locked to show timecode, zero frame drops
- Profile before the show: use TD’s Performance Monitor to find expensive operators
- Common culprits: Python in execute DATs, large numpy operations, unnecessary TOP resolution
- On show day: close everything else on the machine, disable Windows Update, plug in power

**When things go wrong (and they will)**

- Art-Net not reaching fixtures: check IP subnet, check switch config, try broadcast (255.255.255.255)
- Fixtures showing wrong colors: check channel mode on fixture matches your patch
- Universe offset wrong: Art-Net universe numbering can be 0-indexed or 1-indexed depending on the node — check both
- Flickering output: network packet loss — check cable, switch, or reduce refresh rate
- TD frame drops: reduce TOP resolution, move calculations to GPU, check CPU usage

-----

## Closing (5 min)

**What we covered**

- Why DMX exists and what it actually is (a list of numbers)
- How to calculate channel requirements for any rig
- Color theory: RGB, RGBW, white conversion
- TD architecture: CHOP vs TOP approach, GPU vs CPU, the DMX POP
- Building a maintainable patch: patch tables, master dimmer, cue system basics
- Audio-reactive vs generative: terminology and practical implementation
- Real-world: network, redundancy, working with LDs

**What to explore next**

- **RDM**: bidirectional DMX for fixture discovery and configuration
- **MVR / GDTF**: open fixture format for sharing rig data between TD and consoles
- **MA3 integration**: OSC/Art-Net bridge between a lighting console and TD
- **MIDI/OSC control surfaces**: live control of TD parameters from a physical desk
- **Timecode and show control**: SMPTE, MTC, OSC timecode for locked show playback
- **Pixel mapping in 3D**: using SOPs to define fixture positions, then projecting texture onto them

**The thing to remember**

> “The tool is just the tool. What makes someone good at this is understanding what’s happening underneath — the protocol, the data, the signal chain. Once you have that, you can debug anything, adapt to any rig, and build systems that actually work under show conditions.”

-----

## Appendix A — Useful TD nodes for DMX work

|Node          |Family|Use                                      |
|--------------|------|-----------------------------------------|
|Art-Net Out   |CHOP  |Send DMX over Art-Net                    |
|sACN Out      |CHOP  |Send DMX over sACN (E1.31)               |
|DMX Out       |CHOP  |Send DMX over USB (ENTTEC etc.)          |
|Art-Net In    |CHOP  |Receive Art-Net (monitoring, passthrough)|
|Constant      |CHOP  |Static channel values                    |
|Pattern       |CHOP  |Ramps, pulses, waves across channels     |
|LFO           |CHOP  |Oscillating values                       |
|Beat          |CHOP  |Beat detection from audio                |
|Audio Spectrum|CHOP  |Frequency band analysis                  |
|Ableton Link  |CHOP  |Beat-sync with Ableton/Bitwig            |
|Math          |CHOP  |Scale, offset, range conversion          |
|Lag           |CHOP  |Attack/release smoothing                 |
|Filter        |CHOP  |Smoothing / slew rate limiting           |
|Timer         |CHOP  |Cue fade timing                          |
|GLSL          |TOP   |Custom pixel shaders for fixture data    |
|Constant      |TOP   |Static color textures                    |
|Noise         |TOP   |Animated noise textures                  |
|Level         |TOP   |Brightness, contrast, color correction   |
|Lookup        |TOP   |Color palette mapping                    |

-----

## Appendix B — DMX channel value reference

|Value (0–255)|Percentage|Common meaning|
|-------------|----------|--------------|
|0            |0%        |Off / minimum |
|64           |25%       |Quarter       |
|128          |50%       |Half          |
|192          |75%       |Three-quarter |
|255          |100%      |Full / maximum|

In TD, CHOP values are normalized 0–1. Multiply by 255 before DMX Out, or use the CHOP’s built-in range conversion.

-----

## Appendix C — Quick GLSL reference for pixel pipelines

**Basic RGB fixture pixel shader**

```glsl
uniform sampler2D sTD2DInputs[1];
uniform vec4 uColor;
out vec4 fragColor;

void main() {
    vec2 uv = vUV.st;
    fragColor = texture(sTD2DInputs[0], uv);
}
```

**RGB → RGBW conversion**

```glsl
vec3 rgb = texture(sTD2DInputs[0], vUV.st).rgb;
float w = min(rgb.r, min(rgb.g, rgb.b));
vec3 rgbOut = rgb - vec3(w);
// Output: rgbOut.r, rgbOut.g, rgbOut.b, w
fragColor = vec4(rgbOut, w);
```

**Chase pattern**

```glsl
uniform float uTime;
uniform float uSpeed;
uniform int uFixtureCount;
out vec4 fragColor;

void main() {
    float index = floor(vUV.s * float(uFixtureCount));
    float phase = mod(index / float(uFixtureCount) - uTime * uSpeed, 1.0);
    float brightness = step(0.5, phase);
    fragColor = vec4(brightness, brightness, brightness, 1.0);
}
```

**16-bit DMX output (MSB/LSB split)**

```glsl
// For 16-bit parameters (pan, tilt, etc.)
// value: 0.0–1.0 input
// Outputs two channels: MSB (coarse) and LSB (fine)
float value16 = value * 65535.0;
float msb = floor(value16 / 256.0) / 255.0;
float lsb = mod(value16, 256.0) / 255.0;
```