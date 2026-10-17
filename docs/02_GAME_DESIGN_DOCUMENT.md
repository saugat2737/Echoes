# 02 — Game Design Document

Terms and numeric defaults: `00B`. Mechanic rules in detail: `04_MECHANICS_AND_PUZZLES.md`.

## 1. Overview
2D side-scrolling puzzle-platformer. The player controls one hero at a time through five chapters, solving environmental puzzles using movement, hero abilities, and **Echo Trace**, with occasional non-lethal encounters. Offline, single-player, desktop. Each chapter = one theme, one spotlight hero, one boss, one new rule family.

## 2. Design pillars
1. **The past is interactive.** Echo Trace is in every chapter's critical path.
2. **Reason, don't grind.** Puzzles are solved by understanding rules, never by trial-and-error of hidden states.
3. **Abilities are verbs with limits.** Each has a distinct use and a meaningful cost.
4. **The world tells the story.** Mystery is environmental; the immediate goal is always legible.
5. **Failure is cheap.** Retry within seconds; no long walkbacks.

## 3. Main gameplay loop
```
Enter room → Read the rule (visual language) → Plan (movement + abilities + trace)
   → Execute → Feedback (success / readable failure) → Reset in <3 s on failure
   → Progress → Beacon → Story beat (environment-first)
```
Chapter loop: explore hub rooms → solve Gate puzzles → boss/Shard → revelation → next chapter. Optional: Witness Chamber and Memory Fragments.

## 4. Gameplay systems

### 4.1 Exploration
Linear chapters with small branching side rooms. Fog-of-war is a visibility effect, not a map system; Echo's Veil Sight clears it locally. Backtracking is rare and always short (≤ 1 room) except optional Witness Chambers. `[PROPOSAL]`

### 4.2 Platforming
Responsive, forgiving: coyote time, jump buffering, variable jump. See §6.

### 4.3 Puzzle-solving
Every puzzle teaches, tests, or combines a rule (see `04` fairness rules). Puzzle state resets via the room reset contract; solved puzzles stay solved after leaving the room.

### 4.4 Combat (selective action) `[PROPOSAL]` (D-015)
No lethal combat. The player has **Pulse** (short-range stun). Stunned enemies can be **Released** (interact), dissolving them. Bosses are not damaged by Pulse; they are resolved through puzzle conditions. Enemies are mostly environmental problems to bait, hack, or evade.

### 4.5 Narrative progression
Revelation schedule in `01` §9. Dormant heroes' voices comment during play via short lines (≤ 3 s, non-blocking).

## 5. Controls and input mapping `[PROPOSAL]`

| Action | Keyboard | Controller |
|---|---|---|
| Move | A/D or ←/→ | Left stick / D-pad |
| Jump | Space | A |
| Ability (current hero) | Left Shift | B |
| Pulse | J | X |
| Interact | F | Y |
| Echo Trace (record / launch / hold to cancel) | E | Right trigger |
| Hero swap next / previous | Tab / Q | RB / LB |
| Direct hero select | 1–5 | Hold left trigger (radial) |
| Hint | H | Back/View |
| Pause | Esc | Start |

All actions are remappable; hold vs toggle options for ability and trace (see §12). No input requires a microphone or a mouse.

## 6. Player movement and physics requirements

| Requirement | Start | Notes |
|---|---|---|
| Max run speed | ~6 BH/s | Tunable |
| Jump apex | ~2.2 BH | Variable by hold; tunable |
| Coyote time / jump buffer | 0.10 s / 0.10 s | Tunable |
| Gravity | Single fixed scale per zone; zone overrides for Ch2 | |
| Air control | High (≥ 70% ground accel) | |
| One-way platforms | Drop-through with Down + Jump | |
| Moving platforms | Carry the player; traces store world-space | `[PROTOTYPE]` see `10` |
| Physics tick | 60 Hz fixed | Required by Trace determinism |

Rules: movement must not depend on frame rate; the same inputs on the same tick must yield the same position (see `04`, `15`).

## 7. Characters, switching, cooldowns
- One hero active in the world; others are "resting" on the Tether.
- Swap takes 0.2 s animation, then CH-08 cooldown.
- Each hero has an independent ability cooldown that keeps ticking while resting.
- A trace stores *which hero* it records and replays that hero's abilities; so you can record Blitz and then play as Nova.
- Locked heroes are unavailable until the unlock points in `00B` §3.

| Hero | Use | Limit |
|---|---|---|
| Blitz | Cross gaps, pass Pulse Traps, beat timers | Escalating cooldown; collides with walls |
| Nova | Vertical reach, over ground hazards/EMP | Energy meter; no flight in Static Fields |
| Grim | Block projectiles; muffle noise | Slow while shielded; cooldown long |
| Echo | Reveal hidden platforms/paths | Reveals are temporary; radius-limited |
| Volt | Hack machines/turrets/jet parts | Range + line of sight; not on organic things; not in Static Fields |

## 8. Health, damage, checkpoints, death `[PROPOSAL]` (D-025)

| Event | Result |
|---|---|
| Hazard/enemy contact | −1 Integrity, 1.5 s invulnerable |
| Pit fall | −1 Integrity, return to last Safe Ledge |
| Integrity 0 | Respawn at last Beacon with full Integrity; room reset |
| Boss defeat condition not met | No game-over screen; reset phase from boss Beacon |
| Puzzle failure (non-lethal) | Reset button (hold) resets puzzle state only, <1 s |
| Beacon | Autosave + resets Integrity |

Never lose more than ~60 seconds of non-trivial progress. Beacons sit before each major puzzle and boss phase.

## 9. Collectibles and progression
- **Memory Fragments** (lore, optional).
- **Witness Clauses** (one per chapter, ending-gating, via Witness Chambers).
- **Shards** (story-critical).
- No currency, gear, or XP. Hero unlocks are story-driven.

## 10. Save system `[PROPOSAL]`
Three manual slots, autosave at Beacons, save data = chapter, Beacon id, unlocked heroes, solved puzzle ids, collected fragment ids, Witness Clauses, settings, Behavior Ledger summary. Versioned; local only. Spec in `10`.

## 11. UI and HUD
Minimal HUD: Integrity pips, current hero + ability cooldown ring, Trace state indicator (idle / recording with time ring / playing / holding), Noise Meter (Ch3, and Ch3-rule rooms), hint indicator. Dialogue as small bubbles. No minimap. Details in `11`.

## 12. Accessibility `[PROPOSAL]`
| Area | Requirement |
|---|---|
| Input | Full remapping; hold/toggle for ability, trace, shield; no mic; one-handed preset |
| Visual | Colorblind-safe palette variants; high-contrast mode; scalable UI text; reduced glitch/flash; photosensitivity-safe effects (no flashes above 3 per second) |
| Audio | Every audio cue has a visual cue and vice versa; separate sliders |
| Cognitive | 3-tier hints; puzzle goals always shown; skip-trace-timer assist |
| Motor | Adjustable Sync Window, Trace length, noise sensitivity; invulnerability option; slow-motion option for boss telegraphs |
| Presets | Story / Standard / Challenge built on individual toggles |

## 13. Difficulty progression
| Chapter | New rule | Trace slots | Notes |
|---|---|---|---|
| 1 | Trace, seams, sentries | 1 | Teaches every Trace rule individually |
| 2 | Gravity zones, jet assembly | 1 | Hero swap pairing (Nova + Volt) |
| 3 | Noise meter | 1 | Traces as noise decoys |
| 4 | Loops, adaptive clones | 2 | Rooms demand varied solutions |
| 5 | Combination, Fold | 2 | Little new; all rules recombined |

## 14. Level completion conditions
Room complete = exit gate open; chapter complete = boss resolved + Shard collected. Details per chapter in `05`–`09`.

## 15. Endings and replayability
Three endings (see `01` §11). Replay: chapter select after Ch5; New Cycle (post-MVP): Trace-trial remix and a harder-timing variant. No NG+ required for the best ending.

## 16. MVP vs post-MVP

| Feature | Slice (M5) | Full MVP | Post-MVP |
|---|---|---|---|
| Movement/jump | ✔ | ✔ | |
| Blitz dash | ✔ | ✔ | |
| Echo Trace (1 slot) | ✔ | ✔ | |
| Echo Veil Sight | stretch | ✔ | |
| Hero switching | | ✔ | |
| Nova, Volt, Grim | | ✔ | |
| 2 trace slots | | ✔ | |
| Corruptor prototype | ✔ | ✔ (full) | |
| Chapters 2–5 | | ✔ | |
| Witness Chambers/Ending C | | ✔ | |
| New Cycle mode | | | ✔ |
| Co-op | | | ✔ (architecture only) |
| Fragment gallery | | stretch | ✔ |
| Console ports | | | ✔ |
