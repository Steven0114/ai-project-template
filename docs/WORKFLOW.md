# 多模型協作流程

本檔補充 `AGENTS.md`：誰負責什麼、任務如何在模型之間交接。規則衝突時以 `AGENTS.md` 為準。

## 1. 角色

| 角色 | 負責 | 不做 |
|---|---|---|
| **使用者** | GOALS.md 確認；Gate A（規格）、Gate B（合併）；批准一切交易、花錢、對外寄送、隱私事項；手動登入 | — |
| **Claude（Claude Code）** | 深化四問、grill、spec、拆 tickets；核心架構與關鍵模組；整合審查；在 ORCA 擔任 coordinator 派工 | 不自行合併 `main`、不連實盤帳戶 |
| **Codex** | 依 `ready-for-agent` 的 Issue 實作與寫測試（`/implement`、`/tdd`）；修正審查意見 | 不改 spec 和驗收標準 |
| **Grok** | 獨立審查（`/code-review` 的 Standards 與 Spec 兩個面向），結論寫成 PR 留言 | 不直接改程式（審查者不當作者） |
| **Antigravity（Gemini）** | `/research` 一手資料研究；審查意見衝突時當第二審查（裁決）；之後做 UI 與儀表板 | 不碰核心模組與交易模組 |
| **ORCA** | 調度：開 worktree、派工給各模型、收回報；使用者關卡設為 ORCA decision gate；排程 | 不取代任何模型 |

所有模型都裝了 mattpocock-skills（Claude Code 和 Codex 用 plugin 自動更新，Gemini 用 `gemini skills install`），所以技能名稱在各模型之間通用。

## 2. 一個功能從頭到尾

```
使用者提需求
 → Claude：深化四問 → 確認或更新 GOALS.md
 → Claude：/grill-with-docs（或 /wayfinder）→ /to-spec → /to-tickets
 → [Gate A] 使用者確認 spec，在 Issue 加上 ready-for-agent
 → ORCA 開 worktree，派 Codex：/implement <issue>，push 分支，開 PR
 → ORCA 派 Grok：/code-review，結論留言在 PR（BLOCKER / MAJOR / MINOR）
     Codex 不同意某條 → 派 Antigravity 裁決
     有 BLOCKER/MAJOR → 退回 Codex 修（最多 2 輪；第 3 輪交給 Claude 判斷）
 → Claude：整合審查（全部測試、外部驗證、是否違反 AGENTS.md）
 → [Gate B] 使用者看 PR 摘要後批准合併
 → 里程碑完成時：/retro
```

**快速通道**：文件、設定、少於 50 行的小變更，可以 Codex 實作 → Claude 審查 → Gate B，略過 Grok。

**拆單原則**：tickets 用「曳光彈」方式切，每張都是端到端可以跑起來的薄切片，並標明 blocking 關係，讓 ORCA 可以平行派工給沒有互相依賴的 tickets。

## 3. 交接

- **狀態**：放在 Issue 標籤（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）和 PR 狀態，不另外維護狀態檔。
- **需要使用者時**：Issue 加上 `ready-for-human` 標籤，並留言說明要批准什麼、為什麼；同時設 ORCA decision gate。
- **換 session 或換模型**：用 `/handoff` 產生交接文件。
- **決定**：長期的架構決定寫 `docs/adr/`；專案用語寫 `GLOSSARY.md`（`/grill-with-docs` 會自動維護）。

## 4. 電腦操作

- Claude Code 在本機有 shell、檔案、git、`gh` 和瀏覽器工具；需要操作桌面程式時，在 Claude 桌面 app 開啟 computer use，每個 app 第一次使用都要使用者核准。
- 能用 CLI 或 API 完成的事不用 GUI。
- `orca computer` 等桌面操作工具不經過 Claude app 的逐 app 核准，**禁止用於券商交易軟體、網路銀行、密碼管理器**。
