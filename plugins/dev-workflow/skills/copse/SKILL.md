---
name: copse
description: 使用 Copse 管理 Git worktree；建立、列出、切換、刪除或批次清理 worktree，以及在同一儲存庫的 worktree 間複製未提交變更時使用。此技能操作 worktree，不負責開發 Copse 本身。
---

# Copse

使用 [Copse](https://github.com/usira-okay/Copse) 及其非互動子命令管理 Git worktree。依使用者要求操作，完成後回報實際路徑、影響範圍及尚未完成的部分；不要額外 commit 或 push。

## 確認工具與能力

優先尋找 `copse`，也相容執行檔 `git_worktree_copse`。先讀取 `--version`、`--help`，再讀取本次所需子命令的 `--help`，以實際輸出確認命令、旗標與語意；不能只憑檔名或版本號推定功能存在。正常自動化使用非互動子命令。

若找不到工具或缺少所需功能，明確指出缺少項目並提供 Cargo 安裝方式。只有取得使用者明確同意後才執行安裝；更新現有版本時，依工具與 Cargo 回報說明升級方式及所需參數，並先取得同意：

```sh
cargo install git_worktree_copse
```

若沒有 Cargo/Rust 工具鏈，說明需要的條件；不要自行安裝 Rust，也不要自行安裝或升級 Copse。

## 確認目標與命令範例

先辨認目標 repository 與 worktree，再以 `-C '<REPOSITORY_OR_WORKTREE>'` 明確設定 Copse 的有效工作目錄。一般操作先用 `list --json` 確認現況；JSON 中的 `path` 是 worktree 絕對路徑，`branch` 可能為 `null`。跨 repository 的批次刪除則以使用者指定的 `--root` 做預覽。以下尖括號內容是要替換的佔位值；shell 命令中的路徑、分支與 reference 須以安全的參數傳遞方式引用，不要把不可信字串串成 shell 程式碼。

以下各行是命令範例，僅執行本次任務需要的步驟，不要整段執行。範例以 `copse` 表示執行檔；實際使用前替換為前一步確認可用的名稱（`copse` 或 `git_worktree_copse`）。

```sh
copse -C '<REPOSITORY>' list --json
copse -C '<REPOSITORY>' create '<REFERENCE>'
copse -C '<REPOSITORY>' create '<REFERENCE>' --new-branch '<NEW_BRANCH>'
copse -C '<REPOSITORY>' move-to --path '<WORKTREE_PATH>'
copse -C '<REPOSITORY>' move-to --branch '<BRANCH>'
copse -C '<REPOSITORY>' delete --path '<WORKTREE_PATH>' --yes
copse -C '<REPOSITORY>' delete --branch '<BRANCH>' --yes
copse -C '<START_DIRECTORY>' batch-delete --root '<USER_SPECIFIED_ROOT>' --path '<WORKTREE_PATH>' --dry-run
copse -C '<START_DIRECTORY>' batch-delete --root '<USER_SPECIFIED_ROOT>' --path '<WORKTREE_PATH>' --yes
copse -C '<DESTINATION_WORKTREE>' copy-changes --from-path '<SOURCE_WORKTREE>'
copse -C '<DESTINATION_WORKTREE>' copy-changes --from-branch '<SOURCE_BRANCH>'
```

`create` 的 reference 必須是工具支援的既有分支、標籤、`HEAD` 或 commit；以 `--new-branch` 明確要求建立新分支。`move-to` 的 `--path` 與 `--branch` 互斥。`delete` 也只能選一種 selector，可重複指定同類 `--path` 或 `--branch` 以刪除多個目標。

## 刪除範圍與確認

`delete --yes` 是 CLI 必要參數，只表示命令列同意執行；它不代表使用者已授權。沿用對話中已明確給出的授權，只有目標或破壞性範圍不清楚時才詢問。批次刪除一律先以 `--dry-run` 預覽，檢查完整候選清單、排除項目及刪除方式，再依既有授權執行。預覽與執行必須使用相同的 `--root`、selector（`--path` 或 `--all`）及 `--force`／`--delete-branch` 選項，只將 `--dry-run` 換成 `--yes`。若預覽出現授權範圍以外的新目標，先停止並釐清；使用 `--all` 時尤其要確認全部候選都在授權範圍內，不得額外納入未核准目標。

`--force` 可能丟棄未提交變更；`--delete-branch` 會移除本地分支；批次執行的 `--all` 會涵蓋 root 下所有符合條件的候選。這些旗標的實際影響須符合使用者明確授權。若髒 worktree 或工具保護檢查使操作失敗，不要自行加 `--force` 或其他旗標重試；回報阻擋原因與目前結果。

## 切換目錄標記

`create` 與 `move-to` 可能輸出 `COPSE_CD:<path>`。子程序無法更改父 shell 的目前目錄；只精確解析標記行，並將路徑設為後續工具的 workdir。不要把 stdout 當 shell 程式碼執行。不要自行修改 shell rc；只有使用者要求持久 shell 整合時，才依官方專案說明處理。

## 複製未提交變更

以 `-C` 指定目的 worktree，以 `--from-path` 或 `--from-branch` 指定同一 repository 中不同的來源 worktree。預設複製來源相對於 `HEAD` 的 tracked 最終變更；只有任務範圍包含未追蹤檔案時才加 `--include-untracked`。`--3way` 會嘗試三方合併，可能留下衝突或 index 變更；只有使用者要求或明確授權時才使用。這不會保留來源 staged/unstaged 區分，也不等同 Git commit。衝突或部分失敗時，檢視並回報來源、目的與目前狀態；不要自動 reset、清除、反向套用，或自行加 `--3way` 重試。
