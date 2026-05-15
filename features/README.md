# Features

Artifacts for each feature live in their own folder.

**Naming convention:** `[feature-name]-[number]`
- Numbers are sequential per feature name, used for both ordering and disambiguation.
- Do not overwrite a previous version — create a new numbered folder.

Folders and the artifacts inside them are **populated by skills at runtime** (intent, plan, verify, etc.). Do not pre-create empty placeholder files: the build agent uses file existence to detect tier, and empty placeholders break detection (every feature would appear as Tier 3).
