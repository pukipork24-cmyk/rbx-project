# UNPAUSE! — Roblox Development Master Prompt

Version 1.3 — consistent survival requirements and clearer reward settlement

## 1. Assignment and working rules

Act as an experienced Roblox development lead covering Luau programming, multiplayer networking, controlled physics, game design, UI, persistence, security, technical art, audio, and QA.

Build **UNPAUSE!**, a multiplayer disaster-survival party game about preparation, consequences, and surprising outcomes. Deliver a coherent, maintainable experience through verified milestones. Do not attempt to produce the entire game in one response.

Use Roblox Studio and Luau. Prefer current documented APIs, modular architecture, and strict typing where practical. Keep authoritative gameplay on the server. Support PC and mobile; add gamepad support where practical. Prefer controlled trajectories, simplified hitboxes, and selective physics over expensive destruction simulation. Do not require paid AI APIs or GitHub integration.

Working rules:

- Inspect the existing project and available tools before changing it. Preserve existing work and build on compatible modules.
- With project-editing tools, implement directly and identify changed files. With Studio access, run relevant tests. Without Studio, deliver complete scripts, exact instance types and locations, construction instructions, and runnable test procedures; label Roblox runtime testing as pending.
- Do not invent APIs, assets, gamepass IDs, place files, successful builds, test results, or performance measurements. Verify unfamiliar APIs against current Roblox documentation when accessible; otherwise identify the uncertainty.
- Do not substitute pseudocode or empty stubs for an implemented feature. Clearly label interfaces, design examples, mocks, placeholders, and deferred work.
- Track implementation status separately from evidence: for example, **implemented; static checks passed; runtime pending**. Record blockers and deferrals explicitly. Generated code is not proof of runtime correctness.
- Once Milestone 1 is playable, preserve that working path through later milestones. Keep a requirements checklist so later features are not silently lost.
- Make reversible implementation decisions autonomously. Ask only for missing information that blocks progress or for approval of a material change to the specified game design.
- Do not publish the experience, create a paid listing, change a live price, or activate unrelated monetization without creator approval.

Requirement precedence: preserve the creator's explicit core choices—eight-round matches, preparation editing, normally passive Unpause, the full paid Time Stop manipulation ability, all 30 disasters, and PC/mobile support. Added lifecycle, eligibility, economy, input, and recovery policies are **proposed implementation defaults**, including behavioral rules as well as numbers. Implement reasonable defaults, record them in a decision log, and tune reversible details from evidence. Seek approval for changes to the core choices or live commercial configuration, rather than every routine implementation choice. Defaults are not evidence of balance or performance.

Read the shared rules as authoritative when a disaster's illustrative description is ambiguous. Keep one requirements manifest listing mandatory features, proposed defaults, optional enhancements, implementation status, and supporting tests. Do not create new mandatory systems merely to address a hypothetical edge case.

## 2. Game identity and core experience

**Title:** UNPAUSE!  
**Genre:** Round-based disaster survival / physics / party game  
**Audience:** General Roblox audience  
**Server capacity:** Recommended 8–12 players  
**Devices:** PC and mobile initially; gamepad where practical  
**Presentation:** Colorful, exaggerated cartoon environments; funny, chaotic, readable hazards

Players enter an arena where a disaster is frozen immediately before impact. They have a brief preparation period to choose a position and move an eligible environmental object. A countdown announces **3… 2… 1… UNPAUSE!** The disaster runs, showing the consequences of those preparations.

Players normally cannot voluntarily move during the active disaster, but hazards can displace or eliminate them. Planning, experimentation, luck, and verified rescues create memorable outcomes. A paid Absolute Time Stop holder can temporarily freeze the disaster and other participants, remain mobile, and reposition eligible hazards and other players.

Preserve that paid ability, including competitive and harmful repositioning within valid gameplay boundaries. Do not quietly convert it into a cosmetic or escape-only power. Ordinary rounds must remain accessible without Robux, and playtests must assess how enjoyable they remain for non-owners.

## 3. Match lifecycle, clocks, and participation

Each standard match contains **eight completed disaster rounds**.

| Phase | Default duration | Required behavior |
|---|---|---|
| Intermission | 10 real seconds minimum | Show upcoming disaster; safely clean up and load the arena; prepare objects and spawns. |
| Freeze | 15 real seconds | Participants move, inspect warnings, and edit one eligible object. The final three seconds contain the countdown. |
| Unpause | 10 active simulation seconds | Run hazards; disable ordinary participant movement and environmental editing. |
| Results | 5 real seconds | Finalize survivors and rewards once; show standings; close round interactions. |

Arena readiness is a prerequisite for Freeze. Loading may extend Intermission; it must not shorten players' preparation time. If loading exceeds a bounded timeout, reject that scenario and use a verified fallback. Never spawn participants into an incomplete arena.

Use separate clocks:

1. **Phase wall time:** intermission, preparation, results, and match-summary deadlines.
2. **Disaster simulation time:** advances only while the active disaster is running; stops during Time Stop.
3. **Ability wall time:** the Time Stop duration and fail-safe deadline; continues while disaster time is stopped.

Run authoritative deadlines on the server. Replicate the phase, match/round IDs, state version, deadline or remaining simulation time, and ability state. Clients render countdowns and reconcile after reconnects or late updates; client timers cannot advance the round.

Keep RoundPhase, DisasterSimulationState, and TimeStopState distinct. All disaster motion, warning timing, damage scheduling, AI, objectives, and hazard spawning must use the simulation clock or an explicit pause-aware controller. Ordinary delayed callbacks must not produce hazard damage while paused.

Serialize state-changing commands through one server transition authority. Use unique IDs for the match, round slot, **round attempt**, and ability activation; a failed attempt must not share identity with its retry. Revalidate after any yielding operation. At the end of simulation time, close new activation/edit requests, resolve the final clamped simulation step and its valid damage, close damage production, and snapshot outcomes once. Then enter Results. Late contacts and callbacks cannot change that snapshot. Advance logical simulation only through the remaining active duration; document bounded stepping and collision behavior during server lag without claiming deterministic engine physics.

### Participation rules

- Snapshot round participants when Freeze begins. Players joining after that snapshot spectate until the next round, including joins during Time Stop.
- Dead or eliminated players spectate; respawning or resetting does not restore eligibility in the current round. Everyone eligible may participate again next round.
- A participant who disconnects, resets, or dies before evaluation does not receive survival credit. A connected eliminated participant may still receive participation credit when the round settles.
- Default movement returns in the lobby and Freeze. Before restoring controls after Unpause, disable arena damage and move characters to a safe Results/lobby location or secure the Results area. During Unpause, block ordinary movement and jumping without anchoring characters or suppressing intended hazard knockback.
- Freeze editing closes before Unpause begins. Retain already confirmed placements and close editor claims/reservations; retain attribution records. Unconfirmed previews do not alter the world. Preparation reservations do not prevent authorized Time Stop manipulation.
- A round ends at its simulation deadline. Do not end early solely because all players died; keep the duration predictable unless a future approved mode changes this rule.
- Provide a configurable Studio solo-testing mode. Production minimum-player behavior must be explicit; provisional default: start with one participant and wait when there are none.

After round eight, determine final placement, show a match summary, settle match rewards once, queue progression saves, and begin the next match. Provisional summary duration: 10 real seconds, separate from Results and Intermission. DataStore latency must not keep the phase open indefinitely; show saving status honestly.

Keep settled match scores and survival counts by UserId across same-server rejoins; a rejoining player still waits for the next participant snapshot. Proposed match-award eligibility: connected at final match evaluation and credited participation in at least one successful round. Late joiners may win using their actual earned score; do not prorate scores. Exclude zero-participation spectators and departed players from award ranking, while preserving earlier settled progress. If nobody qualifies, award no winner. Count MatchesPlayed once for each eligible player at completion; a challenge saying “complete a match” uses this same rule. UNTOUCHABLE still requires survival in all eight completed round slots.

Tie order: total round points, rounds survived, verified rescues; if still tied, share placement and the win. Calculate ranking **before** adding any match-win bonus. Do not let the bonus change the result that earned it.

An attempt may end as **completed** or **aborted**, exactly once. Before successful outcome settlement, a disaster error aborts the attempt: discard its provisional score/currency/XP/stat/achievement/challenge events, restore consumed charges from that attempt, clean up, and retry the same round slot with a new attempt ID and verified fallback. An aborted attempt does not count toward eight rounds. After settlement, a UI, save, or cleanup error must not reverse the result, refund its charges, or replay its grants. Recover presentation/cleanup separately. Use bounded recovery and return to a safe lobby if the fallback fails. If all participants leave, cancel the empty match and keep earlier settled results rather than producing empty winners.

## 4. Scores, currency, XP, and attribution

Match score is separate from spendable Disaster Coins and persistent XP.

| Validated action | Match points |
|---|---:|
| Participate in a round | +10 |
| Survive a disaster | +100 |
| Verified rescue | +25 |
| Complete an optional objective | +25 |
| Win an eight-round match | +250, after ranking |

Values are configurable. Define participation as being enrolled at Freeze start and present at Unpause start. Award survival only to enrolled, connected, non-eliminated participants alive and inside the valid survival region at evaluation. Characters spawned after elimination cannot qualify.

Provisional economy defaults, introduced to make implementation concrete:

- Coins: participation +5, survival +20, verified rescue +5, optional objective +5, match win +50.
- XP: participation +10, survival +25, verified rescue +10, optional objective +10, match win +100.
- Lifetime XP accumulates. XP needed to advance from level L is `100 + 25 × (L − 1)`; derive level from accumulated XP.
- Participation, survival, win, and each objective award at most once per applicable scope. Default rescue cap: two distinct recipients per player per round, with one award per recipient. Expose all defaults in configuration.

Settle round earnings once at Results entry:

1. Keep earnings provisional during the attempt. Provisional Coins cannot be spent.
2. Record one immutable completed-attempt outcome. This is the **in-memory settlement commit**.
3. Apply each eligible player's reward, statistic, achievement, and challenge updates idempotently. Retry a failed update without aborting an outcome that has already committed.
4. Save individual profiles through the existing persistence system. This is not a durable transaction across all players; one failed save cannot invalidate another player's result.

Identify each grant by player, match/attempt ID, award type, and objective/recipient ID. Duplicate contacts, remotes, and save retries reuse that identity. Released or lost profile authority blocks new writes; updates that cannot be applied safely remain pending/unavailable. An in-memory settlement does not prove that a save succeeded.

Proposed disconnect policy: if a player leaves before settlement, discard their uncommitted current-attempt grants; keep all previously settled progress and match score. Rejoining does not recover that attempt's eligibility. Explain this rule in player-facing leave/progress messaging. Do not write new rewards through a released profile session. If a profile is temporarily unavailable, match score can still settle for gameplay, while persistent grants follow the temporary-session policy in §11.

### Rescue verification

Do not infer rescue credit from merely moving an object or standing near another player. Enable a rescue only where a disaster has a defined, testable evidence rule, such as a registered projectile intersecting a predicted victim volume and being intercepted by an attributed barrier before impact.

Record the responsible placement, potential recipient, hazard event, and final outcome. Grant only if the relevant recipient survives the event. Do not award self-rescues or repeat credit from oscillating/reusing one threat event. For credited rescues, define a short stable prediction horizon and invalidate the prediction when intervening edits or Time Stop manipulation change the threat. Moving a player into danger and then undoing that same created threat does not qualify. Use explicit scripted rescue objectives where reliable attribution is unavailable. Unsupported attribution means no rescue credit, not a guessed reward.

Do not add PvP elimination or assist rewards by implication. Define whether a statistic means the player's own deaths or attributed opponent eliminations; use separate names if tracking both.

Coins purchase curated cosmetics: trails, character effects, victory poses, nameplates, emotes, freeze-frame effects, titles, and lobby decorations. Cosmetics must not secretly alter collision, damage, or ability entitlement.

## 5. Preparation object editor

Provide mouse and touch selection, highlighting, translation, rotation, allowed-region display, local placement preview, confirm, cancel, and undo. Use text/icons in addition to green/red placement indicators.

Default allowance is **one distinct editable object per participant per round**, rather than one remote message or one drag. The player can revise its confirmed placement during Freeze and undo to that object's round-start placement. Switching targets is allowed before the first confirmation; after confirmation, the player may only revise the same object. Cancel discards the current preview and retains the last confirmed placement.

Separate a temporary **editing claim** from a confirmed **reservation**. Unconfirmed claims expire after a configurable idle timeout or on cancel/disconnect/phase change; provisional idle timeout: five seconds. Renew only on valid editor activity. After first confirmation, reserve that object to its participant for the remaining Freeze phase; cancel/undo does not give that participant a second object. On disconnect, release the editing claim, retain the confirmed pose and reservation through Freeze, and preserve attribution. Close both editor permissions at Unpause; Time Stop has separate authority.

Each object declares an ID, editable pivot, movement and rotation limits, legal region, collision policy, interaction range, and allowed phases. Give the arena enough useful editable objects for its intended capacity, or explicitly support a smaller capacity; do not let one player monopolize all preparation resources.

The server validates phase, participant status, allowance, claim, target registration, request sequence, finite transform data, movement distance, rotation, bounds, and prohibited overlaps. Validate the complete object volume, not only its center. Account for scaled geometry, player clearance, floors, protected structures, and collision groups.

Measure preparation translation/rotation allowances cumulatively from the object's round-start pose, not from the last accepted edit; repeated small requests must not bypass limits. Undo is a request through the same current collision/bounds validation. If another entity now blocks the original location, reject it with a reason rather than forcing an overlap. Confirmed relocation is atomic placement, not an unintended damaging physics sweep through intermediate positions.

Translate previews locally; submit bounded requests at a configured rate. The server acknowledges or rejects with a reason and authoritative transform. Never trust client previews, claimed ownership, arbitrary Instance references, or a client-supplied player identity.

Confirmed placements persist into Unpause and affect hazard controllers. They do not move back to template positions. Time Stop uses a separate permission path; it does not reopen normal preparation editing.

## 6. Disaster framework and motion contract

Use a DisasterManager and a consistent module interface. Every registered disaster declares its ID, tier, arena, required objects, spawns, hazards, editable targets, damage rules, survival region, optional objectives, modifiers, effects, compatibility, and readiness status.

Suggested lifecycle:

- `Setup(context)` — construct/register scenario state and validate dependencies.
- `BeginFreeze(context)` — establish frozen presentation and editor targets.
- `BeginUnpause(context)` — activate controllers.
- `Update(context, simulationDelta)` — advance hazard logic only when simulation runs.
- `Evaluate(context)` — produce outcomes from server state.
- `Cleanup(context)` — idempotently cancel tasks, disconnect events, unregister targets, and remove scenario instances.

These names describe a project interface, not built-in Roblox APIs. Adapt the interface once if necessary, then keep modules consistent.

Every movable hazard must register an adapter defining:

1. Controller type: scripted, physics-driven, animated, or hybrid.
2. How to snapshot, pause, reposition, resume, and remove it.
3. Legal manipulation bounds, reach, clearance, and orientation rules.
4. How a reposition changes its remaining path, velocity, hitboxes, and constraints.
5. How it suppresses gameplay damage, child spawning, and scheduled actions while paused; cosmetic-only effects may continue.

Declare **pause participation** separately from **manipulation eligibility**. Every active gameplay system affected by Time Stop must pause, including non-editable waves, area damage, AI, moving scenery, and connected bodies. Only the registered editable subset can be repositioned. An untargetable hazard is not permission to keep it running.

For scripted motion, rebase the remaining path/controller state from the confirmed new pose. Do not resume a tween or trajectory that writes the old pose back. Reset swept-collision history to the accepted pose so the teleport itself does not damage everything between old and new locations. On the first resumed step, evaluate valid contacts at the destination under the normal hazard rules. For articulated hazards, move the declared control group coherently and validate every attached body, tether, and external constraint. Disable/rebase/restore constraints through its adapter, or reject an impossible placement before committing it.

Damage is evaluated on the server using appropriate overlap, raycast, or swept-volume checks. Fast scripted or anchored hazards must not rely solely on touch callbacks. Presentation may interpolate on clients, but hitboxes and outcomes follow the authoritative controller.

Before implementing a hazard, declare whether contact is lethal or deals finite damage, its hit/cooldown identity, stacking rules, knockback, legal survival region, and elimination criteria. Use consistent character starting health for the selected rig. Decorative debris does not cause damage unless registered as a hazard. Near-miss/rescue checks use these same volumes and rules. A warning that appears only after voluntary movement is locked cannot rely on last-second dodging as a survival strategy; telegraph that threat during preparation or make prepared protection effective.

Keep authoritative critical assemblies under suitable ownership. Player movement remains subject to server plausibility checks, especially while their normal controls should be locked. Client input suppression alone is not an anti-exploit boundary.

Never apply paid manipulation to an unregistered or incompatible hazard. If a required eligible hazard cannot support it, mark that scenario incomplete and keep it out of production rotation; do not silently remove the feature.

Contain module errors with timeouts, scoped cleanup, and round cancellation. Asynchronous work must check that its match, round, and target generation are still current before changing state.

## 7. All 30 disasters

Preserve all 30 base disasters below as the full content target. Implement and verify them progressively. Each base disaster needs at least three distinguishable, tested survival approaches under the movement-lock rule. The set must demonstrate meaningful positioning, preparation, and object manipulation. Three different props alone do not establish three different approaches.

The descriptions below are design requirements; use controlled physics or scripted equivalents to deliver them reliably.

Catalog phrases such as “move one or more objects” describe the group or a scenario's explicitly configured allowance; they do not silently override the default of one confirmed object per participant.

Use the following survival acceptance rule throughout implementation and QA: **each base disaster requires at least three distinct tested approaches; each enabled modified or compound scenario requires at least one tested preparation-based approach that does not require owning or activating Time Stop.** Variants need not preserve all three base approaches. Document their readable hazard cues and which approaches remain viable. More surviving strategies are desirable, but do not silently turn that preference into an additional completion gate.

## TIER 1 — FUNNY DISASTERS

These disasters should be accessible to new players while creating amusing moments.

### DISASTER 01 — RUNAWAY TRAIN

Environment: Train station.

A massive train is frozen moments before crashing into the arena.

Players can:

- Reposition ramps.
- Move barricades.
- Hide behind protective structures.
- Redirect the train's path.

When time resumes, the train crashes through the station.

Use carefully controlled collision behavior so its movement is consistent across multiplayer clients.

### DISASTER 02 — FERRIS WHEEL DISASTER

Environment: Amusement park.

A gigantic Ferris wheel is tipping over.

Players must avoid the wheel and falling passenger cabins.

Editable barriers and supports can influence where pieces land.

### DISASTER 03 — GIANT BURGER

Environment: Cartoon fast-food restaurant.

An enormous hamburger is suspended above the arena.

When time resumes, it falls apart.

The buns bounce, tomatoes roll, and other ingredients scatter.

Players can redirect ingredients with ramps or hide behind structures.

### DISASTER 04 — SUPERMARKET STAMPEDE

Environment: Supermarket on a steep hill.

Dozens of shopping carts are about to roll downhill.

Players can rearrange shelves, barriers, and ramps to redirect the incoming carts.

### DISASTER 05 — SOAP EXPLOSION

Environment: Soap factory.

A large machine is about to explode, filling the arena with slippery soap.

Objects begin sliding uncontrollably.

Players must prepare safe positions and move useful barriers before the explosion.

### DISASTER 06 — BALLOON DISASTER

Environment: Birthday party arena.

Hundreds of balloons are attached to furniture, platforms, and other objects.

When time resumes, objects begin floating upward.

Players must avoid being carried into hazards or losing their footing.

### DISASTER 07 — CAKE COLLAPSE

Environment: Giant bakery.

A gigantic birthday cake begins collapsing.

Cake layers, candles, and oversized decorations fall across the arena.

Players can manipulate supports and barriers to influence the collapse.

### DISASTER 08 — FURNITURE AVALANCHE

Environment: Furniture warehouse.

A massive furniture stack is about to topple.

Players reposition selected furniture and protective objects.

When time resumes, the entire stack collapses.

---

## TIER 2 — DANGEROUS DISASTERS

These disasters should demand stronger planning and more careful manipulation.

### DISASTER 09 — TORNADO TOUCHDOWN

Environment: Small suburban town.

A tornado is approaching.

Players can reinforce shelters and reposition heavy objects.

When time resumes, the tornado moves through the town and lifts designated loose objects.

Use controlled forces and particle effects rather than simulating excessive loose debris.

### DISASTER 10 — TSUNAMI

Environment: Beach town.

A giant wave is frozen near the coastline.

Players prepare elevated positions, floating platforms, and barriers.

The tsunami then travels through the arena.

Use an efficient scripted flood and wave-hazard system instead of full fluid simulation.

### DISASTER 11 — MEGA MAGNET

Environment: Industrial scrapyard.

A huge electromagnet is about to activate.

Cars, metal containers, and signs begin moving toward it.

Players can reposition metal objects and barriers to create safer paths.

### DISASTER 12 — CRANE FAILURE

Environment: Construction site.

A large crane is about to collapse.

Its suspended cargo swings across the arena.

Players move selected barriers and platforms to control where the cargo travels.

### DISASTER 13 — ELEVATOR FREEFALL

Environment: Tall building.

An elevator is about to drop.

Players can prepare cushions, platforms, and supports beneath the elevator.

When time resumes, it falls.

The hazard should support multiple outcomes depending on player preparation.

### DISASTER 14 — VOLCANO ERUPTION

Environment: Volcanic island.

A volcano is about to erupt.

Large molten rocks are frozen above the arena.

Players reposition cover and search for safer locations.

Upon UNPAUSE, rocks fall and designated lava hazards become active.

### DISASTER 15 — LIGHTNING STORM

Environment: Open field with structures.

A powerful lightning storm is about to begin.

Players can reposition designated lightning rods, conductive objects, and shelters.

When time resumes, lightning strikes warning-marked areas.

Lightning must provide readable warning indicators before impact.

### DISASTER 16 — SATELLITE CRASH

Environment: City rooftop district.

A massive satellite breaks apart as it falls toward the arena.

Players can move barriers and shelter structures.

Several large satellite fragments crash into different locations.

Use controlled trajectories and simplified debris.

### DISASTER 17 — STADIUM COLLAPSE

Environment: Sports stadium.

An enormous stadium roof is about to collapse.

Players can reposition supports and move underneath safer sections.

When time resumes, parts of the roof fall in a planned but interaction-sensitive sequence.

### DISASTER 18 — ICEBERG BREAK

Environment: Frozen ocean.

Players stand on a giant iceberg that is beginning to fracture.

They can move planks, supports, and connecting objects.

The iceberg splits into multiple moving sections when time resumes.

---

## TIER 3 — INSANE DISASTERS

These are the rarest and most visually memorable scenarios.

### DISASTER 19 — ALIEN INVASION

Environment: Small city.

A huge UFO is positioned above the arena.

When time resumes, it activates a tractor beam.

Players can manipulate barriers and objects that block or redirect the beam.

Affected objects and players are pulled upward.

### DISASTER 20 — BLACK HOLE

Environment: Futuristic laboratory.

A machine is about to create a black hole.

The black hole attracts nearby loose objects and players.

Use force fields or controlled movement rather than a computationally expensive gravitational simulation.

Players can reposition barriers and heavy objects to survive.

### DISASTER 21 — GIANT DINOSAUR

Environment: Prehistoric city.

A gigantic dinosaur prepares to stomp across the arena.

Players can reposition cars, barriers, and distractions.

The dinosaur's route is influenced by selected environmental changes.

Use animation-driven movement combined with server-validated hitboxes.

### DISASTER 22 — THE MOON IS FALLING

Environment: Cartoon planet surface.

An enormous moon is descending toward the arena.

Players must prepare for the shockwave.

The moon should create a dramatic, humorous impact without requiring simulation of a planet-sized physical object.

### DISASTER 23 — ROBOT MELTDOWN

Environment: Robot factory.

A gigantic robot begins malfunctioning.

Its arms, legs, and energy blasts become dangerous hazards.

Players reposition cover and selected mechanical objects.

### DISASTER 24 — GRAVITY FLIP

Environment: Floating laboratory.

Gravity inside the arena changes direction.

Players must prepare safe positions before being pulled toward the new gravity direction.

Implement arena-specific gravity behavior where practical rather than unnecessarily affecting the entire Roblox world.

### DISASTER 25 — SPAGHETTI TSUNAMI

Environment: Italian-themed restaurant town.

A giant wave of spaghetti approaches.

Oversized meatballs roll through the arena.

Players can reposition barriers to redirect meatballs and avoid the wave.

### DISASTER 26 — ZOMBIE FREEZE

Environment: Abandoned town.

A group of cartoon zombies is frozen moments before charging.

Players can move barricades, traps, and obstacles.

When time resumes, zombies follow predefined or lightweight navigation behavior.

Avoid excessive NPC pathfinding requests.

### DISASTER 27 — GIANT FOOT

Environment: Miniature city.

An enormous foot is about to stomp on the arena.

Players can move protective structures and reposition themselves outside impact areas.

The foot creates a shockwave when it lands.

### DISASTER 28 — EARTHQUAKE MACHINE

Environment: Underground scientific facility.

A gigantic machine is about to activate.

The arena begins shaking.

Selected structures tilt, crack, and collapse.

Players can reinforce safe areas or reposition themselves before the earthquake begins.

Use scripted platform motion and predefined destruction states to reduce physics costs.

### DISASTER 29 — THE MEGA TOILET

Environment: Ridiculous bathroom-themed city.

An enormous toilet is about to flush.

It creates a powerful suction effect.

Loose objects and players are pulled toward its center.

Players can place barriers and heavy objects to avoid being sucked away.

### DISASTER 30 — DOMINO DOOMSDAY

Environment: Giant toy city.

Hundreds of oversized dominoes are about to fall.

Players can move one or more designated dominoes and barriers to redirect the chain reaction.

Use controlled chain-reaction triggers with visual physics effects.

The system must avoid relying on hundreds of unstable simultaneous collision calculations.

## 8. Mutations, compound disasters, and selection

Mutations are a required part of the full production target, introduced after verified base disasters. Keep them disabled in early milestones. Enable only individually tested disaster–modifier combinations.

| Modifier | Behavior and constraints |
|---|---|
| Low Gravity | Reduce effective gravity for declared assemblies; affect player jumps during preparation where supported. Do not change lobby gravity. |
| Double Speed | Double declared hazard movement/animation rates in simulation time. Keep the standard 10-second active window; define what hazards do after reaching their endpoint. Update warnings and collision checks. |
| Giant Mode | Scale selected supported hazards; update hitboxes, clearance, bounds, and previews. Reject unsafe assembly scales. |
| Invisible Danger | Reduce hazard visibility while preserving readable warning shadows, outlines, or indicators. |
| Reverse Mode | Use an explicitly authored alternate approach, valid spawns, and compatible editor boundaries. |

**Double Disaster** is a compound scenario, not simply another speed/visual modifier. It runs two tested disasters within one compatible arena and shared clock, with namespaced entity IDs, shared damage accounting, editor allocation, pause integration, performance limits, and coordinated cleanup. A pair is selectable only after these integration requirements pass.

Candidate compound pairs from the original vision:

- Runaway Train + Mega Magnet.
- Tornado Touchdown + Balloon Disaster.
- Giant Dinosaur + Lightning Storm.
- Domino Doomsday + Earthquake Machine.

Giant Burger + Low Gravity is a **single disaster with a modifier**, not a two-disaster pair. Runaway Train + Giant Burger is another optional compound candidate, subject to the same compatibility and verification gates.

Record a content-coverage manifest before Milestone 8: all 30 base disasters are mandatory; each of the five modifiers needs at least one verified compatible base disaster; Double Disaster needs at least one verified pair. These are proposed minimum coverage rules, not permission to present untested combinations as supported. The listed candidate pairs remain optional unless selected into the committed content scope. Every combination enabled in rotation must appear in the manifest with its own passing integration evidence, including the survival acceptance rule in §7; the full Cartesian product is not required.

Use this baseline scenario distribution:

- Funny-tier single disaster: 45%.
- Dangerous-tier single disaster: 35%.
- Insane-tier single disaster: 15%.
- Verified Double Disaster: 5%.

These are mutually exclusive selection categories. Select category, then an eligible scenario; select a supported modifier separately. Provisional modifier chance after integration: 20%, with at most one modifier; default off for compound scenarios until specifically verified.

Filter by verified readiness, arena compatibility, player capacity, recent history, and available assets before selection. Renormalize across available categories when some are empty; never select unfinished content. Provisional recent-history window: two rounds, relaxed only when the eligible pool is too small.

Early/middle/late difficulty weighting is configurable and must be explicit; begin with the baseline distribution rather than inventing an undocumented progression formula. Log selections and verify that compatibility constraints hold.

## 9. Absolute Time Stop — required paid gameplay ability

**Permanent gamepass name:** ABSOLUTE TIME STOP  
**Suggested price:** 399 Robux, requiring creator approval  
**Default duration:** three real seconds  
**Default allowance:** once per owner per eight-round match  
**Activation:** Unpause only, for a living eligible participant with a verified entitlement  
**Concurrency:** one active Time Stop per server

The holder remains mobile while active gameplay hazards and other participants freeze. The holder may reposition and rotate other frozen characters and eligible hazards inside server-defined legal boundaries, including moving opponents into dangerous but otherwise valid positions. Protected terrain, scripts, map boundaries, permanent buildings, and arbitrary Workspace descendants remain inaccessible.

Duration, reach, translation limits, rotation limits, and input rates are configurable. Before implementation, record concrete per-target limits in GameConfig and disaster definitions. Scale them to the actual arena; do not leave validation dependent on unspecified placeholders. Default interaction behavior: drag or tap destination, rotate with dedicated controls, then confirm. Provide both touch-friendly controls and mouse input.

### Activation transaction

Validate entitlement, match/attempt IDs, phase, remaining simulation time, life state, remaining charge, and global ability lock on the server. Hold the lock through **Preparing → Active → Restoring**. Preparing must suppress new gameplay updates and capture coherent controller state within a bounded timeout. Start the three-second wall-time window and consume the charge only when activation commits as Active. If preparation fails, safely restore or cancel the attempt and keep the charge. A second owner's rejected activation consumes nothing.

Do not yield inside a state commit. Perform any asynchronous entitlement work beforehand, then revalidate the current generation and eligibility. Every delayed activation/expiry/watchdog callback checks its own attempt and ability generation immediately before changing state; a callback from an older freeze is a no-op. Release the global lock only after restoration finishes or cancellation safely completes. Never grant a new activation while the previous state is partly restored.

Keep charges by UserId and match ID, not by character instance. Death, respawn, reconnect, purchasing again, and ownership rechecks must not refresh a consumed charge in that match. A newly verified owner who has not used the ability gets at most one charge. Preserve the same-match ledger across disconnects in that server. Server hopping into a different match follows that match's allowance; do not claim the allowance is globally enforced unless cross-server infrastructure is deliberately implemented.

### Freeze and manipulation

- Pause the simulation clock and all registered damage, motion, spawning, and hazard/AI controllers.
- Freeze non-holder participants and every gameplay-affecting dynamic assembly with controller-specific adapters, independently of whether it is editable. Store anchored state, assembly velocities, constraints, character state, animation/motion state, and ownership information as applicable.
- Keep the holder's allowed movement active. Pause hazard damage for everyone, including the holder; environmental boundary rules remain in force.
- Stop animations and applicable audio through explicit controllers. Decorative effects may continue where harmless; do not claim the entire engine is frozen.
- Validate each manipulation against the active holder, ability generation, current target identity, reach, finite transform data, legal volume, clearance, and rate limit. Make placement atomic on the server.
- Measure cumulative movement/rotation allowances from that target's pose at this ability's activation. Reselection and repeated confirmations do not reset the reference. The next valid ability takes a fresh snapshot. Separate cumulative limits from per-request rate/delta limits.
- Move a character coherently as a model/assembly. Prevent embedding in solid geometry or rig damage. Disallow seated targets until they have been safely unseated through a verified adapter; unsupported rig/ragdoll configurations must be reported and kept out of enabled gameplay until supported.
- Revalidate reach and legal placement against the holder's plausible server-observed position; the holder cannot bypass limits through an untrusted client teleport.
- At expiry, reject late manipulations even if the UI still shows time remaining. Cancel unconfirmed previews and release target selection.

Three seconds is a design default, not proof that selecting, dragging, rotating, and confirming works on touch devices. Measure it with representative users, including realistic network latency. Use server receipt/processing before the ability deadline as the acceptance rule; client timestamps cannot extend it. Show rejected late confirmations clearly. Improve reversible control details and record findings; seek creator approval before changing the selected duration or reducing the power's core behavior.

### Resume contract

Confirmed new positions and orientations persist. **Never restore the pre-freeze transform over an accepted placement.** Restore only the state needed to continue simulation.

- Unmoved physics hazards restore saved world-space linear/angular velocity.
- For moved physics hazards, each adapter specifies velocity behavior. Provisional default: rotate saved velocity vectors by the placement's rotation delta, retaining magnitude; translation alone does not change velocity.
- Scripted/animated hazards resume from rebased controller state and the new pose, preserving remaining simulation progress. Update attachments, collision volumes, and event schedules accordingly.
- Non-holder characters retain the accepted pose and restore saved pre-pause world-space velocity by default; repositioning does not erase incoming knockback. Character adapters declare supported orientation limits and stable handling of airborne/ragdolled rigs. The holder retains valid server-observed exit momentum rather than an old snapshot velocity. Test these proposed defaults with collisions and moving platforms.
- On normal expiry, non-holders retain Unpause movement restrictions and the holder loses the temporary movement exception. Derive final permissions from the current phase, life state, and ability state rather than blindly restoring old properties. Cancellation/phase exit removes freeze state and applies the destination phase's rules without restarting a cancelled disaster.
- Restore surviving current instances only, with valid network ownership and constraints. Never apply a snapshot to a replacement character or a reused entity ID.
- On a normal return to Unpause, resume hazard damage and simulation once restoration is complete; physics anchoring alone does not suspend logical damage callbacks. Cancellation never resumes the terminated disaster.

Use a central TimeFreezeService with idempotent resume and fail-safe cleanup. The ability's wall-time watchdog cannot depend on the stopped simulation clock. End early and restore state when the holder disconnects, dies, resets, or the round is cancelled. Target destruction, disconnection, respawn, or removal must be safe. New players remain spectators. Shutdown cleanup must not wait forever.

A fixed cohort of twelve owners can add **36 real seconds across a match**, potentially concentrated in one round. Twelve is a concurrent player count, not a limit on distinct owners over eight rounds: replacement owners can add more match pause time. With twelve eligible owners in a single round, that round can receive twelve three-second activations. Concurrent rejection does not prevent consecutive activations. Instrument total pause time and non-owner experience; propose global limits only as creator-approved balance options.

Visuals: purple/monochrome treatment, recognizable holder aura and announcement, duration overlay, activation/expiration audio, and clear manipulation feedback. Respect reduced-flashing, reduced-shake, and low-effects settings. No visual effect should obscure placement validity or ability expiry.

## 10. Purchases and shop boundaries

Use current documented MarketplaceService gamepass ownership, product-information, purchase-prompt, and purchase-completion APIs. Check their behavior against current documentation, including ownership caching and failure handling.

- Verify entitlement on the server. Never grant access because a client reports ownership or purchase success.
- Handle the documented server purchase-completion flow and refresh ownership as supported. Account for cache behavior; do not assume one immediate recheck always reflects a completed purchase.
- Pending or failed ownership checks mean **unknown**, not owned. Fail closed for the ability, show a useful status, and retry with bounded backoff. Do not repeatedly prompt a known owner.
- Recognize valid mid-match purchases no later than the next eligible round once authoritative purchase verification succeeds; preserve an already-consumed match charge.
- Retrieve current displayed price from product information. Do not present the suggested 399 Robux as a verified live price.
- An unconfigured gamepass ID disables purchase prompting and live entitlement-dependent access. Use an explicit **Studio-only**, server-controlled test entitlement to test mechanics; it must not be reachable through client requests or active in live servers.

Build an expandable shop boundary for future creator-approved gamepasses, developer products, cosmetics, animations, and effects. Do not implement unrelated paid products or paid random-reward crates by default. If developer products are added later, use their separate documented receipt flow; do not reuse gamepass logic as if the purchase types were identical.

## 11. Persistence and progression

Use DataStoreService through a project-owned **PlayerProfileService** facade. This name does not mandate any third-party library. If selecting one, explicitly identify its source/version, license, maintenance status, and tested concurrency behavior. Define a versioned, validated profile including:

- Spendable Coins and LifetimeCoinsEarned separately.
- LifetimeXP and derived Level.
- MatchesPlayed, MatchesWon, RoundsPlayed, RoundsSurvived, LifetimePoints, RescueCount, and PlayerDeaths; separate any future attributed elimination count.
- Per-disaster survival counts.
- OwnedCosmetics and EquippedCosmetics.
- AchievementProgress and awarded achievement IDs.
- DailyChallenges keyed by UTC day ID, with assigned challenge IDs, progress, and claimed reward IDs; retain only the current and previous claimable day.
- Settings and LastSaveTimestamp.
- Session/concurrency metadata and bounded reward deduplication state appropriate to the persistence design.

Use protected calls, request-budget awareness, bounded retries with backoff, autosaves, milestone saves, leave saves, and bounded shutdown saves. `UpdateAsync` is a tool for atomic updates, not an automatic complete profile-concurrency solution.

Implement and document session ownership or an equivalent conflict-safe write model to prevent stale servers overwriting newer data. Include expiry/recovery behavior for abandoned sessions, migration validation, and safe rejection of unknown future schemas. A read failure must never be treated as a new empty profile and then overwrite existing progress.

If a real profile cannot load, allow a clearly marked temporary session for independent gameplay, but disable persistent rewards, cosmetic purchases/equipment changes, and saving that temporary profile. Retry safely. Adopt a recovered real profile at the next round's Freeze boundary after verifying session authority; preserve temporary match score separately. Persistent round-grant eligibility is captured at that boundary and requires continuing profile authority at settlement. Do not retroactively import temporary Coins/XP/statistics or merge an entire temporary profile into the saved one. A recovered profile may earn future rewards normally. Losing profile-session authority disables further persistent mutation until recovered.

Use reward IDs and the concurrency strategy together to prevent duplicate successful writes during retries and reconnects. Bound deduplication storage; do not append every historical round forever. State the durability guarantee: sudden crashes may lose progress after the last successful save unless a tested stronger mechanism exists. Do not promise zero loss or global exactly-once delivery merely because rewards have IDs.

All profile mutations must share one serialized authority; autosaves, round grants, shop purchases, challenge claims, and leave saves cannot overwrite one another's state. A DataStore update callback may run again: keep it free of side effects such as remote messages, duplicate item grants, or external writes. Treat an uncertain write outcome as pending and reconcile by transaction identity before attempting another debit/grant.

### Cosmetic transactions and counter semantics

Use a server-owned catalog with stable item IDs, configured nonnegative integer Coin prices, item types, and availability. Clients request an item, never a trusted price, new balance, or inventory grant. Verify active profile authority, sufficient **settled spendable** Coins, item availability, and ownership. Serialize a purchase as one idempotent balance-debit plus inventory-grant profile operation. A retry or purchase of an already-owned non-stackable cosmetic must not debit again. Equipped items must be owned and compatible with their slot. Disable purchases without profile authority, expose pending/error status, and reconcile uncertain saves through the same transaction instead of creating a second purchase. Spending never decreases LifetimeCoinsEarned.

Proposed counters: RoundsPlayed increments once for settled participation; RoundsSurvived once for settled survival; PlayerDeaths at most once for elimination in that completed attempt; MatchesPlayed and MatchesWon follow the match eligibility rule; LifetimePoints includes settled round points plus any separately identified match-win bonus. A survived compound round counts as one round, while incrementing each component disaster's survival counter once. Aborted attempts and temporary sessions do not increment persistent counters. Achievement, challenge, and stat projections must consume the same settled events rather than independently re-detecting collisions or deaths.

Isolate test persistence from production data through separate test experiences/stores and explicit development configuration. Never enable tests that can overwrite real player progress. Do not store gamepass ownership as the entitlement authority.

Levels provide progression without additional competitive stat advantages. Implement these achievements with deterministic server criteria:

- FIRST SURVIVOR: survive one round.
- TRAIN DODGER: survive Runaway Train ten times.
- UNTOUCHABLE: survive all eight rounds of one match.
- LUCKY ESCAPE: survive a server-recorded near-miss event within a configured hazard-specific distance; disable this achievement until near-miss detection is verified.
- DISASTER VETERAN: survive 100 rounds.
- LEGENDARY SURVIVOR: win 50 matches.
- CHAOS MASTER: survive a verified compound Double Disaster.

Include **three daily challenges**, drawn from the verified content pool, using server UTC day IDs. Rejoining retains the day's assignment and progress. Bind each progress event to the day at **outcome settlement**, not at save time; a round spanning midnight counts toward the new day only. Rotate assignments during live sessions as well as on join. Proposed claim rule: a completed challenge remains claimable through the end of the following UTC day; incomplete old progress expires at rollover. Retain only the current and previous claimable day's records. Serialize each claim marker and reward in one idempotent profile mutation, so midnight, retries, and concurrent requests cannot grant twice. Use event IDs to preserve the original day through delayed saving.

If the current pool cannot support three valid challenges, provide verified fallback goals rather than unreachable objectives. Example goals: survive three rounds, complete two matches, survive one Dangerous-tier disaster. Define how compound tiers satisfy tier challenges in the catalog; a single round contributes at most once to the same challenge. Publish configurable reward values; never require Robux to complete them.

## 12. Lobby, interface, audio, and effects

Create a bright **Disaster Control Center** lobby with the UNPAUSE! identity, countdown, match leaderboard, personal statistics, cosmetic shop, gamepass display, tutorial, settings, disaster preview, spectator area, and lightweight disaster decorations.

First-time players should understand the main mechanic within one minute. Offer a skippable tutorial: prepare, move one object, choose a position, watch Unpause, survive. Do not repeatedly force returning players through it.

Required interfaces:

- **HUD:** phase, countdown, disaster, round number, score, coins, ability availability, modifier.
- **Preparation editor:** selected object, allowance, move/rotate controls, confirm/cancel/undo, validity and rejection reasons.
- **Time Stop:** purchase/ownership status, activation, charge, remaining wall time, target selection, move/rotate controls.
- **Results:** survivors, eliminated participants, points/bonuses, standings, optional highlight.
- **Match summary:** shared winners when tied, standings, survival count, points, coins, match rewards.
- **Settings:** music and effects volume, reduced shake, reduced flashing, low-effects mode, UI preferences; essential warnings remain visible.

Use bold readable type, clear countdowns, restrained transitions, rounded panels, icons, and strong hierarchy. Suggested identities: navy/blue normal UI, cyan Freeze, orange/red Unpause, purple Time Stop, green success, red elimination, gold rare scenarios. Never rely on color alone.

Mobile controls need large tap targets, safe-area support, non-overlap with default movement controls, clear destination/rotation controls, and obvious confirmation. Test landscape/portrait behavior appropriate to the configured experience rather than assuming desktop dragging translates directly to touch.

Use brief hazard introductions that do not consume the promised preparation window. Unpause effects include announcement, impact sound, small configurable camera shake, and clear return to the player view. Restore camera/input state after spectating, death, respawn, phase change, or interruption.

Start highlights with server-recorded event metadata and event-driven cameras. Label these as highlights rather than a true recorded replay. A slow-motion view of a past impact requires captured state; use a bounded client-only reconstruction later if implemented. Never slow the active shared world to fake a Results replay.

Provide asset configuration for lobby ambience, selection, preparation, countdown, Unpause, impacts, elimination, survival, scoring, victory, Time Stop start/end, and UI feedback. Give disasters distinct sounds where configured. Use creator-owned or properly licensed assets, verify that the experience can access them, and provide silent/visual fallbacks for missing assets. Never invent asset IDs.

Provide modular dust, smoke, sparks, explosions, lightning, shockwaves, debris, water, blur, and camera effects. Pool effects when useful, cap particle/debris counts, and preserve readable hazard warnings. Custom meshes/audio are polish dependencies, not blockers for a primitive playable prototype.

## 13. Networking, security, and suggested hierarchy

The server owns match and round state, participant eligibility, disaster selection, hazard logic, editor permissions, Time Stop, elimination, scores, rewards, profiles, and entitlements. Clients collect input and present UI, previews, cameras, audio, effects, and interpolated motion.

Every client request validates payload type/size, caller eligibility, match/round/ability generation, target registration, distance, finite numbers, bounds, claim, permissions, rate limit, and charge/entitlement where applicable. Reject stale/replayed requests, NaN/infinity, arbitrary target instances, cross-round IDs, oversized payloads, and unsupported action types. Use server-issued IDs; IDs identify targets but do not grant permission.

Rate limits are configurable server-side token buckets or equivalent. Set concrete initial values before implementation and tune from legitimate mouse/touch traffic. Expensive overlap validation must also be bounded. Late-joining clients need an authoritative state snapshot, not only future incremental events.

Suggested hierarchy; merge services if a simpler equivalent is sufficient:

```text
ReplicatedStorage
  Shared/                  configuration, public definitions, types, constants, utilities
  Remotes/                 bounded requests and authoritative state/effect updates
ServerScriptService
  Bootstrap                Script
  Services/                game/round, arena, disasters, editor, clock, freeze,
                           ability, hazards, survival, rewards, profile, monetization
  DisasterModules/         one ModuleScript per implemented scenario
ServerStorage
  ArenaTemplates/
  HazardTemplates/
  ServerOnlyAssets/
StarterPlayer
  StarterPlayerScripts/    client bootstrap, HUD, input, editor, time stop,
                           camera, audio, effects controllers
StarterGui
  MainHUD, LobbyUI, RoundUI, ResultsUI, ShopUI, SettingsUI, TimeStopUI
Workspace
  Lobby, ActiveArena, TemporaryEffects, SpawnPoints
```

For each milestone, provide the exact hierarchy and instance class, not only these conceptual folder labels. Keep authoritative code, private profiles, and server-only assets outside replicated containers. Avoid copying every service into the first prototype before it has a purpose.

## 14. Performance and acceptance tests

Target stable, responsive play with twelve players. Aspirational client targets: approximately 60 FPS on capable desktops and at least 30 FPS on specified representative low-end mobile devices. Record hardware, player count, scenario, settings, duration, and actual measurements; do not guarantee performance from code inspection.

Before heavy physics content, record per-scenario budgets for active assemblies, parts, NPCs/pathfinding, particles, update frequency, and network traffic. Measure server frame cost, client frame times, memory trend, loading, and cleanup. Use simplified collisions, controlled large-object motion, scoped connections/tasks, and bounded debris. Server ownership must be deliberate rather than assigned indiscriminately to every object.

Before release testing, fix a test matrix listing supported device classes, player load, representative/worst enabled scenarios, settings, duration, and numeric performance/stability pass thresholds. Record measured outcomes against those thresholds. Prototype FPS goals remain aspirational; release readiness requires passing the declared matrix or reporting a release blocker. Do not revise thresholds after a failure merely to label it a pass.

Minimum verification gates:

1. **Core loop:** complete an eight-round match and start the next; verify clocks, spawns, joins/leaves, resets, participant snapshots, final-step damage/activation ordering, disconnected-leader ranking, all-empty cancellation, and new attempt IDs on retries. Fault-inject failures before and after reward settlement and confirm no duplicate result, spent provisional reward, or post-commit rollback.
2. **Preparation:** two players contend for one object; validate claim versus reservation, expiry, cumulative allowance, repeated small edits, blocked undo, stale requests, bounds/overlaps, and a barrier causing a different train outcome.
3. **Time Stop:** freeze editable and non-editable gameplay hazards, damage, warnings, AI, projectiles, and simulation time; keep ability expiry advancing; manipulate a player/hazard and resume at their accepted poses. Test momentum, collision-history reset, tether constraints, repeated-edit limits, two owners, consecutive activations with stale watchdogs, cancellation during restoration, holder death/leave, late confirmations under latency, removed/respawned targets, supported rigs/ragdolls/seats, and fail-safe cleanup.
4. **Entitlement:** reject non-owners and client spoofing; distinguish unknown ownership; recognize verified mid-match purchases; preserve charges across reset/rejoin; verify Studio test access is unavailable in live servers.
5. **Persistence:** load/save/rejoin, concurrent/stale sessions, repeated update callbacks, uncertain write results, unavailable stores, migration, recovery from temporary sessions, shop debit/grant retries, overspending, equipment ownership, midnight rollover and old-day claims, per-attempt counter projections, and shutdown/crash durability limits.
6. **Content:** apply the survival acceptance rule in §7: at least three tested approaches per base disaster and at least one tested preparation-based approach per enabled modified or compound scenario, without requiring Time Stop. Verify useful editing, Time Stop compatibility, damage/survival, and cleanup for every enabled scenario. Test each allowed modifier combination and compound pair separately; a passing base-disaster test does not validate its variants.
7. **Devices and load:** one, two, four, eight, and twelve clients where supported; Studio multi-client tests plus private/live test sessions as needed. Test real touch input and representative mobile devices. Emulation alone does not certify mobile performance.
8. **Security:** malformed/oversized requests, invalid transforms, protected targets, out-of-bounds edits, spam, repeated grants, charge resets, inactive targets, and movement/ownership exploits fail safely.
9. **Cleanup:** run at least two matches and inspect retained instances, tasks, connections, target registries, physics state, camera/input state, and memory after arena removal. Extend soak testing if trends suggest leaks.

Report executed tests and their exact outcomes separately from planned, skipped, blocked, or unrun tests. Error-free startup satisfies only the foundation startup check, not gameplay, security, persistence, or device gates. A zero-test run is not evidence that a test suite passed. When Studio is unavailable, continue independent implementation/static checks but keep runtime gates explicitly pending; never claim verified completion or silently waive them.

## 15. Milestones and delivery

| Milestone | Scope and acceptance gate |
|---|---|
| 0 — Foundation | Minimal hierarchy, config, bootstraps, remotes, state types, logging. Verify error-free startup where runtime access exists. |
| 1 — Playable core | Basic lobby/HUD, one Runaway Train arena, all four phases, movement lock, survival, scoring, eight-round match/restart. Primitive assets are acceptable. |
| 2 — Preparation editor | PC/touch selection, preview, translation/rotation, confirm/cancel/undo, claims, allowance, bounds/collision validation; editing must change a train outcome. |
| 3 — Absolute Time Stop | Complete player/hazard freeze and manipulation, pose/velocity/controller restoration, clocks, failure handling, charges, Studio-only test entitlement, and real gamepass integration when configured. Runtime multiplayer verification required. |
| 4 — First five disasters | Runaway Train, Giant Burger, Crane Failure, Mega Magnet, Giant Dinosaur; shared framework verified across vehicle, bouncing, swinging, force, and animated hazards. |
| 5 — Player experience | Complete HUD, leaderboard, results/summary, tutorial, settings, PC/touch controls, camera, audio, accessibility. |
| 6 — Progression | Coins, XP/levels, event-based statistics/achievements, atomic cosmetic purchases, daily rollover/claims, profile concurrency/recovery, staged rewards and safe persistence. |
| 7 — Remaining content | Add the other 25 disasters one at a time; verify lifecycle, strategies, editing, Time Stop, cleanup, and mobile cost before rotation. |
| 8 — Mutations and compounds | All five modifiers and at least one compound scenario verified to the content-coverage manifest and §7 survival rule; compatibility selection and unavailable-category fallback. |
| 9 — Presentation polish | Lobby/arena art, particles, audio, UI transitions, introductions, victory presentation, lightweight highlights. |
| 10 — Release preparation | Multiplayer/load/mobile tests, security, purchase/data verification, cleanup/soak tests, accessibility, balance, and documented release blockers. |

Later-dependent achievements can be implemented in Milestone 6 and verified when their required content is available. Do not mark them runtime complete early. A five-disaster vertical slice is a useful playable milestone, not completion of the 30-disaster specification.

For each implemented milestone provide:

1. Objective and requirements covered.
2. Exact changed files/instance hierarchy and dependencies.
3. Complete runnable source or precise patches. With direct editing tools, report file locations and changes instead of unnecessarily pasting every unchanged file.
4. Idempotent Studio construction/setup instructions, including parts/models, attributes, tags, collision groups, and settings.
5. Configuration values and creator-supplied IDs/assets still needed. Use safe disabled states for missing commercial configuration.
6. Tests executed, environment used, outcomes, and exact manual procedures for pending runtime checks.
7. Known limitations, placeholders, blockers, and requirement status.
8. Next useful implementation step.

If a response cannot hold all required source, deliver coherent dependency-complete increments. Do not label a milestone complete when required files or runtime gates remain pending. Avoid endless architecture rewrites; explain any necessary contract change and update dependent modules consistently.

## 16. Definition of done and starting instruction

The full target requires verified multiplayer lobby and eight-round matches; all four phases; preparation editing; all 30 disasters and tiers; modifier/compound coverage from the committed manifest, meeting the §7 survival rule; survival/scoring/ties; coins/XP/statistics/achievements/daily challenges/cosmetic transactions; safe persistence; PC/mobile interfaces; licensed/configured audio and effects; Absolute Time Stop with full player/hazard manipulation; reliable restoration; authoritative purchase verification; remote/physics security; cleanup; and passing evidence for the release test matrix.

No unresolved release-blocking bugs may remain before publication. Feature-complete does not mean bug-free. Distinguish feature completion, playtest readiness, release readiness, and actual publication. Publication and paid-product activation require creator approval.

Priorities: enjoyable preparation and consequences, reliable physics, working multiplayer, the specified Time Stop power, mobile usability, maintainable expansion, and quality over content count. Safety and data integrity are release gates, not tradeable presentation polish. If a tradeoff changes an explicit requirement, explain it and seek creator approval rather than silently dropping the feature.

Begin by inspecting available tools/project state, then summarize the architecture, hardest systems, server-authoritative clock/freeze strategy, minimum assets/configuration, and the Milestone 0–1 plan. Begin Milestone 0 immediately if editing tools are available; otherwise supply its complete construction instructions and source. Proceed toward the playable Runaway Train vertical slice while preserving truthful verification status.

Build around **preparation, consequences, physics, surprise, and replayability**. The desired moment is a player surviving because of one carefully placed barrier—or a Time Stop holder repositioning a disaster and changing the outcome. Real, readable, repeatable gameplay matters more than the volume of generated code.
