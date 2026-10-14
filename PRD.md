> Copied from `/workspace/pokegenie-clone-prd.md` (Oct 13, 2026). Content is unchanged apart from this note. See research-findings.md and sideloading.md for later verification (App Groups are confirmed to work on a free account).

# PRD / Working Plan: On-Device IV Scanner + Rename Keyboard for Pokémon GO (iOS only)

*Version 0.2 — 13 Oct 2026 (CT). Author: Sambhav L. Status: Draft for review.*
*Working name: "IV Keys" (placeholder; avoid "Poke Genie"/"Pokémon" in the app name — App Review 4.1(c), 5.2.1).*

> **Hard rules for this product**
> 1. **iOS only.** No Android build, IME or overlay work.
> 2. **No external AI APIs and no cloud services.** Every bit of image analysis, IV math and name generation runs on the device. No server, no analytics SDK, no ad SDK, no login. Game data ships inside the app bundle.

---

## 1. Background: how Poke Genie works (reference product)

Poke Genie (Poke Genie Inc., free with IAPs such as "Keyboard Pro" and "Scan Pro") is the best-known Pokémon GO IV calculator. What the sources show:

- **Input is screen pixels only.** No game API, no login. The developer says it "gets all necessary information directly and only from the screenshots you provide" and works offline [S1, S2, S3].
- **iOS capture.** The user takes a normal screenshot of the Pokémon detail or appraisal screen. They then either import it in the app, or open the rename field in Pokémon GO and switch to the **Poke Genie Keyboard**, which reads the **latest screenshot** [S2, S4, S5, S6]. Newer builds also have **"IVision"**, which uses iOS Screen Recording (a broadcast extension) for live detection [S5, S11].
- **Android capture, for context only.** A floating overlay button triggers a screen capture through the "screen capture" permission (MediaProjection) [S7, S8, S9].
- **Reading the image.** It recognizes the name, CP and power-up (stardust) cost. The developer says it measures the CP **arc angle "with pixel level precision"** to get the level [S3]. It also auto-detects the appraisal screen to narrow results [S8]. Its exact OCR or template method is **not published**.
- **Output.** All possible IV combinations, the IV range and %, PvP IV ranks for Great/Ultra League, plus power-up and evolution simulators and a scan history [S2, S8].
- **Rename.** The **Name Generator** fills a user template (IV %, the A/D/S combination often written as circled digits like ⓪⑭⑮, league, rank, level, custom text). It can auto-copy the name to the clipboard on each scan, and the user pastes it into Pokémon GO's nickname field [S10, S12, S13]. The iOS keyboard needs **Full Access** to read the latest screenshot, sync scan data with the app, use the clipboard, and fetch stat updates [S5].
- **Free tier.** Community posts say the keyboard has a free scan limit (reported as 50) before Keyboard Pro is needed. *Not confirmed by Poke Genie itself.*

---

## 2. Problem & goals

**Problem.** IVs (0–15 each for Attack, Defense and Stamina) are hidden. To decide what to keep, power up, trade or transfer, players must appraise each Pokémon, read the bars, look up PvP ranks and then hand-type a code into the 12-ish-character nickname. *The exact nickname limit is unverified; confirm it during development.* Doing this for hundreds of Pokémon is slow.

**Goals**
- G1. Exact IVs for any Pokémon from **one screenshot**: the appraisal screen gives exact IVs; the detail screen alone gives candidate combinations.
- G2. A **rename keyboard** that shows the stats by number and inserts a formatted name into Pokémon GO's rename field in one tap, without leaving the game.
- G3. Work across **many species and IV spreads**, with batch-friendly flows: swipe between Pokémon while appraising, screenshot each one.
- G4. **100% on-device.** No network needed to work, and no data leaves the phone.

**Non-goals.** Remote raids, battle simulator, account login, reading game memory or network traffic, automated tapping in the game, Android.

## 3. Personas
- **Casual collector (Maya).** Wants to know "keep or transfer?" and to tag her best ones. Values simplicity and privacy.
- **PvP grinder (Dev).** Hunts high-rank Great/Ultra League spreads. Wants rank, stat product % and the best league in the nickname.
- **Batch sorter after events (Lee).** Community Day aftermath: 100+ catches. Needs speed, good batch screenshot handling and low error rates.

## 4. Scope

| | MVP (v1) | Later |
|---|---|---|
| Input | Screenshot of the **appraisal screen** (bars + CP + HP visible); detail screen (CP/HP/dust/arc) | Live broadcast-extension scanning (an "IVision"-style feature) |
| Classifier | Pixel-based bar reader; Vision OCR on small crops for CP/HP/dust/name; arc-dot level estimate | Digit template matcher, so Vision is not needed in the keyboard |
| Output | Exact IVs (or a candidate list), IV %, level, star tier, GL/UL/ML rank and stat product % | Power-up / evolution / purification simulator; best-buddy handling |
| Keyboard | Shows the latest result; one-tap insert of the template name; candidate picker; Globe/next-keyboard key; minimal typing keys | Multiple templates, history browser, batch queue |
| Templates | Presets + custom tokens | Search-string helpers |
| Data | Bundled base stats and CPM table; updates ship as app updates | User-importable data file (local file, still no network) |

---

## 5. Key user flows (iOS)

**Flow A — Keyboard reads the latest screenshot (main flow, needs Full Access + Photos permission)**
1. One-time setup: install the app → Settings → General → Keyboard → Keyboards → Add → enable **Allow Full Access** → grant Photos access the first time it is asked for. Set trainer level in the app (needed for the arc).
2. In Pokémon GO, open a Pokémon → Appraise → take a screenshot.
3. Tap the name → pencil → switch to the IV Keys keyboard (Globe).
4. The keyboard fetches the newest screenshot from Photos, runs the classifier and shows `A 14 · D 15 · S 13 · 93% · L 27.5 · GL #12`.
5. Tap **Insert** → the keyboard deletes the old name and inserts e.g. `Swa⑭⑮⑬G12`, then the user taps Done.
6. Next Pokémon: swipe in the appraisal screen → screenshot → the keyboard refreshes.

**Flow B — Share/Import, then the keyboard reads the result (works WITHOUT Full Access)**
1. Take a screenshot → tap the thumbnail → Share → "IV Keys" **Share Extension**, or import in the main app (photo picker).
2. The extension or app classifies the image and writes the result JSON to the **App Group** container.
3. In Pokémon GO, switch to the keyboard → it reads the latest result from the App Group (read-only access works without Full Access) → Insert.

**Flow C — Pasteboard image (fallback, needs Full Access)**
The user copies the screenshot image, for example from the screenshot editor or Photos "Copy". The keyboard shows a system **UIPasteControl**, the user taps it, and the keyboard classifies the pasted image.

**Flow D — Ambiguous result.** From the detail screen only, the keyboard shows candidate spreads, for example 4 combinations. The user can pick one, insert a range such as `91-96%`, or take an appraisal screenshot to get exact values.

---

## 6. Functional requirements

**Capture / input**
- FR1. Fetch the most recent screenshot (PHAsset `mediaSubtype` contains `.photoScreenshot`, sorted by `creationDate`, `fetchLimit = 1`) from inside the keyboard when Full Access and Photos permission are both granted [A4, A6].
- FR2. A Share Extension accepts an image and classifies it. The main app has a photo picker import (`PHPickerViewController`, which needs no library permission) [A7].
- FR3. Ignore screenshots older than N minutes (default 10) or ones already processed (keep the asset ID in the App Group).

**Recognition** (details in §8)
- FR4. Classify the screen type: appraisal / detail / unknown.
- FR5. Appraisal: read the Attack/Defense/HP bars to integer IVs 0–15, and read the star tier.
- FR6. Read the CP, the HP "x / y", the power-up stardust cost, and the name text. Estimate the level from the arc dot using trainer level.
- FR7. Identify the species: from the OCR'd name if it is not nicknamed. Otherwise use the stored mapping from the last rename, or ask the user (species picker with search). Handle forms (Alolan, Galarian, etc.) through the picker.
- FR8. Cross-validate with the CP formula (§8.6). Report a confidence level and never show an "exact" result that is inconsistent.

**Calculation**
- FR9. Enumerate all (level, A, D, S) combinations consistent with CP, HP, the dust bracket and the arc. Intersect them with the appraisal bars or star tier when present.
- FR10. Apply IV floors by origin when the user sets it (wild 0, weather-boosted 4, raid/egg/research 10, lucky trade 12, etc.) [S16].
- FR11. PvP: for GL (1500) and UL (2500), compute the best level ≤ cap (L50/51), the stat product and the rank among all 4,096 spreads. ML: IV %.
- FR12. Shadow/purified flag. Purification adds +2 to each IV, capped at 15 (needs verification against a primary source).

**Keyboard & naming**
- FR13. The keyboard shows the latest result card, the inserted-name preview, Insert, Copy, Delete-name, a minimal ABC/123/symbol layout and a next-keyboard key (required) [A1, A3].
- FR14. Insert works through `textDocumentProxy.insertText`. Clearing the old name works through `deleteBackward`, repeated based on `documentContextBeforeInput`. If the host does not report context, fall back to a fixed-count delete [A1].
- FR15. Template engine with tokens: `{name:N}`, `{iv%}`, `{a}{d}{s}` (plain / circled ⓪–⑮ / negative-circled ⓿–⓯), `{lvl}`, `{gl_rank}`, `{ul_rank}`, `{league_best}`, `{stars}`, `{shadow}`, literal text. Live length counter against the nickname limit; auto-shorten in priority order.
- FR16. A local scan history (App Group SQLite/JSON) shared by the app and the keyboard.

**Privacy & compliance**
- FR17. No networking code in any target. Leave out `NSAllowsArbitraryLoads`, and audit that no networking APIs are linked in. Privacy nutrition label: "Data Not Collected".
- FR18. The keyboard stays useful without Full Access: typing keys, plus reading results through Flow B (App Review 4.4.1) [A8].
- FR19. No ads or IAP UI inside the keyboard (App Review 2.5.18, 4.4) [A8].

---

## 7. Technical considerations (general)

- **Stack.** Swift, UIKit keyboard (`UIInputViewController`), SwiftUI app, Share Extension. Shared Swift package `IVCore` holds the math, data and classifier, built for both small memory (keyboard) and full (app) use.
- **Game data.** Base Attack/Defense/Stamina per species and form, plus the CPM table (levels 1–51 in half steps), **bundled offline**. Candidate sources: the PvPoke `gamemaster.json` (MIT-licensed repo, actively updated) [D1] and the community-extracted Game Master dumps (PokeMiners; the repo has no declared license, so check before use) [D2]. The CPM table is also published by GO Hub [S14]. A build-time script turns this into a compact binary table, under 300 KB. Data refreshes ship as **App Store updates**: no runtime downloads, per the no-cloud rule.
- **Pokémon GO rename UI.** The nickname field is a normal (not secure) text field. Poke Genie's keyboard is proof that third-party keyboards work there [S4, S5]. Risks: the game could block custom keyboards (`application(_:shouldAllowExtensionPointIdentifier:)` returning false) or make the field secure. Either would disable the keyboard; Flow B with Copy is the fallback [A1].
- **ToS.** Pokémon GO is now published by Scopely (Niantic until 2025, per Serebii) [S15]. The help-center Player Guidelines forbid "modified or unofficial software" and cheating, and allow suspension [S17]. Screenshot-only tools without login or API access are widely used and treated as low risk by the community [S1, S3]. **There is no explicit official endorsement.** Mitigations: no game automation, no memory or network reading, no credentials, user-initiated screenshots only.
- **Trademarks.** Avoid Pokémon artwork and sprites. Use species names as data only. Add a disclaimer saying the app is not affiliated with the game's owners.

---

## 8. On-device classifier & iOS keyboard extension feasibility

### 8.1 Design principle
Use **deterministic, non-neural pixel methods first**, and Apple's on-device Vision text recognition only for digits and names. No custom ML model unless validation shows it is needed. Even then, use a tiny Core ML digit classifier under 200 KB. Everything runs locally. Vision's text recognition runs on the device [A9, A10].

### 8.2 Screen-type detection
- Downsample to a working width of 360 px for detection. Keep the native-resolution crops for measurement.
- **Appraisal screen:** the lower part of the screen has a white/light card. Inside it are three horizontal bar tracks, gray with orange or red fill, stacked at regular spacing, plus a star row. Detect the card by row-wise brightness profile. Detect the bars by scanning rows for long runs of "track gray" or "fill orange/red" pixels within a hue/saturation window.
- **Detail screen:** big CP digits near the top, a semicircular arc around the model, a "Power up" row with a stardust icon and cost.
- Output the type plus a confidence. If unknown, tell the user "take the screenshot on the appraisal screen".

### 8.3 Reading the IV bars (core, non-neural)
Facts: each of the three bars (Attack, Defense, HP) is split into **3 segments of 5 IV points**. The fill moves in steps of 1/15 of the bar, and a full bar (15) is shown in a different color (red) [S16a, S16b, S18].

Algorithm per bar:
1. **Locate the track.** For each candidate row band, find the leftmost and rightmost pixels belonging to the track or fill. Use rounded-end tolerance: take the extent at the band's vertical center row, then average 3–5 rows.
2. **Calibrate.** Detect the **two segment dividers** (thin vertical gaps or lighter lines) at about 1/3 and 2/3. The dividers give an internal ruler, so the reading does not depend on scale or density (@2x/@3x, any device).
3. **Measure the fill.** Find the run length of "fill" pixels from the left edge. Use a color-distance threshold in Lab or HSV against a fill-color prototype sampled at runtime from the leftmost filled pixels (if any). This adapts to dark mode or tone-mapped screenshots.
4. **Classify.** `iv = round(15 × fill_len / track_len)`. The divider positions give 5 and 10 exactly, which helps snap neighbors. With a typical bar of about 250–330 px on modern iPhones, each IV step is about 17–22 px, so ±3 px of anti-aliasing never changes the result.
5. **The 15 check:** fill reaches the right end **and** the fill hue matches "red" rather than "orange" → 15. A full-length but orange fill → error or re-check.
6. **0:** no fill pixels found but the track is present → 0.
7. **Alternative / backup classifier:** compute a 1-D profile vector of the bar (fill-probability per column, resampled to 60 bins). Classify it by **nearest neighbor** against 16 stored reference profiles (0–15), built from calibration screenshots. That handles odd rendering without heuristics drifting.

*Exact colors and pixel geometry in the current game build must be sampled from real screenshots during calibration. Third-party hex values (e.g., Hundo Hunter's "#E64545") are unverified [S18].*

### 8.4 Reading numbers & text
- **Crop first**, using layout anchors found in 8.2: the CP area above the model, HP text below the HP bar, the dust cost in the power-up row, the name under the model.
- **Option 1 (MVP): Apple Vision `VNRecognizeTextRequest`.** Use `.accurate` in the app/Share Extension and `.fast` in the keyboard, with `usesLanguageCorrection = false` for digits. The species list as `customWords` with correction on applies to the name crop only. On-device, no network [A9, A10]. Parse with regexes: `CP\s*(\d+)`, `(\d+)\s*/\s*(\d+)\s*HP`, dust from a whitelist of valid costs.
- **Option 2 (later / keyboard-light): digit template matching.** Binarize the crop (Otsu), split connected components into glyphs, normalize each to 16×24. Classify against stored glyph templates of the game's font by normalized cross-correlation or kNN on the pixel vector. The digits are one font in a few sizes, so 10 templates × 2–3 sizes is enough. This avoids loading Vision in the keyboard.
- **Dust cost → level bracket.** The cost per power-up is fixed per level band, so it narrows the level to a few half-levels (GO Hub dust table) [S14]. Lucky (half cost), shadow (+20%) and purified (−10%) change the cost. Detect a lucky background or shadow marker, or ask the user.
- **Arc → level.** Find the white dot on the semicircle (bright, near-white blob along an arc fitted from the arc track pixels). Angle θ ∈ [0, π] → fraction of the way from level 1 to the max level for the trainer's level. *The exact mapping (max = trainer level + 2, capped at 50/51, and whether it is linear in CPM) must be checked against real screenshots.* Poke Genie needs trainer level for this [S3, S8].

### 8.5 Device resolutions & aspect ratios
- Never hard-code pixel coordinates. Locate anchors (card edges, bar tracks, arc) by search, and express regions as fractions of the detected card or screen width.
- Expect native screenshot sizes: notch/Dynamic Island iPhones (about 19.5:9, @3x), SE-class (16:9, @2x), iPad (4:3; Pokémon GO's iPad layout may differ, so test it). Bars are measured against their own dividers, so scale doesn't matter.
- Calibration set: at least one appraisal and one detail screenshot per device class, at 0/5/10/15 fills. Keep per-layout parameter sets keyed by aspect bucket. Let users run "Calibrate with a known Pokémon" as a backup.
- Robustness: downscaled shares (Share Sheet may give a reduced JPEG), HDR or Night Shift tint, the screenshot markup overlay. Reject crops whose quality score is too low.

### 8.6 Accuracy & validation against the CP formula
- `CP = max(10, floor((Atk+A) × sqrt(Def+D) × sqrt(Sta+S) × CPM(L)² / 10))` [S14]. `HP = max(10, floor((Sta+S) × CPM(L)))`. *The min-10 floors are standard community formulas; verify them.*
- After the bars give (A, D, S), search L in the dust bracket (or all 1–51) for an exact CP **and** HP match. One match gives an exact, verified result; that L is the level.
- No match means a misread somewhere. Try neighbors: each IV ±1 (with 3 stats this is 27 combinations), CP OCR alternates (top-3 Vision candidates). Pick the combination that satisfies the formula, and mark it "corrected".
- Still no match: show "can't verify" plus the detail-screen candidate list. Never insert silently.
- Detail screen only: enumerate all 4,096 IV spreads × levels in the bracket, keep those matching CP + HP (and the arc level), intersect with the star tier if known. Show the range and count.
- **Targets:** bar-read exact accuracy ≥ 99.5% per stat. CP/HP OCR ≥ 99.5%. End-to-end verified-exact rate ≥ 98% on appraisal screenshots. False "exact" (wrong but shown as exact) < 0.1%.
- **Validation set:** 500+ labeled screenshots across 6+ device models, all 16 values per stat, light/dark, lucky/shadow/purified, a regression suite run in CI (XCTest) on stored PNGs.

### 8.7 Keyboard extension: what it can and cannot do (Apple docs)

| Question | Answer | Source |
|---|---|---|
| Can a keyboard work on its own? | It must be shipped inside a containing app, and must work **without** Full Access (App Review 4.4.1). It cannot launch other apps (only Settings). | A1, A8 |
| Can it capture the screen or read the host app's UI? | **No.** It draws only inside its own view, cannot see or select host text beyond `textDocumentProxy` context, and has no screen-capture API. Live capture is only possible through a separate ReplayKit broadcast extension started by the user (Later scope). | A1 |
| Default sandbox (no Full Access) | No network, **can read but not write** the App Group shared container, no pasteboard. | A1, A2 |
| With `RequestsOpenAccess = YES` + user "Allow Full Access" | Shared container read/write, `UIPasteboard`, **Camera Roll/Photos access (with its own permission prompt)**, network (we won't use it), Location/Contacts. Check at runtime with `hasFullAccess`. | A1, A2, A3 |
| Photo library from the keyboard | Possible with Full Access + `NSPhotoLibraryUsageDescription` in the extension's Info.plist (developers report crashes without it). Poke Genie's keyboard does exactly this ("access the latest screenshot"). | A1, A6, S5 |
| Pasteboard images | Needs Full Access. Since iOS 16 a programmatic read shows an "Allow Paste" prompt. A user tap on **`UIPasteControl`** avoids it. | A1, A11, A12 |
| Secure fields / phone pads | The system swaps in the Apple keyboard; custom keyboards can't type there. Host apps can ban custom keyboards entirely. | A1 |
| Memory limit | **Not officially published.** Apple says only that extensions have "a much lower per-process memory limit than foreground apps". Developers report about 48–77 MB depending on device/OS (KeyboardKit: ~60–70 MB; others ~48–66 MB). Exceeding it means a silent jetsam kill. | A13, C1, C2, C3 |
| Vision / Core ML in the keyboard | No documented ban. Feasibility depends on memory. Pixel bar-reading on a downsampled crop costs < 5 MB. A full-res screenshot decode is ~1290×2796×4 ≈ 14 MB, so decode **cropped or downsampled regions only** with ImageIO (`CGImageSourceCreateThumbnailAtIndex` / crop-on-decode), and release them right away. Vision `.fast` on a few small crops is *probably* feasible; this is **unverified, so a spike is needed (Phase 0)**. A tiny Core ML digit model (<200 KB) is cheap. | A9, A13, C1 |
| Ads / IAP in keyboard | Not allowed. | A8 |

### 8.8 Architectures (ranked)
1. **Hybrid (recommended).** The keyboard runs the **lightweight pixel classifier + template or Vision-fast digit OCR** on the latest screenshot when it has Full Access + Photos (Flow A). Without Full Access, or if the in-keyboard scan exceeds a memory/time budget (target < 35 MB peak, < 700 ms), it shows results that the **main app or Share Extension** wrote to the App Group (Flow B). The Share Extension has a much larger budget and can use Vision `.accurate`.
2. **App/Share-Extension-only classification.** The keyboard is a pure reader of App Group results. It needs no Full Access and no Photos access in the keyboard, so it's the most private and App-Review-safe. It adds 2–3 taps per Pokémon.
3. **Pasteboard image.** Copy the screenshot → tap the UIPasteControl in the keyboard. Needs Full Access. Clunkier than Flow A, and not every route gives a "copy image" action (iOS screenshot editor options vary by version; verify).
4. **Live broadcast extension (Later).** ReplayKit Broadcast Upload Extension (an "IVision"-style feature) analyzes frames while the user browses and writes results to the App Group. It needs explicit start and a visible recording indicator (App Review 2.5.14). Broadcast extensions also have tight memory limits.

### 8.9 Phase 0 spikes (must pass before Phase 1)
- S-1: Measure keyboard peak `phys_footprint` on an iPhone SE-class and a Pro Max-class device: (a) PhotoKit fetch + crop decode, (b) bar reader, (c) Vision `.fast` on 3 crops. Pass: < 40 MB peak, no jetsam in 200 runs.
- S-2: Confirm the Pokémon GO nickname field accepts custom-keyboard `insertText`, reports `documentContextBeforeInput`, and its max length.
- S-3: Bar-reader accuracy on 100 labeled screenshots ≥ 99%.

---

## 9. Data model (local, App Group)

```text
Species { id, dex, name, form, baseAtk, baseDef, baseSta, family, isShadowEligible }
CPMTable { level(1.0..51.0 step .5) -> cpm }
DustTable { levelBand -> stardust, candy }
Scan {
  id, createdAt, sourceAssetId?, screenType(appraisal|detail),
  speciesId?, ocrName?, cp, hp, hpMax, dustCost?, arcAngle?, isLucky?, isShadow?, isPurified?,
  ivA?, ivD?, ivS?,            // exact from bars when verified
  candidates: [ {level, a, d, s} ],
  level?, ivPercent?, stars?,
  pvp: { gl:{rank, statProdPct, level, cp}, ul:{...} },
  confidence(0..1), verifiedByFormula(bool), corrections:[String]
}
Template { id, name, pattern, symbolStyle(plain|circled|negCircled), maxLen, isDefault }
Settings { trainerLevel, defaultOrigin(ivFloor), templateId, maxScreenshotAgeMin }
CalibrationProfile { aspectBucket, scale, barFillHue, barMaxHue, trackHue, offsets }
```
Storage: SQLite (GRDB) or JSON files in the App Group container. Writes come from the app/extension. The keyboard writes only with Full Access; otherwise it is read-only.

## 10. Success metrics
- Activation: % of installs that enable the keyboard (target 50%) and Full Access (target 35%, never required).
- Time per Pokémon (screenshot → name inserted): ≤ 6 s median (Flow A), ≤ 10 s (Flow B).
- Accuracy: verified-exact rate ≥ 98%; user "wrong result" reports < 0.5% of scans.
- Stability: keyboard jetsam/crash-free sessions ≥ 99.8%.
- Retention: D7 ≥ 30% among keyboard-enabled users.
- Measuring with no cloud: no remote telemetry. Use App Store Connect's built-in (opt-in, Apple-provided) metrics, local self-diagnostics the user can export manually, and TestFlight feedback. *Whether App Store Connect analytics fits "no cloud services" is a judgment call for the owner.*

## 11. Risks & mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Game UI update changes appraisal/detail layout (community reports scan breakage after a game update in 2026) | High | High | Anchor-based detection, calibration profiles, fast app update, regression suite [S11] |
| Keyboard memory jetsam | Med | High | Crop-decode, pixel-first, Flow B fallback, Phase 0 spike |
| Game blocks custom keyboards / secure field | Low | High | Copy-to-clipboard + paste fallback |
| ToS / enforcement shift under the new publisher | Low–Med | High | Screenshot-only, no automation, clear disclaimer; monitor ToS |
| App Review rejection (keyboard needs Full Access to be useful, Photos usage) | Med | Med | Meet 4.4.1 via Flow B; clear purpose strings; use the photo picker in the app |
| Stale base stats for new species/forms | Med | Med | Bundle updates each release; user-picker fallback; local data import (Later) |
| Wrong "exact" result | Low | High | Formula verification gate; never insert unverified names without a warning |
| Trademark complaints | Low | Med | Neutral name/icon, no sprites, disclaimer |

## 12. Phased plan
- **Phase 0 — Spikes (1–2 wks):** S-1 memory, S-2 rename field, S-3 bar reader; collect a 300-screenshot calibration set.
- **Phase 1 — Core engine (3 wks):** `IVCore` math (CP/HP, candidate enumeration, PvP ranks), bundled data pipeline, bar reader, Vision crops, formula validation, XCTest suite.
- **Phase 2 — App + Share Extension (2 wks):** import, result screen, history, templates, App Group storage (Flow B end to end).
- **Phase 3 — Keyboard (3 wks):** UI, Flow A (PhotoKit latest screenshot), insert/delete name, candidate picker, no-Full-Access mode, UIPasteControl Flow C.
- **Phase 4 — Hardening & beta (2–3 wks):** device matrix, dark/HDR, lucky/shadow, TestFlight with 50 players, accuracy dashboard from exported local logs.
- **Phase 5 — Launch + Later:** App Store submission; then digit template matcher, broadcast-extension live scan, power-up simulator, batch queue.

## 13. Open questions / unverified items
- The nickname maximum length; whether the game reports text context to custom keyboards.
- The arc-angle ↔ level mapping and the trainer-level cap.
- The exact bar colors in the current build; iPad layout differences.
- Real keyboard memory ceiling on current iOS (not documented by Apple).
- Poke Genie's free keyboard scan limit (community-reported 50) and its internal OCR method (unpublished).
- How Scopely's current ToS treats screenshot-based companion tools.

---

## Sources

**Poke Genie / game mechanics**
- S1. r/pokemongo, "PokeGenie finally allows measuring IVs within the Pokemon GO app" — https://www.reddit.com/r/pokemongo/comments/c1qo4z/
- S2. App Store listing, Poke Genie — https://apps.apple.com/us/app/poke-genie-remote-raid-iv-pvp/id1143920524 (UK: …/gb/app/…)
- S3. Pokémon GO Hub, "Poke Genie review – smart IV calculator for iOS" (2016) — https://pokemongohub.net/post/featured/poke-genie-review-iv-calculator-ios/
- S4. TheGamer, "Everything You Need To Know About Poke Genie" — https://www.thegamer.com/pokemon-go-poke-genie-guide/
- S5. Poke Genie Privacy Policy (iOS): Keyboard Full Access, IVision — https://pokegenie.net/privacypolicy-I/index.html
- S6. r/PokeGenie, "Cannot get name generator to work" (2026) — https://www.reddit.com/r/PokeGenie/comments/1wfc3u4/
- S7. Poke Genie Privacy Policy (Android): screen capture, floating button — https://pokegenie.net/privacypolicy-A/
- S8. Paperblog, "Poke Genie – Overlay IV Calculator" (store-text mirror) — https://en.paperblog.com/poke-genie-overlay-iv-calculator-for-pokemon-go-1714944/
- S9. Google Play listing — https://play.google.com/store/apps/details?id=com.cjin.pokegenie.standard
- S10. APKMirror release notes, Name Generator — https://www.apkmirror.com/apk/poke-genie/poke-genie-safe-iv-calculator-2/poke-genie-safe-iv-calculator-2-2-1-1-release/
- S11. r/PokeGenie, "iOS IV scan issues since PGo v0.427.0" — https://www.reddit.com/r/PokeGenie/comments/1w5fg69/ (title via search; page blocked to fetch)
- S12. r/PokeGenie, "What are your preferred naming conventions…" — https://www.reddit.com/r/PokeGenie/comments/1o1yjmb/
- S13. r/PokeGenie, "Name Generator Setups" — https://www.reddit.com/r/PokeGenie/comments/11p65jf/
- S14. Pokémon GO Hub, "Combat Power: Ultimate Guide" (CP formula, CPM table) — https://pokemongohub.net/post/wiki/cp-mechanics/ ; CPM list — https://pokemongohub.net/post/article/pokemon-go-cpm-list/
- S15. Serebii, Appraisal (star ranges; publisher note) — https://serebii.net/pokemongo/appraisal.shtml
- S16. IV floors by encounter type — https://www.dittobase.com/pokemon-go/iv-floors (secondary; verify)
- S16a. Pokémon GO Hub, "New Appraisal System Guide" (3 chunks of 5) — https://pokemongohub.net/post/guide/new-pokemon-go-appraisal-system-guide/
- S16b. Serebii appraisal (bars in 20% intervals per segment) — same as S15
- S17. Pokémon GO Help Center, Player Guidelines — https://niantic.helpshift.com/hc/en/6-pokemon-go/faq/1797-player-guidelines/
- S18. Hundo Hunter, "Reading IVs from the appraisal screen" (third-party; colors unverified) — https://www.hundo-hunter.com/methodology/appraisal-iv-reading

**Apple documentation**
- A1. App Extension Programming Guide: Custom Keyboard — https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/CustomKeyboard.html
- A2. Configuring open access for a custom keyboard — https://developer.apple.com/documentation/uikit/configuring-open-access-for-a-custom-keyboard (mirror: https://apple-docs.everest.mt/docs/uikit/configuring-open-access-for-a-custom-keyboard/)
- A3. UIInputViewController / hasFullAccess — https://developer.apple.com/documentation/uikit/uiinputviewcontroller/hasfullaccess
- A4. PHAsset / PHAssetMediaSubtype.photoScreenshot — https://developer.apple.com/documentation/photokit/phassetmediasubtype/photoscreenshot
- A5. Configuring app groups — https://developer.apple.com/documentation/xcode/configuring-app-groups ; containerURL(forSecurityApplicationGroupIdentifier:) — https://developer.apple.com/documentation/foundation/filemanager/containerurl(forsecurityapplicationgroupidentifier:)
- A6. Stack Overflow, keyboard extension + PHImageManager (NSPhotoLibraryUsageDescription requirement) — https://stackoverflow.com/questions/44326413/
- A7. PHPickerViewController — https://developer.apple.com/documentation/photokit/phpickerviewcontroller
- A8. App Review Guidelines (2.5.14, 2.5.18, 4.4, 4.4.1, 5.1.1) — https://developer.apple.com/app-store/review/guidelines/
- A9. Recognizing Text in Images (Vision; on-device) — https://developer.apple.com/documentation/vision/recognizing-text-in-images ; VNRecognizeTextRequest — https://developer.apple.com/documentation/vision/vnrecognizetextrequest
- A10. WWDC19 "Text Recognition in Vision Framework" ("All on Device") — https://developer.apple.com/videos/play/wwdc2019/234/
- A11. UIPasteControl — https://developer.apple.com/documentation/uikit/uipastecontrol
- A12. Sarunw, "UIPasteboard's privacy change in iOS 16" — https://sarunw.com/posts/uipasteboard-privacy-change-ios16/
- A13. Identifying high-memory use with jetsam event reports (extensions have much lower limits) — https://developer.apple.com/documentation/xcode/identifying-high-memory-use-with-jetsam-event-reports

**Community engineering (memory limits)**
- C1. KeyboardKit, Memory Management (~60–70 MB) — https://docs.keyboardkit.com/documentation/keyboardkit/developer-memory-management/
- C2. Wurstfinger commit (~48–66 MB) — https://github.com/cl445/wurstfinger/commit/5cd78f43c800ef2d650c83b92c3b86814706d2f4
- C3. SwiftKey/Medium, CoreText memory in keyboard (~77 MB total) — https://medium.com/@mohasalah/core-text-emoji-rendering-memory-issue-1c6a227d592d

**Data**
- D1. PvPoke repo (MIT), `src/data/gamemaster.json` — https://github.com/pvpoke/pvpoke
- D2. PokeMiners Game Master dumps (no declared license) — https://github.com/PokeMiners/game_masters
