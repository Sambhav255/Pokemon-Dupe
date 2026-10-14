# Bulk scanning (~800 Pokémon)

**Recommendation:** a ReplayKit **broadcast upload extension**. The user starts a broadcast and swipes through appraisal screens. The extension keeps stable frames, then runs Vision and the solver on-device and writes results to the App Group. The fallback is **batch import** from Photos.

| Option | iOS? | Automation | Effort |
|---|---|---|---|
| Poke Genie IVision | Yes | iOS Screen Recording while you swipe ([privacy](https://pokegenie.net/privacypolicy-I/index.html), [r/PokeGenie](https://www.reddit.com/r/PokeGenie/comments/1elfl71/scan_vision_on_iphone/)) | Swiping only |
| Poke Genie batch import | Yes | Take screenshots, then "import all" ([thread](https://www.reddit.com/r/PokeGenie/comments/1wt43p3/best_way_to_clean_up_and_optimize_my_pok%C3%A9genie/)) | 800 screenshots |
| PokeDup broadcast extension | Yes | Pick stable frames, run Vision | Medium; ~50 MB memory limit *(unverified)*, so sample and downscale frames |
| Screen-record video → frames | Yes | AVAssetImageGenerator in the app | Low–medium |
| Photos batch processing | Yes | PhotoKit | Low |
| iOS Shortcuts | Partly | Can screenshot; can't tap inside other apps *(unverified)* | Doesn't help with swiping |
| Calcy IV scan all, overlays, adb/scrcpy | Android only | — | n/a |
| pogo-ipad-renamer | iPad + Mac | Automated taps ([repo](https://github.com/widmonstertony/pogo-ipad-renamer)) | High; risky |

## Terms of service
Niantic bans "modified or unofficial third party software", unauthorized access to the game, bots and scraping:
- [ToS §3.1 as quoted by IGN](https://pk.ign.com/pokemon-go/241380/news/amid-fan-confusion-around-the-risks-of-using-a-well-known-pokemon-go-exploit-the-games-developer-res)
- [Player Guidelines](https://niantic.helpshift.com/hc/en/6-pokemon-go/faq/1797-player-guidelines/?p=Web)
- [Fairness Policy](https://niantic.helpshift.com/hc/en/6-pokemon-go/faq/39-gameplay-fairness-policy-1701994322/)
- [2016 bans post](https://pokemongolive.com/en/post/update-082916)

Passive reading of screenshots or recordings doesn't touch the game. Poke Genie calls itself "fully compliant", but that is its own claim, not Niantic's. **Auto-tapping is riskier, so PokeDup stays passive.**

## Repo / branch status
The `auto-scan` branch **has not been created**. The GitHub repo **Pokemon-Dupe doesn't exist yet**, and creating it is waiting on a GitHub token because the box's `gh` isn't logged in. The user first asked for a private repo, then changed it to public.
