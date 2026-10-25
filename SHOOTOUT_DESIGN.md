# SHOOTOUT — Design Bible

> **For any AI or developer working on this repo:** read this file in full before changing anything in SHOOTOUT.
> Rules in §2 are hard rules. Never break them, even if a task seems to require it — stop and ask instead.
> If a task conflicts with this file, say so before writing code.
> Items marked `OPEN` are undecided. Do not guess — ask the owner.

---

## 0. What SHOOTOUT is

A mobile-first co-op zombie shooter set in the NGMI Pandas universe.
Feeling: **big chaos, massive hordes, explosions everywhere — but always smooth.**
Reference feel: Left 4 Dead horde mode, played on a phone.

Priority order when things conflict:
1. **Smooth on mobile** (no stutter, no frame drops)
2. **Fun chaos** (lots of zombies, big explosions)
3. **Visual quality** (realistic look)

If a visual upgrade costs smoothness on mobile, smoothness wins.

---

## 1. Design pillars

| Pillar | Meaning |
|---|---|
| Power fantasy | The player feels strong. Good weapons are never taken away. |
| Horde chaos | Many zombies on screen; explosions clear groups. |
| Flow | Nothing interrupts play: no freezes, no hit-stop, no slow-mo, no frozen victory screens. |
| Grounded look | Semi-realistic, not cartoon. See §6. |

---

## 2. HARD RULES (never break)

1. **No freeze / stutter effects.** No hit-stop, no slow-motion, no frame-freeze on kills or victory.
2. **No announcer voice.**
3. **Zombies exist only in SHOOTOUT.** Never add zombies to BEATDOWN.
4. **Never auto-replace a better weapon with a worse one.** (See §3.)
5. **Grenades are always available.** (See §4.)
6. **Mobile first.** Every change must work with touch controls and run smoothly on a mid-range phone.
7. **Do not touch BEATDOWN** code or assets when working on SHOOTOUT, unless the task explicitly says so.
8. **Work on `main`** and push directly (owner's preference). Before any large change, create a git tag (e.g. `shootout-v0.4-before-weapons`) so it can be rolled back.

---

## 3. Weapon system

### 3.1 Weapon ranking (low → high)
`pistol < SMG < shotgun < rifle < RPG < dual RPG`
`OPEN:` confirm the exact weapon list and order.

### 3.2 Pickup rules (IMPLEMENTED — `WEAPON_RANK`, `addWeapon`)
Rank: pistol 1 · revolver/smg 2 · shotgun/rifle/flamer 3 · ak/sniper 4 · gatling 5 · golden 6 · RPG 9.

| Holding | Picks up | Result |
|---|---|---|
| Any gun | A **higher-rank** gun | Added, and you auto-switch to it (never if you are holding the RPG) |
| Any gun | A **lower/equal** gun | Added to a free slot, **no switch** |
| Any gun | A gun you already own | Ammo refill only, no forced switch |
| RPG | Anything | **Stays on RPG.** Never auto-switches away |
| RPG | RPG | **DUAL RPG** (`w.dual`): 2 rockets per shot, mag 6 |
| Dual RPG | RPG | Refill only |
| 5 slots full | Better gun | Replaces the **weakest** gun, never the RPG |
| 5 slots full | Gun no better than your weakest | Not picked up (stays on the floor, "SLOTS FULL" hint) |

### 3.3 RPG behaviour
- Splash damage clears groups of zombies.
- `OPEN:` self-damage from own RPG splash? Recommended for mobile: **none or very low**, because zombies swarm the player at close range.
- `OPEN:` does RPG ammo run out? If infinite, zombie pressure must scale up to keep it challenging.

### 3.4 Reload & magazines
- Reloads are fast; magazines are large (owner prefers this).
- Reload must never lock the player out of moving.

### 3.5 Melee, shove, accuracy (IMPLEMENTED 2026-10-25)
- **Right click / ✋ button / E = SHOVE.** Free, no ammo, 0.7 s cooldown. Cone in front, up to 8 bodies: pushes zombies back and **staggers** them (cannot bite, move at 12% speed for ~1 s). Bosses are immune to knockback. ADS moved to **hold Z** (touch: ADS button unchanged).
- **Hand weapons** (`melee: true` in `SHOOT_WEAPONS`): knife, machete, katana, bat, crowbar, shovel, axe, sledgehammer. Infinite, no reload, rank 0 (never auto-selected, never counted in the 5 gun slots). One hand weapon at a time, lives in the last slot; **key 6** or ⇄ selects it. A new pickup swaps it. Cone strike from the eye: `reach`, `arc`, `cleave` (bodies per swing), `knock`; crosshair target takes headshot x1.8; axe/sledge can gib.
- **Graze:** `traceShot` returns `graze` when the bullet line passes outside ~90% of the true hit sphere radius (touch: only the aim-assist band). Graze = 45% damage, no headshot bonus, pink `~12` number, ricochet "ping". Touch aim-assist stays generous, so it only trims the edge.
- **Accuracy:** `me.shots` / `me.hits` (rockets + flamer not counted) -> `ACC %` in the HUD. **Spread bloom** while holding the trigger (`me.bloom`), settles when you ease off.
- **FX realism:** impacts take the colour of the surface hit (chips + dust), pooled bullet-hole decals replace paint splats, dark-red blood spray carries along the bullet direction, muzzle gas jet + sparks, rocket smoke trail, explosions have fireball column, ground shockwave dust, flying debris and a layered sound (crack, boom, rumble, falling debris). Explosion ring/glow are now **pooled** (no materials created mid-fight).

---

## 4. Grenades

- **Always available**, never lost.
- To protect performance and keep it fun: grenades use a **charge system** — e.g. 3 charges, each recharges in a few seconds. `OPEN:` confirm numbers, or truly unlimited?
- Grenade button is a dedicated, always-visible touch button.
- Explosions use **pooled** effects (re-used objects), never create new objects per explosion.

---

## 5. Zombies & chaos

### 5.1 Modes
- **Horde / Incoming:** endless waves, rising count and speed.
- **Co-op boss:** team works together to kill a big boss panda zombie.

### 5.2 Zombie types (starting set)
| Type | Role |
|---|---|
| Walker | Slow, shambling, comes in big numbers |
| Runner | Fast, low health |
| Brute | Big, slow, tanky |
| Boss | Co-op target, large, attack patterns |
`OPEN:` add or remove types.

### 5.3 Chaos without breaking flow
Chaos comes from **numbers + explosions + ragdoll/knockback**, not from screen freezes or effects that drop frames.

---

## 6. Art direction — "grounded, not cartoon"

Applies to **zombies AND player pandas** (all characters get upgraded together, so styles match).

**Target look:** semi-realistic / stylised-realistic. Think a grounded survival game, not a kids' cartoon.

**Do:**
- Realistic proportions (less big-head / chibi)
- PBR materials (roughness, normal maps) — fur should look like fur, not flat colour
- Desaturated, muted colour palette; darker scenes; fog and atmospheric lighting
- Zombie pandas: matted / torn fur, dirt, exposed patches, cloudy eyes, hunched posture
- Zombie animation: shambling, limping, uneven gait, reacting to hits
- Player pandas: tactical gear (vests, straps, helmets), believable weapon holding
- Rim lighting so characters stay readable against dark backgrounds

**Don't:**
- Bright saturated primary colours
- Round, bouncy, smiling cartoon shapes
- Gore so heavy it's unpleasant for friends/students to play — keep it "horror-lite"

**Technical honesty:** procedural code-built geometry has a ceiling. For true realism, use proper 3D model files (GLB/GLTF) + rigged animations. `OPEN:` decide asset source (AI 3D generation, asset store, or commission).

---

## 7. Performance budget (mobile)

Starting targets — tune on a real phone:

| Item | Mobile target | Desktop target |
|---|---|---|
| Frame rate | 60 fps target, never below ~45 | 60 fps |
| Max active zombies | ~40 | ~80 |
| Max simultaneous explosions | ~6 | ~12 |
| Particles | hard cap, pooled | hard cap, pooled |
| Zombie rendering | instanced meshes / LOD | instanced meshes / LOD |

Rules:
- Object pooling for zombies, bullets, rockets, explosions.
- Dead zombies fade out and return to the pool.
- If frame rate drops, auto-reduce: fewer particles first, then fewer zombies — never freeze.

---

## 8. Regression checklist (check after every change)

- [ ] Game loads on mobile and desktop
- [ ] Touch controls: move, aim, shoot, grenade all work
- [ ] Picking up a lower weapon while holding RPG does NOT switch weapon
- [ ] RPG + RPG = dual RPG (HUD shows DUAL RPG, 2 rockets per shot)
- [ ] Hold left click next to a downed teammate revives them (progress bar fills)
- [ ] Squad panel shows every teammate's HP + kills; fonts readable on a phone
- [ ] Grenades still available after use / after death / new wave
- [ ] No freezes, hit-stop, slow-mo, or announcer
- [ ] Heavy horde + many explosions stays smooth on phone
- [ ] BEATDOWN unaffected

---

## 9. Report format (after every task)

1. **Fixed / built:** what changed
2. **Found but not fixed:** problems noticed, not addressed
3. **Unverified:** what could not be tested (e.g. real phone performance)
4. **Design questions:** anything that conflicted with this file or was `OPEN`

---

## 10. Decision log

Record every decided `OPEN` item here with a date, then update the section above.

| Date | Decision |
|---|---|
| 2026-10 | RPG is sticky; RPG + RPG = dual RPG (built 2026-10-25) |
| 2026-10 | Grenades always available |
| 2026-10 | All characters move to grounded semi-realistic style (NOT started) |
| 2026-10-25 | Hold left click (FIRE on touch) next to a downed mate = revive; range 3.6 m, same for bots; prompt and progress bar are big |
| 2026-10-25 | HUD minimum sizes: nothing important under 13 px on phones; squad panel shows each mate's HP, kills, medkits; "YOU KILLED" + "SQUAD KILLS" counters |
| 2026-10-25 | Touch action buttons are 54-104 px; 💣 and ➕ are big buttons in the cluster |
| 2026-10-25 | Squad AI teamwork: one bot revives (two for a human), the rest cover; focus fire; medkit run = medic + escort fetch a kit (`planMedRun`) |
| 2026-10-25 | Right click = shove (ADS -> hold Z); 8 hand weapons in slot 6; graze + accuracy + spread bloom; pooled realistic FX |
| 2026-10-25 | Music engine upgraded (kick-ducked bass/pads, shared room reverb, tempo-synced echo, detuned voices, risers/crashes); Shootout FFA/siege gets its own 3-song set + SUDDEN DEATH rush |
| 2026-10-25 | Music busy-fight mode + SFX voice budget (crackle with big hordes); 📞 Call the office: dynamite run in zombie mode (60 s cd, squad-safe) |
| 2026-10-25 | Squad bots tuned for waves 6+ (dmg tracks zombie HP, faster bursts/reload, nade resupply); Delta Force reinforcement at team-kill milestones (host-damage, scripted sweep) |
| 2026-10-26 | Layered onboarding after community feedback (surface is easy, depth arrives gradually): coach tips, late-unlocked touch buttons, ?fps=1 meter, auto clips for Delta/dynamite |
| 2026-10-26 | Visual pass: detailed zombies baked with grime, sharper post-processing, anisotropic filtering; physics fixes: blasts blocked by walls, dynamite lands on roofs, Delta only chases zombies it can see |
| 2026-10-26 | Horror zombies (no cute), stronger/faster stats + runner pounce; smarter squad (shove, kiting, retreat, idle reload, bot dynamite) |
