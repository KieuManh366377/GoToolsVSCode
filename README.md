# 🛠️ GoToolsVSCode

> **A native Windows toolset for Go development in VS Code — built with Delphi**
>
> Set up once. Works across every Go workspace automatically.

---

## ✨ Overview

**GoToolsVSCode** is a set of 7 standalone `.exe` tools that integrate deeply with VS Code's User-level `tasks.json`.  
Unlike shell scripts or Makefiles, these are native Win32 executables — no PowerShell wrapper, no encoding issues, no dependencies.

Run `GoSetup.exe` once → `Ctrl+Shift+B` shows a full task picker in every Go project you open.

---

## 📦 Tools

| Tool | Description |
|------|-------------|
| 🔧 `GoSetup.exe` | One-time setup — scans tools, merges into VS Code User tasks |
| 📦 `GoBuildDLL.exe` | Build Go DLL for amd64 + x86 with Garble obfuscation |
| 🚀 `GoBuildExe.exe` | Build Go EXE for amd64 + x86 with Garble obfuscation |
| 🔍 `GoCheckALL.exe` | Parallel multi-project checker: go vet + staticcheck + build |
| 👀 `GoWatcher.exe` | File watcher — auto-check on `*.go` save, debounce 800ms |
| 🧹 `GoClean.exe` | Remove build artifacts + `go clean -cache` |
| 📋 `GoModSync.exe` | Run `go mod tidy` in parallel across all projects |
| 🆕 `GoNewProject.exe` | Create a new Go project from template |

---

## 🚀 Quick Start

### Step 1 — Place tools in one folder

```
C:\GoToolsVSCode\
    GoSetup.exe
    GoBuildDLL.exe
    GoBuildExe.exe
    GoCheckALL.exe
    GoWatcher.exe
    GoClean.exe
    GoModSync.exe
    GoNewProject.exe
```

### Step 2 — Run GoSetup.exe (once only)

```
C:\GoToolsVSCode\GoSetup.exe
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

![Task Picker](https://i.imgur.com/placeholder.png)

---

## ⚙️ How It Works

```
GoSetup.exe
    │
    ├── Scans Go*.exe in its folder
    ├── Merges into %APPDATA%\Code\User\tasks.json
    │       (applies to ALL workspaces automatically)
    └── Creates .gobuild\*.cfg sample configs

VS Code (any workspace)
    │
    └── Ctrl+Shift+B
            │
            ├── GoBuildDLL  →  reads .gobuild\GoBuildDLL.cfg
            ├── GoBuildExe  →  reads .gobuild\GoBuildExe.cfg
            ├── GoCheckALL  →  scans workspace for go.mod files
            ├── GoWatcher   →  watches *.go, debounce 800ms
            ├── GoClean     →  removes artifacts
            ├── GoModSync   →  go mod tidy all projects
            └── GoNewProject→  scaffold new project
```

---

## 📁 Project Config (.cfg)

Each build tool reads a `.cfg` file from the `.gobuild\` subfolder of your project:

```ini
# GoBuildDLL.cfg
ProjectPath = D:\MyGoProjects\MyLib
OutputDir   = D:\MyGoProjects\MyLib\bin
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

Scans all `go.mod` under `${workspaceFolder}` and runs in parallel threads:

```
┌─ [MyProject] File changed — checking...
│  [OK]   go.mod found
│  [OK]   No duplicate functions
│  [OK]   go vet passed
│  [OK]   staticcheck passed
│  [OK]   build amd64 OK
│  [OK]   build 386 OK
└─ [MyProject] OK  (3.8s)
```

Checks performed:
- ✅ `go.mod` validation
- ✅ Package declaration consistency
- ✅ Duplicate function names
- ✅ Duplicate `//export` names
- ✅ `go vet ./...`
- ✅ `staticcheck ./...`
- ✅ Build for `amd64` and `386`

---

## 👀 GoWatcher — Live File Watcher

Uses Win32 `ReadDirectoryChangesW` (not polling) — zero CPU when idle.

```
[INFO] Watching: D:\MyGoProjects\MyLib
[INFO] Edit any .go file to trigger check

┌─ [MyLib] File changed — checking...
│  ...
└─ [MyLib] OK  (2.1s)
```

- Debounce: 800ms (configurable via arg)
- Watches subdirectories recursively
- Press `Ctrl+C` to stop cleanly

---

## 🏗️ Technical Notes

- Built with **Delphi 13.1** — native Win32 executables
- All output via `WriteFile` directly — **no ACP encoding issues**
- VS Code tasks use `"type": "process"` — **no PowerShell wrapper**
- `GoWatcher` uses `isBackground: true` + `problemMatcher: []` — terminal never hangs
- Parallel execution via `TThread` + `TCriticalSection`
- Go build uses `Garble` for obfuscation + `UPX` for compression

---

## 📋 Requirements

| Requirement | Version |
|-------------|---------|
| Windows | 10 / 11 |
| VS Code | Any (stable / Insiders / Cursor) |
| Go | 1.21+ |
| staticcheck | Optional (auto-detected) |
| Garble | Optional (for obfuscated builds) |

---

## 📂 Folder Structure

```
C:\GoToolsVSCode\
    GoSetup.exe
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
```

---

## 📜 License

Binaries only — source code is not distributed.  
Free for personal and commercial use.

---

## 👤 Author

**Kieu Manh**  
📧 kieumanh366377@gmail.com

---

*GoToolsVSCode — Native Go tooling for VS Code on Windows*
