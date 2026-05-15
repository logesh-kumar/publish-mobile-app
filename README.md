# publish-mobile-app

> A Claude Code skill that automates iOS App Store + Google Play Store publishing for Capacitor, Expo, or Flutter apps.

Shipping a mobile app to both stores is full of paper cuts: subtle filename rules, undocumented APIs, locale gotchas, rejection emails that quote guideline numbers nobody memorises. This skill bundles every paper cut learned shipping a real app to v1.0 — so the next one ships clean.

## What it does

When you invoke `/publish-mobile-app`, Claude routes to one of these sub-commands:

| Sub-command | What happens |
|---|---|
| `status` | Query both stores' APIs, report what's live vs missing |
| `setup` | Wire fastlane + gradle-play-publisher + App Store Connect API key + reviewer user (one-time, ~5 min interactive) |
| `push` | Sync listing copy + screenshots to both stores |
| `ipa` | Build signed iOS IPA → upload to TestFlight |
| `aab` | Build signed Android AAB → upload to Play internal track |
| `release` | Full lifecycle (bump + build + upload + listing push) |
| `fix-rejection` | Paste a rejection email; Claude maps to the recipe and applies the fix |
| `checklist` | Refresh a `RELEASE_CHECKLIST.md` at repo root |

It also **auto-triggers** when you mention Play Store, App Store Connect, fastlane, TestFlight, or paste a store rejection email — even without the slash command.

## Install

### As a Claude Code plugin (recommended)

```
/plugin marketplace add logesh-kumar/publish-mobile-app
/plugin install publish-mobile-app@publish-mobile-app
```

### As a raw skill

```bash
git clone https://github.com/logesh-kumar/publish-mobile-app.git /tmp/pma
cp -r /tmp/pma/plugins/publish-mobile-app/skills/publish-mobile-app ~/.claude/skills/
cp /tmp/pma/plugins/publish-mobile-app/commands/publish-mobile-app.md ~/.claude/commands/
```

## What's in the box

```
publish-mobile-app/
├── SKILL.md           Entry point and sub-command router
├── REFERENCE.md       Framework specifics (Capacitor / Expo / Flutter), Fastfile templates, every gotcha learned the hard way
├── REJECTIONS.md      Recipes per Apple/Google rejection (Guideline 2.1, 2.3.8, 4.0, Data Safety mismatch, etc.)
├── MANUAL_FORMS.md    Walkthroughs for Age Rating, App Privacy, Data Safety questionnaires
└── scripts/
    ├── bootstrap-app-store-key.ts    Assembles ASC key JSON from .p8 + IDs, moves .p8 to safe location
    ├── check-ios-state.ts            Queries ASC API for ground truth (fastlane logs lie sometimes)
    ├── create-app-review-user.ts     Idempotently provisions reviewer user (Supabase auto-detected)
    ├── print-reviewer-creds.ts       Copy-paste output for Play Console (which has no API for App Access)
    └── validate-metadata.ts          Local char-limit + Kids Category trap detector
```

## Gotchas it captures (so you don't have to learn them again)

- `LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8` must be forced when invoking fastlane or Ruby crashes on em-dashes/bullets in `description.txt`
- App Store Connect API key path in `Fastfile` must be `File.expand_path("../app-store-connect-key.json", __dir__)` — relative paths silently fail
- iPad 2048×2732 screenshots route to the wrong slot unless filename literally contains `iPad Pro (12.9-inch) (3rd generation)`
- `copyright.txt` is **non-localized** (lives at `metadata/copyright.txt` root, not per-locale)
- Phone numbers in App Review Information need `+<country>` format; no spaces
- Subtitle lives on `appInfoLocalizations`, NOT `appStoreVersionLocalizations`
- Play Console default locale is often `en-GB`, not `en-US` — wrong folder = silently ignored
- "kids" / "children" in App Store subtitle → Guideline 2.3.8 rejection unless in Kids Category
- China book/magazine apps need an Internet Publishing License — easier to remove China from territories (via UI; the v2 API is undocumented)
- Fastlane's "Successfully uploaded" log is NOT equivalent to "appears in correct UI slot" — always verify via API

Full list with workarounds in [REFERENCE.md](plugins/publish-mobile-app/skills/publish-mobile-app/REFERENCE.md).

## Supported stacks

- **Capacitor** — `ios/App/App.xcodeproj`, `cap sync` in build pipeline
- **Expo** — works after `expo prebuild`; native EAS Build/Submit also supported as an alternative path
- **Flutter** — `ios/Runner.xcodeproj`, `flutter build appbundle` for Android

## Prerequisites

- `fastlane` (Homebrew or RubyGems)
- `bun` or Node 18+ (for the TypeScript helper scripts)
- For iOS: Xcode + an Apple Developer account
- For Android: Android Studio + a Google Play Console developer account + a service account JSON
- For automated reviewer-user provisioning: Supabase (other backends fall back to manual instructions)

## Contributing

PRs welcome — especially for new rejection recipes. The skill's value compounds as more papercuts get documented. To add a recipe:

1. Add an entry to `REJECTIONS.md` under the relevant store + guideline number
2. Include the rejection email's verbatim wording (helps Claude pattern-match)
3. Include the fix as concrete steps
4. Open a PR

## License

[MIT](LICENSE)

## Acknowledgements

Built and battle-tested while shipping a real app to v1.0 on both stores. Every gotcha in `REFERENCE.md` came from a real failure, not a hypothetical.
