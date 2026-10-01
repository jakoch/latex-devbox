# Changelog

All changes to the project will be documented in this file.

- The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
- The date format is YYYY-MM-DD.
- The upcoming release version is named `vNext` and links to the changes between latest version tag and git HEAD.

## [vNext] - unreleased

- "It was a bright day in April, and the clocks were striking thirteen." - 1984

## [1.0.3] - 2026-10-01

### Fixed

- `CURL_OPTIONS` was misspelled as `CURL_OPTIONS_BAR`, so the JDK, ltex-ls-plus and
  tex-fmt downloads ran without `--retry`, `--silent` and `--show-error`
- dropped the `date`-generated `org.opencontainers.image.created` label, which invalidated
  the layer cache on every build (the label is now supplied by the CI workflow)
- corrected the OCI labels: `image.title` pointed at `cpp-devbox`, and `image.url` used a
  registry hostname instead of a URL
- stopped wiping `/tmp` in the cleanup layers
- removed the dead `fonts-powerline` `.deb` download block (the package comes from apt now)
- generate the locales before setting the default locale, and no longer force `LC_ALL`
- removed the contradictory `ZSH_THEME=agnoster` environment variable
- the hadolint ignore list contained a YAML mapping instead of plain strings, so hadolint
  discarded every ignore rule; now that hadolint is a blocking check, that failed the
  release build before the image was built

### Added

- multi-architecture support (`linux/amd64` and `linux/arm64`) via `TARGETARCH`, covering
  the JDK, ltex-ls-plus and tex-fmt downloads
- added a `WORKDIR` so the image no longer starts in `/`
- arm64 is verified in CI with a smoke test of the natively downloaded binaries

### Changed

- hadolint is now a blocking CI check (`no-fail` removed), and `trivy-action` is pinned to
  a released version instead of `@master`
- Dockle now fails on hard errors instead of only reporting them
- images are pushed directly by buildx, because multi-platform manifest lists cannot be
  loaded into the local image store

## [1.0.2] - 2026-01-11

### Changed

- fetch fonts-powerline package using apt

## [1.0.1] - 2025-10-26

### Changed

- updated to Java SDK 25, added checksum verification
- disabled zsh update prompt and auto update feature (DISABLE_UPDATE_PROMPT & DISABLE_AUTO_UPDATE)

## [1.0.0] - 2025-06-14

- No changes: stable release

## [0.0.3] - 2025-03-25

### Changed

- fixed curl options (to be silent, instead of spamming the logs)

## [0.0.2] - 2025-01-26

### Changed

- fixed installation of "fonts-powerline"

## [0.0.1] - 2024-12-26

- Initial Release

<!-- Section for Reference Links -->

[vNext]: https://github.com/jakoch/latex-devbox/compare/v1.0.3...HEAD
[1.0.3]: https://github.com/jakoch/latex-devbox/compare/v1.0.2...v1.0.3
[1.0.2]: https://github.com/jakoch/latex-devbox/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/jakoch/latex-devbox/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/jakoch/latex-devbox/compare/v0.0.3...v1.0.0
[0.0.3]: https://github.com/jakoch/latex-devbox/compare/v0.0.2...v0.0.3
[0.0.2]: https://github.com/jakoch/latex-devbox/compare/v0.0.1...v0.0.2
[0.0.1]: https://github.com/jakoch/latex-devbox/releases/tag/v0.0.1
