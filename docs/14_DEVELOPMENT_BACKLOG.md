# 14 — Development Backlog

Effort scale (solo, focused; *assumption*): **S** ≤ 1 day · **M** 2–3 days · **L** 4–7 days · **XL** > 1 week. Priority: **P0** blocks the slice · **P1** needed for full MVP · **P2** stretch.

## Epics
E0 Setup · E1 Movement · E2 Echo Trace · E3 Puzzle framework · E4 Characters/switching · E5 Chapter 1 levels · E6 Enemies · E7 Boss · E8 Save/checkpoints · E9 UI · E10 Audio · E11 Art/VFX · E12 Accessibility · E13 QA tooling · E14 Deployment

## First 20 tasks (ordered, implementable without further design discussion)

### BL-001 — Create repository, pin Godot, folder structure — E0 · P0 · S
- **Desc:** Create repo with tree from `10` §2, `.gitignore`, `README`, pinned Godot version noted in `version.txt`.
- **Rationale:** Everything depends on it. **Deps:** none.
- **Acceptance:** project opens; empty `Main.tscn` runs; tree matches. **Test:** fresh clone opens on a second machine.

### BL-002 — Project settings: 60 Hz physics, named layers, input actions — E0 · P0 · S
- **Desc:** Set `physics_ticks_per_second=60`; name collision layers (`10` §6); create all InputMap actions (`02` §5).
- **Deps:** BL-001. **Acceptance:** actions appear with defaults for keyboard and controller. **Test:** input printout test scene.

### BL-003 — InputService (hold/toggle, jump buffer) — E1 · P0 · M
- **Deps:** BL-002. **Acceptance:** toggle vs hold for `ability`, `trace`; buffer 0.10 s works. **Test:** unit test with synthetic events.

### BL-004 — Player controller graybox — E1 · P0 · L
- **Desc:** Run, jump (variable), coyote, buffer, one-way drop-through. Parameters in a `MovementConfig` Resource.
- **Deps:** BL-003. **Acceptance:** `02` §6 values met; frame-rate independent. **Test:** movement test room; automated run 10 s scripted input yields identical end position twice.

### BL-005 — Movement test room — E1 · P0 · S
- **Desc:** Gaps, steps, one-way, spikes, moving platform (disabled by default).
- **Deps:** BL-004. **Acceptance:** each element exercisable. **Test:** manual checklist.

### BL-006 — Camera and scale decision — E1 · P0 · M
- **Desc:** Follow camera with lookahead; decide BH→pixel scale, zoom, resolution policy; record in D-030.
- **Deps:** BL-005. **Acceptance:** decision logged with screenshots; camera does not jitter. **Test:** 5-minute playtest.

### BL-007 — StateMachine + AbilityBase — E4 · P0 · M
- **Deps:** BL-004. **Acceptance:** states Idle/Run/Jump/Fall/Hurt; `AbilityBase` with cooldown API. **Test:** unit tests for cooldown.

### BL-008 — Blitz Dash — E4 · P0 · M
- **Desc:** CH-01/CH-02 behavior incl. escalating cooldown, wall stop, pulse-trap pass-through flag.
- **Deps:** BL-007. **Acceptance:** AC-BL-1..3. **Test:** unit test of cooldown formula; manual dash gap room.

### BL-009 — Debug overlay and logger — E13 · P0 · S
- **Desc:** Tick counter, state, trace buffer length; `Debug.log` categories.
- **Deps:** BL-001. **Acceptance:** toggle with F3; stripped in release. **Test:** export build lacks overlay.

### BL-010 — TraceBuffer / TraceData + unit tests — E2 · P0 · M
- **Desc:** Packed arrays for pos/facing/anim/hero/events; append, read(tick), duration.
- **Deps:** BL-001. **Acceptance:** 480 ticks stored; read-back exact. **Test:** unit tests.

### BL-011 — TraceRecorder — E2 · P0 · M
- **Desc:** F-T1..F-T4, cooldown F-T8, cancel F-T3.
- **Deps:** BL-004, BL-010. **Acceptance:** records at 60 Hz, max TR-01, short discard cue. **Test:** scripted input; buffer length equals ticks elapsed.

### BL-012 — TraceGhost and TracePlayer — E2 · P0 · L
- **Desc:** Playback from Origin Mark, hold (TR-03), dissolve (TR-06), slots (TR-05).
- **Deps:** BL-011. **Acceptance:** F-T5, F-T7. **Test:** replay positions equal recorded per tick.

### BL-013 — WeightSensor, PressurePlate, Door — E3 · P0 · M
- **Desc:** Plate counts player + ghost weight; door open while active; countdown ring when ghost-held.
- **Deps:** BL-012. **Acceptance:** F-T9, F-T10; door closing warning 0.5 s. **Test:** room 1.3 graybox solved by trace.

### BL-014 — Resettable contract and RoomController — E3 · P0 · M
- **Desc:** `reset()`, snapshot/restore; hold-reset input; clears traces on reset.
- **Deps:** BL-013. **Acceptance:** reset in <1 s; no residual state. **Test:** reset-after-solve test.

### BL-015 — Trace determinism golden test — E13 · P0 · M
- **Desc:** Scripted inputs recorded, replayed, compared per tick over 100 runs.
- **Deps:** BL-012. **Acceptance:** zero deviation. **Test:** automated; fail the build on mismatch.

### BL-016 — Trace HUD — E9 · P0 · S
- **Desc:** Ring, icon states, cooldown.
- **Deps:** BL-011. **Acceptance:** reflects recording/playing/holding. **Test:** manual.

### BL-017 — Integrity, hazards, pits, Safe Ledges — E1 · P0 · M
- **Deps:** BL-004. **Acceptance:** SV-01/SV-02. **Test:** manual + unit for damage rules.

### BL-018 — Beacon and respawn — E8 · P0 · M
- **Deps:** BL-014, BL-017. **Acceptance:** death → last Beacon, Integrity full, room reset. **Test:** die in each graybox room.

### BL-019 — Graybox rooms 1.1–1.3 — E5 · P0 · M
- **Deps:** BL-008, BL-013, BL-018. **Acceptance:** PF-1…PF-10 checklist per room. **Test:** testers solve 1.3 unaided.

### BL-020 — Playtest checkpoint #1 (Gate A prep) — E13 · P0 · S
- **Desc:** 3–5 testers; template with observations, time-to-solve, confusion points.
- **Deps:** BL-019. **Acceptance:** report stored in `docs/playtests/`. **Test:** Gate A decision recorded.

## Remaining tasks (compact)

| ID | Task | Epic | Pri | Effort | Deps | Acceptance | Test |
|---|---|---|---|---|---|---|---|
| BL-021 | Pulse + Release | E6 | P0 | M | BL-007 | CH-07 stun; release dissolves | unit + manual |
| BL-022 | Memory Sentry AI | E6 | P0 | L | BL-021, BL-012 | Patrol, cone sight, trace bait | AI test room |
| BL-023 | Seam system + layout states | E5 | P0 | L | BL-012 | Deterministic toggle; traces dissolve | scripted crossings |
| BL-024 | Graybox rooms 1.4–1.6 | E5 | P0 | L | BL-022, BL-023 | Rooms complete | playtest |
| BL-025 | Pylons + Sync Window | E7 | P0 | M | BL-013 | SV-06 window; trace strike works | unit |
| BL-026 | Corruptor prototype (P1–P2) | E7 | P0 | XL | BL-025 | AC-CO-1..3 | scripted boss test |
| BL-027 | SaveService v1 (slots, autosave at Beacon) | E8 | P0 | L | BL-018 | Save/load round-trip | `15` §8 |
| BL-028 | Menus (continue/new/settings) | E9 | P0 | M | BL-027 | Navigable by keyboard + pad | manual |
| BL-029 | HUD (pips, ability ring) | E9 | P0 | S | BL-017 | Reads state | manual |
| BL-030 | Placeholder audio + AudioDirector | E10 | P0 | M | BL-001 | Cues by id | manual |
| BL-031 | Slice art pass (Blitz, tileset, Corruptor) | E11 | P0 | XL | BL-024 | Style-guide compliant | review |
| BL-032 | Slice VFX (dash, trace, seam, glitch-soft) | E11 | P0 | L | BL-031 | Reduced-effects works | manual |
| BL-033 | Settings: remap, hold/toggle, volume | E12 | P0 | M | BL-028 | Persisted | manual |
| BL-034 | Export presets Win/Linux + build script | E14 | P0 | M | BL-027 | Builds run on clean machines | manual |
| BL-035 | Slice acceptance run | E13 | P0 | M | all slice | `15` §13 passes | checklist |
| BL-036 | Hero swap + HeroData | E4 | P1 | L | BL-007 | CH-08; per-hero cooldowns | unit |
| BL-037 | Echo Veil Sight + hidden platforms | E4 | P1 | L | BL-036 | AC-EC-1..2 | room test |
| BL-038 | Second trace slot | E2 | P1 | M | BL-012 | F-T7 | unit |
| BL-039 | Nova flight / Volt hack / Grim shield | E4 | P1 | XL | BL-036 | AC-NO/VO/GR | unit + rooms |
| BL-040 | Noise system + meter | E3 | P1 | M | BL-007 | SV-03/04 | unit |
| BL-041 | Gravity zones / portals | E3 | P1 | M | BL-004 | `04` §10, §12 | rooms |
| BL-042 | Behavior Ledger + Reflection | E7 | P1 | XL | BL-036 | AC-RE-1..3 | unit |
| BL-043 | Accessibility baseline (colorblind, reduced fx, assist) | E12 | P1 | L | BL-033 | `15` §11 | checklist |
| BL-044 | Witness Chamber framework + Protocol Assembly | E3 | P1 | L | BL-014 | Clauses persist | integration |
| BL-045 | Fragment collectibles + gallery | E3 | P2 | M | BL-027 | Persist | manual |
| BL-046 | License audit tooling (`CREDITS.md` generator) | E14 | P1 | S | `12` | Matches register | script |
