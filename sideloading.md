# Installing PokeDup without the $99/yr Apple Developer Program

## Free Xcode Personal Team (recommended)
- Profiles expire after **7 days**; rebuild to renew. Data is kept if the bundle ID stays the same. Up to **3 apps** per device and **10 App IDs per 7 days**. Each extension uses up an App ID. *(Limits per [SideStore FAQ](https://docs.sidestore.io/docs/faq); Xcode behavior from memory.)*
- Keyboard extensions work. **Full Access** is the Info.plist key `RequestsOpenAccess`; the user turns it on in Settings. It is not a provisioned capability *(unverified)*.
- **App Groups: VERIFIED.** Apple's table, parsed from the raw HTML, gives free accounts exactly these: App groups, Background modes, Data protection, HealthKit, HomeKit, Inter-App Audio, Keychain sharing, Maps, Wireless Accessory Configuration. Push, iCloud and Associated Domains are not available. https://developer.apple.com/help/account/reference/supported-capabilities-ios
- Reinstalling over Wi-Fi from Xcode works after pairing *(unverified)*.
- I didn't search forums for recent signing errors with Xcode 16/26. Do a quick test build.

## Re-signers
- **AltStore / SideStore:** use your free Apple ID (same limits) and refresh in the background over Wi-Fi. SideStore needs a pairing file + VPN and no computer. Both **rewrite App Group IDs** to per-team IDs ([SideStore entitlements](https://github.com/SideStore/SideStore/blob/develop/AltStore/AltStore.entitlements), [FAQ](https://docs.sidestore.io/docs/faq), [issue #1245](https://github.com/SideStore/SideStore/issues/1245), [Impactor PR](https://github.com/CLARATION/Impactor/pull/165)). **Read the group ID at runtime** from the entitlements or Info.plist; don't hardcode it.
- SideStore can reuse the main app's profile for extensions to save App IDs ([PR #1012](https://github.com/SideStore/SideStore/pull/1012)).
- **Sideloadly:** needs a computer; supports extensions *(unverified)*.

## Dead ends
- **TestFlight:** paid program required; builds expire after 90 days *(unverified)*.
- **AltStore PAL, EU marketplaces, web distribution:** paid program plus notarization required *(unverified)*.
- **TrollStore:** iOS 14.0b2–16.6.1, 16.7 RC, 17.0 only; never 17.0.1+ ([README](https://github.com/opa334/TrollStore)) — verified.
- **Enterprise certificates / signing services:** against Apple's Enterprise program terms and revoked regularly *(unverified)*.

## Untested fallbacks if App Groups fail
- Keychain sharing: allowed on free accounts, but only for small data.
- Pasteboard with Full Access: works, may trigger paste prompts.
- Keyboard reads Photos directly: unverified.
- Network/localhost: banned by the no-cloud rule.
- URL schemes: keyboards can't open apps.
