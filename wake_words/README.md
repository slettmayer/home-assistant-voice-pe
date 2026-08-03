# Vendored microWakeWord models

Offline copies of the two wake word models this firmware pulls from
[`kahrendt/microWakeWord`](https://github.com/kahrendt/microWakeWord) release assets.

**Nothing here is wired up.** `home-assistant-voice.yaml` still references the upstream
release URLs. These files exist purely as a fallback for the day those URLs stop
resolving — see [Restoring from these copies](#restoring-from-these-copies).

## Why

On 2026-07-08 the `okay_nabu_20241226.3` release 404'd and every `esphome config` run
against this firmware failed. It came back the same day — GitHub reports the assets were
re-uploaded at 14:06 UTC — so the release had been deleted and recreated under the same
tag, not merely gone missing.

Both releases are marked `immutable: false` and `okay_nabu_20241226.3` is additionally a
prerelease, so the same thing can happen again at any time, and the bytes behind a given
tag are not guaranteed to stay the same.

## Contents

Downloaded 2026-08-03. The SHA-256 values below were cross-checked against GitHub's own
release-asset digests, not just against the local download.

| File | Size | SHA-256 | Upstream |
| --- | --- | --- | --- |
| `okay_nabu.json` | 416 B | `f8026e2ce93a0855ca23483036fd2d74ee924a2588d9ea6a6c6c7478fbf4be57` | [`okay_nabu_20241226.3`](https://github.com/kahrendt/microWakeWord/releases/tag/okay_nabu_20241226.3) |
| `okay_nabu.tflite` | 80,824 B | `d89128c4d16a72de429119fb2254ce46649553c2a24f5dd840175c80d7b9d094` | same |
| `stop.json` | 375 B | `bd13aeb1b83852649dc4fb6135cb160ff68716d14612b06f6a405342c57447aa` | [`stop`](https://github.com/kahrendt/microWakeWord/releases/tag/stop) |
| `stop.tflite` | 45,744 B | `b5a18c4ad681a89950dfade31011e1631bdcb333e93c84519a1a63ff4f071146` | same |

`stop` is included because it comes from the same repository and carries the same risk;
archiving only `okay_nabu` would leave half the exposure in place.

The `hey_jarvis`, `hey_mycroft` and `vad` models are deliberately *not* vendored. They
resolve through ESPHome's bare-name shorthand to
[`esphome/micro-wake-word-models`](https://github.com/esphome/micro-wake-word-models),
an org-owned repository on a stable path.

To re-verify the files:

```sh
shasum -a 256 -c <<'EOF'
f8026e2ce93a0855ca23483036fd2d74ee924a2588d9ea6a6c6c7478fbf4be57  okay_nabu.json
d89128c4d16a72de429119fb2254ce46649553c2a24f5dd840175c80d7b9d094  okay_nabu.tflite
bd13aeb1b83852649dc4fb6135cb160ff68716d14612b06f6a405342c57447aa  stop.json
b5a18c4ad681a89950dfade31011e1631bdcb333e93c84519a1a63ff4f071146  stop.tflite
EOF
```

## Do not substitute the `okay_nabu` shorthand

The obvious-looking fix when the URL breaks is to swap `model:` for the bare name
`okay_nabu`. **That is a different model**, not a mirror of this one:

| | `okay_nabu_20241226.3` (vendored here) | `okay_nabu` shorthand |
| --- | --- | --- |
| tflite size | 80,824 B | 60,264 B |
| trained languages | en, nl, fr, de, it, es, sv | en |
| `probability_cutoff` | 0.85 | 0.97 |
| `tensor_arena_size` | 37,000 | 26,080 |

Beyond losing six languages, the `Wake word sensitivity` select in
`home-assistant-voice.yaml` overrides the cutoff with values (217 / 176 / 143) measured
specifically against `okay_nabu@20241226.3`; they are meaningless applied to a different
model. This swap was tried in 885347a and reverted in 09d1a90.

## Restoring from these copies

ESPHome's `micro_wake_word` reads the `.json` manifest and then loads the `.tflite`
named by its `model` key, resolved *relative to the manifest*. So both files must travel
together — pointing at the `.json` alone is enough, as long as the `.tflite` sits beside
it.

If the upstream release breaks again, change only the `model:` lines under
`micro_wake_word:` in `home-assistant-voice.yaml`:

```yaml
micro_wake_word:
  models:
    - model: github://slettmayer/home-assistant-voice-pe/wake_words/okay_nabu.json@dev
      id: okay_nabu
    ...
    - model: github://slettmayer/home-assistant-voice-pe/wake_words/stop.json@dev
      id: stop
      internal: true
```

Use the `github://` form when the firmware is consumed as a remote package or via
`dashboard_import`. A plain relative path resolves against the *user's* config
directory, not the package checkout, so it will not work there.

When compiling from a local checkout of this repository, a relative path is fine and is
the only variant that survives GitHub itself being unreachable:

```yaml
    - model: wake_words/okay_nabu.json
      id: okay_nabu
```

## License

The models are build outputs of [microWakeWord](https://github.com/kahrendt/microWakeWord)
by Kevin Ahrendt, licensed under the Apache License 2.0. The upstream license text is
copied here as [`LICENSE`](LICENSE). The files are redistributed unmodified.
