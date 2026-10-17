# 17 — Decision Log and Open Questions

Status: **Decided** (in force), **Proposed** (needs confirmation), **Open**. IDs referenced across docs.

## Decisions

| ID | Topic | Decision | Reasoning | Alternatives | Consequences | Status |
|---|---|---|---|---|---|---|
| D-001 | Ownership designation | Saugat Rauniyar = designated project lead; not a statement of verified IP ownership | Brief instruction; provenance unresolved | Silent assumption of full ownership | `12` §6 tracks Qs; credits separate current/historical | Decided |
| D-002 | Engine/language | Godot 4.x (pinned at M1), GDScript | Brief default; small team fit | Unity, custom | Pin version; upgrade between milestones | Decided |
| D-003 | "Echo" naming | Hero = **Echo**; ability = **Veil Sight**; mechanic = **Echo Trace**; boss = **Titan Echo**; Warden = **Prime Echo** | Reduce confusion; retain source names | Rename hero | UI uses "Veil Sight", "Trace" | Decided |
| D-004 | Party model | One hero in world; unlock Ch1 Blitz+Echo, Ch2 Nova+Volt, Ch3 Grim; dormant heroes speak via Tether | Teach abilities gradually; single-player | All five from start; AI companions | Chapter design uses unlocked sets | Proposed |
| D-005 | Trace model | Recorded-transform playback with tick events; traces affect sensors, not hazards | Determinism; testability | Input replay; physics resim | Moving-platform risk; hazards ignored by traces | Decided `[PROTOTYPE]` |
| D-006 | Shifting corridors | Seam-triggered layout states; traces dissolve on change | Single-player adaptation of "players split" | Distance-based tether | Clear visual states; no dead ends | Decided |
| D-007 | Blitz ability | Dash with escalating cooldown (map.docx) over 30 s speed boost (story doc) | Fits puzzle platforming | Speed boost | `00B` CH-01/02 | Decided |
| D-008 | Cooldowns | Short cooldowns (seconds), not minutes | Puzzle pacing | "Once every few minutes" | Abilities used frequently | Decided |
| D-009 | Noise | Visual Noise Meter driven by game events; no mic | Accessibility; brief rule | Mic input | `04` §11 | Decided |
| D-010 | When Shards are found | Shards I–IV at the end of Ch1–4; Shard V in Ch5 Act 3 | Resolve source contradiction | All at Ch5 | Chapter exits tied to Shards | Decided |
| D-011 | Shards ↔ pillars | One-to-one; pillar N trial recombines chapter N's rule; Pillar V (Volt) last | Clean structure | Free mapping | `09` | Decided |
| D-012 | Hero ↔ Prime Echo | Heroes are the Final Draft's five strategies, independent after running | Fits twist | Fragments; clones | Revealed Ch4 | Decided |
| D-013 | Trace lifetime | Play once, hold TR-03, dissolve; Loop Resonator for looping | Predictable; simple | Always loop | Plate holds readable | Decided |
| D-014 | Corruptor weakness | Pylon sync with Sync Window; trace replaces second player | Preserves "dual attack" | HP bar | `03` C1 | Decided |
| D-015 | Combat model | Non-lethal Pulse + Release; bosses puzzle-resolved | Theme and scope | Lethal combat | No weapons/loot | Proposed |
| D-016 | Stormhawk | Preserved flight-record of pilot Kesh Arandor `[PROPOSAL]` | Original "old friend" retained | Generic AI | Release request found Ch2 | Proposed |
| D-017 | Titan Echo | Guardian of the Vault; reacts to noise; not born of the hero's fear | Resolve source conflict | Echo-fear monster | Fear theme still present | Decided |
| D-018 | Reflection | Division Log self-image of Prime Echo; non-lethal; approach variety | "Mirror Prime Echo" retained | Pure combat | Behavior Ledger | Decided |
| D-019 | Voidcaller | Boundary Keeper protecting Release Seals | Brief | Corrupted mind | Mid-boss framing; choice ending | Decided |
| D-020 | Pillar defense | Five pillar trials + 90 s Stability Hold (tunable) | Single-player puzzle fit | 3-min wave defense | `09` | Proposed `[PROTOTYPE]` |
| D-021 | Ending C gating | Five Witness Clauses + Protocol Assembly; chambers revisitable | Not a trivial count | Collectible total | No missable gate | Decided |
| D-022 | Slice scope | Blitz + Trace + Pulse + Sentry + Seam + Corruptor P1–P2; Echo Veil stretch | Validates fun | Include swap | `13` M5 | Decided |
| D-023 | Co-op | Deferred; no co-op dependencies | Brief | Include | `input_source` abstraction | Decided |
| D-024 | Reboot Memory | Replaced by post-MVP "New Cycle" | Scope | Replay button now | `02` §16 | Decided |
| D-025 | Health/death | 3 pips; Beacon respawn; no game over | Failure cheap | Hardcore | `02` §8 | Proposed |
| D-026 | Behavior Ledger | Local, bounded, optional, authored fallback | Privacy and fairness | Raw input recording | Toggle in Settings | Proposed |
| D-027 | Opening line | "This isn't the beginning. It's the return." reused as opening | Preserve source | Drop | Foreshadows Draft | Decided |
| D-028 | Rewards | No gold/gear/XP | Theme | Currency | Memory Fragments/Clauses | Decided |
| D-029 | Asset policy | Original or verified-license only; evidence stored | Licensing risk | Use blanket claim | `12` | Decided |
| D-030 | Units | Use BH until M2 fixes scale | Avoid arbitrary px | Fixed px now | Close at BL-006 | Decided |

## Original contradictions explicitly resolved
| Contradiction | Resolved by |
|---|---|
| When Shards are found | D-010 |
| How Nexus pillars relate to Shards | D-011 |
| How each hero connects to Prime Echo | D-012 |
| How the final Voidcaller encounter works | D-019, `03` C5, `09` |
| Titan Echo origin | D-017 |
| Blitz ability | D-007 |

## Parked non-canon ideas (from the source expansion hooks)
Ice realm, underwater city, "digital hell", PVP arena, betrayal among the five. Not planned; revisit only after release.

## Open questions
| ID | Question | Needed by |
|---|---|---|
| Q-001…Q-010 | Provenance questions in `12` §6 | M1 |
| Q-011 | Confirm Stormhawk pilot name/backstory | M7 |
| Q-012 | Do we want voiced lines or text only? | M5 |
| Q-013 | Final Ending-C clause order and puzzle design | M8 |
| Q-014 | Minimum PC spec | M5 |
| Q-015 | Release platforms/storefront | M9 |
