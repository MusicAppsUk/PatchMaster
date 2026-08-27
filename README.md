# PatchMaster v19.2.3 — MODX M Mapping Foundation

Foundation repair based on Yamaha MIDI architecture and legacy compatibility.

- Keeps the v19.2.1 MIDI recovery path unchanged.
- Corrects core legacy Single-Part bank/program coordinates used by PatchMaster.
- Built-in factory mappings now take precedence over stale user/learned mappings with the same factory name.
- Raw MODX M Performance-list ordinal arithmetic is no longer marked verified and is blocked from automatic Send.
- Penny Whistle uses the retained legacy Single-Part address MSB 63 / LSB 3 / Program 16.
- Corrected core coordinates include CFX Concert, Rd 1 Gallery, Wr Amp, All 9 Bars!, The Preacher, Seattle Sections, Penny Whistle and The Synth Brass 1.

Important: Yamaha's MODX M Performance List numbering is not the same namespace as Single-Part MIDI bank/program selection. New M-series Performances are interleaved in the catalogue. PatchMaster must not derive MIDI addresses from catalogue row numbers.
