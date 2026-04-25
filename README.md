# OK-Composer

A browser-based songwriting and music production tool. Write lyrics, build chord progressions, program beats, record audio, arrange sections, and mix everything — all in a single HTML file with no installation required.

**Live app:** https://okimawake.github.io/ok-composer/

---

## Tabs

### Lyrics
- Full-screen lyric editor with autosave
- Section markers (Verse, Chorus, Bridge, etc.) with colour coding
- Section-level loop and arrangement controls
- Tag system for organising songs by genre, mood, or project

### Audio
- Microphone recording via the browser (requires HTTPS — use the live URL on mobile)
- Named recordings saved per-song
- Playback, trim, and gain controls
- Waveform display with trim-start / trim-end handles

### Compose
Seven sub-tabs, all playing in sync with a shared BPM and loop length:

**Chords**
- Tap a piano keyboard to build chord progressions
- Chord types: major, minor, 7th, maj7, min7, sus2, sus4, dim, aug, add9, and more
- Inversions (root, 1st, 2nd, 3rd)
- Custom chords: pick up to 7 notes manually
- Instruments: Rhodes EP, Electric Piano, Acoustic Piano, Organ, Warm Pad
- Save and reload named progressions per song

**Sampler**
- Upload audio clips (WAV, MP3, etc.) and trigger them on a mini-keyboard
- Per-clip trim-start / trim-end, fade in / fade out, gain, and pitch controls
- Loops in sync with the transport

**Bass**
- 11 synthesised bass voices: Electric, Upright, Synth, Sub, 808, Slap, Muted, Acid, Fretless, Rubber, Picked
- Follow chord roots automatically or set an independent bass progression
- Octave and gain controls

**Beat**
- 50+ factory drum patterns: Rock, Pop, Hip-Hop, Trap, House, Techno, Funk, Swing, Bossa Nova, and many more
- Save, name, and reload custom patterns per song
- Factory / User pattern toggle
- 16-step grid for kick, snare, and hi-hat with per-step editing
- Per-drum gain and pitch controls (collapsed sub-menu)

**Sequencer**
- 16-step sequencer for kick, snare, and hi-hat
- Straight, dotted, and triplet rhythm modes
- Per-drum gain and pitch controls (same state shared with Beat tab)

**Melody**
- Piano-roll editor with scrollable piano-key column synced to the grid
- 8 melody voices: Flute, Rhodes, Piano, Pluck, Mallet, Pad, Organ, EP Keys
- Draw, move, and delete notes on the roll
- Scales and key helpers

**Progressions**
- Save and manage chord progressions independently of the current arrangement
- Copy progressions across songs

### Mixer
DAW-style channel mixer with:

**Channel strips** (Chords, Bass, Drums, Audio Loop, Sampler, Melody, Recordings)
- Vertical drag fader (double-click to reset to unity)
- 4 send knobs per channel: **Rev** (reverb), **Dly** (delay), **Cho** (chorus), **Dst** (distortion)
- Mute button

**Master strip** (gold fader, rightmost)
- Same 4 send knobs (Rev, Dly, Cho, Dst) for global effect amounts
- Fader controls master output gain

**Master FX section**
- **Reverb shape**: Decay, Damping, Width knobs
- **Delay shape**: Time knob, Feedback knob, Sync-to-tempo toggle with note-division selector (1/32 – 1 bar)
- **12 insert effects** (click name to toggle on/off, LED dot shows state):
  EQ, Compressor, Distortion, Overdrive, Chorus, Flanger, Phaser, Auto-Wah, Auto-Filter, Auto-Pan, Tremolo, Limiter
- All controls use drag rotary knobs; double-click any knob to reset to default
- Reset All button restores mixer to defaults

### Notes
Free-text scratch pad per song, autosaved.

### Arranger
- Arrange song sections (Verse, Chorus, etc.) into a timeline
- Drag to reorder; set loop counts per section
- Preview the full arrangement

### Launcher
- 6 × 8 trigger pad grid
- Assign saved progressions and beats to pads for live triggering

---

## Data & Storage

- Songs are saved to `localStorage` on every edit — no account required
- Each song stores: lyrics, sections, tags, chord progressions, beat patterns, bass settings, sampler clips, melody notes, mixer state, and notes
- **Backup / Restore**: export a `.zip` bundle (audio clips + MIDI + lyrics); import to restore on any device
- Songs are **per-device** — use Backup to move work between iPhone and Mac

---

## Mobile / PWA

- Add to Home Screen on iOS for fullscreen launch
- Portrait and landscape supported
- Mic recording requires the live HTTPS URL (not `localhost`)

---

## Tech

Single `index.html` file — no build step, no framework, no server.

- Audio engine: Web Audio API (synthesised voices, no samples except user uploads)
- Parallel FX buses: Reverb (convolver IR), Delay (feedback loop), Chorus (modulated delay), Distortion (waveshaper + LP filter)
- Serial master insert chain: EQ → Compressor → Distortion → Overdrive → Chorus → Flanger → Phaser → Auto-Wah → Auto-Filter → Auto-Pan → Tremolo → Limiter
- Backup format: JSZip bundle
- Deploy: GitHub Pages (auto-deploys ~60s after push to `main`)
