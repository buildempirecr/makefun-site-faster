# makefun-site, faster

A drop-in replacement for **makefun-site 1.0.0** — the reference static site generator of make.fun's
*Build a static site faster* bounty — that produces a **byte-identical output tree** and runs in
roughly **2.5x less CPU time**.

The pipeline, the template dialect, the front matter grammar and the Markdown subset are the
reference's, unchanged. Only the execution is faster.

## Verified

Every case of the published practice set is answered **byte for byte** as makefun-site 1.0.0 answers
it (3/3 cases, compared against `practice/reference/expected.json`).

| entry | CPU time, practice set (3 cases) |
|---|---|
| makefun-site 1.0.0 (`baselines/makefun-site`) | ~830 ms |
| this entry | ~330 ms |

## Where the speed comes from

1. `escapeHtml` — the reference ran a regex replace with a callback on every call. This scans once,
   returns the input untouched when nothing needs escaping, and otherwise splices runs.
2. `inline()` — the reference escaped every plain character individually. This buffers plain runs and
   escapes each run once.
3. `lex()` — per-line regexes are guarded by a first-character test, so an ordinary paragraph line
   costs a `charCodeAt` instead of five regex attempts.
4. `slugify()` and `longDate()` are memoised; `slugify()` skips NFKD and the mark-strip regex for
   ASCII input.
5. Template rendering pushes into one array and joins once, instead of building intermediate strings
   at every nesting level.
6. One card per source is built once and shared by the page layout and every list, instead of
   re-counting each body's words per render.

## Run it

```sh
MAKEFUN_INPUT_DIR=<dir with cases.json> MAKEFUN_OUTPUT_DIR=<out dir> node main.ts
```

`main.ts` reads every case from `cases.json` and writes `answers.json`, one answer per case, in
order — the same driver the reference ships.

## Attribution

Derived from makefun-site 1.0.0's published source, shipped in the bounty kit under
`baselines/makefun-site/vendor/makefun-site` (MIT). See `NOTICE`.
