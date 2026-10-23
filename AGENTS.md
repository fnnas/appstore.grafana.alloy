# Working in this repository

This repository packages upstream Grafana Alloy Linux binaries for fnOS; it does not build Alloy from source.

## Layout

- `.github/workflows/main.yml`: manual `Test Release` workflow, version settings, artifact collection and optional GitHub prerelease.
- `.github/workflows/pack_amd64.yml` and `pack_arm64.yml`: download, checksum verification, manifest substitution and tar packaging.
- `app/`: default Alloy configuration and runtime UI assets. The downloaded executable becomes `app/alloy-linux`.
- `package/`: manifest template, icons, lifecycle Bash scripts, privilege/resource JSON and `i18n/{zh-CN,en,ja-JP}` metadata.
- `ui/` and `package/wizard/`: retained placeholder directories used by packaging.

## Packaging changes

- Read `target_version` and `this_pack_manifest_version` in `main.yml`; update both for an upstream release and keep README version information consistent. Confirm the latest stable release and its assets on `grafana/alloy` before changing versions.
- Keep the two architecture workflows aligned. Upstream `amd64` maps to manifest `arch=x86_64`, `platform=x86`; upstream `arm64` maps to `arch=arm`, `platform=arm`.
- Preserve checked-in `this_pack_manifest_*` placeholders; replace them only in build copies.
- Verify upstream `alloy-linux-<arch>.zip` against its release's `SHA256SUMS` before extraction. Keep artifact upload/download action versions compatible.
- Cache only downloaded ZIP files under `tmp/downloads/<upstream-version>/` with `actions/cache@v4`; key by runner OS, upstream architecture and upstream version. Do not key by package version or commit, or use fallback keys across upstream versions. Always fetch the official `SHA256SUMS` again and verify restored files before extraction.
- Use the existing direct tar process; do not introduce `fnpack` or official packager downloads. Inner `app.tgz` contains `app/` and `ui/`; outer `appstore.grafana.alloy.<package-version>.<arch>.tgz` contains `package/`, including `package/app.tgz`.
- Running `main.yml` with `make_release=false` builds artifacts; `true` also publishes a GitHub prerelease.

## Runtime conventions

- `package/cmd/main` runs `${TRIM_APPDEST}/app/alloy-linux`; `package/config/privilege` intentionally specifies root access for host logs and Docker discovery.
- Persistent files live under `/var/apps/grafana.alloy/shares/grafana.alloy/`: `config.alloy.conf`, `storage/` and `grafana.alloy.log`. Use these script-defined paths when updating documentation.
- Copy the default configuration only when the user's configuration is absent. Preserve existing configuration and storage during upgrades.
- The default template collects Docker and host logs and sends them to Loki at `http://127.0.0.1:3100/loki/api/v1/push`.

## Validation and hygiene

- There is no standalone build script or test suite. For packaging changes, reproduce each architecture workflow's commands in disposable Linux/WSL staging directories; its `sed` and `mv` steps mutate files.
- Check shell syntax with `for script in package/cmd/*; do bash -n "$script"; done`, parse changed workflow YAML and run `git diff --check`.
- With a matching host architecture and the downloaded binary in place, run `./app/alloy-linux --version` and `./app/alloy-linux validate app/config.alloy.conf.template`.
- Inspect both archives: resolved manifest version/architecture/platform, nested archive layout, ELF architecture and executable permissions. Validate fnOS installation and lifecycle behavior on a real device; WSL build checks do not cover these.
- Keep Bash files UTF-8 with LF endings. Keep translated metadata consistent when editing descriptions.
- Put local downloads, binaries and build outputs under ignored `tmp/`; retain only useful outputs. Preserve README files, avoid unrelated files or standalone license material, and do not overwrite unrelated worktree changes.

## Commit messages

- Follow `appstore.grafana`'s Chinese `feat: ...` title style. State the concrete change; avoid generic titles such as `upload <URL>` or `update`.
- Start the body with overview paragraphs describing the resulting behavior and purpose.
- Follow the overview with changed file paths, each followed by bullets in the form `- <变更点> 新增/修改/修复/删除：<具体变化>`; select the applicable change verb. Group closely related files when they share the same changes.
- Describe only changes included in the commit. Keep validation results in the PR description or work report; do not add a separate validation section to commit messages.

Example:

```text
feat: 更新 Grafana Alloy 版本并校验官方发布包

升级 Grafana Alloy 的打包版本，保留 amd64 和 arm64 支持。下载后校验官方 SHA256，通过校验后再解压打包。

.github/workflows/main.yml
- 版本配置 修改：同步上游版本与应用包版本。

.github/workflows/pack_amd64.yml、.github/workflows/pack_arm64.yml
- 下载校验 新增：使用所选版本的 SHA256SUMS 校验对应架构压缩包。
```
