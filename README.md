# AutoCAD MCP｜繁體中文說明版

讓支援 MCP 的 AI 助理透過自然語言操作 AutoCAD，也能在未安裝 AutoCAD 的環境產生 DXF 圖檔。

本儲存庫由 **allen2123231** 建立為個人分支，基於 [puran-water/autocad-mcp](https://github.com/puran-water/autocad-mcp)。本次調整為繁體中文文件整理，核心程式沿用上游；原作者及貢獻者的版權與 [MIT 授權](LICENSE) 完整保留。頁面下方另保留英文原文供對照。

## 可以做什麼？

| 工具 | 中文用途 |
| --- | --- |
| `drawing` | 開啟與儲存圖檔、取得圖面資訊、匯出 DXF、出圖 PDF、復原與重做 |
| `entity` | 建立直線、圓、聚合線、矩形、圓弧、橢圓、多行文字與填充；移動、複製、旋轉等 |
| `layer` | 建立圖層、設定目前圖層、調整屬性、凍結、解凍及鎖定 |
| `block` | 查詢與插入圖塊、讀取及更新圖塊屬性 |
| `annotation` | 建立文字、線性尺寸、對齊尺寸、角度尺寸、半徑尺寸及引線 |
| `pid` | 管線與儀表流程圖的符號、設備標籤、流程線、閥門、泵浦與儲槽 |
| `view` | 縮放至全圖、指定視窗範圍及取得 PNG 截圖 |
| `system` | 檢查連線、查詢後端及執行環境、重新初始化、執行 AutoLISP |

工具以 MCP 的標準輸入／輸出（stdio）提供給 AI 用戶端。實際可用操作依後端而異。

## 選擇執行模式

| 模式 | 適合用途 | 環境 |
| --- | --- | --- |
| `file_ipc` | 操作已開啟的 AutoCAD 圖面 | Windows、AutoCAD 與已載入的 LISP 派送器 |
| `ezdxf` | 離線建立或編輯 DXF | 不需要 AutoCAD，可在 Windows、Linux 或 macOS 使用 |
| `auto` | 自動選擇 | 預設先嘗試 File IPC，失敗時改用 ezdxf |

需要操作眼前的 AutoCAD 圖面時，設定 `file_ipc`，並確認 `system` 回傳的後端確實是 `file_ipc`。`ezdxf` 不代表已連上 AutoCAD，也不直接提供 DWG 編輯。

## 安裝與連線

### 1. 準備環境

- Windows 10／11。
- 上游文件指定 AutoCAD LT 2024 以上 Windows 版；此版本起支援 AutoLISP。其他 AutoCAD 版本請在實際環境驗證。
- Python 3.10 以上，使用 Windows 原生 Python。
- [uv 套件管理工具](https://docs.astral.sh/uv/getting-started/installation/)。

若只使用 ezdxf 離線模式，無須安裝 AutoCAD，也不受限於 Windows。

### 2. 下載你的專案

在 PowerShell 執行：

```powershell
git clone https://github.com/allen2123231/autocad-mcp.git
cd autocad-mcp
uv sync
```

### 3. 在 AutoCAD 載入 LISP

1. 開啟 AutoCAD 與要操作的圖面。
2. 在命令列輸入 `APPLOAD`。
3. 選擇專案內的 `lisp-code/mcp_dispatch.lsp` 並載入。
4. 確認命令列顯示 `MCP Dispatch v3.1 loaded` 及 `Ready for commands via (c:mcp-dispatch)`。

可將檔案加入 APPLOAD 的啟動套件，讓之後開啟圖面時自動載入。

### 4. 設定 MCP 用戶端

以下為使用 `mcpServers` 格式之用戶端的 JSON 範例。請將 `C:\\path\\to` 換成實際安裝位置；不同用戶端的設定檔格式可能不同。

```json
{
  "mcpServers": {
    "autocad-mcp": {
      "command": "C:\\path\\to\\autocad-mcp\\.venv\\Scripts\\python.exe",
      "args": ["-m", "autocad_mcp"],
      "env": {
        "AUTOCAD_MCP_BACKEND": "file_ipc"
      }
    }
  }
}
```

離線產生 DXF 時，將 `AUTOCAD_MCP_BACKEND` 改成 `ezdxf`。用戶端在 WSL 執行時，File IPC 仍需啟動 Windows 端 Python，完整範例見下方英文說明。

### 5. 驗證是否連線成功

由 MCP 用戶端呼叫：

```text
system(operation="status")
drawing(operation="info")
```

先確認後端為 `file_ipc`，再檢查回傳圖面資訊是否符合目前工作。開始修改前，先儲存或備份圖檔。

## 中文使用範例

連線後，可以向 AI 助理提出這類要求：

- 「先確認目前 AutoCAD 的圖面資訊與圖層。」
- 「在測試圖面畫一個 100 × 50 的矩形，放到新圖層 TEST。」
- 「替這條斜邊建立對齊尺寸，並讀回尺寸值。」
- 「列出圖塊名稱與屬性，先不要修改。」
- 「將目前圖面另存成 DXF，並確認輸出檔案。」

請在需求中明確指定圖面、物件、尺寸與儲存路徑；座標數值使用目前圖面單位，需先確認單位設定。

## AutoLISP 呼叫方式

此儲存庫目前的 `system` 工具接收 `data.code`，範例：

```json
{
  "operation": "execute_lisp",
  "data": { "code": "(+ 1 2)" },
  "include_screenshot": false
}
```

這個操作限 File IPC 後端。Python 端會處理暫存程式檔；對本版本的 MCP 工具應傳入 `code`。若使用其他修改版，請依該版本實際工具定義確認參數。

## 設定與常見問題

| 環境變數 | 預設值 | 說明 |
| --- | --- | --- |
| `AUTOCAD_MCP_BACKEND` | `auto` | `auto`、`file_ipc` 或 `ezdxf` |
| `AUTOCAD_MCP_IPC_DIR` | `C:/temp` | 命令與結果 JSON 的交換資料夾 |
| `AUTOCAD_MCP_IPC_TIMEOUT` | `10.0` | 等待秒數，可設定 1 至 300 |
| `AUTOCAD_MCP_ONLY_TEXT` | `false` | 設為 `true` 時只回傳文字、不擷取畫面 |

- **找不到工具：** 先確認用戶端是否啟用 MCP 設定、Python 路徑是否存在及伺服器能否啟動。沒有顯示工具，不等於本機未安裝。
- **回傳 ezdxf：** 可能是 `auto` 模式找不到 AutoCAD；檢查 AutoCAD、LISP 載入狀態與後端設定。
- **操作逾時：** 檢查 AutoCAD 是否停在對話框或命令提示中，再依工作量調整等待秒數。
- **修改 IPC 資料夾：** Python 的環境變數與 LISP 內的 `*mcp-ipc-dir*` 必須一致。
- **功能差異：** `offset`、`fillet`、`chamfer`、PDF 出圖與復原／重做限 File IPC；`block.define` 為 ezdxf 功能。
- **P&ID 符號：** 部分操作需另外安裝 CTO 符號庫或 LISP 輔助程式；一般線條、圖層與尺寸操作不需要該符號庫。
- **清空圖面：** `drawing.create` 會清除目前圖面物件並清理未使用項目，請只在準備重設圖面時使用。

## 專案結構與開發

```text
lisp-code/          AutoCAD 端 LISP 派送器
src/autocad_mcp/    Python MCP 伺服器與後端
tests/             測試程式
pyproject.toml     套件資訊與依賴
LICENSE            原始 MIT 授權
```

開發測試可執行：

```powershell
uv sync
uv run --with pytest --with pytest-asyncio pytest tests/ -v
```

本次為文件中文化，並未宣稱已在所有 AutoCAD 版本完成實機測試。上游 README 稱此功能版本為 v3.1，`pyproject.toml` 的套件版本仍為 3.0.0。

## 來源與授權

感謝 [puran-water/autocad-mcp](https://github.com/puran-water/autocad-mcp) 原作者與所有貢獻者。本專案依 [MIT License](LICENSE) 使用、修改及散布，請保留原始版權與授權聲明。

---

## 英文原始說明

<details>
<summary>展開上游英文 README（保留原文供對照）</summary>


# AutoCAD MCP Server

MCP server for AutoCAD LT automation and headless DXF generation.

Two backends, one API:

| Backend | Runtime | Requires AutoCAD? | Screenshot |
|---------|---------|-------------------|------------|
| **File IPC** | Windows Python | Yes — AutoCAD LT 2024+ (Windows) | Win32 PrintWindow |
| **ezdxf** | Any platform | No (headless) | matplotlib render |

The server exposes **8 consolidated tools** (`drawing`, `entity`, `layer`, `block`, `annotation`, `pid`, `view`, `system`) over the MCP stdio transport. An MCP client (Claude Desktop, Claude Code, etc.) connects and drives AutoCAD through natural-language requests.

## Prerequisites (File IPC backend)

- **Windows 10/11** (the File IPC backend uses Win32 APIs for focus-free window messaging)
- **AutoCAD LT 2024 or newer** — AutoLISP support was added in LT 2024 for Windows. AutoCAD LT for Mac exists but does **not** support AutoLISP.
- **Python 3.10+** (Windows native — not WSL Python)
- **uv** package manager ([install guide](https://docs.astral.sh/uv/getting-started/installation/))

> The ezdxf headless backend works on any platform (Linux, macOS, WSL) for offline DXF generation without AutoCAD installed.

## Quick Start

### 1. Clone and install

```powershell
git clone https://github.com/puran-water/autocad-mcp.git
cd autocad-mcp
uv sync
```

### 2. Load the LISP dispatcher in AutoCAD LT

Open AutoCAD LT and load `mcp_dispatch.lsp` using **APPLOAD**:

1. Type `APPLOAD` in the AutoCAD command line
2. Browse to `<repo>/lisp-code/mcp_dispatch.lsp`
3. Click **Load**
4. You should see: `=== MCP Dispatch v3.1 loaded ===` and `Ready for commands via (c:mcp-dispatch)`

> **Tip:** Add the file to your AutoCAD Startup Suite (in the APPLOAD dialog) so it loads automatically with every drawing.

### 3. Configure your MCP client

Add to your MCP client configuration (e.g. Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "autocad-mcp": {
      "command": "C:\\path\\to\\autocad-mcp\\.venv\\Scripts\\python.exe",
      "args": ["-m", "autocad_mcp"],
      "env": { "AUTOCAD_MCP_BACKEND": "auto" }
    }
  }
}
```

**Key points:**

- The `command` must point to the **Windows Python** inside the project venv (not WSL python).
- `AUTOCAD_MCP_BACKEND` can be `auto` (default — tries File IPC, falls back to ezdxf), `file_ipc` (requires AutoCAD), or `ezdxf` (headless only).

#### Running from WSL

If your MCP client runs in WSL (e.g. Claude Code), launch the server through `cmd.exe` so it runs as a native Windows process:

```json
{
  "mcpServers": {
    "autocad-mcp": {
      "type": "stdio",
      "command": "cmd.exe",
      "args": ["/d", "/s", "/c", "cd /d C:\\path\\to\\autocad-mcp && .venv\\Scripts\\python.exe -m autocad_mcp"],
      "env": { "AUTOCAD_MCP_BACKEND": "auto" }
    }
  }
}
```

### 4. Verify

From your MCP client, call:

```
system(operation="status")
```

You should see `backend: "file_ipc"` if AutoCAD is running, or `backend: "ezdxf"` for headless mode.

## Tools

### `drawing` — File/drawing management

| Operation | Description | File IPC | ezdxf |
|-----------|-------------|----------|-------|
| `create` | Reset to clean drawing (erase all + purge) | Yes | Yes |
| `open` | Open an existing drawing | Yes | Yes (DXF) |
| `info` | Get entity count and layers | Yes | Yes |
| `save` | Save current drawing (to path if given) | Yes | Yes |
| `save_as_dxf` | Export as DXF | Yes | Yes |
| `plot_pdf` | Plot to PDF | Yes | No |
| `purge` | Purge unused objects | Yes | Yes |
| `get_variables` | Get system variables by name | Yes | Yes |
| `undo` | Undo last operation | Yes | No |
| `redo` | Redo last undone operation | Yes | No |

### `entity` — Entity CRUD + modification

**Create:** `create_line`, `create_circle`, `create_polyline`, `create_rectangle`, `create_arc`, `create_ellipse`, `create_mtext`, `create_hatch`

**Read:** `list`, `count`, `get`

**Modify:** `copy`, `move`, `rotate`, `scale`, `mirror`, `offset`\*, `array`, `fillet`\*, `chamfer`\*, `erase`

> \* `offset`, `fillet`, `chamfer` are File IPC only (not supported in ezdxf headless backend).

### `layer` — Layer management

`list`, `create`, `set_current`, `set_properties`, `freeze`, `thaw`, `lock`, `unlock`

### `block` — Block operations

| Operation | File IPC | ezdxf |
|-----------|----------|-------|
| `list` | Yes | Yes |
| `insert` | Yes | Yes |
| `insert_with_attributes` | Yes | Yes |
| `get_attributes` | Yes | Yes |
| `update_attribute` | Yes | Yes |
| `define` | No | Yes |

### `annotation` — Text, dimensions, leaders

`create_text`, `create_dimension_linear`, `create_dimension_aligned`, `create_dimension_angular`, `create_dimension_radius`, `create_leader`

### `pid` — P&ID operations (CTO symbol library)

`setup_layers`, `insert_symbol`, `list_symbols`, `draw_process_line`, `connect_equipment`, `add_flow_arrow`, `add_equipment_tag`, `add_line_number`, `insert_valve`, `insert_instrument`, `insert_pump`, `insert_tank`

> P&ID symbol insertion requires the [CAD Tools Online](https://www.cadtoolsonline.com/) (CTO) P&ID Symbol Library installed at `C:\PIDv4-CTO\`. The ezdxf backend has built-in CTO library support. For the File IPC backend, some P&ID operations require additional LISP helpers — see the P&ID section in the wiki for setup details.

### `view` — Viewport and screenshot

| Operation | Description |
|-----------|-------------|
| `zoom_extents` | Zoom to show all entities |
| `zoom_window` | Zoom to a specified window |
| `get_screenshot` | Capture current AutoCAD view as PNG |

Screenshots use `PrintWindow` (Win32) for the File IPC backend — works even when AutoCAD is minimized or in the background. The ezdxf backend renders via matplotlib.

### `system` — Server management

`status`, `health`, `get_backend`, `runtime`, `init`, `execute_lisp`

> `execute_lisp` runs arbitrary AutoLISP code (File IPC only). Pass `data: {code: "(+ 1 2)"}`. This turns the server into an extensible automation platform — any valid AutoLISP expression can be executed.

## Architecture

```
MCP Client (Claude)
    │  stdio (JSON-RPC)
    ▼
Python MCP Server (autocad_mcp)
    │
    ├── File IPC Backend ──► C:/temp/*.json ──► mcp_dispatch.lsp (AutoCAD LT)
    │   PostMessageW(WM_CHAR) to MDIClient — no focus steal
    │
    └── ezdxf Backend ──► in-memory DXF (headless, no AutoCAD needed)
```

The File IPC backend sends keystrokes to AutoCAD's MDIClient window via `PostMessageW(WM_CHAR)`, triggering the `(c:mcp-dispatch)` AutoLISP command. This approach does **not** steal window focus — you can continue working in other applications while automation runs.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `AUTOCAD_MCP_BACKEND` | `auto` | Backend selection: `auto`, `file_ipc`, `ezdxf` |
| `AUTOCAD_MCP_IPC_DIR` | `C:/temp` | Directory for IPC command/result JSON files (must match on both Python and LISP sides) |
| `AUTOCAD_MCP_IPC_TIMEOUT` | `10.0` | IPC command timeout in seconds (1-300) |
| `AUTOCAD_MCP_ONLY_TEXT` | `false` | Disable screenshot capture (text feedback only) |

> **Note:** If you change `AUTOCAD_MCP_IPC_DIR`, you must also update the `*mcp-ipc-dir*` variable in `mcp_dispatch.lsp` to match.

## Development

```powershell
uv sync
uv run pytest tests/ -v
```

## AutoCAD LT AutoLISP Compatibility

AutoLISP was added to AutoCAD LT in the **2024 release (Windows only)**. AutoCAD LT for Mac does not support AutoLISP.

| Supported (LT 2024+ Windows) | Not Supported |
|-------------------------------|---------------|
| `.lsp` / `.fas` / `.vlx` / `.dcl` | VLIDE (Visual LISP IDE) |
| All `vl-*` utility functions | `vlax-*` (ActiveX/COM) |
| File I/O (`open`, `read-line`, etc.) | Express Tools |
| Entity access (`entget`, `entmod`, etc.) | 3D operations |
| Selection sets | AutoLISP on Mac |

The `mcp_dispatch.lsp` dispatcher is fully compatible with LT 2024+.

## What's New in v3.1

- **`execute_lisp`** — Run arbitrary AutoLISP code via temp file pattern. Turns the server from a fixed command set into an extensible automation platform.
- **Undo / Redo** — Single-step undo and redo via `drawing` tool.
- **Drawing open** — Open existing `.dwg` files programmatically (FILEDIA suppressed).
- **Drawing create** — Now resets current drawing (erase all + purge) instead of `_.NEW`, preserving the LISP dispatcher namespace.
- **Drawing save with path** — `save` with a `path` parameter uses SAVEAS; without path uses QSAVE.
- **`get_variables` fix** — Respects the `names` parameter; returns requested variables with proper type handling.
- **Polyline/leader fix** — Point arrays properly encoded via semicolon-delimited format.
- **ESC prefix** — Sends 2x ESC before each dispatch to cancel stale pending commands from prior timeouts.
- **UTF-8/cp1252 fallback** — Handles non-ASCII characters in LISP result files (AutoCAD writes Windows-1252).
- **Configurable IPC timeout** — `AUTOCAD_MCP_IPC_TIMEOUT` env var (1–300 seconds, default 10).
- **Thread-safe backend init** — `asyncio.Lock` prevents parallel initialization races.

## License

MIT

</details>
