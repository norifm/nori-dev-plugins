# nori dev plugins

Plugins for nori's developers: the ones its checks install, examples of each extension point, and
plugins for hardware of our own. nori's plugins for listeners are in
[nori-plugins](https://github.com/norifm/nori-plugins), the repo nori lists by default. Each plugin is a
WebAssembly component built with nori's plugin SDK; how plugins work is in
[Plugins](https://github.com/norifm/nori/blob/master/docs/developers/plugins.md).

## The plugins

| Plugin | What it adds | Extension points |
|---|---|---|
| `orbit` | A sound effect: the music circles your head | `sound-stage` |
| `playback` | Speed, pitch, skipping silence and what previous does | `lifecycle` (overrides) |
| `remote-switch` | A switch on **Playing on** that turns remote control and jams off | `lifecycle` (overrides) |
| `showcase` | One of every kind of row and place a plugin can fill | `lifecycle`, `slots` |
| `speaker` | Casting to a nori speaker on your network | `lifecycle`, `hooks` |

nori's checks use them: `tools/feature-e2e.sh` installs orbit and the showcase from `dist/`
(`NORI_PLUGINS_DIST`), and `web/check/plugins.mjs` installs orbit and playback from this repo.

## Use the repo

In nori, open **Settings › Plugins › Get plugins › Repos**, choose **Add repo** and enter:

```
https://raw.githubusercontent.com/norifm/nori-dev-plugins/main/index.json
```

## Build a plugin

The toolchain is pinned in `rust-toolchain.toml`, with the `wasm32-wasip2` target. To build one plugin
and pack it into `dist/`, run:

```sh
python3 tools/pack.py orbit
```

`tools/pack.py` with no names packs every plugin, into `dist/<plugin>-<version>.noriplugin`. To try one,
install the file in nori with **Install from file** on the **Get plugins** page.

The SDK comes from nori's repository, at the commit that `Cargo.lock` names. To move it to nori's newest
commit, run `cargo update -p nori-plugin-sdk`. To build against your own nori checkout while you change
the SDK, run:

```sh
cargo build --config 'patch."https://github.com/norifm/nori".nori-plugin-sdk.path="../nori/crates/plugin-sdk"'
```

That command rewrites `Cargo.lock` to the local path. Do not commit that change.

## Release a plugin

1. In the plugin's `Cargo.toml`, set the new `version`, and commit.
2. Push the tag `<plugin>-<version>`, for example `orbit-0.8.0`.

The release workflow (`.github/workflows/release.yml`) checks that the tag matches the plugin's
`Cargo.toml` and runs the plugin's tests. It then packs the plugin, makes a GitHub release with the
package, and commits the new version to `index.json` on `main`. Nobody writes `index.json` by hand.

Packing is deterministic. To check a release, pack its tag again and compare the sha256 with the one in
`index.json`.

## License

AGPL-3.0-or-later, as `Cargo.toml` says.
