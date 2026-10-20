# cursor-plugin-layout-spike-2

A throwaway Cursor plugin marketplace for testing plugin folder layouts. This is not a Hootsuite plugin.

## What the first spike missed

The first spike ([cursor-plugin-layout-spike](https://github.com/jasonmatta-hs/cursor-plugin-layout-spike)) moved a plugin from `plugins/layout-probe` to `plugins/nested/layout-probe`. It went wrong in three ways:

1. It changed the folder path and the marker text in the same commit, so there was no control to tell which change caused the failure.
2. It never tested a nested path on a fresh install.
3. Cursor remembers each plugin's folder path from the first install (by plugin name) and reuses it for later commits. This holds even after restart, uninstall, and re-adding the marketplace. So a path change breaks existing installs: the plugin page errors with ENOENT on the old path.

## Rule this spike is testing

A plugin's name plus its folder path is its identity.

- Never change the path of an installed plugin.
- To reorganize, keep the old path working and ship the new location under a new plugin name.

## Phases

Each phase changes one thing.

### Phase 1 (passed)

Result: both plugins installed and replied with the V1 markers.

Fresh install of two plugins:

- `flat-probe` at `plugins/flat-probe` (flat control).
- `nested-probe` at `plugins/platform/nested-probe` (nested from its first commit).

Ask these in a new chat and expect these exact replies:

| Question | Expected reply |
| --- | --- |
| Is flat-probe installed? | `SPIKE2_FLAT_V1` |
| Is nested-probe installed? | `SPIKE2_NESTED_V1` |
| Which flat-probe rule is loaded? (optional) | `SPIKE2_FLAT_RULE_V1` |
| Which nested-probe rule is loaded? (optional) | `SPIKE2_NESTED_RULE_V1` |

Passing proves a nested path installs cleanly.

### Phase 2 (passed, control)

Result: both plugins updated to the V2 markers without uninstalling. The plugin cache moved to the phase 2 commit.

Change only the markers to V2. Keep both plugins at the same paths.

Do not uninstall. Wait for Cursor to refresh the marketplace, then open a new chat. Each chat loads its skill list when it opens, so an old chat keeps the old marker. If a new chat still shows V1, restart Cursor and try again.

Ask these in the new chat and expect these exact replies:

| Question | Expected reply |
| --- | --- |
| Is flat-probe installed? | `SPIKE2_FLAT_V2` |
| Is nested-probe installed? | `SPIKE2_NESTED_V2` |
| Which flat-probe rule is loaded? (optional) | `SPIKE2_FLAT_RULE_V2` |
| Which nested-probe rule is loaded? (optional) | `SPIKE2_NESTED_RULE_V2` |

A chat may echo a line-number prefix such as `10|` from its file reader. That prefix is not part of the marker.

Passing proves content updates reach existing installs when the path does not move. If this fails, the problem is caching, not paths.

### Phase 3 (this commit, migration hack)

- `flat-probe` stays at `plugins/flat-probe` as a forwarding stub. Its skill replies with a V3 marker that tells the user to install `platform-flat-probe`.
- The new plugin `platform-flat-probe` lives at `plugins/platform/platform-flat-probe`.
- `nested-probe` does not change.

Steps:

1. Do not uninstall anything. Restart Cursor so it fetches the new commit.
2. In a new chat, ask "Is flat-probe installed?" Expect `SPIKE2_FLAT_V3_MOVED install platform-flat-probe`. This proves the existing install still loads and got the stub.
3. Install `platform-flat-probe` from the marketplace.
4. In a new chat, ask "Is platform-flat-probe installed?" Expect `SPIKE2_PLATFORM_FLAT_V1`.
5. Control: ask "Is nested-probe installed?" Expect `SPIKE2_NESTED_V2`.
6. Optional: uninstall `flat-probe`, open a new chat, and confirm `platform-flat-probe` still answers `SPIKE2_PLATFORM_FLAT_V1`.

| Question | Expected reply |
| --- | --- |
| Is flat-probe installed? | `SPIKE2_FLAT_V3_MOVED install platform-flat-probe` |
| Is platform-flat-probe installed? | `SPIKE2_PLATFORM_FLAT_V1` |
| Is nested-probe installed? | `SPIKE2_NESTED_V2` |
| Which flat-probe rule is loaded? (optional) | `SPIKE2_FLAT_RULE_V3_MOVED` |
| Which platform-flat-probe rule is loaded? (optional) | `SPIKE2_PLATFORM_FLAT_RULE_V1` |

As in phase 2, open a new chat after each refresh, and ignore any `10|` line-number prefix.

What a pass means: existing installs keep working through a move, and the new location installs cleanly under its new name. So a real move should keep the old folder as a stub and publish the new path under a new name.

## Install steps

1. In Cursor, add `https://github.com/jasonmatta-hs/cursor-plugin-layout-spike-2` as a marketplace.
2. Install `flat-probe` and `nested-probe`.
3. Open a new chat and ask the phase 1 questions above.
4. Follow the phase 2 and phase 3 steps above in order.
