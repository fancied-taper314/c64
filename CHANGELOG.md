# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [0.1.2] - 2026-03-27

### Added

- Background update checker (checks npm once/day, notifies on next run)
- Homebrew formula (`brew tap jeffsand/tools && brew install c64`)
- GitHub Actions release workflow (tag push triggers npm publish + GitHub Release)
- Demo GIF for README

### Fixed

- FTP upload timeout handling for C64 Ultimate devices
- Version sync between package.json and version.ts

## [0.1.0] - 2026-03-27

### Added

- **Core commands**: `c64 info`, `c64 drives`, `c64 mount`, `c64 eject`,
  `c64 run`, `c64 play`, `c64 reset`, `c64 reboot`, `c64 type`
- **File operations**: `c64 ls` (FTP directory listing), `c64 upload` (FTP upload)
- **Smart input resolution**: local files, URLs, ZIP archives, and directories
  are all accepted by mount/run/play -- resolved automatically
- **Full play sequence**: `c64 play` does mount, reset, LOAD"*",8,1, and RUN
  with configurable timing delays
- **PETSCII keyboard injection**: `c64 type` converts text to PETSCII and
  injects into the C64 keyboard buffer via TCP port 64
- **Network discovery**: `c64 discover` scans the local subnet for C64
  Ultimate devices with --save, --all, and --subnet options
- **Data disk management**: `c64 disk create`, `c64 disk list`, `c64 disk dir`
  for creating and inspecting blank D64 save disks
- **D64 parser**: parse, create, and rename D64 disk images. Handles standard
  (174848) and extended (175531, 196608, 197376) sizes
- **Watch mode**: `c64 watch` polls drive status every 2 seconds and prints
  changes. Clean SIGINT handling
- **Shell completions**: `c64 completions bash|zsh|fish` generates tab
  completion scripts with command descriptions
- **Configuration**: `c64 config show`, `c64 config set`, `c64 config init`
  with XDG-compatible config at ~/.config/c64/config.json
- **Global flags**: --host, --json, --quiet, --timeout, --no-color, --no-input
- **Environment variables**: C64_HOST, C64_TIMEOUT, NO_COLOR
- **JSON output**: every data command supports --json for scripting and agents
- **Error handling**: every error has what/why/what-to-do guidance, distinct
  exit codes (0=success, 1=general, 2=usage, 3=device, 4=network)
- **FTP timeout workaround**: handles the Ultimate FTP server not sending
  transfer-complete acknowledgments
- **71 tests** across 13 test suites
- **GitHub Actions CI** building and testing on macOS, Linux, and Windows
