# AutoCAD MCP

讓支援 MCP 的 AI 助理透過自然語言操作 AutoCAD，也能在未安裝 AutoCAD 的環境產生 DXF 圖檔。

本儲存庫由 **allen2123231** 建立為個人分支，基於 [puran-water/autocad-mcp](https://github.com/puran-water/autocad-mcp)。本次調整為繁體中文文件整理，核心程式沿用上游；原作者及貢獻者的版權與 [MIT 授權](LICENSE) 完整保留。

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

離線產生 DXF 時，將 `AUTOCAD_MCP_BACKEND` 改成 `ezdxf`。用戶端在 WSL 執行時，File IPC 仍需啟動 Windows 端 Python，並使用 Windows 檔案路徑設定伺服器。

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

## 後續開發方向

以下為規劃中的開發項目，尚未代表已完成功能；實作順序可依使用需求調整。目前優先改善連線可靠性，再擴充加工繪圖與批次出圖流程。

### 第一階段：連線穩定與操作確認

- [ ] **連線診斷工具：** 一次檢查 Python、MCP 啟動、AutoCAD 視窗、LISP 載入狀態與 IPC 資料夾，提供中文排除步驟。
- [ ] **多圖面與多執行個體選擇：** 列出已開啟的 AutoCAD 視窗和圖檔，讓操作明確綁定指定文件。
- [ ] **命令佇列與逾時處理：** 依序派送修改命令，逾時後查明執行結果，避免重複建立物件。
- [ ] **備份與結果核對：** 修改前建立備份，完成後讀回圖元、尺寸及儲存狀態，留下可追查的操作紀錄。

驗收重點：在切換圖面、同時開啟多個視窗及操作逾時時，仍能確認目標與結果，且不重複執行修改。

### 第二階段：中文使用與安裝體驗

- [ ] **中文工具說明與錯誤訊息：** 保留既有 API 名稱，補齊繁體中文參數解釋、單位及範例。
- [ ] **Windows 安裝與設定精靈：** 協助建立虛擬環境、檢查路徑及產生 MCP 用戶端設定。
- [ ] **範例圖面與教學：** 提供從連線檢查、基本繪圖、尺寸標註到另存輸出的練習檔與操作流程。
- [ ] **版本相容性紀錄：** 逐一記錄 AutoCAD／AutoCAD LT 版本、Windows 環境及已驗證的功能範圍。

驗收重點：使用者能依中文步驟完成安裝，並在測試圖面完成繪圖、標註、儲存與讀回確認。

### 第三階段：加工繪圖與批次出圖

- [ ] **尺寸標註規則：** 支援指定尺寸樣式、精度、文字高度與標註間距，檢查尺寸重疊及漏標。
- [ ] **鋁板與鈑金加工圖流程：** 針對已有平面輪廓整理板件編號、孔位、折線及加工註記；涉及展開尺寸時，明確輸入板厚、折彎半徑與扣料規則。
- [ ] **圖框與圖塊屬性批次更新：** 依指定範圍套用圖框、板號、版本及圖名，保留範圍外物件。
- [ ] **批次 DWG／DXF／PDF 輸出：** 支援命名規則、出圖範圍、比例及資料夾設定，產出成功與失敗清單。
- [ ] **表格資料對接：** 匯入 CSV／Excel 的板件尺寸與編號，先檢查欄位、單位和幾何條件，再建立或更新圖面。

驗收重點：先完成單一板件樣本並確認尺寸，再執行批次；輸出後核對板號、數量、比例及檔案是否可重新開啟。

### 第四階段：測試與持續維護

- [ ] **自動化測試：** 建立工具參數、錯誤處理、ezdxf 輸出與 IPC 通訊的回歸測試。
- [ ] **AutoCAD 實機驗證：** 在指定版本測試圖面讀寫、尺寸、截圖及批次流程，將實機結果與離線測試分開記錄。
- [ ] **版本發布與變更紀錄：** 統一套件版本、文件版本與發布標籤，清楚列出新增功能、修正及已知限制。
- [ ] **追蹤上游更新：** 比較上游修正，確認相容性後合併，持續維護本分支的中文文件與擴充功能。

驗收重點：每次發布都有對應測試結果、變更紀錄與可重現的安裝方式。

## 來源與授權

感謝 [puran-water/autocad-mcp](https://github.com/puran-water/autocad-mcp) 原作者與所有貢獻者。本專案依 [MIT License](LICENSE) 使用、修改及散布，請保留原始版權與授權聲明。
