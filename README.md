# 🛠️ GoToolsVSCode

> **A native Windows toolset for Go development in VS Code — built with Delphi**
>
> Set up once. Works across every Go workspace automatically.

---

## ✨ Overview

**GoToolsVSCode** is a set of 8 standalone `.exe` tools that integrate deeply with VS Code's User-level `tasks.json`.  
Unlike shell scripts or Makefiles, these are native Win32 executables — no PowerShell wrapper, no encoding issues, no dependencies.

Run `GoConfig.exe` once → `Ctrl+Shift+B` shows a full task picker in every Go project you open.

---
https://raw.githubusercontent.com/KieuManh366377/GoToolsVSCode/main/GoToolsVSCode.png

## 📦 Tools

| Tool | Version | Description |
|------|---------|-------------|
| ⚙️ `GoConfig.exe` | v1.3 | One-time setup — scans tools, merges into VS Code User tasks |
| 📦 `GoBuildDLL.exe` | v1.1 | Build Go DLL for amd64 + x86 with Garble obfuscation |
| 🚀 `GoBuildExe.exe` | v1.1 | Build Go EXE for amd64 + x86 with Garble obfuscation |
| 🔍 `GoCheckALL.exe` | v1.2 | Parallel multi-project checker: go vet + staticcheck + build |
| 👀 `GoWatcher.exe` | v1.2 | File watcher — auto-check + auto-build on `*.go` save |
| 🧹 `GoClean.exe` | v1.3 | Remove build artifacts + `go clean -cache` |
| 📋 `GoModSync.exe` | v1.2 | Run `go mod tidy` in parallel across all projects |
| 🆕 `GoNewProject.exe` | v1.3 | Create a new Go project from template, open in VS Code |

---

## 🚀 Quick Start

### Step 1 — Place tools in one folder

```
C:\GoToolsVSCode\
    GoConfig.exe
    GoBuildDLL.exe
    GoBuildExe.exe
    GoCheckALL.exe
    GoWatcher.exe
    GoClean.exe
    GoModSync.exe
    GoNewProject.exe
```

### Step 2 — Run GoConfig.exe (once only)

```
C:\GoToolsVSCode\GoConfig.exe
```

This automatically:
- ✅ Detects all `Go*.exe` tools in the folder
- ✅ Merges tasks into `%APPDATA%\Code\User\tasks.json`
- ✅ Creates `.gobuild\*.cfg` sample config files
- ✅ Generates `HUONG_DAN.md` usage guide

> Supports **VS Code stable**, **VS Code Insiders**, and **Cursor**

### Step 3 — Configure your Go project

Copy the sample `.cfg` into your project:

```
<your-project>\.gobuild\GoBuildDLL.cfg
```

Edit `ProjectPath` in the `.cfg` file to point to your Go project.

### Step 4 — Use in VS Code

Open any Go project in VS Code:

```
Ctrl + Shift + B  →  task picker appears  →  select tool
```

---

## ⚙️ How It Works

```
GoConfig.exe
    │
    ├── Scans Go*.exe in its folder
    ├── Merges into %APPDATA%\Code\User\tasks.json
    │       (applies to ALL workspaces automatically)
    └── Creates .gobuild\*.cfg sample configs

VS Code (any workspace)
    │
    └── Ctrl+Shift+B
            │
            ├── GoBuildDLL   →  reads .gobuild\GoBuildDLL.cfg
            ├── GoBuildExe   →  reads .gobuild\GoBuildExe.cfg
            ├── GoCheckALL   →  scans workspace for go.mod files
            ├── GoWatcher    →  watches *.go, debounce 800ms
            ├── GoClean      →  removes artifacts
            ├── GoModSync    →  go mod tidy all projects
            └── GoNewProject →  scaffold new project
```

---

## 📁 Project Config (.cfg)

Each build tool reads a `.cfg` file from the `.gobuild\` subfolder of your project.  
`GoNewProject.exe` creates this file automatically with `ProjectPath` pre-filled.

```ini
# GoBuildDLL.cfg
ProjectPath = D:\MyGoProjects\MyLib
OutputDir   = D:\MyGoProjects\MyLib\release
DLLName     = MyLib
```

```ini
# GoBuildExe.cfg
ProjectPath = D:\MyGoProjects\MyApp
OutputDir   = D:\MyGoProjects\MyApp\bin
ExeName     = MyApp
UseGarble   = true
LDFlags     = -s -w
```

---

## 🔍 GoCheckALL — Parallel Checker

Scans all `go.mod` under `${workspaceFolder}` and runs checks in parallel threads:

```
[16:17:46] [INFO] === GO TOOLKIT CHECKER ===
[16:17:46] [OK]   go.mod found
[16:17:46] [OK]   No duplicate functions
[16:17:46] [OK]   go vet passed
[16:17:46] [OK]   staticcheck passed
[16:17:48] [OK]   build amd64 OK
[16:17:49] [OK]   build 386 OK

=====================================
[OK]   MyProject  (2.8s)
KET QUA: TAT CA OK
=====================================
```

Checks performed:
- ✅ `go.mod` validation
- ✅ Package declaration consistency
- ✅ Duplicate function names
- ✅ Duplicate `//export` names
- ✅ `go vet ./...`
- ✅ `staticcheck ./...`
- ✅ Build for `amd64` and `386`
- ✅ On FAIL → opens VS Code at exact error line (`code --goto file.go:line`)
- ✅ Log saved to `%TEMP%\GoCheckALL_<project>.log`

---

## 👀 GoWatcher — Live File Watcher + Auto Build

Uses Win32 `ReadDirectoryChangesW` (not polling) — zero CPU when idle.

```
[14:51:50] [INFO] Watching : D:\MyGoProjects\MyLib
[14:51:50] [INFO] Log FAIL : C:\Users\...\GoWatcher_MyLib.log

[14:52:39] ┌─ [MyLib] File changed — checking...
[14:52:42] └─ [MyLib] CHECK OK  (3.1s) → building...

[14:52:42] │  [INFO] Auto-build: GoBuildDLL.exe ...
[14:53:02] │  [OK]   Build complete: GoBuildDLL.exe
[14:53:02] └─ [MyLib] BUILD done  (20.1s)
```

- Debounce: 800ms (configurable via arg)
- On FAIL → opens VS Code at exact error line automatically
- On OK → auto-build via `GoBuildDLL.exe` or `GoBuildExe.exe` (reads `.gobuild\*.cfg`)
- Log saved to `%TEMP%\GoWatcher_<project>.log`
- Press `Ctrl+C` to stop cleanly

---

## 🆕 GoNewProject — Project Scaffolding

Creates a complete Go project structure from template, then opens Explorer and VS Code automatically.

**3 project types:**

| Type | Description |
|------|-------------|
| `console` | Console Application |
| `dll` | DLL Library (CGo, buildmode=c-shared) |
| `webapi` | Web API (net/http, port 8080) |

**Files created automatically:**

```
MyLib\
    main.go              ← DLL template with sample exports
    go.mod
    .gitignore
    README.md
    versioninfo.json     ← DLL only
    .gobuild\
        GoBuildDLL.cfg   ← ProjectPath pre-filled, ready to use
```

After creation → Explorer opens + VS Code opens → `Ctrl+Shift+B` to build immediately.

---

## 🏗️ Technical Notes

- Built with **Delphi 13.1** — native Win32 executables, no runtime required
- All output via `WriteFile` directly — **no ACP encoding issues**
- VS Code tasks use `"type": "process"` — **no PowerShell wrapper**
- `GoWatcher` uses `isBackground: true` + `problemMatcher: []` — terminal never hangs
- Parallel builds via `TThread` + `TCriticalSection` + `MemoryBarrier`
- Go build uses `Garble -tiny -seed=random` for obfuscation
- Timestamp `[HH:MM:SS]` on every output line for precise debugging

---

## 📋 Requirements

| Requirement | Notes |
|-------------|-------|
| Windows 10 / 11 | x64 |
| VS Code | stable / Insiders / Cursor |
| Go 1.21+ | [go.dev/dl](https://go.dev/dl/) |
| GCC (MSYS2) | Required for DLL builds only |
| staticcheck | Optional — auto-detected |
| Garble | Optional — auto-detected, fallback to `go build` |
| goversioninfo | Optional — for embedding version info in DLL |

---

## 📂 Folder Structure

```
C:\GoToolsVSCode\
    GoConfig.exe
    GoBuildDLL.exe
    GoBuildExe.exe
    GoCheckALL.exe
    GoWatcher.exe
    GoClean.exe
    GoModSync.exe
    GoNewProject.exe
    .gobuild\
        GoBuildDLL.cfg   ← sample, copy to your project
        GoBuildExe.cfg   ← sample, copy to your project
    HUONG_DAN.md
    GoConfig.marker      ← created after first setup
```

---

## 💡 Why GoToolsVSCode?

| | Shell Script / Makefile | Go Extension | GoToolsVSCode |
|--|------------------------|--------------|---------------|
| UTF-8 encoding | ❌ Often broken | ⚠️ Occasional | ✅ WriteFile direct |
| PowerShell | ❌ Required | ✅ No | ✅ No |
| Multi-project parallel | ❌ Complex | ❌ No | ✅ TThread |
| File watcher CPU | ❌ Polling | ✅ Low | ✅ Win32 event ~0% |
| User-level config | ❌ Per project | ❌ Per project | ✅ Once for all |
| Auto-build on save | ❌ No | ❌ No | ✅ Yes |
| Open VS Code on error | ❌ No | ✅ Yes | ✅ Yes |
| Deploy new machine | ❌ Reinstall all | ✅ Extension | ✅ Copy folder + GoConfig |

---

## 📜 License

Binaries only — source code is not distributed.  
Free for personal and commercial use.  
MIT License — Copyright 2026 Kieu Manh

---

## 👤 Author

**Kieu Manh**  
📧 kieumanh366377@gmail.com  
🔗 [github.com/KieuManh366377](https://github.com/KieuManh366377)

---

*GoToolsVSCode — Native Go tooling for VS Code on Windows*
