# PatchMaster — Yamaha Compatibility & Safety Matrix

**Version:** v18.2 · **Core rule:** *No unprofiled SysEx writes, ever.*
When uncertain, PatchMaster degrades rather than guesses.

---

## 1. Architecture (unchanged detection, new layers above it)

```
MIDI connection
   └─ existing auto-discovery + remembered port  (pm-midi-portname)
       └─ existing identity probe                (F0 7E 7F 06 01)
           └─ existing dialect probes            (M-series 4-byte / classic 3-byte)
               └─ stores pm-sysex-model + pm-sysex-addrlen
                   └─ [NEW] PM_PROFILE  → family, level, capabilities
                       └─ [NEW] PM_ALLOW → per-packet gate inside send()
```

The manual keyboard selector is consulted **only** when detection has produced nothing.
A positively-detected but unprofiled model does **not** fall back to the selector —
it degrades and names the model byte it saw.

Musical rules stay hardware-agnostic. `Hammond → needs Rotary Slow/Fast` contains no
model-specific assumptions; the profile decides whether and how that is implemented.

---

## 2. Compatibility levels

| Level | Meaning |
|---|---|
| **Verified Full** | All PatchMaster functions used on this family are documented/verified, including specific SysEx writes. |
| **Verified Safe MIDI** | Bank Select, Program Change and known CCs only. Read-only SysEx permitted. **No SysEx writes.** |
| **Sound Setup Only** | Sound identification/selection only. Controllers and Scenes withheld. |
| **Detected / Read Only** | Model recognised; identity and read requests only; no configuration changes. |
| **Unknown Yamaha** | Identify if possible; universally safe traffic only. |

---

## 3. Priority 1 — current profiles

| Capability | MONTAGE M / MODX M | MONTAGE / MODX / MODX+ | MOTIF XF / XS |
|---|---|---|---|
| Profile id | `m-series` | `montage-modx` | `motif` |
| Model bytes | 0x0C–0x10 (0x0D typical) | 0x07 | 0x02 |
| SysEx dialect | 4-byte address | 3-byte address | 3-byte address |
| **Level** | **Verified Full** | **Verified Safe MIDI** | **Verified Safe MIDI** |
| Factory sound map | ✅ 3,182 presets embedded | ⚠️ MODX-M map reused, unverified | ⚠️ not mapped |
| Architecture | Performance, 16 Parts, KBD CTRL 1–8 | Performance, 16 Parts, KBD CTRL 1–8 | Voice / Performance / Mixing |
| Channel routing | MIDI I/O Ch (KBD CTRL on) or Part Tx/Rx Ch; Multi / Single / Hybrid | same | Part channel |
| Scenes | ✅ 8, learned Scene CC + SysEx dialect fallback | ⚠️ CC only, unverified | ❌ none — withheld |
| Rotary behaviour | ✅ CC1 → InsA SpdCtrl (factory) | ✅ CC1 (factory convention) | ✅ CC1 (factory convention) |
| Read-only SysEx | ✅ Identity, param request (0x30), bulk request (0x20) | ✅ same | ✅ same |
| **Writable SysEx** | ✅ parameter change 0x10, address a1 ∈ 0x10–0x1F only | ❌ none verified | ❌ none verified |
| Blocked always | bulk write (0x00), a1 outside 0x10–0x1F, non-Yamaha SysEx, GM Reset, unlisted CCs | all SysEx writes | all SysEx writes |

### Existing PatchMaster operations, per family

| Operation | M-series | MONTAGE/MODX/+ | MOTIF XF/XS |
|---|---|---|---|
| Auto-discovery / reconnect | ✅ | ✅ | ✅ |
| Identity + dialect probe | ✅ | ✅ | ✅ |
| Bank Select + Program Change | ✅ | ✅ | ✅ |
| CC7 part volumes | ✅ | ✅ | ✅ |
| CC11 expression | ✅ | ✅ | ✅ |
| Scene recall (learned CC) | ✅ | ✅ CC only | ❌ |
| Scene learning (Sync Doctor) | ✅ CC + SysEx dialect | ✅ CC branch | ⚠️ CC branch only |
| Rotary Slow/Fast (CC1) | ✅ | ✅ | ✅ |
| Read from keyboard (dump) | ✅ | ✅ | ⚠️ untested |
| Performance name write | ✅ | ❌ blocked | ❌ blocked |
| Scene table write | ✅ | ❌ blocked | ❌ blocked |
| Part volume write (T2/0x10) | ⚠️ allowed by address, **still unproven** — flag for individual verification | ❌ | ❌ |
| Pedal router write | ⚠️ same caveat | ❌ | ❌ |

---

## 4. The output gate (`PM_ALLOW`, inside `send()`)

Verified against 25 packet classes — 25/25 correct.

**Allowed**
- Universal Identity Request `F0 7E 7F 06 01 F7` — always, every level
- MIDI-CI discovery `F0 7E 7F 0D …` — always
- Yamaha parameter request `F0 43 3n 7F 1C …` (read-only)
- Yamaha bulk dump request `F0 43 2n 7F 1C …` (read-only)
- Note On / Note Off / Program Change
- Control Change on the allow-list: **0, 1, 7, 11, 22, 32, 64, 86, 92, 121, 123**, plus the learned Scene CC, learned pedal CC and selected rotary CC
- Yamaha parameter change `F0 43 1n 7F 1C …` **only if** the profile defines a write permission **and** the address passes it

**Denied — always, regardless of profile**
- Bulk-memory writes (`F0 43 0n …`)
- Any SysEx address not allow-listed (system/global 0x00, live set 0x09, song/pattern 0x0C–0x0D, anything outside 0x10–0x1F)
- Unknown Yamaha SysEx classes
- Non-Yamaha SysEx (any other manufacturer ID)
- GM Reset and other universal SysEx
- Unlisted CCs, pitch bend, channel/poly aftertouch
- Firmware / initialise / format / reset operations (no address for these is on any allow-list)

Refusals are recorded in a rolling `PM_BLOCKED` buffer and logged, never thrown —
a mistake upstream degrades to silence, not to a crash.

---

## 5. Priority 2 — gaps to close before any write behaviour

| Family | Known | Unknown / to establish |
|---|---|---|
| **MOXF6 / MOXF8** | Motif XF-derived engine; Voice/Performance/Song-Mixing; SysEx param change + bulk documented in its Data List | Model byte; whether the classic 3-byte dialect probe (0x07/0x02) answers; sound map; whether the identity reply distinguishes it from Motif XF |
| **MOX6 / MOX8** | Motif XS-derived | Same three: model byte, probe response, sound map |
| **S90 XS / S70 XS** | Motif XS-derived; Performance = 4 Parts | Model byte; Part/channel routing differs from Motif's 16-part Mixing; no Scene concept |
| **MX49 / MX61 / MX88** | Data List published; supports Bulk Dump, Parameter Change, Bulk/Parameter Request; Voice-based with Bank Select MSB 0 (Normal), 63 (drum/other), 127 (drum) | Model byte; Performance is only 2 layers, so PatchMaster's 8-part model does **not** map; needs a reduced architecture profile |

**Recommended initial level for all four: `Sound Setup Only`** — identify, select sounds by
Bank Select + Program Change, withhold Scenes and controller programming — until a model
byte and a Data List address are each confirmed on real hardware.

**Blocking question for MX specifically:** its 2-layer Performance architecture means
PatchMaster's Part 1–8 blueprint cannot be sent as-is. That family needs a *reduced*
architecture profile (single Voice + optional layer), not merely a permission change.

### Priority 3 notes
- **MOTIF ES / original MOTIF** — older dialect, likely `Sound Setup Only`.
- **YC61 / 73 / 88** — organ-first; rotary is a *hardware* control, so the rotary rule may need a "no MIDI needed" implementation.
- **CP73 / CP88** — no Performance architecture in the MONTAGE sense; `Sound Setup Only` at best.
- **reface** — separate, tiny architecture; treat as its own family.
- **Genos / PSR** — arranger architecture, explicitly out of scope for the synth profile.

---

## 6. Verification method for adding a family

1. Connect, run the existing identity probe, record the **model byte** and which dialect answered.
2. Confirm the factory sound map (Bank Select MSB/LSB + PC) against the instrument's Data List.
3. Test read-only first: parameter request, bulk request. No writes.
4. Only then, one write at a time, using the **diff-two-dumps** method (dump → edit by hand → dump → diff)
   rather than trusting an address from documentation alone.
5. Add the confirmed address to that profile's write permission. Never widen the range speculatively.
