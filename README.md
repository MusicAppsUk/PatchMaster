# PatchMaster v19.2.5 — Signature Instrument Correction

PatchMaster is a musician-facing Yamaha performance builder. Its governing workflow is:

**SEARCH → RESEARCH → BUILD → SEND → PLAY**

v19.2 carries forward the v19.1 stabilisation work and adds the first signature-instrument research correction. It does not add speculative controller features.

## v19.1 stabilisation carried forward

- Preserved the existing MIDI safety gate, Yamaha identity/dialect detection and "degrade, never guess" behaviour.
- Removed the destructive startup behaviour that unregistered service workers and deleted caches every time the app opened.
- Restored a controlled service-worker registration path.
- Versioned the application-shell cache as `patchmaster-v19.1` and made navigation network-first with offline fallback, reducing stale-build risk.
- Kept cross-origin research/API traffic outside the service-worker cache.
- Aligned PWA manifest background/theme colours with the application's actual light shell.
- Replaced the effectively empty README with this technical handover.
- Left `patchmaster-theme.css` unlinked intentionally: it is a legacy/alternate theme asset and linking it during a stabilisation release would change the live UI without a controlled visual regression pass.

## Protected architecture

Do not weaken these areas without an explicit, reviewed reason:

1. **Final MIDI safety gate (`PM_ALLOW`)** before normal outbound MIDI packets.
2. **Unknown Yamaha models degrade safely** rather than inheriting an assumed family map.
3. **M-series vs classic Yamaha SysEx dialect detection** remains evidence-led.
4. **Bulk/unsafe writes remain blocked** unless specifically allow-listed.
5. **Research evidence and sound-map authority remain separate concerns.** A plausible sound name is not permission to send an unverified program mapping.

## Known work after v19.2

These are intentionally *not* silently solved in this stabilisation build:

- Broaden evidence-led song research beyond the currently implemented sources.
- Complete verified preset/sound maps for promised Yamaha families rather than borrowing across families.
- Refine Yamaha family entries into accurate individual model capabilities/keybeds.
- Move engineering/MIDI diagnostic tools behind a clearly separated expert/support surface.
- Decide the final PatchMaster visual identity and either integrate or retire `patchmaster-theme.css` after visual regression testing.
- Add automated regression fixtures for the full SEARCH → RESEARCH → BUILD → SEND path and known reference songs.

## Release discipline

v19.0 remains the frozen pre-stabilisation baseline. v19.1 is the stabilisation baseline. v19.2 is the signature-instrument correction build described below. Future feature requests should remain separate unless required to correct safety, integrity or a broken core workflow.

## v19.2 — signature-instrument research correction

- Added a verified MODX M `Penny Whistle` sound entry (preset #631; MSB 63 / LSB 4 / PC 118).
- Research now recognises tin whistle / pennywhistle / whistle evidence before generic flute matching.
- Added an explicit *My Heart Will Go On* signature-arrangement guardrail so its iconic whistle cannot disappear behind a generic ballad rig when source parsing is incomplete.
- Added tin-whistle musician-role labelling and search semantics.
- MIDI safety, identity detection, SysEx allow-lists and the v19.1 PWA stabilisation remain unchanged.


## v19.2.5 MIDI recovery
- Restores automatic Web MIDI reconnection when a previously selected MIDI port is stored, including browsers where the Permissions API MIDI query is unavailable or unreliable.
- Preserves the existing manual Connect path, SysEx fallback, port selection, Yamaha detection, and PM_ALLOW safety gate.
- No research or patch-mapping changes from v19.2.


## v19.2.5 — Yamaha Multi/GM foundation
- Branches from known-good v19.2.1 MIDI recovery baseline.
- Before constructing an M-series song Performance, sends the documented universal GM System On message to recall Yamaha Multi/GM, activating all 16 Part slots. This fixes the architectural error where Program Change could replace an existing Part but could not create an empty Part.
- Waits 350 ms after reset (Yamaha guidance: GM setup data follows 150–300 ms later), then sends each Part's MSB, LSB and Program Change in order with settling time after Program Change.
- Keeps the existing verified M-series name-stamp path, so the resulting edit buffer is renamed to the song after construction.
- Does not alter v19.2.1 MIDI reconnect/identity logic or bulk-write safety gate.
- Important: MSB 63 selects Part 1 of the referenced factory Performance into the addressed Part. Multi-Part donor Performances therefore contribute Part 1 only; they are not silently treated as a complete multi-Part instrument.

## v19.2.6 — verified single-Part foundation
- Stops treating every MODX M Performance-list row as a verified one-Part MIDI voice.
- Dynamic catalogue arithmetic is lookup-only and cannot transmit until Single-Part status is verified.
- Known Multi-Part presets CFX Concert, All 9 Bars!, and Seattle Sections are blocked from one-slot insertion.
- My Heart Will Go On signature uses Penny Whistle plus single-Part CFX Stage and Seattle Violins 1 support.
- Factory mappings take precedence over stale learned mappings.
- Keeps the v19.2.1 MIDI recovery path and v19.2.5 Multi/GM slot activation.
- After sending on M-series, PatchMaster now reads back Part Name blocks 1–8 from the edit buffer and records a hardware receipt; it no longer relies only on the outbound MIDI log.
