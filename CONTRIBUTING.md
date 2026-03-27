# Contributing to c64

Thanks for your interest! Whether it is a bug report, feature request, or code
contribution, we appreciate your help making c64 better.

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 20 or later
- A C64 Ultimate device is helpful for testing but not required --
  the test suite uses mocks and fixtures

### Setup

```bash
git clone https://github.com/jeffsand/c64.git
cd c64
npm install
npm run build
npm test
```

### Development workflow

```bash
# Make changes in src/
npm run build          # compile TypeScript
npm test               # run all tests
npm run dev -- info    # run without building (uses tsx)
```

## Making Changes

1. Fork the repo and create a branch from `main`
2. Make your changes in `src/`
3. Add tests for new functionality in `src/tests/`
4. Run `npm run build` -- must compile with zero errors
5. Run `npm test` -- all tests must pass
6. Update `CHANGELOG.md` with your changes
7. Open a PR with a clear description of what and why

## Code Style

### TypeScript

- Strict mode (`strict: true` in tsconfig)
- No `any` types -- use `unknown` and narrow with type guards
- Every exported function has a JSDoc comment
- Prefer `const` over `let`

### Output conventions

- **Data goes to stdout** (`console.log`, `printData`, `printTable`)
- **Messages go to stderr** (`console.error`, `printInfo`, `printSuccess`, `printError`)
- This ensures `c64 info --json | jq` works correctly
- All data commands must support `--json` for structured output

### Error messages

Every error should have three parts:
1. **What happened** -- clear description of the failure
2. **Why** -- likely cause
3. **What to do** -- specific command to run or action to take

```typescript
// Good
printError(
  "Cannot connect to C64 Ultimate at 192.168.1.42",
  "The device is not responding. Try:\n  c64 discover\n  c64 config set device.host <IP>"
);

// Bad
printError("Connection refused");
```

### Testing

- Tests use Node.js built-in test runner (`node:test`)
- CLI integration tests run the compiled binary via `execFileSync`
- Unit tests import modules directly
- Use `fixtures/` for test data (D64 images, etc.)
- Use temp directories for tests that write files

## What Makes a Good PR

- **One feature or fix per PR** -- easier to review and revert if needed
- **Tests included** -- if you add a command, add a test
- **CHANGELOG.md updated** -- describe what changed under `[Unreleased]`
- **Docs updated** -- if behavior changes, update README/ARCHITECTURE
- **Commit messages are clear** -- describe what and why, not just what

## Reporting Bugs

Open an issue with:
- What you tried to do
- What happened instead
- Output of `c64 --version`
- Your OS and Node.js version
- Your C64 Ultimate model and firmware version (if relevant)

## Feature Requests

Open an issue describing:
- What you would like the CLI to do
- Why it would be useful
- Example of how you would use it (command + expected output)

## Questions

Open an issue! No question is too basic. If you are confused by something,
others probably are too, and your question helps us improve the docs.
