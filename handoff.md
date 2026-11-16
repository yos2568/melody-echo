# Melody Echo (เล่นตามทำนอง) · handoff

Built from `specs/new/23-melody-echo.md` with `specs/new/20-new-apps-shared.md` and `specs/v2/10-v2-shared-upgrades.md`
sections 1 to 4. One file, `index.html`, assembled by `node tools/build-new-apps.mjs melody-echo` from the parts in
`tools/new-apps/` (the parts are development-only and are not shipped).

## What was built

- Four modes: `grow` (3 lives, tempo rises every 3 rounds), `tune` (14 tunes split into phrases, tune card, song book),
  `blind` (no lights on try 1, first key outlined, needs sound), `free` (readout of the last note).
- Home, play, result, how-to, badges and song book screens; settings with theme, reduced motion, key labels, volume, clear.
- On-screen keyboard C4–C5 (8 white buttons, 5 black keys positioned by a CSS variable), pointer input on `pointerdown`,
  computer keys, and Enter/Space through the native button click.
- Playback engine on a `requestAnimationFrame` loop that only runs while something plays, using `performance.now()`;
  sounds are scheduled `lookaheadMs` ahead on the audio clock when it exists. Late items beyond `maxLateMs` are dropped.
- Self-check with 12 tests, independent of the editable CONFIG and zone A tables (it uses fixed spec values).
- Five slots, eleven events, all helpers named in the spec, plus the shared helpers.

## Deviations and decisions

1. **Added Thai strings** (all in `TEXT`, zone A): `tunePickLabel`, `learnedMark`, `readoutNone`, `statStars`, `tuneUnknownKey`,
   `tuneBadBeats`, `tuneTooShort`, `tuneSkipped`, `cntKeys`, `cntPools`, `cntTunes`, `cntModes`, `cntBadges`, `cntScreens`,
   `scoreLabel` (defined, unused), `badgeHad`, `badgeNot`, `badgeAllDone`, `extensionLine`, `backHome`, `storageOk`,
   `storageBlocked`.
2. **`tune` mode has no replay limit** (the spec gives limits only for `grow` and `blind`), so no counter is shown there.
3. **`tune` mode keys:** only the 8 white keys are active (all 14 tunes use white keys only); black keys are dimmed.
4. **Mystery tune numbers** are the position in `TUNES` (1 to 14), not the position inside the level, so a number does not
   change when another tune is learned.
5. **`CONFIG.seed`, `wrongPauseMs`, `slowFactor`, `keyFlashMs`** were added (the last three are pauses and the speed of the
   "shown slowly" playback after a failed phrase).
6. **Rounds counter:** `stats.rounds` counts every completed grow round and is saved at once, so the badge `firstEcho`
   (rule `roundsAtLeast`) is awarded at the end of the first game that finished at least one round.
7. **`gameEnd` also carries `total` and `firstTry`** (needed by the badge rule `tuneAllFirstTry` and by stars).
8. **Cut-off behaviour:** a tab hidden during playback cancels it and, on return, restarts it from the beginning without using
   a replay. Playback requested while the tab is hidden is queued the same way.
9. **Feature menu item 4 ("my sound")** has no UI in the app (it is an exam item); `Game.setWave` exists and is saved.
10. **`CONFIG.litRatio`** (0.7) makes a key stay lit for only that share of the note's sounding time, so two equal notes in
    a row blink separately (with the full length the gap was about 27 ms at 112 BPM and the lights merged).
11. **Black-key labels** show the note letter (`C♯`) even when labels are set to `โด เร มี` because black keys have no solfege.

## Tests run in this session (from `final-exam/`)

| Command | Result |
|---|---|
| `cd tests && node --test melody-echo-core.test.cjs melody-echo-ui.test.cjs` | 32 tests, 32 passed (8 core, 24 UI) |
| `cd tests && node --test new-apps-packaging.test.cjs` | 21 tests, 21 passed (7 for this app) |
| `node tools/melody-echo-browser.mjs` | passed: every mode with and without audio using real key presses, note spacing measured at 100 BPM, 66 screenshots |
| `node tools/new-apps-contrast.mjs` | passed: HTML and SVG text contrast in light and dark |
| `Game.runSelfCheck()` in jsdom and in headless Chrome | 12 of 12 |

The overall record, including the nine stale older tests that make `npm test` red, is in `REPORT.md`.

## What was not verified

- Real sound on speakers (tones were checked against a fake `AudioContext`; headless Chrome plays nothing audible).
- Touch feel and multi-touch on a real phone (pointer events were simulated).
- The school lab Windows PCs (key sizes were measured only in macOS Chrome at 320 px: white 40 px, black 27 px).
- Deployment to GitHub Pages or any host.
- Musical content: tune notes, rhythms, titles and composer credits (for example Spring, Largo, Minuet in G attributed to
  Petzold) are copied from the spec and were only checked for internal consistency (valid key ids, positive durations,
  `ode` = 15 notes / 16 beats). A musician must review them.
- Audio-clock timing accuracy: the pacing test measures when keys light on `performance.now()`, not when sound is emitted.
- Whether a free AI model can build feature-menu item 2 from the README in the exam window.
