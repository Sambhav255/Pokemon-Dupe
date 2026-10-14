# Conversation summary (through Oct 13, 2026)

1. **Initial research + PRD:** a Poke Genie-style IV scanner with a rename keyboard → `PRD.md`.
2. **Constraints added:**
   - iOS only
   - fully on-device; no cloud or external AI
   - non-neural classifier / pixel measurement for the IV bars
   - results in a custom iOS keyboard extension
3. **Free sideloading:** Xcode Personal Team is the path; TestFlight, the EU routes, TrollStore and enterprise certificates are dead ends → `sideloading.md`.
4. **App Groups:** confirmed for free accounts from Apple's capabilities table. Re-signers rewrite group IDs, so read them at runtime.
5. **Open-source search:** TrainerKit (MIT) is the best base. GoIV is GPL and reference only. pogo-ipad-renamer has no license. rowan is excluded because it uses cloud AI. The search included who reads appraisal bars by pixel vs classifier → `research-findings.md`.
6. **Poke Genie reverse-engineering dropped** as too costly. Scope narrowed to the screenshot-to-IV pipeline: OCR, arc/stardust level, CP/HP formulas, 4,096-combination brute force.
7. **Bulk scanning of ~800 Pokémon:** recommend a ReplayKit broadcast extension, with batch import as fallback; passive only for ToS reasons → `auto-scan.md`.
8. **GitHub:** the repo Pokemon-Dupe was requested private, then public. Blocked on a GitHub token because box `gh` isn't logged in. The `auto-scan` branch is pending.

*(No record of the phone calls was in this agent's context. This summary covers the chat-delegated research only.)*

## Decisions
- No neural nets and no network.
- Port TrainerKit's math and scanner to Swift.
- Free Personal Team signing.
- Passive scanning only.

## Open items
- Phase 0 device spikes (README).
- GitHub token → create the repo and the `auto-scan` branch.
- Unverified items marked throughout.
