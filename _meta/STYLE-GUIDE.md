# Style Guide — Pedagogical DNA

Every module must hold these. Derived from the DMX specimen; they are *why* it works.

1. **Protocol/fundamentals before tool.** Teach the thing underneath, then the node.
2. **Optimization as a worldview, not a chapter.** GPU/CPU, where work belongs,
   "never numpy in perform mode." It lives in *slot 2* of every module — never its own section.
3. **Physical/analog anchors.** Flashlight chase, workshop-vs-warehouse, deus ex machina.
   Ground abstractions in tangible mechanisms.
4. **"It's just X."** Reduce to primitives to demystify (DMX is just 512 numbers at 44 Hz).
5. **Production reality as the payoff.** End on what breaks on a real job.
6. **Conceptual precision.** Get distinctions right (reactive vs generative), not just "make it work."

## The through-line

Every module quietly answers: **where should this computation live, and what is it costing you?**
Cost *is* cooking. State it in the module's own local terms — **rhyme, not cross-reference.**
A module never says "remember the decision from Module 1." It re-grounds "don't overcook" fresh,
in its own data family.

## Per-module skeleton (the same shape every time)

1. **What this data type *is*** — the "it's just X" reduction.
2. **Where the work belongs and what the boundary costs** — the through-line, restated locally.
3. **The right way vs the way that bites you on a show** — correct technique vs the trap.
4. **Hands-on exercise** — makes the point land physically.

## Voice

- Direct, second person, lightly irreverent where it earns it. The specimen's "unpaid labor, like Apple" register.
- Short declarative landings after a build-up. Use blockquotes for the line you want them to carry out the door.
- Code blocks are real and runnable, not pseudo. GLSL/Python as appropriate.

## Verify-before-teaching flags

- **POPs (Module 4):** recent, moved fast across builds. Verify operator names, conversion
  paths, perf characteristics against current Derivative docs. Note target build.
- **Module 0 cook-inspection tools:** exact UI names drift across builds. Verify.
- Always note the **target TD build** when a claim is build-sensitive.
