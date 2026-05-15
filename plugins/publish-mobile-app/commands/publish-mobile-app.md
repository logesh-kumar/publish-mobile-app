---
description: Automate iOS App Store + Google Play Store publishing — listing, screenshots, IPA/AAB, reviewer creds, rejection fixes
argument-hint: [setup|push|ipa|aab|release|fix-rejection|status|checklist]
---

Invoke the `publish-mobile-app` skill to handle iOS App Store + Google Play Store publishing for the current project.

User argument: $ARGUMENTS

Use the Skill tool to load `publish-mobile-app`, then route based on the argument:

- `setup` — wire fastlane, gradle-play-publisher, App Store Connect API key, reviewer user
- `push` — sync listing copy + screenshots to both stores
- `ipa` — build signed iOS IPA and upload to TestFlight
- `aab` — build signed Android AAB and upload to Play internal track
- `release` — full lifecycle (bump + build + upload + listing push)
- `fix-rejection` — user will paste rejection email; map to recipe in REJECTIONS.md
- `status` — query both stores' APIs, report what's live vs missing
- `checklist` — refresh RELEASE_CHECKLIST.md at repo root

If no argument is given, default to `status` to orient first, then ask the user which sub-command to run next.

The skill's scripts (`bootstrap-app-store-key.ts`, `check-ios-state.ts`, `create-app-review-user.ts`, etc.) live in `~/.claude/skills/publish-mobile-app/scripts/`. Copy them into the project's `scripts/` directory as needed during `setup`.

Always verify outcomes via the App Store Connect API after a push — fastlane's "Successfully uploaded" log can be misleading. The `check-ios-state.ts` script is the source of truth.
