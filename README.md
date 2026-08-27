# PatchMaster v19.2 — Signature Instrument Correction

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
