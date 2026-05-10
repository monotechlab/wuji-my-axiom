# huggingface/ml-intern 深度研究

> 日期：2026-04-25 ｜ source: https://github.com/huggingface/ml-intern ｜ depth=1 clone @ HEAD `2a2e170 feat(observability)`
> 任務描述提及 5.9k★ / 373 commits — GitHub API 即時數據為 `stargazers_count: 5947`、`forks_count: 529`，commit 總數無法以 shallow clone 驗證 [未驗證]。

## 1. License + 採用門檻

| 項目 | 結果 | 出處 |
|---|---|---|
| LICENSE 檔 | **不存在** | `find . -iname LICENSE*` 空；GitHub API `"license": null` |
| 預設模型 | `bedrock/us.anthropic.claude-opus-4-6-v1` | `configs/main_agent_config.json:2` |
| 必要 env | `HF_TOKEN`（必）+ `ANTHROPIC_API_KEY` 或 bedrock IAM | `README.md:29-32` |
| Python | `>=3.11` | `pyproject.toml:6` |
| 重依賴 | `litellm`、`fastmcp`、`fastapi`、`huggingface-hub`、`apscheduler`、`boto3` | `pyproject.toml:8-30` |
| GPU | Agent 端**不需要**，計算外包給 HF Jobs/Space | `agent/tools/jobs_tool.py`、`sandbox_client.py` |

**結論**：無 LICENSE = 預設「保留所有權利」，不能合法 fork/再發佈，只能 clone 自用；想借鑑必須抄寫重實作而非複製貼上。RPi5 跑得動 orchestrator（純 Python，無 CUDA），但會把 GPU 帳單推到 HF Jobs。

## 2. Agent loop 架構

- **iteration cap = 300**：`config.py:35` `max_iterations: int = 300  # -1 = unlimited`，主迴圈 `agent_loop.py:564 while max_iterations == -1 or iteration < max_iterations`。
- **ToolRouter**：`agent/core/tools.py:126`，註解 `Based on codex-rs/core/src/tools/router.rs`（OpenAI codex-cli 血統）。`register_tool` / `register_mcp_tools` / `register_openapi_tool` 三軌共存，MCP server 在 enter 時 lazy-load。
- **Context compaction**：`context_manager/manager.py:335 _COMPACT_THRESHOLD_RATIO = 0.9`，`needs_compaction` 在 `running_context_usage > 0.9 * model_max_tokens`（預設 180k） 時觸發；`compact_size = 0.1 * max`（預留 18k）。`_COMPACT_PROMPT`（manager.py:73）寫法 = 「未經歷者也能接手」，而 `_RESTORE_PROMPT`（manager.py:85）= 第一人稱續寫，列工具呼叫+檔案+下一步。
- **Approval gate**（`agent_loop.py:51 _needs_approval`）觸發來源：`sandbox_create` 必過、`hf_jobs` GPU job 必過 / CPU job 看 `confirm_cpu_jobs`、`hf_private_repos` 的 `upload_file`+`create_repo`、`hf_repo_files` 的 `upload`+`delete`、`hf_repo_git` 的 `delete_branch/delete_tag/merge_pr/create_repo/update_repo`。`yolo_mode=True` 全跳過（`config.py:34`）。
- **Doom Loop Detector**（`doom_loop.py:103`，僅 135 行純函式）：lookback=30 訊息，連續 3 次相同 tool+args hash 或 2-5 長度的重複序列出現 ≥2 次就注入「STOP repeating, change strategy」prompt。

## 3. HF Hub 整合深度

- **Session 上傳**：`agent/core/session_uploader.py` 為**獨立子進程**，預設 repo `smolagents/ml-intern-sessions`（`config.py:27`），路徑 `sessions/YYYY-MM-DD/{session_id}.jsonl`（line 116），**先過 `redact.scrub` 洗 token**（line 84-86）再上傳；失敗本地存 `upload_status: failed`。
- **HeartbeatSaver**：`heartbeat_interval_s=60`（config.py:33），長 turn 中途存檔避免崩潰丟資料。
- **HF Jobs**：`agent/tools/jobs_tool.py` 直接派遠端訓練 job；CPU/GPU flavor 走不同 approval 分支。
- **Sandbox = HF Space**：`sandbox_client.py:1` 註解「duplicate template Space `burtenshaw/sandbox`」→ 等上線 → HTTPS 操作 bash/read/write/edit。**不是 Docker，不是 e2b**，是 HF Space dev mode。
- 沒有專屬 `trainer.py` 或 `model_card` 工具 — model card 由 LLM 自行產出後透過 `hf_repo_files.upload` 推上去 [未驗證但合理推論]。

## 4. vs 既有 agent

| 維度 | ml-intern | Claude Code | OpenHands/Devin | smolagents (HF) |
|---|---|---|---|---|
| 定位 | 垂直 ML engineer | 通用 dev | 通用 dev | 通用 agent 框架 |
| Sandbox | HF Space 雲端 | 本機 | Docker | 看實作 |
| Compute | HF Jobs | 本機 | 本機/雲 | 本機 |
| Approval | 寫死 + yolo flag | 互動式 + 規則 | per-action | 無強制 |
| LLM 層 | litellm 多家 | Anthropic only | LiteLLM | LiteLLM |

ml-intern 的差異化在「假設你已有 HF 帳號 + 想跑 ML 工作流」，把 Hub/Datasets/Spaces/Jobs 當原生 OS。

## 5. 對共腦/太極的可借鏡（附證）

1. **`_needs_approval` 清單 → 太極「不可逆的不賭」操作字典**：`agent_loop.py:51-118` 列出的 destructive op taxonomy（delete_branch / merge_pr / upload-overwrite / GPU job）就是「不可逆」的工程定義，可直接寫入 `feedback_no_reflex_backup.md` 同層的「強制 approval 清單」。
2. **`doom_loop.py`（135 行純函式）→ 共腦 worker 監督器**：偵測 worker tmux 內 LLM 卡迴圈（同樣參數重複呼叫），可作為 `scripts/start-worker.sh` 的衛兵，無外部依賴可整段移植。
3. **`_RESTORE_PROMPT`（manager.py:85）→ memory 壓縮策略**：第一人稱續寫格式比目前 lesson 摘要更完整，適合套到 `archive/*/DIGEST.md` 產生流程。
4. **`heartbeat_interval_s=60` → PHASCHNG checkpoint**：長任務每 60 秒落地，比目前共腦只在 worker_done 才存更穩。
5. **300 iteration cap**：對應太極「不生即死」— 跑滿就強制 vendor handoff，可作 worker 上限預設值。

## 6. 風險

- **無 LICENSE**：法律上不能 fork / 不能整段抄；只能借鑑邏輯重寫。
- **`yolo_mode=True`**（`config.py:34`）= 一鍵繞過 approval，部署時必須鎖死。
- **Session 預設上傳公開 dataset** `smolagents/ml-intern-sessions`，雖有 `redact.scrub` 但 regex-based 是 best-effort（`session_uploader.py:68-86`），可能漏掉自訂 secret 格式。
- **HF Jobs/Space 帳單失控**：approval 機制在，但 yolo+long iteration 組合會燒錢。
- **強耦合 HF 生態**：移植到非 HF 環境要拆 sandbox/jobs/uploader 三大塊。

## 7. Owner 三條具體下一步

1. **借鑑（高 ROI 低風險）**：把 `agent_loop.py:51-118` 的 destructive op taxonomy 抄寫進 `wuji-my-axiom/templates/` 一份 `irreversible-ops.md` 清單，作為太極「不可逆」工程定義；把 `doom_loop.py` 邏輯重實作（不是複製）成 `scripts/worker-doom-guard.sh`，掛上 `start-worker.sh`。
2. **觀察（待時機）**：等 HF 加 LICENSE 再評估是否值得 fork；目前以 `research/ml-intern.md`（本檔）入庫，後續發 paper 提 EPOCH 演化時引為「2026 業界 vertical agent 設計樣本」。
3. **跳過**：sandbox-via-HF-Space 不採用（共腦 RPi5 + tmux 已有等價隔離且自主可控）；session 上傳機制不採用（共腦走 Gitea 私有 = single source of truth，不上 HF dataset）。
