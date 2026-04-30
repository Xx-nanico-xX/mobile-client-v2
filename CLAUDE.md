# Memory

## Me
Guillaume, developer of Streamflix (fork of streamflix-reborn). Android app for streaming, supports mobile + TV layouts.

## Project
| Key | Value |
|-----|-------|
| **App ID** | com.streamfr.app |
| **Namespace** | com.streamflixreborn.streamflix |
| **Repo** | GitHub (private fork of streamflix-reborn/streamflix) |
| **Upstream** | https://github.com/streamflix-reborn/streamflix |
| **CI/CD** | GitHub Actions, `.github/workflows/release.yml` |
| **Signing** | `streamflix-release.jks` in repo root |
| **Test device** | Honor 200 (AJ4UVB4531012023, ELI-NX9) |

## Critical Rules - DO NOT BREAK

1. **NEVER create GitHub releases manually** - Always push a `v*` tag and let the CI workflow (`release.yml`) build, sign, and upload APKs automatically.
2. **NEVER push all local tags** - Use `git push origin v1.x.x` for the specific tag only. Never `git push --tags` which floods old tags.
3. **NEVER make broad code changes without per-change testing** - Test each individual change on the target device before combining. The "memory optimization" disaster of April 2025 proved this.
4. **Always clean Gradle caches** when manifest or build config changes don't take effect - `./gradlew clean` + delete `.gradle/` and `build/` if needed.

## Lessons Learned
| Date | Mistake | Impact | Rule |
|------|---------|--------|------|
| 2025-04 | Applied 6+ memory optimizations at once (onTrimMemory, Glide cache, destination listener, sharedPool, ArtworkLoader, NetworkClient) | App crashed on Honor 200 Films tab | Test changes individually |
| 2025-04 | Created GitHub release manually via API with unsigned APKs | Release had wrong APKs, had to delete and redo | Let CI handle releases |
| 2025-04 | Pushed ~80 old tags with `--tags` | CI didn't trigger for the right tag | Push only specific tag |

## Build & Release Process
1. Bump `versionCode` and `versionName` in `app/build.gradle`
2. Commit and push to `main`
3. Create tag: `git tag v1.x.x`
4. Push ONLY that tag: `git push origin v1.x.x`
5. Wait for CI to build 3 APKs (default, mobile, TV), sign them, and create the release
6. Verify on GitHub that release has all 3 APKs

## Architecture
- **3 APK variants**: default (both), mobile-only, TV-only (controlled by `APP_LAYOUT` in `local.properties`)
- **Manifest selection**: Gradle picks manifest based on `APP_LAYOUT` value
- **TV layout**: Uses `VerticalGridView`, Leanback
- **Mobile layout**: Uses `RecyclerView` + `GridLayoutManager`
- **Both share**: `MoviesViewModel`, `AppAdapter`, same providers

## Current State (v1.7.139)
- **Latest release**: v1.7.139 (versionCode 162) — Fix crash release ViewPager2 R8 + fix Search TV DPAD_CENTER
- **Upstream version**: v1.7.116 (streamflix-reborn)
- **We are AHEAD of upstream** on French providers (Wiflix, Frembed, FrenchStream, VOE aliases)

## Upstream Comparison (as of 2026-04-29)
- **PR #423 (Wiflix/Frembed/FrenchStream)**: All changes already in our code. We have MORE VOE aliases than them.
- **PR #424 (manga français search fix)**: Not evaluated yet
- **PR #420 (overlay épisode suivant + préchargement)**: Interesting feature, not yet integrated
- **PR #302 (refonte settings/thèmes/bypass)**: Large PR, not yet evaluated
- **PR #335 (persistance/ordonnancement/cache)**: Not yet evaluated
- Only French-relevant changes matter — ignore non-French providers (CB01, Einschalten, Polish providers, etc.)

## Stash Warning
- `git stash list` contains `memory-fixes-and-optimizations` — DO NOT pop/apply this. It contains the 6+ changes that crashed the app on Honor 200.

## Preferences
- Ship updates quickly, don't overthink
- Test on Honor 200 before releasing
- French-speaking user — only care about French providers
- Don't waste time on non-French provider fixes
