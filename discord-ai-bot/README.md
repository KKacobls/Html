# Discord AI Bot · Qwen / Ollama

本機 Discord AI 聊天機器人，使用 Python `discord.py`、FastAPI 與 Ollama。管理後台可以逐一選擇 Bot 有權查看及發言的頻道。**預設沒有勾選任何頻道 = 不自動回覆。**

## 功能

- 自動列出 Bot 加入的 Discord 伺服器、可見頻道、發言權限；新頻道預設不勾選。
- 依伺服器儲存回覆模式和頻道清單：**指定頻道**（預設）、**@Bot 模式**、**混合模式**；另外支援 `/chat` 手動提問。
- 在勾選頻道新訊息上使用 Discord **原生 Reply**，先回覆「🤔 思考中…」，模型完成後編輯該則 Reply。
- 初次回覆時由 Discord API 即時抓取頻道最近 50 則訊息當作上下文（可在後台改為 0–100）；**不會回覆歷史訊息**。各頻道／討論串分開使用上下文。
- Ollama Qwen3.5-35B-A3B 或 9B，關閉 Thinking，預設 `num_ctx=8192`。
- 管理員 Basic Auth 密碼、127.0.0.1 本機限定的後台、SSH 轉發、Bot/其他機器人訊息過濾、10 秒冷卻、單 GPU 併發鎖。
- 設定存在 `data/settings.json`，重新啟動仍保留；`.env`、`.venv`、`data/` 在 `.gitignore`。

## 伺服器（目前環境 Ubuntu WSL2）

假設使用者已經安裝 `uv`、建立 `~/discord-ai-bot/.venv`，且 `discord.py fastapi uvicorn` 已安裝：

1. 取得程式碼放入 `~/discord-ai-bot/`，並保留原來的 `.env`、`.venv` 和 `.tools`；請勿覆蓋 `.env`。
2. `.env` 內容（敏感資訊不得 commit）：
   ```env
   DISCORD_TOKEN=your_actual_bot_token
   OLLAMA_URL=http://127.0.0.1:11555
   ```
3. Discord Developer Portal → Bot → **Message Content Intent** 啟用；OAuth2 安裝需要 `bot`、`applications.commands`，Bot 需要 `View Channels`、`Send Messages`、`Read Message History`（以及 threads 所需的發訊權限）。
4. Ollama 需在 `127.0.0.1:11555` 啟動，下載 `qwen3.5:35b-a3b`；如需改成 9B 可在管理後台切換。
5. 啟動：
   ```bash
   cd ~/discord-ai-bot
   .venv/bin/python app.py
   ```
6. **首次啟動後**，在 Ubuntu 查看管理員密碼（只在自己的終端機查看，不要貼到聊天或 GitHub）：
   ```bash
   cat ~/discord-ai-bot/data/admin_password.txt
   ```
   帳號固定為 `admin`。可自行在 `.env` 指定 `ADMIN_PASSWORD`（高強度密碼），重啟後生效。
7. **Windows PowerShell** 開啟 SSH tunnel：
   ```powershell
   ssh -L 8765:127.0.0.1:8765 -p <SSH_PORT> -i "$env:USERPROFILE\.ssh\id_ed25519" <SSH_USER>@<SSH_HOST>
   ```
   在本機瀏覽器開啟 <http://127.0.0.1:8765>，以 `admin` 與密碼登入，勾選頻道並儲存。

### 背景執行（確認正常後再用）

```bash
cd ~/discord-ai-bot
nohup .venv/bin/python app.py > bot.log 2>&1 < /dev/null &
```

僅對一般 SSH 斷線有保護，WSL2 主機停止、重開機或共享 GPU 被收回仍會中斷；正式常駐使用請先符合學校伺服器政策。只允許一份 Bot 程序登入，避免重複回覆。

## 設定細節

- **指定頻道（預設）**：沒有打勾就完全不會自動回覆。勾選的頻道收到新訊息才回覆。
- **@Bot**：只在提及 Bot 的訊息回覆；`/chat` 是額外的手動呼叫。
- **混合**：勾選頻道自動回覆；其餘頻道僅在 @Bot 時回覆。
- 管理頁從 Discord 實時取可見文字頻道，無法發言者不可選；討論串會跟隨其父頻道設定。
- 讀取歷史訊息用於模型輸入，不是完整永久性儲存。Discord 歷史訊息可能受權限影響；只保留最近 50 則、裁切上下文至有限長度。
- `/chat` 需要 `applications.commands`，全球指令剛同步後可能要等 Discord 分發；Slash 回覆是互動回覆，文字聊天是原生 Reply。
- 模型使用單張共用 GPU：一次只推論一個請求，繁忙時使用者會等待；可按需要縮小歷史訊息數量。
- 管理頁 **只綁定 127.0.0.1**，切勿將帳密、Token 或 Ollama 服務埠直接暴露於網際網路。

## 授權

此實作使用 `discord.py`、FastAPI、Uvicorn、aiohttp（保留各原始套件授權）。主程式由本專案撰寫，並未從其他 Discord Bot 倉庫直接複製原始碼。