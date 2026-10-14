# PokeDup: research pack for Codex

PokeDup is a personal Pokémon GO IV calculator. It is **iOS only** and runs **fully on-device**.
- **Input:** screenshots of the Pokémon screen and the appraisal screen.
- **IV bars:** read by pixel measurement, or by a lightweight non-neural classifier. No neural net.
- **CP/HP/stardust:** read with Apple Vision text recognition (on-device).
- **IV solve:** CP/HP formulas plus the CPM table.
- **Output:** results shown in a **custom keyboard extension** for the rename field. Data is shared with the app through an App Group.
- **No external AI or cloud APIs, ever.**
- **Distribution:** free Xcode Personal Team signing on the user's own iPhone.

## Files
- [PRD.md](PRD.md): full product requirements. §8 covers classifier and keyboard feasibility.
- [research-findings.md](research-findings.md): how Poke Genie works, open-source projects and licenses, Reddit threads, the screenshot-to-IV method.
- [sideloading.md](sideloading.md): installing without the $99 program, App Groups on a free account.
- [auto-scan.md](auto-scan.md): bulk scanning about 800 Pokémon, ToS considerations, repo/branch status.
- [chat-summary.md](chat-summary.md): conversation history, decisions, open items.

## Open questions & next steps for Codex
**Phase 0 spikes (test on device before Phase 1):**
1. Keyboard extension memory. The limit is roughly 30–70 MB (unverified). Can Vision `VNRecognizeTextRequest` and image decoding run inside the keyboard with Full Access? Poke Genie's developer said RAM limits stopped the keyboard from merging scan history ([source](https://www.reddit.com/r/TheSilphRoad/comments/bzoffy/how_to_use_new_poke_genie_ios_keyboard_for_iv/)).
2. App Groups on a free Personal Team. Apple's table says yes (verified). Build the app plus keyboard in Xcode with a shared `group.` container and confirm it works.
3. Can the keyboard read the latest screenshot from Photos with Full Access? The fallback is the main app processing it and writing results to the App Group.
4. Appraisal bar colors, segment geometry and calibration on the **current** game build, across device resolutions.
5. Nickname length limit (believed to be 12 characters, unverified) and the naming format.
6. Mapping arc dot angle to level (relative to trainer level); stardust cost to level range.
7. Feasibility of a ReplayKit broadcast extension for bulk scans (see auto-scan.md).

**Then:**
- Port TrainerKit's `packages/core/src/cp.ts`, `iv.ts` and `scan.ts` (MIT, code only) to Swift.
- Use base stats and the CPM table from pvpoke or alexelgt/game_masters, bundled offline.
- Check against GoIV's logic as a reference only. It is GPL-3.0, so don't copy its code.
- Write unit tests: known screenshots → expected IVs.
