# NGMI Pandas — working notes

Single-file game: `index.html` (React 18 UMD + Babel standalone + Three.js r149 + MQTT). No build step. Deployed by Vercel from `main` (https://ngmipandas-beatdown.vercel.app/). Push to `main` = live.

## Map of index.html (grep these names; line numbers drift)

- `MATCH_SECONDS` (Beatdown clock) and `SHOOT_SECONDS` (Shootout clock): top of the script.
- Audio: `sfx` object (cases like `'call'`, `'dash'`, `'cannon'`), music via `musicTimer`, `kickAudio` global listeners.
- Story: `CHAPTERS` (5 entries, each with `mode`, `check(s)`, `intro`, `outro`), `storyEval(idx, st)`, `StoryComic`, `StoryResult`, `STORY_PROLOGUE`, `STORY_EPILOGUE`.
- Profile: `defaultProfile`, `loadProfile`, `saveProfile`, `awardXp`, key `PROFILE_KEY` (localStorage `ngmi_profile_v1`).
- Lobby: `App` component (screens: lobby, playing, shootout). `LobbyBackdrop`, `Showcase`, `ClipGallery`.
- Shootout: `ShootoutGame`. Match init `const match = { left: SHOOT_SECONDS ...}`, `endMatch`. Variants: `ffa`, `siege`, `zombie` (lobby `soVariant`; zombie sub-mode `soZMode` = `raid` | `incoming`, passed as `zmode`).
- Weapons: `SHOOT_WEAPONS` (incl. `sniper` with `pierce`, `flamer` with `flame`/`range`, `revolver`), pickups `SHOOT_PICKUPS`, models in `makeGunModel`.
- Zombie mode: constants `ZOMBIE_KINDS`, `ZKIND_IDS`, `ZOMBIE_RAID_WAVES`, mesh `buildZombieMesh` (shared geometry, pooled per kind). Inside `ShootoutGame` search `ZOMBIES: the host runs the horde`: `zd` director state (phases `boot/prep/wave/clear/over/won`, horde 0–3), `zTick` (host only), `updateZombie` (host AI), `animZombies` (everyone), `hurtZombie` → `applyZDamage` → `killZombie`, boss `bossAct`/`throwRock`/`slamFx`, `bloaterPop`. Downed/revive: `goDown`, `tickDowned`, `reviveFighter` (owner-side: you on your client, bots on the host). Net: host packet carries `zs`/`zd`/`zfx`; clients send `zh` (damage), host sends `zk` (kill + credit); `dw`/`rv` on player packets. Zombie rooms use their own MQTT topic (`.../shootout/v1/zombie/ROOM`). Squad is capped at 4 (bots retire as friends join).
- Zombie map: `buildShootoutMap(scene, cinematic, zmap)` — with `zmap` the perimeter grows to ±86, six buildings become walk-ins (`WALK` table) and six new outskirts buildings are added via `walkIn(...)`. It returns `rooms`/`portals` (door graph used by `navStep`/`regionOf`), `ladders` and `zspawns`. Colliders can be switched off with `c.off` (barrels); `moveBody` stops heads at ceilings.
- Zombie extras inside `ShootoutGame`: corpses (`toCorpse`, pooled meshes), `gibFx`, `splat` decals, spitter acid (`acids`, `throwRock(..., 'spit')`), grenades/medkits (`doThrow`, `throwables`, net `th`/`mkx`/`heal`), explosive barrels (`barrels`, `detonateBarrel`, net `bx`), callouts (`callout`, `CALLS`, speech via `voiceRef`, packet `cl`), hold-fire revive (`me.revTo`, packet `rt`), squad AI `updateSquadBot`, minimap `drawMini` (canvas `miniRef`), bottomless mags `world.frenzyT`. Raid = `ZOMBIE_RAID_WAVES` (10), bosses on waves divisible by 5.
- Audio: `sfx.play(name, vol, p, pan, muffle)` routes through a stereo panner + lowpass when `pan`/`muffle` are given (`curBus`). In `ShootoutGame` use `play3d(name, pos, vol, pitch, range, occlude)` / `spatial(pos)`. Zombie sounds: `zStep` (footsteps synced to the walk cycle, budget 4/frame), `zbreath`, `zswipe`, `hordeRumble` from `zd.hordeAt`. Music has themes: `sfx.startMusic('zombie')` plays `ZTRACKS`/`ZRUSH` via `scheduleHorror`; everything else uses the original `TRACKS`. Zombie mode drives `sfx.setIntensity` from the nearest zombie.
- Litter (zombie map, cosmetic, local only): `LITTER` kinds as InstancedMesh, `tickLitter` kicks items when players/zombies walk through (`kickT` cooldown).
- Beatdown: effect inside `App` with `const match = { left: MATCH_SECONDS ...}`. Gang logic: `addMinionToFighter`, `spawnWildPanda`, `wildMinions`. Moves: `doDash`, `fireCannon`, `startTornado`, `doSmash`. Action flags: `actionTriggerRef.current.{jump,smash,dash,cannon,ult}`. SMASH/DASH/THROW go through an input buffer (`pressBuf`, `INPUT_BUFFER_MS`) so a tap stays live for ~0.3 s. The `call` and `swarm` flags are dead code.
- GPU memory: gang/wild panda meshes come from `takePanda` and go back with `releasePanda` (pooled). Use `disposeObj` for anything else you remove from the scene.
- Multiplayer (Beatdown): `ROOM_SLOTS` = 8 fighters; humans first, the host (smallest peer id) fills the rest with bots via `botsWanted` and retires bots as friends join. `livePeers()` = peers seen in the last 5 s. The MQTT client is kept across rounds (`client.__topic`, `onNetMessage` swapped per round) and closed when leaving the playing screen. Party matches (`results.party`) auto-start the next round after 12 s (`nextRound`). Lobby presence: `useRoomPresence`; shared loader `loadMqtt`, broker `MQTT_URL`. Name and room are saved in localStorage (`ngmi_name`, `ngmi_room`).
- Multiplayer testing: run a local aedes broker over ws, point `MQTT_URL` at it in a test copy, and gate `renderer.render` behind a flag so several pages run at real speed.
- Network input is untrusted: clamp numbers with `netNum`, room codes with `cleanRoom`. Story chapters never connect to MQTT (`storyMode`).
- Mobile controls: the `[ {key:'jump'...}, {key:'smash'...} ]` array in the action cluster, plus the floating joystick and `releaseJoy()`.
- Desktop ability cards: the array with `keyLabel` (E, SHIFT, C, T).
- Clips: `createClipper`, `clipDB` (IndexedDB `ngmi_clips`), `clipStore`, `shareClip`, `downloadClip`.
- Results screens: Beatdown and Shootout results cards near the bottom of the JSX. Buttons: PLAY AGAIN, LOBBY.

## Rules for working here

- Work directly on `main` and push when done (owner's standing preference: no feature branches, he tests on the live site). Run the compile check and a browser smoke test first, since push = live.

- Compile check before pushing: extract the `<script type="text/babel">` block and run `@babel/standalone` `transform` with the `react` preset.
- Browser tests use Playwright with `/opt/pw-browsers/chromium` and CDN routing to `/tmp/bchk/node_modules`. Rendering is about 1 fps in the sandbox, so shorten clocks in a test copy rather than waiting for real match lengths.
- Never run the full game without a timeout. Run long tests in the background and poll their log.
- Keep dev notes and test artefacts out of the repo. Use the session scratchpad.
- Commit messages end with the attribution lines the session asks for.

## Known open items

- Real-device feel for dash, tornado, throw and the joystick is not verified.
- Gang growth was tuned down (9 starting grey pandas, pickup radius 2.6, Tornado tops up to 5). Needs real-play feedback.
- iPhone has no vibration API, so `buzz()` only works on Android.
- Story chapters 2–4 are Beatdown-only by design. Shootout is deferred until mobile shooting feels fun.
- Zombie mode balance (wave sizes, boss HP, horde size) is only bot-tested. Needs real squad feedback, especially on phones (zombie cap `ZMAX` is 16 on touch, 26 on desktop).
