# TouchDesigner Course

A course about **understanding the data and where computation belongs** — not a node manual.

> **Don't overcook. Make the PC's life easier.**

The reader should leave able to debug anything, adapt to any setup, and build systems
that survive real conditions — because they understand what's happening underneath the
nodes, not just which node to drag.

## How this repo works

- **`_meta/master-spec.md`** is the source of truth. Read it first. It defines the thesis,
  the through-line, the pedagogical DNA, and the per-module skeleton.
- **`_meta/reference-specimens/`** holds fully-built artifacts used as *voice and structure*
  references only — not course content. The DMX masterclass lives here; it is the specimen
  the pedagogical DNA was derived from. In the course itself, DMX is just one application
  inside Module 6, nothing more.
- Each module lives in `modules/NN-name/` with:
  - `src/` — Markdown source (the editable truth)
  - `build/` — generated HTML deliverables (do not hand-edit)
  - `assets/` — diagrams, images, `.toe` files

## Course spine

| # | Module | Focus |
|---|--------|-------|
| 0 | The mental model | What TD *is*: dataflow + cooking. Pure abstraction. Plants "don't overcook." |
| 1 | CHOPs | Thinking in channels; the cook chain; when channel count forces you out. |
| 2 | TOPs & the GPU | Textures as parallel processing; GLSL; transfer cost. |
| 3 | SOPs / geometry | Geometry as data; instancing; the render pipeline. Stay tight. |
| 4 | POPs | Points on the GPU; SOP→POP mirrors CHOP→TOP; readback tax. |
| 5 | DATs, Python, architecture | Where Python belongs vs where it kills you. Systems, not patches. |
| 6 | Inputs & the outside world | Audio, MIDI, OSC, DMX/Art-Net, sensors. Reactive-vs-generative principle. |
| 7 | Generative systems | Noise, feedback, particles, simulation. State, not patches. |
| 8 | Production & deployment | Profiling, perform mode, networking, redundancy. What breaks and why. |

## Deliverable format

Each module ships **both** forms, like the DMX specimen:
- a written reference doc (read solo), and
- live teaching notes.

## Building

See `_build-tools/README.md`. Source is Markdown; HTML is generated into each module's `build/`.

## Status

See `_meta/STATUS.md` for module progress and open questions.
