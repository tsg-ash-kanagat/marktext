# Developer Documentation

## 1. Project Setup

### 1.1 Pre-Requisites

- Python (`>= 3.12`)

- Node.JS (`22.x LTS`) — **must match the Electron runtime version**
  - Electron 39 bundles Node 22 internally; using Node 25+ will cause native module compilation failures
  - Install via `brew install node@22` (macOS) or `nvm install 22`
  - Verify with `node -v` — should show `v22.x.x`

- A C++ compiler with **C++17** support (for `npm install`) and **C++20** support (for `@electron/rebuild`)
  - macOS: Apple Clang 17+ (included with Xcode 16+)
  - Linux: GCC 10+ or Clang 10+
  - Windows: MSVC 2022

### 1.2 Linux Specific Pre-requisites

- Linux environments require additional dependencies, please see [Linux Specific Pre-reqs](LINUX_DEV.md)

### 1.3 Windows Specific Pre-requisites

- You will need [Build Tools for Visual Studio 2022](https://visualstudio.microsoft.com/downloads/) (Scroll all the way to the bottom)
  - Additionally, you need **spectre-mitigated MSVC**, go to "Individual Components" and select "MSVC ... - VS2022 C++ Spectre-Mitigated Libs"
  - Many native libraries do not support ClangCL well yet, hence we force it to use MSVC in our `.npmrc`

### 1.4 macOS Specific Pre-requisites

- **Node.js 22 LTS** is required. If you have a newer version (e.g. Node 25) installed via Homebrew:
  ```bash
  brew install node@22
  brew unlink node
  brew link node@22 --force --overwrite
  ```
- Native modules (`native-keymap`, `ced`) require the C++ standard flag to be set explicitly:
  - `npm install` requires **C++17** (for Node 22's V8 headers)
  - `@electron/rebuild` requires **C++20** (for Electron 39's V8 headers)
- On **macOS 26 (Tahoe)**, packaged app bundles must be re-signed after building to fix Team ID mismatches between Electron helper processes (see [Building for macOS](#17-building-for-macos) below)

### 1.5 Clone and Install

```bash
git clone https://github.com/peterjthomson/marktext.git
cd marktext

# macOS/Linux: set C++17 standard for native module compilation
CXXFLAGS="-std=c++17" npm install

# Windows: npm install (MSVC handles standards automatically)
npm install
```

### 1.6 Create minified locale files

- This is **automatically ran** when building for production, but not for dev for performance

```
npm run minify-locales
```

### 1.7 Run in Development

```bash
npm run dev
```

#### 1.7.1 Some Points to Note:

- The `main` and `preload` processes are **NOT** automatically hot-loaded on edit, you need to **reload the development process** on each edit unfortunately
  - The good news is Vite bundles it _really really quickly_ so it shouldnt be too big of a hassle
- Although the `renderer` process is hot-loaded, loss of states can often lead to **weird errors**. I recommend doing a full reload if this happens
- Compile targets:
  - `main` and `preload` still compile to `CommonJS`
  - `renderer` is `ESModules` only (take note when using any legacy `CommonJS` libraries)

### 1.8 Build for Production

```bash
# For Windows
$ npm run build:win

# For macOS (see section below for details)
$ npm run build:mac

# For Linux
$ npm run build:linux
```

### 1.9 Building for macOS

The macOS build requires extra steps due to native module compilation and code signing.

#### Step 1: Rebuild native modules for Electron (requires C++20)

```bash
CXXFLAGS="-std=c++20" npx @electron/rebuild
```

#### Step 2: Build and package

```bash
npm run minify-locales && npm run build && npx electron-builder --mac --publish never
```

Or as a single command (if `@electron/rebuild` has already been run):

```bash
npm run minify-locales && npm run build && npx electron-builder --mac --publish never
```

#### Step 3: Re-sign the app bundle (required on macOS 15+ / Sequoia / Tahoe)

macOS 15+ enforces that all binaries in an app bundle share the same code signing Team ID. The default electron-builder ad-hoc signing can produce mismatched signatures across helper processes (GPU, Renderer, Network), causing the app to crash on launch with `dyld` errors about "different Team IDs".

Fix by re-signing the entire bundle:

```bash
codesign --deep --force --sign - dist/mac-arm64/marktext.app
```

Verify the signature:

```bash
codesign --verify --deep --strict dist/mac-arm64/marktext.app
```

#### Step 4: Launch

```bash
open dist/mac-arm64/marktext.app
```

The DMG is also available at `dist/marktext-mac-arm64-1.3.0.dmg`.

> **Note:** If macOS blocks the app (unidentified developer), clear the quarantine attribute:
> ```bash
> xattr -cr dist/mac-arm64/marktext.app
> ```

## 2. Sub-sections

- [Project architecture](ARCHITECTURE.md)
- [Build instructions](BUILD.md)
- [Debugging](DEBUGGING.md)
- [Interface](INTERFACE.md)
- [Steps to release MarkText](RELEASE.md)
- [Prepare a hotfix](RELEASE_HOTFIX.md)
- [Internal documentation](code/README.md)
