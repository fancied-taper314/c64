# Architecture

## Overview

The c64 CLI is a TypeScript application built with Commander.js. It communicates
with C64 Ultimate devices over three protocols: HTTP REST (port 80),
TCP binary (port 64), and FTP (port 21).

## Source Layout

```
src/
  index.ts              Entry point -- parses args, handles top-level errors
  cli.ts                All command definitions (Commander.js derive pattern)
  config.ts             Config file at ~/.config/c64/config.json
  error.ts              Error types with exit codes and help text
  output.ts             Output formatting (JSON, table, TTY detection)
  version.ts            Package version constant
  resolve.ts            Smart input resolution (URL, ZIP, directory, file)
  d64.ts                D64 disk image parser (parse, create, rename)
  completions.ts        Shell completion generators (bash, zsh, fish)

  api/                  Protocol clients
    rest.ts             HTTP REST client for /v1/* endpoints
    socket.ts           TCP port 64 binary protocol (keyboard, reset)
    petscii.ts          PETSCII character encoding (encode/decode)
    types.ts            TypeScript interfaces for API responses

  commands/             One file per command (or command group)
    info.ts             c64 info
    drives.ts           c64 drives
    mount.ts            c64 mount (resolves input, uploads via FTP, mounts via REST)
    eject.ts            c64 eject
    run.ts              c64 run (auto-detects file type)
    play.ts             c64 play (full mount + reset + LOAD + RUN sequence)
    reset.ts            c64 reset
    reboot.ts           c64 reboot
    type.ts             c64 type (PETSCII keyboard injection)
    ls.ts               c64 ls (FTP directory listing)
    upload.ts           c64 upload (FTP file upload + helper used by mount/run/play)
    discover.ts         c64 discover (subnet scanning)
    watch.ts            c64 watch (polling drive status)
    disk.ts             c64 disk list/create/dir (data disk management)
    config.ts           c64 config show/set/init

  tests/                Node.js test runner (node:test)
    cli.test.ts         End-to-end CLI tests (runs the compiled binary)
    config.test.ts      Config file unit tests
    petscii.test.ts     PETSCII encoding/decoding tests
    d64.test.ts         D64 parser tests
    resolve.test.ts     Input resolution tests (file, zip, directory)
    disk.test.ts        Data disk management tests
    discover.test.ts    Discovery module tests

fixtures/               Test data
  test.d64              Hand-built D64 with BASIC "HELLO FROM C64 CLI" program
  paradroid.d64         Paradroid (1985) for integration testing
  Badlands.crt          Badlands cartridge for CRT testing
  llamas.crt            Metagalactic Llamas cartridge
```

## Key Design Decisions

**One command per file.** Each command in `src/commands/` exports a single async
function. This makes it easy to find the code for any command, and easy to add
new commands -- just create a file and wire it up in `cli.ts`.

**Lazy imports.** Commands are imported dynamically (`await import(...)`) in
`cli.ts` so that only the code for the invoked command is loaded. This keeps
startup time fast.

**Errors have three parts.** Every error in `error.ts` includes: what happened,
why it likely happened, and what to do about it. This is critical for both
human users and AI agents that invoke the CLI.

**stdout is for data, stderr is for humans.** All informational messages
(`printInfo`, `printSuccess`, `printError`) go to stderr. Data output
(`printData`, `printTable`) goes to stdout. This means `c64 info --json | jq`
works correctly -- the JSON goes to jq, status messages don't.

**FTP timeout workaround.** The Ultimate's FTP server sometimes does not send
"226 Transfer complete" after a successful upload. The upload code treats curl
exit code 28 (timeout) as success, since the data transfer completes before
the acknowledgment times out.

## Adding a New Command

1. Create `src/commands/mycommand.ts`:
   ```typescript
   import { resolveHost, resolveTimeout } from "../config.js";
   import { NoHostConfiguredError } from "../error.js";
   import { printSuccess } from "../output.js";

   export async function mycommand(opts: Record<string, unknown>): Promise<void> {
     const host = resolveHost(opts);
     if (!host) throw new NoHostConfiguredError();
     // ... your logic here
     printSuccess("Done", opts);
   }
   ```

2. Add the command definition in `src/cli.ts`:
   ```typescript
   program
     .command("mycommand")
     .description("What it does")
     .addHelpText("after", "\nExamples:\n  c64 mycommand")
     .action(async (_opts, cmd) => {
       const { mycommand } = await import("./commands/mycommand.js");
       await mycommand(cmd.optsWithGlobals());
     });
   ```

3. Add tests in `src/tests/mycommand.test.ts`

4. Update `src/completions.ts` to include the new command

5. Update CHANGELOG.md

## Protocol Reference

### REST API (port 80)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v1/info` | Device info |
| GET | `/v1/drives` | Drive status (non-standard JSON format) |
| PUT | `/v1/drives/{a\|b}:mount?image=<path>` | Mount disk image |
| PUT | `/v1/drives/{a\|b}:remove` | Eject disk |
| PUT | `/v1/runners:run_crt?file=<path>` | Run cartridge |
| PUT | `/v1/runners:run_prg?file=<path>` | Run PRG |
| PUT | `/v1/machine:reset` | Reset C64 |
| PUT | `/v1/machine:reboot` | Reboot Ultimate |

### TCP Binary Protocol (port 64)

Wire format: `[command: uint16 LE] [payload_length: uint16 LE] [payload]`

| Command | Code | Description |
|---------|------|-------------|
| CMD_KEYB | 0xFF03 | Inject up to 10 bytes into keyboard buffer |
| CMD_RESET | 0xFF04 | Reset the C64 |

### FTP (port 21)

Anonymous access. Used for file uploads (`curl -T`) and directory listings
(`curl --list-only`). Upload destination is typically `/Temp/` for staging
before mount/run.
