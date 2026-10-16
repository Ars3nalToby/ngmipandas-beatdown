# NGMI Pandas — working notes

Single-file game: `index.html` (React 18 UMD + Babel standalone + Three.js r149 + MQTT). No build step. Deployed by Vercel from `main` (https://ngmipandas-beatdown.vercel.app/). Push to `main` = live.

## Map of index.html (grep these names; line numbers drift)

- `MATCH_SECONDS` (Beatdown clock) and `SHOOT_SECONDS` (Shootout clock): top of the script.
- Audio: `sfx` object (cases like `'call'`, `'dash'`, `'cannon'`), music via `musicTimer`, `kickAudio` global listeners.
- Story: `CHAPTERS` (5 entries, each with `mode`, `check(s)`, `intro`, `outro`), `storyEval(idx, st)`, `StoryComic`, `StoryResult`, `STORY_PROLOGUE`, `STORY_EPILOGUE`.
- Profile: `defaultProfile`, `loadProfile`, `saveProfile`, `awardXp`, key `PROFILE_KEY` (localStorage `ngmi_profile_v1`).
- Lobby: `App` component (screens: lobby, playing, shootout). `LobbyBackdrop`, `Showcase`, `ClipGallery`.
- Shootout: `ShootoutGame`. Match init `const match = { left: SHOOT_SECONDS ...}`, `endMatch`.
- Beatdown: effect inside `App` with `const match = { left: MATCH_SECONDS ...}`. Gang logic: `addMinionToFighter`, `spawnWildPanda`, `wildMinions`. Moves: `doDash`, `fireCannon`, `startTornado`, `doSmash`. Action flags: `actionTriggerRef.current.{jump,smash,dash,cannon,ult}`. The `call` flag is dead code.
- Mobile controls: the `[ {key:'jump'...}, {key:'smash'...} ]` array in the action cluster, plus the floating joystick and `releaseJoy()`.
- Desktop ability cards: the array with `keyLabel` (E, SHIFT, C, T).
- Clips: `createClipper`, `clipDB` (IndexedDB `ngmi_clips`), `clipStore`, `shareClip`, `downloadClip`.
- Results screens: Beatdown and Shootout results cards near the bottom of the JSX. Buttons: PLAY AGAIN, LOBBY.

## Rules for working here

- Compile check before pushing: extract the `<script type="text/babel">` block and run `@babel/standalone` `transform` with the `react` preset.
- Browser tests use Playwright with `/opt/pw-browsers/chromium` and CDN routing to `/tmp/bchk/node_modules`. Rendering is about 1 fps in the sandbox, so shorten clocks in a test copy rather than waiting for real match lengths.
- Never run the full game without a timeout. Run long tests in the background and poll their log.
- Keep dev notes and test artefacts out of the repo. Use the session scratchpad.
- Commit messages end with the attribution lines the session asks for.

## Known open items

- Real-device feel for dash, tornado, throw and the joystick is not verified.
- Grey-panda auto-recruit can grow gangs fast (5 to 8 in 4 seconds in tests).
- Story chapters 2–4 are Beatdown-only by design. Shootout is deferred until mobile shooting feels fun.
