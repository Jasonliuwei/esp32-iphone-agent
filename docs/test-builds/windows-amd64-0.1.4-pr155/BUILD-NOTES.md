# TEST build: Windows amd64 esp32agent (not a release)

This is a labeled **TEST** executable for a later local check of one hypothesis: official Windows `esp32agent` 0.1.4 returns `{"devices":[]}` because tinygo bluetooth v0.16.0 corrupts 128-bit service UUIDs, and upstream PR 155 (bluetooth v0.17.0) is the fix.

It is **not** official CLI 0.1.4, **not** official CLI 0.1.5, and **not** a GitHub Release. No `cli-v0.1.5` tag exists. The file name and the embedded version string both say test.

| | |
|---|---|
| File | `esp32agent-TEST-0.1.4-pr155-windows-amd64.exe` |
| SHA-256 | `0c07a657c9ef96bdf791a573dc6ae702d096139434b5d408c0d3acacf2c11e84` |
| Size | 9651712 bytes |
| `version` string | `0.1.4+test.pr155.eac30394` |
| Built | 2026-11-23, Linux host, Go 1.26.0, `GOOS=windows` `GOARCH=amd64` |
| Signed | No. No Authenticode signature. |
| Hardware | Not tested. No Windows PC, BLE adapter, ESP32, or iPhone was used. |

The zip next to this note contains the exe, this note, `LICENSE`, `NOTICE`, and `pr155-go-mod-sum.patch`. Digests of the delivered files are in `SHA256SUMS`.

## What was verified about the hypothesis

Official Windows 0.1.4 is tag `cli-v0.1.4`.

- Annotated tag object: `cce5edc09ae531b2459652cc73cb2d46fd89c793`
- Commit: `e36a9a34d3e9e59487f1ac6f3e3aee270d423d17`
- `cli/go.mod` at that commit requires `tinygo.org/x/bluetooth v0.16.0`

`cli/internal/ble/ble.go` keeps a scan hit only when `HasServiceUUID` matches `6e400001-a9e8-4b1b-9c39-6d6f8a3c0010` (`svcUUID`, base + `0010`). That file's git blob is `f6d060f023f04dc83ce10ed8b2f97a9dcbf469ba` at `cli-v0.1.4`, at the PR 155 commit, and at fork `main` `6af87ec7a5ad2f19e2d98d39222e19ca4fd1958b`. The filter did not change. The library that fills the UUID did.

Checked in the module cache after `go mod download`:

- v0.16.0 `gap_windows.go` takes service UUIDs from generic `IVector.GetAt`. The comment there says the value is a GUID, then the code casts that pointer-sized result to `syscall.GUID`. A GUID is 16 bytes, so `Data4` is not actually read. Upstream PR 155 describes the corrupted id as `6e400001-a9e8-4b1b-d073-dfc4e8000000` instead of `6e400001-a9e8-4b1b-9c39-6d6f8a3c0010`.
- v0.17.0 `gap_windows.go` adds `guidVectorAt`. Its comment states that `GetAt` on `IVector<GUID>` writes a 16-byte GUID, and the function passes a real `syscall.GUID` to the vtable `GetAt` via `syscall.SyscallN`. Module sum recorded in this binary: `h1:jB93QDKgjw7jLarR/3hrzm1aIgpyzqTQye7QuLJ4BSY=`.

PR 155 is the change that moves the CLI from v0.16.0 to v0.17.0.

- https://github.com/onehumanbeing/esp32-iphone-agent/pull/155
- Merged 2026-11-14T14:47:45Z
- Squash commit: `5d4061ee75adfbcc54bca136f9a50af04b5df700`
- Parent: `cc566b9d946bac7980177d83fc92e398553d919d`
- The commit changes only `cli/go.mod` and `cli/go.sum`
- Issue discussed there: https://github.com/onehumanbeing/esp32-iphone-agent/issues/114

`cli/go.mod` and `cli/go.sum` at `cli-v0.1.4` are byte-identical to those at the parent of `5d4061ee`. Copying the two files from `5d4061ee` onto the tag is the official patch, with no other edits. `git apply --check` of `pr155-go-mod-sum.patch` succeeds on a clean `e36a9a34` tree.

v0.17.0 is the whole module, not a one-function fork of v0.16.0. The same `gap_windows.go` also caches and merges advertisement fields during a scan. That behavior comes along with the official dependency bump. The Windows build info links `tinygo.org/x/bluetooth v0.17.0` and `github.com/saltosystems/winrt-go v0.0.0-20261015065339-8ef619c103a3`.

The PR report says the author's machine was Windows 11, a MediaTek MT7925, and firmware 0.2.5, and that Windows itself already saw the board. Jason's later check is a different adapter (Realtek) and official dual firmware 0.2.7. The UUID decode bug does not depend on which adapter Windows uses, so this is the relevant official fix to try. This build does not show that his board will appear.

## Source that was compiled

Minimal tree: tag `cli-v0.1.4` plus the two dependency files from `5d4061ee`. Nothing else.

| Item | Value |
|---|---|
| Base commit | `e36a9a34d3e9e59487f1ac6f3e3aee270d423d17` |
| Tag | `cli-v0.1.4` (`cce5edc09ae531b2459652cc73cb2d46fd89c793`) |
| Files replaced | `cli/go.mod`, `cli/go.sum`, taken from `5d4061ee75adfbcc54bca136f9a50af04b5df700` |
| `cli/go.mod` blob | `26afd00e79549104e2aaaba83c31c75285148c11` |
| `cli/go.sum` blob | `0e40b1cc7bc9467c17feac50c54132705af6c49e` |
| Resulting tree | `eac303941c5d1de2337fee82d9438e7aef61274e` |
| Other patches | None |
| Fork `main` when this note was written | `6af87ec7a5ad2f19e2d98d39222e19ca4fd1958b` (already contains `5d4061ee`) |

`go version -m` on the exe prints the main module as `(devel)` because `-buildvcs=false` was set. That flag is the one addition to the official link line. A default VCS stamp would have recorded `e36a9a34` and a dirty worktree, which is not this tree. The tree id is in the version string instead: `0.1.4+test.pr155.eac30394`.

### Later upstream changes that are not in this exe

Every commit after `e36a9a34` is absent except the two dependency files from `5d4061ee`. Commits that touch `cli/` and are not in this binary:

| Commit | What it changes in `cli/` | In this exe |
|---|---|---|
| `0a7c073` CLI 0.1.4 npm/dsh version bump (#129) | `cli/npm/package.json` and docs | No |
| `8574ef7` daemon test (#126) | `cli/internal/daemon/conn_test.go` | No (test only) |
| `39aa23e` App 1.0.6 (#153) | memory commands and their tests | No |
| `5d4061ee` PR 155 | `cli/go.mod`, `cli/go.sum` only | Yes, those two files only |
| `7947101` Rebrand to Manina AI (#167) | Handsy strings, `support@manina.ai`, `intents@manina.ai` | No. This exe still says Handsy in those strings |
| `d4d34c0` App 1.0.7 (#173) | `.esp32intent` wording becomes `.intent` in help text | No |
| `b1b80c7` Bluetooth-mode skill docs | docs / skill text | No |

App, firmware, Hermes, and the rest of `main` after the tag are not in the binary. `cli/internal/ble/ble.go` would have been the same if `main` had been built. Building `main` would still have shipped the memory commands and the rebrand strings, so it is not the minimal revision.

Current `main` has no published CLI tag after `cli-v0.1.4`. The changelog line that says npm package `esp32agent` 0.1.5 is the npm wrapper version beside CLI binary 0.1.4, not a CLI 0.1.5 release.

## Toolchain and flags

Official build rules used:

- `cli/README.md` Development, and `.github/workflows/cli.yml` job `build`, Windows row: `runner: ubuntu-latest`, `goos: windows`, `goarch: amd64`, `cgo: "0"`. The workflow comment says WinRT is pure Go.
- `actions/setup-go@v6` with `go-version-file: cli/go.mod`. That file says `go 1.26.0`. `check-latest` is not set.
- Link line from the workflow: `go build -trimpath -ldflags "-s -w -X github.com/onehumanbeing/esp32-iphone-agent/cli/internal/app.Version=$version" -o ... ./cmd/esp32agent`

Compiler actually used:

- `go version go1.26.0 linux/amd64`
- Tarball: https://go.dev/dl/go1.26.0.linux-amd64.tar.gz
- SHA-256: `aac1b08a0fb0c4e0a7c1555beb7b59180b05dfc5a3d62e40e9de90cd42f88235`
- `GOTOOLCHAIN=local` so the `go` command did not download another toolchain

Command:

```sh
cd cli
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -trimpath -buildvcs=false \
  -ldflags '-s -w -X github.com/onehumanbeing/esp32-iphone-agent/cli/internal/app.Version=0.1.4+test.pr155.eac30394' \
  -o esp32agent-TEST-0.1.4-pr155-windows-amd64.exe ./cmd/esp32agent
```

`go version -m` on the exe:

- `go1.26.0`
- path `github.com/onehumanbeing/esp32-iphone-agent/cli/cmd/esp32agent`
- mod `github.com/onehumanbeing/esp32-iphone-agent/cli` `(devel)`
- `CGO_ENABLED=0`
- `GOOS=windows`
- `GOARCH=amd64`
- `GOAMD64=v1`
- `-trimpath=true`
- dep `tinygo.org/x/bluetooth v0.17.0`

The version string uses `+` so `upgrade.Valid` is false (`parse` only treats `-` as a pre-release, and `4+test` is not an integer). `version` therefore cannot be mistaken for a plain `0.1.4` or `0.1.5`. Plain `upgrade` refuses a development build. `upgrade --force` would download the published release over **this test file**. Do not run it.

## Tests and file checks

Run on the Linux build host with the same Go 1.26.0, in `cli/` of the tree above. These are host tests of that source. They are not a run of the Windows exe and not a BLE test.

| Command | Result |
|---|---|
| `go vet ./...` | pass |
| `go test ./...` | pass |
| `go test -race ./...` | pass (Linux test binaries, CGO on for the race detector only) |
| `go test -count=1 -v ./internal/video/ -run TestDecoderOneInOneOut` | PASS in 0.09s (ffmpeg and libx264 present) |

Packages with no test files: `cli`, `cmd/esp32agent`, `internal/ble`, `internal/config`, `internal/upgrade`. Nothing in-tree tests the Windows service-UUID decode.

The Windows exe was not launched. Checked as a file:

- `file`: `PE32+ executable (console) x86-64, for MS Windows, 8 sections`
- PE header: machine `0x8664` (AMD64), optional magic `0x20b` (PE32+), subsystem `3` (`IMAGE_SUBSYSTEM_WINDOWS_CUI`), 8 sections, characteristics `0x0022` (executable, large-address aware)
- Embedded bytes contain `0.1.4+test.pr155.eac30394`
- `go version -m` reports `GOOS=windows` and `GOARCH=amd64`

## License

The compiled tree is MIT, Copyright (c) 2026 Henry Yuan. The zip includes `LICENSE` and `NOTICE` from that tree (`NOTICE` there still says "Handsy AI / ESP32 iPhone Agent"; later main renamed the product to Manina AI). Keep those notices with the exe.

Based on [ESP32 iPhone Agent](https://github.com/onehumanbeing/esp32-iphone-agent) by Henry Yuan.

## Reproduce

```sh
git clone https://github.com/onehumanbeing/esp32-iphone-agent.git
cd esp32-iphone-agent
git checkout e36a9a34d3e9e59487f1ac6f3e3aee270d423d17
git apply pr155-go-mod-sum.patch
# expected tree: eac303941c5d1de2337fee82d9438e7aef61274e
#   git add -A && git write-tree
```

Install Go 1.26.0 from the tarball above (SHA-256 listed above), then run the build command in [Toolchain and flags](#toolchain-and-flags) from `cli/`. `go.mod` and `go.sum` after the patch must match blobs `26afd00e79549104e2aaaba83c31c75285148c11` and `0e40b1cc7bc9467c17feac50c54132705af6c49e`.

This note branch lives only on the fork https://github.com/Jasonliuwei/esp32-iphone-agent (`cursor/windows-amd64-ble-test-cli-a2d0`). Nothing in this build was pushed to `onehumanbeing/esp32-iphone-agent`. No tag and no release were created.

## Windows commands for a later local check

Do this on the Windows PC. This build environment did not.

Leave the official file where it is. Do not copy this test exe over it, and do not rename this test exe to `esp32agent.exe` in that folder.

Official file, untouched rollback target:

`C:\Users\webfi\esp32agent-cli\esp32agent.exe`

Put the test exe in a **different** folder. Power the board. Within 3 minutes of power-on (the window in `cli/README.md`):

```powershell
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe version
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe devices
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe pair
```

`version` does not use Bluetooth. It should print JSON whose `version` is `0.1.4+test.pr155.eac30394`.

`devices` is the check for the empty list. `pair` is the same 3-minute window. In Bluetooth mode, pair the board on the iPhone in that same window (Settings > Accessibility > Touch > AssistiveTouch > Devices > Bluetooth Devices). `ffmpeg` is for a later `screenshot`, not for `version`, `devices`, or `pair`.

Both binaries use `%AppData%\esp32agent` (keys, `daemon.sock`, `daemon.log`). If a daemon is already running and its version string differs, this exe asks it to stop and starts its own from this test file. That still does not modify `C:\Users\webfi\esp32agent-cli\esp32agent.exe`.

When finished:

```powershell
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe daemon stop
```

Rollback is to run the original file:

`C:\Users\webfi\esp32agent-cli\esp32agent.exe`

Do not run `upgrade` or `upgrade --force` on the test exe.

The exe is unsigned. An unknown-publisher or SmartScreen prompt is expected. This file is an unofficial test build.

## 在 Windows 上怎么试、怎么退回

这一份是测试文件，不是官方 0.1.4，也不是官方 0.1.5。没有在真机上跑过。

不要覆盖 `C:\Users\webfi\esp32agent-cli\esp32agent.exe`。测试 exe 放在另一个目录。给板子上电后 3 分钟内：

```powershell
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe version
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe devices
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe pair
```

`version` 应显示 `0.1.4+test.pr155.eac30394`。`devices` 用来看是不是还会得到空列表。蓝牙模式下，同一段 3 分钟里还要在 iPhone 上配对板子（设置 > 辅助功能 > 触控 > 辅助触控 > 设备 > 蓝牙设备）。

试完后：

```powershell
.\esp32agent-TEST-0.1.4-pr155-windows-amd64.exe daemon stop
```

退回时继续用原来的官方文件：`C:\Users\webfi\esp32agent-cli\esp32agent.exe`。不要对测试 exe 执行 `upgrade` 或 `upgrade --force`。两个程序共用 `%AppData%\esp32agent`，但测试程序不会改官方 exe 那个文件。文件没有代码签名，系统提示未知发布者是正常的。
