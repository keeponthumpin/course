# TouchDesigner Course — Master Spec

> **Purpose of this doc:** Portable source of truth for the whole course. Paste or attach this at the top of each per-module chat so the work stays consistent without depending on cross-chat memory. This chat (the master thread) is where thinking happens; this doc is the memory.

-----

## The thesis

This is not a tour of operator families or a node manual. The course teaches one thing in many costumes:

> **Understand the data, and where computation belongs. Everything else is application.**

Distilled to the sentence a student carries everywhere:

> **Don’t overcook. Make the PC’s life easier.**

The reader should leave able to debug anything, adapt to any setup, and build systems that survive real conditions — because they understand what’s happening underneath the nodes, not just which node to drag.

-----

## The through-line (the spine)

Every module quietly answers the same question:

> **Where should this computation live, and what is it costing you?**

Mechanically, this is a question about **cooking** — what recooks, when, and how expensively. “Cost” *is* cooking. This is the CPU/GPU divide, the single-threaded reality of TD, the transfer tax at boundaries, and “do the right thing for performance and maintainability.” It is **never a chapter** — it lives inside every module.

### The course shape

- **Module 0 plants the principle, purely abstractly.** Understand cooking well enough to feel why overcooking is the enemy — then stop. No node names, no techniques, no “how.”
- **Modules 1–8 are each a worked demonstration of the principle in one data type.** Each answers: *“okay, but how do I not overcook **here**?”* — TOPs (keep work on GPU, TOP→CHOP done right), SOPs (instancing, the geometry wall), DATs/Python (per-frame Python is overcooking incarnate), POPs (points on GPU, the readback tax), and so on.

So: **Module 0 = the principle. Modules 1–8 = the principle applied, one family at a time.**

### The structural rule that makes modules independent AND coherent

Modules are self-contained — someone can drop into any one without having read the others. Coherence comes from **rhyme, not cross-reference.**

- A module **never** says “remember the CHOP→TOP decision from Module 1.”
- It **restates the principle in its own local terms** and re-grounds it fresh: *“don’t overcook — here’s how in this family.”*
- The student who only reads the SOP module gets “don’t overcook” fresh; the student who reads all eight hears the same drumbeat in eight costumes. That repetition **is** the coherence.

Result: each module stands alone; the whole reads as one argument for anyone going front-to-back. The arc is there for whoever wants it, but it is **not load-bearing** for drop-in readers.

-----

## Pedagogical DNA (the “vibe” — derived from the DMX sample)

Keep every module true to these:

1. **Protocol/fundamentals before tool.** Teach the thing underneath, then the node.
1. **Optimization as a worldview, not a chapter.** GPU/CPU, where work belongs, “never numpy in perform mode.”
1. **Physical/analog anchors.** Flashlight chase, workshop-vs-warehouse, deus ex machina. Ground abstractions in tangible mechanisms.
1. **“It’s just X.”** Reduce to primitives to demystify (DMX is just 512 numbers at 44Hz).
1. **Production reality as the payoff.** End on what breaks on a real job and what theory doesn’t tell you.
1. **Conceptual precision.** Care about getting distinctions right (e.g. reactive vs generative), not just “make it work.”

-----

## Per-module skeleton

Every module carries the same internal shape. The through-line lives in **slot 2** — always, never as an explicit cross-reference.

1. **What this data type *is*** — the “it’s just X” reduction.
1. **Where the work belongs and what the boundary costs** — the through-line, restated locally.
1. **The right way vs the way that bites you on a show** — correct technique vs the trap.
1. **Hands-on exercise** — makes the point land physically.

-----

## Course spine

|#|Module                        |One-line focus                                                                                                                                          |
|-|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|0|The mental model              |What TD *is*: dataflow + **cooking**. CPU/GPU divide, single-threaded reality, families as data types. Pure abstraction — plants “don’t overcook.”      |
|1|CHOPs: thinking in channels   |Signal flow, the cook chain, time vs sample; when channel count forces you out of CHOP-land.                                                            |
|2|TOPs and the GPU              |Textures as parallel processing, GLSL (“describe one pixel, run it 10,000×”), transfer cost. Heart of the optimization philosophy.                      |
|3|SOPs / geometry / 3D          |Geometry as data, instancing, the render pipeline. Tight and purposeful — resist bloat.                                                                 |
|4|POPs: geometry on the GPU     |The SOP wall; SOP→POP decision mirrors CHOP→TOP; the GPU→CPU readback tax for point data.                                                               |
|5|DATs, Python, and architecture|Where Python *belongs* (config, structure, orchestration) vs where it kills you (per-frame). Building systems, not patches.                             |
|6|Inputs & the outside world    |Audio, MIDI, OSC, DMX/Art-Net, sensors, NDI. **DMX masterclass lives here as one deep-dive application.** Reactive-vs-generative as a general principle.|
|7|Generative systems            |Noise, feedback, particles, simulation. Systems-with-state vs patches.                                                                                  |
|8|Production & deployment       |Profiling, perform mode, networking, redundancy, show conditions. The “what breaks and why” payoff, generalized.                                        |

### Module 0 — scoped (cooking is the spine)

**Cooking is the most misunderstood core concept in TD, so it IS Module 0** — not one slot inside it. Module 0 is **pure abstraction: zero implementation, no node names, no techniques.** Understand cooking, feel why overcooking is the enemy, stop.

The misconceptions to attack head-on:

- TD does **not** run top-to-bottom like a script every frame. It’s **pull-based, dirty-flag driven** — a node cooks when something downstream needs it *and* its inputs changed.
- An operator that exists but isn’t cooking is **nearly free**. The real tax is **unnecessary cooking**, not node count. (People delete nodes to “optimize” when that’s not the cost.)
- Cooking is **invisible** until it’s a problem — students feel the frame drop but can’t locate it.
- CHOPs cook on CPU per-sample every frame they’re pulled — cheap small, brutal at scale. (The *why* under the whole course’s “leave CHOP-land” rule — but Module 0 only plants the mechanism, doesn’t name the fix.)

Internal shape (against the skeleton):

1. **What TD is:** a dataflow graph that *cooks*. Not a timeline, not a top-down script. “It’s just X” → *operators that recook when their inputs go dirty.*
1. **Cooking properly (the core; through-line lives here):** pull-based + dirty flags; what triggers a cook; **existing ≠ cooking; the cost is the cook.** Plants “don’t overcook — make the PC’s life easier.”
1. **Where cooking happens:** families as *data types* (CHOP/TOP/SOP/DAT = shape of your data, not “what you want to do”) + CPU/GPU. Workshop-vs-warehouse lands here as “where the cook physically runs.”
1. **Exercise (the ONE hands-on, or cut entirely):** make cooking *visible* — cook-time/probe overlay, trivial chain, change one param, watch what recooks vs what doesn’t; then build a node cooking every frame for no reason and watch the cost. Teaches the reflex *look at what’s cooking*, not a technique. **DECISION NEEDED:** keep this, or make Module 0 literally zero-hands-on (first TD touch = Module 1).

**Verify against current Derivative docs before teaching:** exact names/locations of cook-inspection tools (cook flags, probe/perform CHOP, network cook indicators, cook-time readouts) — UI specifics drift across builds. Note target build.

- **Module 0** is scoped in detail above. Pure abstraction; cooking is its spine; plants “don’t overcook.”
- **Module 3 (SOPs)** is where TD courses usually bloat. Keep it tight: geometry as data, instancing, SOP→render pipeline. Tie back to “where does the work happen.”
- **Module 4 (POPs)** is recent and has moved fast across builds. **Verify all technical claims (operator names, stable features, SOP↔POP conversion paths, perf characteristics) against current Derivative docs before it becomes teaching material.** Note the target TD build when scoping.
- **Module 6** absorbs the existing DMX masterclass as a deep-dive application. The reactive-vs-generative distinction is sharp — it belongs here as a general principle, feeding into Module 7.
- **Modules 4 (POPs) and 7 (generative)** are the two most likely to resist clean structure — pressure-test their scope early.

-----

## Decisions locked

- **Through-line phrasing:** “Don’t overcook. Make the PC’s life easier.” Cost = cooking.
- **Course shape:** Module 0 = the principle (pure abstraction). Modules 1–8 = the principle applied, one family at a time, each answering “how do I not overcook *here*?”
- **Module 0 scope:** pure abstraction, no implementation; cooking is its spine.
- **No “GPU trilogy” packaging.** Modules 2/3/4 are NOT bundled. They rhyme; they are not branded as a unit.
- **Modules stay independent** (drop-in friendly) **but keep the through-line** via rhyme, not cross-reference.
- **One chat per module**; this master thread stays as the reference/working thread.
- **This doc is the portable memory** — carry it into each module chat.

-----

## Open questions (to resolve as we go)

- [ ] **Module 0 hands-on:** keep the “make cooking visible” exercise, or make Module 0 literally zero-hands-on (first TD touch = Module 1)?
- [ ] **Format:** written reference doc (read solo), live teaching notes, or both? Changes how front-loaded theory is vs interleaved with exercises.
- [ ] **DMX’s role:** flagship case study the whole course builds toward, or just one application among several in Module 6?
- [ ] **Target TD build/version** (affects POPs especially, and current operator names throughout).
- [ ] Audience entry point — confirmed “TD users with some experience, no [domain] background” per the DMX sample. Does this hold course-wide?

-----

## How to start a module chat

Open the per-module chat with something like:

> “Continue from the course master thread. Here’s the master spec [attach this doc]. Let’s scope **Module N — [name]**.”

Then we build that module’s internal structure against the skeleton above, keep the through-line in slot 2, and flag anything that needs verifying against current TD docs.

-----

## Changelog

- **v2** — Through-line distilled to “Don’t overcook. Make the PC’s life easier.” Cost = cooking. Locked course shape (M0 = principle, M1–8 = principle applied per family). Scoped Module 0 in detail: pure abstraction, cooking as its spine, misconceptions to attack, internal shape, hands-on decision flagged.
- **v1** — Initial spine (Modules 0–8 with POPs inserted at 4), through-line, pedagogical DNA, per-module skeleton, locked decisions, open questions.