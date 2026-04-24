# OKIMAWAKE — Songwriter's Studio

Single-file HTML songwriting app. Everything lives in `index.html` — markup, CSS, and JavaScript. No build step. No external framework. The only CDN dep is JSZip for backup export.

Deploy target: GitHub Pages (public repo with an unguessable name). Live URL redeploys ~30-60s after every push to `main`.

## How to work on this

- Edit `index.html` in place.
- To test locally: `python3 -m http.server 8000` in the repo root, open `http://localhost:8000` in a browser.
- To test on iPhone over the same Wi-Fi: find your Mac's local IP (`ipconfig getifaddr en0`), open `http://<ip>:8000` in Safari. Note: `getUserMedia` (mic recording in the AUDIO tab) only works over `https://`, so use the live Pages URL for that.
- To ship: `git add index.html && git commit -m "…" && git push`.

## Verification before shipping

The file is large (~400KB, 8300+ lines). Run syntax check on the extracted script blocks before claiming a change is done:

```bash
python3 -c "
import re
html = open('index.html').read()
for i, s in enumerate(re.findall(r'<script[^>]*>(.*?)</script>', html, re.DOTALL)):
    open(f'/tmp/s_{i}.js', 'w').write(s)
    print(f'{i}: {len(s)} chars')
"
for f in /tmp/s_*.js; do node --check "$f" || echo "FAIL: $f"; done
```

For structural changes, also do a tag-balance sanity check (count opening vs. closing tags, excluding void + self-closing).

## Architecture — what lives where

**Top of `<body>`:** header + tab switcher (LIBRARY / WRITE / COMPOSE / AUDIO / ARRANGER). Each tab is a pane that's shown/hidden via CSS.

**COMPOSE tab** (the biggest one) has sub-tabs: CHORDS, BASS, BEAT, SAMPLER, MELODY, SEQUENCER, PROGRESSIONS, MIXER, INSTRUMENTS.

**Audio engine:** Web Audio API. One `composeCtx` (AudioContext) for the COMPOSE playback engine, a separate `recAudioCtx` for the AUDIO-tab recorder. Voices are synthesized inline (no samples except user-uploaded Sampler clips and audio recordings).

**Mixer graph (master bus):**
```
channels → composeFxChain.in
  → eq → compressor → distortion → overdrive → chorus → flanger
  → phaser → autoWah → autoFilter → autoPan → tremolo → limiter
  → composeFxChain.out → composeMaster → destination
```
Each FX module is a parallel dry/wet wrapper built by `_fxModule()`. Reverb and Delay are channel sends in parallel to the insert chain, not part of it. FX defaults + params: `_defaultFx()`, `FX_ORDER`, `FX_LABELS`, `FX_PARAMS`. Runtime param application: `applyMixerToNodes()`.

**Scheduler:** `playCompose()` schedules ahead by one loop; live edits (add/remove chord, tempo change) call `rescheduleTimelineFromNow()` which cancels only future-scheduled nodes via the `_startT` tag — currently-playing notes continue unbroken.

**Piano rendering:** `buildPianoInto(host, onTap, startMidi, endMidi)`. Black keys are positioned with pure CSS percentages (`left: calc((i+1)/total * 100% - 11px)`) — no layout measurement, works at any width.

**State:** per-song state lives in `state.songs[]`, selected via `state.activeId`. `getActive()` returns the current song. Persisted to `localStorage` on every edit via `touchActive()` → `save()`. `_defaultMixer()`, `_defaultFx()` self-heal missing fields on older saves.

**Backup/restore:** JSZip bundles BACKUP export (audio clips + MIDI + lyrics + notes). Import reconstructs `state.songs`.

## Mobile / iOS

Viewport + PWA meta tags are set. The app launches fullscreen when Added to Home Screen on iOS. Portrait + landscape both supported; breakpoints at 1100px and 700px. `touch-action` isn't explicitly set on most elements — watch for accidental double-tap-to-zoom on the keyboard and transport buttons; if it appears, add `touch-action: manipulation` on the affected selectors.

Songs are stored per-device in `localStorage`. iPhone and Mac do NOT share state — use BACKUP export/import to move work between devices.

## Pending features (unprioritized, from user's earlier request)

- **Per-instrument saved progressions.** Currently only CHORDS has saved progressions. Extend to BASS, SAMPLER, BEAT. Beat version should support combining multiple saved beats into a single stored pattern.
- **Saved-progression UX:** Save button should prompt overwrite-existing vs. save-as-new. Add load dropdowns per instrument.
- **LYRICS > SECTIONS:** per-section loop counts; per-section assignments for progression, beat, audio clip.
- **Compositions tab** (partially wired): dropdowns that swap in a specific saved progression / beat / audio clip; double-click title to rename a composition; assign compositions to sections.
- **Cross-song copy:** copy a progression from another song — dropdowns should show both song name and progression name.
- **Sampler & Audio-recorder waveform visualization:** draw the waveform; draggable trim-start / trim-end handles, draggable fade-in / fade-out handles.
- **Per-block length + override on non-chord instruments:** NOTE OVERRIDE (BASS/SAMPLER/MELODY), BEAT OVERRIDE (BEAT).
- **Rhythm options per instrument:** dotted, triplet, swing. Custom 16-step sequencer popup for arbitrary patterns.
- **Randomness slider on BEATS.**
- **Beat-repeat + note-repeat sliders** with division dropdown (1/32 through 1 whole).

Outstanding in-progress items from task list:
- #61 ADSR envelope controls wired into all voices (partially done — verify coverage)
- #62 Live Loops: independent per-track loop lengths
- #63 Final verification pass after any new round of changes

## Conventions

- Keep it one file. Don't split into modules unless absolutely necessary — the user values the "one file, no build" property.
- Don't add frameworks. Vanilla JS + hand-rolled DOM.
- Don't add emojis to the UI.
- Preserve the dark / red / grey color palette defined in `:root`.
- Prefer CSS percentage / flexbox / grid layouts over measured layouts (the keyboard history explains why).
- When adding UI, follow the existing sub-tab pattern and `mixer-fx-card` styling conventions.
- After any non-trivial change, run the syntax check above before committing.
