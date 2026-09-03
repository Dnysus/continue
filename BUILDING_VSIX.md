# Building and Installing the Continue VSIX

Steps to build the Continue VS Code extension from source and install the packaged `.vsix` in VS Code.

## Prerequisites

- **Node.js `v26.8.1`** (see [.nvmrc](.nvmrc)) — if you use nvm, run `nvm use` from the repo root
- **npm** (comes with Node.js)
- **VS Code** with the `code` CLI available on your PATH (in VS Code: `Cmd+Shift+P` → "Shell Command: Install 'code' command in PATH")

## Build the VSIX

### Option 1: One-shot script (recommended)

From the repo root, run the install-dependencies script. It installs all dependencies, builds the internal packages, builds the GUI, and packages the extension:

```sh
./scripts/install-dependencies.sh
```

### Option 2: Manual steps

If you prefer to run each step yourself (or already have dependencies installed):

```sh
# 1. Install root-level dependencies
npm install

# 2. Build internal packages (fetch, openai-adapters, config-yaml, etc.)
node ./scripts/build-packages.js

# 3. Install core dependencies
cd core
PUPPETEER_SKIP_DOWNLOAD=true npm install
npm link
cd ..

# 4. Install GUI dependencies and build the GUI
cd gui
npm install
npm link @continuedev/core
NODE_OPTIONS="--max-old-space-size=4096" npm run build
cd ..

# 5. Install extension dependencies and package the VSIX
cd extensions/vscode
npm install
npm link @continuedev/core
npm run package
```

> `npm run package` automatically runs the `prepackage` step (bundling the GUI, native modules, etc.) and then calls `@vscode/vsce package`, so you don't need to run `vsce` directly.

### Output location

The packaged VSIX is written to:

```
extensions/vscode/build/continue-<version>.vsix
```

For example: `extensions/vscode/build/continue-1.3.40.vsix`

## Install the VSIX in VS Code

### Via the command line

```sh
code --install-extension extensions/vscode/build/continue-<version>.vsix
```

### Via the VS Code UI

1. Open the Extensions view (`Cmd+Shift+X`)
2. Click the `...` menu in the top-right of the Extensions view
3. Select **Install from VSIX...**
4. Choose the `.vsix` file from `extensions/vscode/build/`

Reload/restart VS Code after installing to make sure the new build is active.

## Rebuilding after code changes

If you've already run the full setup once and only changed extension/core/GUI code:

```sh
# Rebuild the GUI if you changed anything under gui/
cd gui && npm run build && cd ..

# Repackage the extension
cd extensions/vscode
npm run package
```

Then reinstall the new VSIX:

```sh
code --install-extension extensions/vscode/build/continue-<version>.vsix
```

## Troubleshooting

- **Node version warning**: the build scripts check your Node version against [.nvmrc](.nvmrc). Mismatched versions can cause native module (e.g. sqlite3, esbuild) issues — run `nvm use` first.
- **Out-of-memory during GUI build**: use `NODE_OPTIONS="--max-old-space-size=4096" npm run build` in `gui/`.
- **Stale artifacts**: run `node ./scripts/uninstall.js` (the `clean` task) to remove build artifacts, then rebuild from the top.
- **Pre-release build**: use `npm run package:pre-release` in `extensions/vscode/` to build a pre-release-flagged VSIX.
