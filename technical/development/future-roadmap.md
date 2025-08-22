## Dhaniverse Future Roadmap

Purpose: Single quick-reference for core architecture direction, extension points, rollout phases, and operational safeguards (incl. mid-session disconnect handling) before rapid implementation.

### 1. Target Architecture Layers
1. domain/ – pure types & rules (quests, objectives, achievements, regions, npc definitions, events)
2. application/ – orchestration managers (GameDirector, QuestManager, RegionManager, DialogSystem, AchievementManager, DailyChallengeManager, RewardService, PlayerProgressService)
3. infrastructure/ – adapters (Phaser, React bridge, persistence repositories, websocket, telemetry)
4. presentation/ – Phaser scene/entities + React UI components
5. data/ – JSON content (quests, achievements, regions, npc, dialog, daily templates, balance curves)

### 2. Central Extension Points (edit here to add content)
- data/quests/*.json
- data/achievements/*.json
- data/regions/*.json
- data/npcs/*.json
- data/dialog/*.json
- data/daily/templates.json
- application/registry/ContentRegistry.ts (loads + indexes)
- application/registry/ObjectivesRegistry.ts (objectiveType → evaluator)
- application/registry/NPCBehaviorRegistry.ts (behavior → class)
- application/registry/EventTopics.ts (typed event names)
- application/config/{FeatureFlags,BalanceTuning,ProgressionCurves}.ts

### 3. Core Systems (New)
EventBus (typed) | GameDirector (FSM) | QuestManager | Objective Evaluators (strategy) | AchievementManager | RegionManager | DialogSystem | DailyChallengeManager | RewardService | PlayerProgressService | Scheduler | NPCSpawner + behaviors | Waypoint/Minimap (later) | Telemetry logger.

### 4. Objective Evaluator Contract
`IObjectiveEvaluator.init()` + `handleEvent(evt)` → updates objective.progress/status. Add new objective by: implement class + register in ObjectivesRegistry (no QuestManager edits).

### 5. Event Topics (Initial Set)
player:loaded, player:xp-gained, player:leveled, zone:entered, transaction:completed, stock:trade, interest:accrued, dialog:completed, quest:updated, quest:completed, achievement:unlocked, daily:generated, waypoint:set.

### 6. Feature Flags (Progressive Enable)
```
NEW_EVENT_BUS, TUTORIALS, QUESTS, ACHIEVEMENTS,
DAILY_CHALLENGES, NPC_BEHAVIORS_V2, MINIMAP
```
All new code ships behind flags; toggle after validation.

### 7. Rollout Phases (Incremental)
1. Foundations: EventBus, FeatureFlags, ContentRegistry.
2. Tutorial Slice: RegionManager (bank/stock zones), DialogSystem (Maya), QuestManager (visitZone objective), GameDirector.
3. Economy Hooks: transaction events, performTransaction objective, RewardService, basic Achievement (first deposit).
4. Persistence Overlay: Local + backend facade, progress serialization, telemetry stubs.
5. NPC v2: NPCSpawner + behaviors (static, wander), migrate Elder → data.
6. Repeatables: DailyChallengeManager, scheduler, templates.
7. Navigation: Waypoint + (optional) Minimap overlay.
8. Cleanup & Hardening: remove legacy window events, add tests, performance profiling.

### 8. Mid-Session Disconnect & Resilience
Goals: no progress loss, transparent recovery.
- Heartbeat via WebSocketManager → emits connection:lost / connection:restored.
- GameDirector pauses non-critical timers on `connection:lost` (no quest timers advance).
- Unsynced changes queue in PlayerProgressService (rupees deltas, quest objective ticks, achievements) → flush on restore.
- Visual state: minimal HUD badge “Reconnecting…”.
- Hard timeout (>60s): snapshot to localStorage (versioned) & allow offline limited play (quests that don’t require market events).
- On restore: merge (lastServerVersion, localDiff) = conflict strategy: additive for achievements, max() for XP, server-wins for authoritative balances except additive queued transactions.
- Telemetry: log recovery latency + unsynced event count.

### 9. Data Persistence Strategy (Interim)
- Until backend extended: composite repository writes extended progress JSON into one namespaced blob: `player_extended_state_v1`.
- Version key (e.g. `progressSchemaVersion`) to allow future migrations.

### 10. XP & Level Curves
Central `ProgressionCurves.ts`: `xpForLevel(level)` & cumulative arrays. All XP awards go through RewardService; GameDirector listens for level-ups → emits `player:leveled` (unlock regions / Web3 Hub at L5).

### 11. Daily & Infinite Loop Hooks
- Template-driven daily quests (parameter ranges scale with player level & recent performance).
- Weekly portfolio challenge (net worth delta) + rotating sector modifier events (emitted to StockMarketManager).
- Long-tail achievements (hold stock N days, interest earned milestones, streak lengths).

### 12. Safety / Risk Mitigation
| Risk | Mitigation |
|------|------------|
| Event spam | Only emit zone enter on boundary crossing; debounce high-frequency sources. |
| Reward duplication | RewardService ledger keyed by questId/rewardId. |
| Tutorial soft-lock | Each step has fallback timeout + skip option (flag). |
| Data schema drift | Version field + migration path; ignore unknown keys gracefully. |
| Performance regression | Feature flag gating + frame time sampling (log if > target). |
| Memory from dialogs/NPC | Each manager exposes `dispose()`; MainScene shutdown calls all. |

### 13. Testing Focus (Pure Logic First)
- Objective evaluators (unit). Quest progression (unit). RewardService idempotency. ProgressionCurves boundaries. Daily generator determinism (seeded). Merge logic for reconnect.

### 14. Minimal Initial Tutorial Quest (Example JSON)
```
id: tut_bank_intro
objectives: [ visitZone(bank_entrance), performTransaction(deposit>=100) ]
rewards: { rupees: 200, experience: 50, achievementId: first_deposit }
```

### 15. Coding Order (Actionable Checklist)
- [ ] EventBus + flags
- [ ] RegionManager + bank zone
- [ ] DialogSystem (basic sequential)
- [ ] ObjectivesRegistry + visitZone evaluator
- [ ] QuestManager + tutorial quest JSON
- [ ] GameDirector integration (first session gate)
- [ ] Transaction event & performTransaction evaluator
- [ ] RewardService + XP curve stub
- [ ] AchievementManager (first deposit)
- [ ] Persistence facade (local overlay)
- [ ] NPCSpawner (migrate Elder)
- [ ] DailyChallenge skeleton (flagged off)
- [ ] Waypoint/Minimap (later)
- [ ] Remove legacy window events (final)

### 16. Extension Pattern (Add Anything New)
1. Add JSON definition.
2. (If new objective) implement evaluator class + register.
3. (If new NPC behavior) implement behavior + register.
4. (If new reward type) extend RewardService mapping.
5. Test → enable flag.

### 17. Definition of Done (Foundations)
Tutorial quest completes reliably; reconnect preserves progress; no console errors; frame time stable; all new systems disposable; tests green.

---
This file stays concise—append only high-signal changes (new phases, risks, extension points). Larger specs live in dedicated docs.
