# Fork notes

Personal fork for evaluating Handy as a potential replacement for, or
reference implementation against, `WhisperDog` (Martin's own dictation
app, vibe-coded starting December, still WIP as of September).

- **Upstream:** https://github.com/cjpais/Handy
- **This fork:** https://github.com/SailorMartin/Handy
- **Local path:** `~/projects/Handy`
- **Remotes:** `origin` = this fork, `upstream` = cjpais/Handy (for
  pulling in updates: `git fetch upstream && git merge upstream/main`)

## Why

WhisperDog hallucination/punctuation/paste-ordering work has been going
on for months. Handy is a mature (23k+ stars, MIT, actively developed)
open-source alternative built with Tauri + Rust, supporting both
Whisper and Parakeet models locally. Worth a real trial before sinking
more time into WhisperDog's remaining bugs.

## Setup status (2026-09-17)

- [x] Forked to SailorMartin/Handy, cloned locally
- [x] `upstream` remote wired up (`origin` uses SSH — HTTPS had no
      credentials configured)
- [x] Rust installed (`rustup`, stable, 1.98.1)
- [x] Bun installed (1.4.2) — note: this project uses Bun, not npm
- [x] Xcode Command Line Tools license agreed
- [x] `bun install` — 343 packages, clean
- [x] `cmake` installed via Homebrew (4.4.3) — `transcribe-cpp-sys`'s
      CMake build script needs it; first `bun tauri dev` attempt
      failed with `is 'cmake' not installed?` before this
- [ ] `bun tauri dev` — retrying after the cmake fix. Needs a real
      Terminal session: first launch will prompt for microphone +
      accessibility permissions, which can't be granted headlessly.

## Next step (manual)

```bash
cd ~/projects/Handy
export PATH="$HOME/.bun/bin:$PATH"
source "$HOME/.cargo/env"
bun tauri dev
```

Grant mic + accessibility permissions when prompted. First build will
be slow (full Rust compile of whisper-rs/transcribe-rs, plus the
transcribe-cpp-sys CMake/ggml build); subsequent runs are fast.

## Evaluation questions to answer once it's running

- Does it handle German dictation acceptably (Whisper model, and/or
  Parakeet if the multilingual quality holds up)?
- How does its punctuation/casing compare to WhisperDog's current
  output on the same kind of disfluent speech?
- Does paste-ordering hold up under the same burst conditions that
  broke WhisperDog's typed injection (see WhisperDog's
  `docs/tasks/paste-ordering-defect.md`)?
- Is Handy's out-of-the-box vocabulary handling (custom terms like
  `TinkerBuddy`, `maisig`, `gVisor`) usable, or does WhisperDog's
  ADR 016 correction-map + case-restoration approach still add real
  value on top?
