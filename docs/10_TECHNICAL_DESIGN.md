# 10 — Technical Design Document

Godot 4.x, GDScript. **Pin one stable 4.x minor version at M1 (D-002)**; do not upgrade mid-milestone. Nothing here has been built or benchmarked.

## 1. Principles
- Small, testable systems with narrow interfaces; composition over inheritance.
- Signals for events; direct calls only parent→child.
- Data in Resources (`.tres`), logic in scripts.
- Fixed 60 Hz physics tick is a hard requirement for Trace (`physics_ticks_per_second = 60`).
- No framework without a stated purpose (see §4 for each).

## 2. Repository tree
```
echoes/
  project.godot
  README.md
  docs/                      # this package
  addons/                    # pinned third-party (test framework only, if used)
  assets/
    art/{characters,enemies,environment,ui,vfx}/
    audio/{music,sfx,ambience}/
    fonts/
  data/                      # Resources (.tres): abilities, enemies, trace, noise, rooms
  src/
    core/                    # autoloads, event bus, game state, save, settings
    input/                   # input abstraction
    player/                  # controller, state machine, hero data
    abilities/               # one script per ability + base
    trace/                   # recorder, player, ghost, buffer
    puzzle/                  # plates, switches, doors, resettable contract
    world/                   # seams, gravity zones, portals, noise system
    enemies/                 # base + each enemy
    bosses/                  # boss scripts + phase data
    ui/                      # HUD, menus, dialogue
    fx/                      # glitch, trace visuals
  scenes/
    rooms/ch01 … ch05/
    actors/ ui/ fx/
  tests/                     # unit + integration
  tools/                     # export scripts, validators
  licenses/                  # asset license copies
```

## 3. Scene organization and transitions
- `Main.tscn` (root): holds `RoomHost`, `UIRoot`, `Overlay`.
- Each room is a self-contained scene with a `RoomController` (extends Node) implementing the **Resettable contract** (`reset_room()`, `get_state()`).
- `SceneRouter` (autoload) loads rooms (threaded), fades, and positions the player at an entry marker. Rooms never reference other rooms by path; they emit `exit_requested(exit_id)`.

## 4. Autoloads (kept to five)
| Autoload | Purpose |
|---|---|
| `Events` | Typed signal hub (room_entered, trace_launched, noise_emitted, boss_phase_changed …) |
| `GameState` | Current chapter, unlocked heroes, solved puzzle ids, clauses; serialization |
| `SaveService` | Slots, autosave, versioning, migrations |
| `Settings` | Accessibility, audio, input bindings |
| `Debug` | Logger + overlay (stripped in release) |

## 5. Player controller and abilities
- `Player` (CharacterBody2D) + `StateMachine` node with state nodes: Idle, Run, Jump, Fall, Dash, Hurt, Interact, AbilityActive.
- `HeroData` Resource: speed, jump, ability scene, color, voice id.
- `AbilityBase` (Node): `can_activate()`, `activate()`, `tick(delta)`, `cooldown_left`. Subclasses: `BlitzDash`, `NovaFlight`, `GrimShield`, `EchoVeil`, `VoltHack`. Purpose of the base class: uniform HUD, trace-replay hooks (`replay_event(e)`), and tests.
- **Hero swap:** `HeroController` swaps `HeroData` + ability; cooldown timers live on per-hero ability instances kept alive.

## 6. Physics and collision
- Layers (named in project settings): World, Player, OneWay, Hazard, Interactable, EnemySensor, TraceSensor, Hidden, Static.
- Player moves with `move_and_slide()` in `_physics_process`.
- Hidden platforms toggle collision layer, not node visibility.
- Moving platforms: `AnimatableBody2D`, driven by a deterministic clock `RoomClock` (ticks since room start) so platform position is a pure function of tick. `[PROTOTYPE]`

## 7. Echo Trace architecture `[PROTOTYPE]`
```
TraceRecorder (Node, child of Player)
  - buffer: TraceBuffer (PackedVector2Array pos, PackedByteArray facing/anim/hero, PackedInt32Array events)
  - start(), stop() -> TraceData, cancel()
TraceData (Resource): origin, duration_ticks, arrays, hero_id
TracePlayer (Node in room): spawn(TraceData) -> TraceGhost; manages slots, dissolve, hold
TraceGhost (Node2D with Area2D "TraceSensor"): set position from buffer[tick]; dispatch events
```
- **Determinism design:** record in `_physics_process` at the 60 Hz tick; playback also in `_physics_process`; all event dispatch keyed by integer tick, never wall-clock; no floating accumulators.
- **Triggers:** ghost has an Area2D that can enter plate areas; plates count both Player bodies and TraceGhost via a `weight_provider` group (not physics bodies), to avoid solid collisions.
- **Replaying abilities:** each `AbilityBase` implements `replay_event(event, ghost)`; unsupported = skipped (F-T15).
- **Memory:** 8 s × 60 = 480 ticks per trace; data is tiny.
- **Risks:** moving platforms, changing physics zones (gravity), hit-stop, swap during playback, scene hitches that desync ticks (mitigated by tick-based timeline), Veil remote reveal.

## 8. Puzzle and environment state
- Elements implement `Resettable`: `reset()`, `snapshot()`, `restore(s)`.
- `PuzzleController` per puzzle: collects element signals, evaluates solution, emits `solved`. Persisted by id in `GameState`.
- Dynamic corridors: `SeamController` toggles `TileMapLayer` groups and collision; emits `layout_changed` (TracePlayer listens to dissolve).
- Gravity zones: `GravityZone` (Area2D) sets a gravity override on bodies; player reads `effective_gravity`.

## 9. Enemy AI and boss state machines
- `StateMachine` (generic, ~60 lines) + state nodes; no behavior trees.
- Perception: `Perception` component with `see(cone)`, `hear(NoiseBus)`, `sense_ground()`; sees TraceGhost per flag.
- Bosses: `BossBase` with `PhaseData` Resource (list of phases: states, conditions). Boss scripts keep decisions in `decide()` so tests can drive them with mock time.
- `Replayer` component (shared by Corruptor Shade and Reflection): consumes a TraceData-like buffer.

## 10. Signals and reusable components
Components: `Hurtbox`, `Hitbox`, `Interactable`, `WeightSensor`, `Resettable`, `NoiseEmitter`, `Telegraph` (draws warning shapes). Rule: components expose signals and a minimal API; scenes wire them.

## 11. Save/load and checkpoints
- JSON at `user://saves/slot_N.json` through `FileAccess`; write to temp then rename; `schema_version` and a migration table.
- Contents: chapter, beacon id, heroes, solved ids, fragments, clauses, ledger summary, settings pointer.
- Beacon: `Beacon` node triggers `SaveService.autosave()`; respawn restores Integrity and calls `RoomController.reset_room()`.
- Corrupt/missing file: load backup (`.bak`), else report in UI and offer new game; never crash.

## 12. Input abstraction
Use Godot's InputMap with named actions (`move_left`, `jump`, `ability`, `pulse`, `interact`, `trace`, `swap_next` …). A thin `InputService` handles hold/toggle modes and buffering (jump buffer). Remapping saved in `Settings`. Co-op later: actions suffixed per device index; do not hard-code single-player globals in gameplay scripts (pass an `input_source`).

## 13. Audio and VFX
- Audio buses: Master, Music, SFX, Ambience, UI. `AudioDirector` plays cues by id from a data table; music layers cross-fade by state (explore/tension/boss).
- VFX: shaders for glitch, trace outline, fog; all gated by `Settings.reduced_effects`.

## 14. UI architecture
Control scenes with theme resource; HUD reads state through `Events` signals, never polls gameplay nodes. Dialogue system: JSON/`.tres` lines with speaker, text, duration, priority; non-blocking by default.

## 15. Logging and debug tools
`Debug.log(category, msg)`; categories: trace, input, puzzle, ai, save. Debug overlay: tick counter, trace buffer length, noise meter, room state. Debug actions: skip room, set hero, force Beacon, replay seed. Stripped from release builds.

## 16. Performance considerations `[VERIFY]`
Assumptions: 60 FPS target at 1080p on a mid-range desktop; min spec TBD. Keep particle counts bounded, avoid per-frame allocations in `_physics_process`, pool ghosts/VFX, use atlases, limit simultaneous shaders. Profile with the built-in profiler at each milestone; record numbers in `15`.

## 17. Error handling
Fail soft: invalid references logged and skipped; assertions in debug builds; exported builds log to `user://logs/`. Trace replay events skip with a cue, never crash.

## 18. Testing strategy
Unit tests for pure logic (TraceBuffer, noise meter, cooldown, save serialization); integration tests for rooms (headless scene run with scripted inputs). Test framework: GUT or gdUnit4 `[VERIFY compatibility with the pinned Godot version]`. Determinism test: record scripted inputs → replay → compare positions per tick (see `15`).

## 19. Export and build
Export presets for Windows and Linux first; macOS later (code signing/notarization needed) `[VERIFY]`. Version string from a `version.txt`; build script in `tools/` runs tests, exports, packages licenses and credits. CI optional; manual checklist in `13`.

## 20. Technical risks
| # | Risk | Mitigation |
|---|---|---|
| T1 | Trace determinism | Tick-based timeline; golden-run tests; Prototype M3 |
| T2 | Moving platforms | RoomClock; avoid in slice |
| T3 | Collision edge cases | Kinematic ghost; sensor-based triggers |
| T4 | Changing physics (gravity zones) | Record world positions; zones don't re-simulate |
| T5 | Hero swap and ability state | Per-hero ability instances; events keyed by hero id |
| T6 | Reflection ledger fairness | Bounded counters + authored fallback |
| T7 | Save corruption | Temp+rename, backup, versioned schema |
