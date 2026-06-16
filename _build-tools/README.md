# Build tools

Source of truth is Markdown in each `modules/NN-name/src/`. HTML is generated into `build/`.

The DMX specimen shows the target output: a self-contained styled HTML doc (sticky nav,
diagrams, exercises with answers). A converter that takes `src/*.md` + `shared/css` and
emits styled HTML into `build/` goes here. Not yet written — placeholder.

Suggested approach: a small Python script (pandoc-free) or pandoc with a shared template.
Decide when the first module is drafted enough to need a real build.
