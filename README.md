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

### Phase 2 (this commit, control)

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

### Phase 3 (later commit, migration hack)

- Leave `plugins/flat-probe` in place as a forwarding stub. Its skill replies with a V3 marker that tells the user to install `platform-flat-probe`.
- Add a new plugin `platform-flat-probe` at `plugins/platform/platform-flat-probe`.

Passing proves existing installs keep loading and the new location installs cleanly under a new name.

## Install steps

1. In Cursor, add `https://github.com/jasonmatta-hs/cursor-plugin-layout-spike-2` as a marketplace.
2. Install `flat-probe` and `nested-probe`.
3. Open a new chat and ask the phase 1 questions above.
4. Stop after phase 1.
