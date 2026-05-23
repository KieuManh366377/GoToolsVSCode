# 🛠️ GoToolsVSCode

> **Bộ công cụ Go thuần Windows cho VS Code — viết bằng Delphi**
>
> Cài đặt một lần. Tự động áp dụng cho mọi workspace Go.

---

## ✨ Giới Thiệu

**GoToolsVSCode** là bộ 8 file `.exe` độc lập tích hợp sâu vào `tasks.json` User-level của VS Code.  
Khác với shell script hay Makefile, đây là file thực thi Win32 thuần — không qua PowerShell, không lỗi encoding, không phụ thuộc thêm gì.

Chạy `GoConfig.exe` một lần → `Ctrl+Shift+B` hiện danh sách task picker trong mọi project Go bạn mở.

---

## 📦 Danh Sách Tools

| Tool | Phiên bản | Chức năng |
|------|-----------|-----------|
| ⚙️ `GoConfig.exe` | v1.3 | Cài đặt một lần — quét tools, merge vào VS Code User tasks |
| 📦 `GoBuildDLL.exe` | v1.1 | Build Go DLL cho amd64 + x86 với Garble obfuscation |
| 🚀 `GoBuildExe.exe` | v1.1 | Build Go EXE cho amd64 + x86 với Garble obfuscation |
| 🔍 `GoCheckALL.exe` | v1.2 | Kiểm tra song song: go vet + staticcheck + build |
| 👀 `GoWatcher.exe` | v1.2 | Theo dõi file — tự check + tự build khi lưu `*.go` |
| 🧹 `GoClean.exe` | v1.3 | Xóa build artifacts + `go clean -cache` |
| 📋 `GoModSync.exe` | v1.2 | Chạy `go mod tidy` song song cho tất cả project |
| 🆕 `GoNewProject.exe` | v1.3 | Tạo project Go mới từ template, tự mở VS Code |

---

## 🚀 Hướng Dẫn Sử Dụng

### Bước 1 — Đặt tools vào một thư mục

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

### Bước 2 — Chạy GoConfig.exe (chỉ một lần)

```
C:\GoToolsVSCode\GoConfig.exe
```

GoConfig tự động:
- ✅ Phát hiện tất cả `Go*.exe` trong thư mục
- ✅ Merge tasks vào `%APPDATA%\Code\User\tasks.json`
- ✅ Tạo file `.gobuild\*.cfg` mẫu cho từng build tool
- ✅ Tạo file hướng dẫn `HUONG_DAN.md`

> Hỗ trợ **VS Code stable**, **VS Code Insiders** và **Cursor**

### Bước 3 — Cấu hình project Go của bạn

Copy file `.cfg` mẫu vào project:

```
<project-của-bạn>\.gobuild\GoBuildDLL.cfg
```

Chỉnh `ProjectPath` trong file `.cfg` trỏ đến thư mục Go project.

### Bước 4 — Dùng trong VS Code

M�� bất kỳ project Go nào trong VS Code:

```
Ctrl + Shift + B  →  danh sách task picker hiện ra  →  chọn tool
```

---

## ⚙️ Cách Hoạt Động

```
GoConfig.exe
    │
    ├── Quét Go*.exe trong thư mục
    ├── Merge vào %APPDATA%\Code\User\tasks.json
    │       (áp dụng cho MỌI workspace tự động)
    └── Tạo .gobuild\*.cfg mẫu

VS Code (bất kỳ workspace nào)
    │
    └── Ctrl+Shift+B
            │
            ├── GoBuildDLL   →  đọc .gobuild\GoBuildDLL.cfg
            ├── GoBuildExe   →  đọc .gobuild\GoBuildExe.cfg
            ├── GoCheckALL   →  quét workspace tìm go.mod
            ├── GoWatcher    →  theo dõi *.go, debounce 800ms
            ├── GoClean      →  xóa artifacts
            ├── GoModSync    →  go mod tidy tất cả project
            └── GoNewProject →  tạo project mới
```

---

## 📁 File Cấu Hình (.cfg)

M��i build tool đọc file `.cfg` từ thư mục `.gobuild\` trong project.  
`GoNewProject.exe` tạo file này tự động với `ProjectPath` đã điền sẵn.

```ini
# GoBuildDLL.cfg
ProjectPath = D:\GoProjects\MyLib
OutputDir   = D:\GoProjects\MyLib\release
DLLName     = MyLib
```

```ini
# GoBuildExe.cfg
ProjectPath = D:\GoProjects\MyApp
OutputDir   = D:\GoProjects\MyApp\bin
ExeName     = MyApp
UseGarble   = true
LDFlags     = -s -w
```

---

## 🔍 GoCheckALL — Kiểm Tra Song Song

Quét tất cả `go.mod` trong `${workspaceFolder}` và chạy song song nhiều thread:

```
[16:17:46] [INFO] === GO TOOLKIT CHECKER ===
[16:17:46] [OK]   go.mod tìm thấy
[16:17:46] [OK]   Không có hàm trùng tên
[16:17:46] [OK]   go vet passed
[16:17:47] [OK]   staticcheck passed
[16:17:48] [OK]   build amd64 OK
[16:17:49] [OK]   build 386 OK

=====================================
[OK]   MyProject  (2.8s)
KET QUA: TAT CA OK
=====================================
```

Các bước kiểm tra:
- ✅ Kiểm tra `go.mod`
- ✅ Khai báo package nhất quán
- ✅ Hàm trùng tên
- ✅ `//export` trùng tên
- ✅ `go vet ./...`
- ✅ `staticcheck ./...`
- ✅ Build cho `amd64` và `386`
- ✅ FAIL → tự mở VS Code tại đúng dòng lỗi (`code --goto file.go:line`)
- ✅ Ghi log vào `%TEMP%\GoCheckALL_<project>.log`

---

## 👀 GoWatcher — Theo Dõi File + Tự Động Build

Dùng Win32 `ReadDirectoryChangesW` (không polling) — CPU gần bằng 0 khi nhàn rỗi.

```
[14:51:50] [INFO] Watching : D:\GoProjects\MyLib
[14:51:50] [INFO] Log FAIL : C:\Users\...\GoWatcher_MyLib.log

[14:52:39] ┌─ [MyLib] File thay doi — bat dau kiem tra...
[14:52:42] └─ [MyLib] CHECK OK  (3.1s) → dang build...

[14:52:42] │  [INFO] Auto-build: GoBuildDLL.exe ...
[14:53:02] │  [OK]   Build hoan thanh: GoBuildDLL.exe
[14:53:02] └─ [MyLib] BUILD xong  (20.1s)
```

- Debounce: 800ms (có thể chỉnh qua tham số)
- FAIL → tự mở VS Code tại đúng dòng lỗi
- OK → tự động build DLL/EXE qua `GoBuildDLL.exe` hoặc `GoBuildExe.exe`
- Ghi log vào `%TEMP%\GoWatcher_<project>.log`
- Nhấn `Ctrl+C` để dừng sạch

---

## 🆕 GoNewProject — Tạo Project Từ Template

Tạo cấu trúc Go project hoàn chỉnh từ template, sau đó tự mở Explorer và VS Code.

**3 loại project:**

| Loại | Mô tả |
|------|-------|
| `console` | Console Application |
| `dll` | DLL Library (CGo, buildmode=c-shared) |
| `webapi` | Web API (net/http, port 8080) |

**Files tự động tạo:**

```
MyLib\
    main.go              ← template DLL với các hàm mẫu
    go.mod
    .gitignore
    README.md
    versioninfo.json     ← chỉ DLL
    .gobuild\
        GoBuildDLL.cfg   ← ProjectPath đã điền sẵn, dùng được ngay
```

Sau khi tạo → Explorer mở + VS Code mở → `Ctrl+Shift+B` là build được ngay.

---

## 🏗️ Kỹ Thuật

- Viết bằng **Delphi 13.1** — file exe Win32 thuần, không cần runtime
- Tất cả output dùng `WriteFile` trực tiếp — **không lỗi encoding**
- VS Code tasks dùng `"type": "process"` — **không qua PowerShell**
- `GoWatcher` dùng `isBackground: true` + `problemMatcher: []` — terminal không bị treo
- Build song song qua `TThread` + `TCriticalSection` + `MemoryBarrier`
- Go build dùng `Garble -tiny -seed=random` để obfuscate
- Timestamp `[HH:MM:SS]` trên mọi dòng output để debug chính xác

---

## 📋 Yêu Cầu

| Thành phần | Ghi chú |
|------------|---------|
| Windows 10 / 11 | x64 |
| VS Code | stable / Insiders / Cursor |
| Go 1.21+ | [go.dev/dl](https://go.dev/dl/) |
| GCC (MSYS2) | Chỉ cần khi build DLL |
| staticcheck | Tùy chọn — tự phát hiện |
| Garble | Tùy chọn — tự phát hiện, fallback về `go build` |
| goversioninfo | Tùy chọn — nhúng version vào DLL |

---

## 📂 Cấu Trúc Thư Mục

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
        GoBuildDLL.cfg   ← mẫu, copy vào project của bạn
        GoBuildExe.cfg   ← mẫu, copy vào project của bạn
    HUONG_DAN.md
    GoConfig.marker      ← tạo sau lần setup đầu tiên
```

---

## 💡 Tại Sao Dùng GoToolsVSCode?

| | Shell Script / Makefile | Go Extension | GoToolsVSCode |
|--|------------------------|--------------|---------------|
| Encoding UTF-8 | ❌ Hay lỗi | ⚠️ Thỉnh thoảng | ✅ WriteFile thẳng |
| PowerShell | ❌ Phụ thuộc | ✅ Không | ✅ Không |
| Multi-project song song | ❌ Phức tạp | ❌ Không | ✅ TThread |
| File watcher CPU | ❌ Polling | ✅ Thấp | ✅ Win32 event ~0% |
| Cấu hình User-level | ❌ Từng project | ❌ Từng project | ✅ 1 lần cho tất cả |
| Tự build khi lưu | ❌ Không | ❌ Không | ✅ Có |
| Mở VS Code khi lỗi | ❌ Không | ✅ Có | ✅ Có |
| Deploy máy mới | ❌ Cài lại | ✅ Extension | ✅ Copy folder + GoConfig |

---

## 📜 Giấy Phép

Chỉ phân phối file thực thi — không chia sẻ mã nguồn.  
Miễn phí cho cá nhân và thương mại.  
MIT License — Copyright 2026 Kiều Mạnh

---

## 👤 Tác Giả

**Kiều Mạnh**  
📧 kieumanh366377@gmail.com  
🔗 [github.com/KieuManh366377](https://github.com/KieuManh366377)

---

*GoToolsVSCode — Công cụ Go thuần Windows cho VS Code*
