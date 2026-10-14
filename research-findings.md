# Research findings

## Poke Genie (reference product), from public sources only
- iOS IV calculator. It reads screenshots with OCR (name, CP, HP, upgrade cost) and measures the CP arc "with pixel level precision". Works offline. ([GO Hub](https://pokemongohub.net/post/featured/poke-genie-review-iv-calculator-ios/), [r/pokemongodev](https://www.reddit.com/r/pokemongodev/comments/4zfyvy/implementation_poke_genie_for_ios_auto_iv/))
- Keyboard: needs Full Access to read the latest screenshot, sync with the app, use the clipboard and fetch stat updates. IVision uses iOS Screen Recording. ([privacy policy](https://pokegenie.net/privacypolicy-I/index.html))
- Its keyboard could not merge with the app's scan history "due to iOS RAM constraints" (developer tweet, 2019, quoted in [this thread](https://www.reddit.com/r/TheSilphRoad/comments/bzoffy/how_to_use_new_poke_genie_ios_keyboard_for_iv/)).
- Reverse-engineering was **dropped from scope**. Don't decompile anything; that would raise EULA/DMCA issues.

## Open-source projects (licenses from the GitHub API; activity dates as of Oct 2026)
| Project | Platform | License | Last push | Reads bars | Method | Bars + CP math | Reuse |
|---|---|---|---|---|---|---|---|
| [NspxMiguel/TrainerKit](https://github.com/NspxMiguel/TrainerKit) | Web / React Native (TS) | MIT, code only (game data excluded) | 2026-10-06 | Yes | Pixel measurement, `scanAppraisalBars()` in `packages/core/src/scan.ts` ("no model") | Partly: `iv.ts` `matchesAppraisal()` | **High**: cp.ts, iv.ts, scan.ts. Skip its `groq.ts` (cloud AI) |
| [farkam135/GoIV](https://github.com/farkam135/GoIV) | Android (Java) | GPL-3.0 (`LICENSE.md`) | 2022-07-11 | Yes | Pixel colors, `AppraisalManager.java` | Yes (not verified line by line) | Reference only: `ArcView.java`, `OcrHelper.java`, `LevelRange.java` |
| [widmonstertony/pogo-ipad-renamer](https://github.com/widmonstertony/pogo-ipad-renamer) | Python on macOS → iPad | **None** | 2026-10-09 | Yes | Pixel measurement, `appraisal_cv.py`, `appraisal_decoder.py` | No, bars only (not verified) | Algorithm ideas only; no license |
| [tmwilder/pogoiv](https://github.com/tmwilder/pogoiv) | Python library | MIT | 2017-01 | No | n/a | Library for CP/IV math | Math reference |
| [dnafigator/pogo-autorenamer](https://github.com/dnafigator/pogo-autorenamer) | adb + OCR | ISC | 2019-09 | Old appraisal text | OCR | Not checked | Low |
| [danielcodin/SnapIV](https://github.com/danielcodin/SnapIV) | iOS (ObjC + Tesseract) | None | 2016 | Not checked | — | — | Dead |
| [novemberalpha/rowan](https://github.com/novemberalpha/rowan) | ADB + Gemini | None | 2026-08 | Yes | **Cloud AI**: excluded | No | None |
| [pvpoke/pvpoke](https://github.com/pvpoke/pvpoke) | Web | MIT | 2026-10-12 | — | — | — | Base stats data (owned by Niantic) |
| [alexelgt/game_masters](https://github.com/alexelgt/game_masters) | Data | "Educational use only" notice | — | — | — | — | GAME_MASTER source for stats and CPM |

No open-source Pokegenie clone or iOS keyboard tool was found. The searches were not exhaustive: r/pokemongo and GitHub code search were not done.

## Reddit threads (all 2016–2019)
- [Autorenamer](https://www.reddit.com/r/TheSilphRoad/comments/d4gi7k/autorenamer_when_you_can_not_sleep_in_the_morning/)
- [PoGoIV Android](https://www.reddit.com/r/TheSilphRoad/comments/4trry4/android_iv_calculator_app_pogoiv/)
- [First appraisal calculator](https://www.reddit.com/r/TheSilphRoad/comments/4zd3tm/first_calculator_to_utilise_appraisal/)
- [aggixx IV sheet](https://www.reddit.com/r/TheSilphRoad/comments/4tkk75/updated_iv_calculator_automatically_calculate_ivs/)
- [Poke Genie keyboard](https://www.reddit.com/r/TheSilphRoad/comments/bzoffy/how_to_use_new_poke_genie_ios_keyboard_for_iv/)

## Screenshot-to-IV method
1. **OCR** CP, HP and stardust cost with Vision `VNRecognizeTextRequest` from fixed crop regions, scaled per resolution.
2. **Level from the arc:** the angle of the white dot maps to level relative to trainer level (max trainer level + 10 at level 50). *Unverified*, so calibrate.
3. **Level from stardust:** the power-up cost gives a small range of levels. *Unverified tables*; take them from game data.
4. **Formulas** (standard, matching TrainerKit `cp.ts`):
   - CP = max(10, ⌊(A+a)·√(D+d)·√(S+s)·CPM²/10⌋)
   - HP = max(10, ⌊(S+s)·CPM⌋)
   - Half-level CPM = √((CPM_lo²+CPM_hi²)/2)
5. **Brute force:** for each candidate level, try all 16³ = 4,096 IV combinations and keep those matching CP and HP.
6. **Filter** by the appraisal bars (each 0–15, exact when read cleanly) or by the star/total range.
Caveats: crop positions, bar colors and arc geometry change with game builds and devices. Validate every bar reading against the CP formula.
