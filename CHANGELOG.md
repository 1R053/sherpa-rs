# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.7.1] — unreleased

### Changed

- **sherpa-onnx bumped v1.12.38 -> v1.13.8** (218 upstream commits). The one this bump is
  for: "Fix Cohere Transcribe hallucinating sentences on silent audio" (k2-fsa/sherpa-onnx
  #3924, 1.13.8) — a feature-level silence gate in front of the decoder, so a digitally
  silent clip returns an empty result instead of a sentence such as "Herr Präsident, meine
  Damen und Herren!". Also carried: FunASR-Nano/Moonshine-v2/Qwen3-ASR silent-audio fixes
  (#3921, #3923, #3907), ScaleSilence guards (1.13.5), TopkIndex heap-buffer-overflow
  (1.13.3), the Qwen3-ASR PRNG data race (1.13.8).
- **onnxruntime rides along: 1.24.4 -> 1.28.2.** The macOS shared archive at v1.13.8 is
  `sherpa-onnx-v1.13.8-osx-arm64-shared.tar.bz2` (upstream no longer publishes the
  `-onnxruntime-<ver>-` named variant at this tag) and ships `libonnxruntime.dylib` without a
  version suffix.
- **iOS prebuilts moved.** From v1.13.5 upstream publishes no iOS archive under the versioned
  release; the iOS xcframeworks live in the rolling `xcframework` release as
  `sherpa-onnx-v<tag>-ios-static.xcframework.zip` (one static `SherpaOnnxC` per slice, with
  onnxruntime linked in). `dist.json` targets may now carry their own `url`; `.zip` archives
  are extracted with the host's `unzip`; a `Name.framework/Name` static binary gets a
  `libName.a` placed beside it so `-l static=Name` resolves.
- `diarize.rs` fills the two fields the 1.13 C API added (`compute_confidence`,
  `window_shift_ratio`) with the values that keep 1.12 behaviour.
- `checksum.txt` now lists only the archives `dist.json` can select on Apple targets
  (osx-arm64 shared, osx-x64 shared, iOS static xcframework); other targets fall back to the
  manual build as before.

## [0.7.0] — unreleased

### Added

- **Cohere Transcribe support** — new `cohere_transcribe` module exposing
  `CohereTranscribeRecognizer` / `CohereTranscribeConfig`. Wraps the
  `SherpaOnnxOfflineCohereTranscribeModelConfig` C API added upstream in
  sherpa-onnx v1.12.x. Supports 14 languages with native punctuation and
  inverse-text-normalization toggles.
- **Integration test suite and model-download helper** — first
  cargo-runnable tests in the repo (previously only `examples/` existed
  and required manual model setup). `tests/offline_recognizers.rs`
  exercises both the Whisper path (validating that existing C API fields
  still work after the v1.12.38 bump) and the new Cohere Transcribe
  module end-to-end against real audio. `tests/test_utils.rs` is a
  reusable helper exposing `ensure_model(&ModelArchive)` /
  `ensure_motivation_wav()` — it resolves a model cache directory
  (`SHERPA_TEST_MODELS` env var or workspace-root `test_data/`), skips
  tests gracefully when files are missing (CI-friendly default), and
  auto-downloads from the k2-fsa/sherpa-onnx release assets when
  `SHERPA_DOWNLOAD_MODELS=1` is set. Downloads are serialised via
  `std::sync::Once` so the suite is safe under `cargo test`'s default
  parallel runner. Covers Whisper-tiny (~250 MB) and Cohere Transcribe
  int8 (~2.7 GB); adding further model archives is a one-const entry.
- `ZipVoiceTtsConfig`: new `encoder`, `decoder`, `lexicon` fields
  matching the upstream C API layout.

### Changed

- **Bumped bundled sherpa-onnx from v1.12.15 → v1.12.38.** Updated
  submodule, `dist.json` tag, and `checksum.txt`.
- **Apple Silicon performance fix.** `dist.json` for `aarch64-apple-darwin`
  now pulls `sherpa-onnx-{tag}-onnxruntime-1.24.4-osx-arm64-shared.tar.bz2`
  instead of `sherpa-onnx-{tag}-osx-arm64-shared.tar.bz2`. The former
  bundles the full-optimization onnxruntime build (same as the Python
  pip wheel); the latter shipped a smaller variant missing
  graph-optimization paths, which blocked the post-first-inference
  kernel-cache warmup. Net effect on our spot-check with Cohere
  Transcribe int8: a multi-x speedup on warm inferences, bringing
  warm-path parity with the Python `sherpa_onnx` pip wheel. Actual
  numbers will vary by chip generation, thread count, and model.
- Windows `dist.json` entry updated to
  `sherpa-onnx-{tag}-win-x64-shared-MD-Release.tar.bz2` (upstream renamed
  the asset to include build configuration).
- `aarch64-apple-darwin` / `x86_64-apple-darwin` split: each arch now
  points at its native single-arch tarball instead of the universal2
  fat binary. Smaller downloads, no lipo-imposed compiler compromises.

### Breaking

- `ZipVoiceTtsConfig` dropped `flow_matching_model`, `text_model`, and
  `pinyin_dict` fields — upstream replaced them with `encoder`,
  `decoder`, and `lexicon` between v1.12.15 and v1.12.38. Callers must
  rename their field usage.
- Minimum bundled sherpa-onnx is v1.12.38. Consumers pinning
  `sherpa-rs` in a workspace that also vendors an older sherpa-onnx
  must upgrade.

## [0.6.8] — 2025-xx-xx

- See GitHub releases.
