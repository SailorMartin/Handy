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

## Setup status (2026-09-17) — RUNNING

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
- [x] `bun tauri dev` — clean run 2026-09-16 22:37. Mic + accessibility
      + input-simulation permissions granted, shortcuts initialized,
      auto-downloaded `handy-computer/parakeet-unified-en-0.6b-gguf`
      (Q8_0), bound to Metal on the M2 Pro, registered as a login item.
      No errors.

## Known limitation to test around

The model Handy auto-selected on first run, **`parakeet-unified-en-0.6b`,
is English-only** (log shows `supports_translate=false`,
`supports_language_detect=false`). German dictation needs a model swap
in Handy's settings — either a multilingual Parakeet variant (if
bundled) or a Whisper model. Untested which options Handy actually
offers for German.

## Evaluation questions (in progress)

- [ ] Does it handle German dictation acceptably once switched to a
      multilingual model (Whisper, and/or Parakeet if quality holds)?
- [ ] How does its punctuation/casing compare to WhisperDog's current
      output on the same kind of disfluent speech?
- [ ] Does paste-ordering hold up under the same burst conditions that
      broke WhisperDog's typed injection (see WhisperDog's
      `docs/tasks/paste-ordering-defect.md`)?
- [ ] Is Handy's out-of-the-box vocabulary handling (custom terms like
      `TinkerBuddy`, `maisig`, `gVisor`) usable, or does WhisperDog's
      ADR 016 correction-map + case-restoration approach still add
      real value on top?

## Day-to-day run command

```bash
cd ~/projects/Handy
export PATH="$HOME/.bun/bin:$PATH"
source "$HOME/.cargo/env"
bun tauri dev
```

## Native app build (2026-09-28)

Built as a real `.app` so it gets its own entry in Privacy & Security
(Accessibility, Microphone) instead of borrowing Terminal's:

```bash
cd ~/projects/Handy
export PATH="$HOME/.bun/bin:/opt/homebrew/bin:$PATH"; source "$HOME/.cargo/env"
bun tauri build --bundles app --config '{"bundle":{"createUpdaterArtifacts":false}}'
ditto src-tauri/target/release/bundle/macos/Handy.app /Applications/Handy.app
```

- `createUpdaterArtifacts:false` because the stock config demands an
  updater signing key we don't have; `--bundles app` skips the DMG.
- Signed ad-hoc (`signingIdentity: "-"`), not notarized. Consequence:
  every rebuild changes the code signature, and macOS may drop the
  Accessibility grant. If paste silently stops after a rebuild, remove
  Handy from Accessibility and re-add it.
- Release build takes several minutes (LTO, codegen-units=1).
- Same bundle id (`com.pais.handy`) as the dev run, so settings and
  downloaded models carry over.

## Wrong-text-pasted bug: root cause (`src-tauri/src/clipboard.rs`)

Default path (`PasteMethod::CtrlV`): save clipboard -> write transcript
-> sleep `paste_delay_ms` (60) -> send Cmd+V -> sleep
`paste_delay_after_ms` (60) -> restore old clipboard. If the target app
handles Cmd+V later than ~60ms after the keystroke, it reads the
*restored* clipboard and pastes the old content. Timer-based race.

Options, best first:
1. `reliable_paste` (Debug panel, Cmd+Shift+D; macOS/Windows beta,
   issue #502): restores the clipboard only after the target app has
   actually read the transcript. Fixes the race rather than widening it.
2. Raise the paste delays in the Debug panel (100-300ms). Crude.
3. `PasteMethod::Direct` (types via enigo). Avoids the clipboard
   entirely, but upstream docs warn of garbled output on non-US
   layouts; untested on QWERTZ.
