# labelle-nullfixture-backend

An **out-of-tree** (third-party-style) backend package for
[labelle](https://github.com/labelle-toolkit), used to exercise the
**pluggable-backends** remote-fetch pipeline (labelle-assembler epic #386,
RFC #378).

It is a standalone-repo clone of the assembler's built-in `null` backend: a
**headless** backend with no window, GPU, input, or audio — every backend
module is a silent no-op. That makes it the smallest backend that still
produces a game binary which actually compiles and runs (a fixed-frame tick
loop that exits cleanly), so it can validate the whole
**fetch → resolve → generate → build** path with zero native dependencies.

## How a game uses it

Instead of a built-in `.backend = .raylib`, a project names this package:

```zig
.{
    .name = "my_game",
    // Fetched into the assembler's package cache like a plugin (#386 Phase 6a),
    // then codegen is driven entirely by this repo's backend.manifest.zon.
    .backend_package = .{
        .name = "nullfixture",
        .repo = "github.com/labelle-toolkit/labelle-nullfixture-backend",
        .version = "0.1.0",
    },
    // ...
}
```

A `local:` repo (`.repo = "local:../labelle-nullfixture-backend"`) works too,
for monorepo / offline development.

## What's in here

| Path | Role |
|------|------|
| `backend.manifest.zon` | Declares the codegen contract the assembler reads (run-loop style, main-loop template, build fragments). **Mandatory** for an external backend — there is no enum-path fallback. |
| `templates/headless.txt` | The generated `main()`: a bounded `while (frame < max_frames)` tick loop. |
| `build_fragments/backend_dep.txt` | The `build.zig` lines that wire the four no-op modules into the game. |
| `build_fragments/link.txt` | Empty — a headless backend links nothing native. |
| `src/{gfx,input,audio,window}.zig` | No-op modules satisfying the engine's backend contracts. |

## Tests

```bash
zig build test   # smoke-tests each no-op module's import surface
```
