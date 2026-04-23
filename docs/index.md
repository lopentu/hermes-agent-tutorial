# 生成式 AI 的人文導論

*2026 年 4 月 24 日*

### 打造你的 AI Agent：用 Hermes Agent + Gemini + Telegram 從零開始

> 這份教學帶你在自己的電腦上架設 AI Agent，透過 Telegram 隨時跟它對話，還能叫它執行程式碼、操作檔案。

---

## 為什麼要學這個？

### 一般 LLM 和 AI Agent 有什麼不同？

你可能已經用過 ChatGPT、Claude 或 Gemini 這些 AI 聊天工具。它們很聰明，能回答問題、寫文章、翻譯——但它們**只能跟你聊天**，沒辦法真的幫你做事。

**AI Agent 不一樣。** 它不只會說，還會**動手做**：

| | 一般 LLM App | AI Agent |
|---|---|---|
| 回答問題 | ✅ | ✅ |
| 執行終端機指令 | ❌ | ✅ **直接在電腦上執行** |
| 操作檔案 | ❌ | ✅ 建立、讀取、修改檔案 |
| 定時自動執行任務 | ❌ | ✅ 排程功能 |
| 記住你的偏好 | 有限 | ✅ 跨對話記憶 |
| 從手機操控電腦 | ❌ | ✅ 透過 Telegram |

想像你要跑一個耗時的實驗或備份，不想乾等在電腦前。你掏出手機，傳一句話給你的 Bot：「開始備份桌面資料夾」或「執行這個分析程式」。Bot 立刻在你的電腦上動起來——你去吃飯、去上課，回來就完成了。這不是告訴你怎麼做，而是**直接幫你做完**。

### 為什麼選 Hermes Agent？

[Hermes Agent](https://github.com/NousResearch/hermes-agent) 是由 [Nous Research](https://nousresearch.com/) 開發的開源 AI Agent 框架，有幾個特色讓它適合學習和使用：

- **開源免費**：MIT 授權，完全免費使用
- **不綁定模型**：支援多種 AI 模型，可以用免費的 Gemini API，也能跑本地模型
- **多平台串接**：一個 Agent 可以同時透過 Telegram、Discord、Slack、WhatsApp 等平台操作
- **記憶功能**：記住你的習慣和偏好，下次不用重複說明
- **自動排程**：用自然語言設定定時任務，例如「每天早上 9 點幫我摘要新聞」
- **技能系統**：每次完成任務後會分析自己的表現並更新記憶，越用越聰明

### 今天的目標

**基本目標**：完成以下三件事就算成功——

1. 串接成功：Telegram Bot 上線
2. 可以正常對話
3. 執行終端機指令並回傳結果

**進階目標**：基本目標完成後，我們再來挑戰——

1. 自訂 Agent 的行為規則，例如叫它永遠用繁體中文回覆、或扮演特定角色
2. 讓 Agent 自己決定要用哪個工具完成任務，例如中文斷詞、查今天台北的天氣

---

## 環境需求

- **作業系統**：macOS 或 Linux（Windows 需要 [WSL2](https://learn.microsoft.com/zh-tw/windows/wsl/install)）
- **記憶體**：2GB 以上（使用雲端模型不需要大記憶體）
- **網路**：需要連網
- **帳號**：Google 帳號（申請 Gemini API Key）、Telegram 帳號

---

## Part 1：基本準備

> ⚠️ **Windows 使用者請注意**：Hermes Agent 不支援原生 Windows，需要先安裝 WSL2。請先跳到本文最後的[「附錄：Windows 使用者完整指南」](#附錄windows-使用者完整指南)完成 WSL2 設定後，再回來繼續。

### 1.1 安裝 Telegram

如果你還沒有 Telegram，到 App Store 或 Google Play 搜尋「Telegram」下載並註冊帳號。

### 1.2 申請 Gemini API Key

Gemini 是 Google 的 AI 模型，提供免費的 API 額度，足夠我們上課使用。

1. 前往 [Google AI Studio](https://aistudio.google.com/apikey)
2. 用你的 Google 帳號登入
3. 點擊「Create API Key」建立一組 API Key
4. **複製並保存好這組 Key**（格式像 `AIzaSy...`），上課會用到

> ⚠️ API Key 等同密碼，不要分享給別人，也不要上傳到 GitHub。

### 1.3 確認系統環境

打開終端機（Terminal）：

- **Mac**：按 `Cmd + 空格`，輸入「Terminal」，按 Enter
- **Windows**：開啟 Ubuntu（WSL）終端（如果還沒安裝 WSL2，請先看[附錄](#附錄windows-使用者完整指南)）
- **Linux**：打開你的 Terminal

輸入以下指令確認 Python 版本：

```bash
python3 --version
```

需要 Python 3.11 以上。如果版本太舊或沒安裝，請先更新。

---

## Part 2：建立 Telegram Bot

我們需要一個 Telegram Bot 作為 AI Agent 的入口。

### 2.1 跟 BotFather 申請 Bot

1. 在 Telegram 搜尋 **@BotFather**，點進去開始對話
2. 輸入 `/newbot`
3. BotFather 會問你 Bot 的**名稱**（顯示名稱，隨意取），例如：`My AI Agent`
4. 再問你 Bot 的**使用者名稱**（必須以 `bot` 結尾），例如：`my_ai_agent_2026_bot`
5. 建立成功後，BotFather 會給你一組 **Bot Token**（格式像 `123456789:ABCdefGHI...`）

> ⚠️ **複製並保存好這組 Token**，等一下設定 Hermes 時會用到。

### 2.2 測試 Bot

在 Telegram 搜尋你剛才設定的 Bot 使用者名稱，點進去傳一則訊息。此時 Bot 不會回覆（因為還沒串接 Hermes），這是正常的。

---

## Part 3：安裝 Hermes Agent

### 3.1 一鍵安裝

在終端機輸入：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安裝程式會自動處理所有相依套件，大約需要 2-5 分鐘。

### 3.2 設定精靈

安裝完成後，會自動進入設定精靈。依照以下步驟操作：

**選擇 Provider：**

用方向鍵移到 `Google AI Studio (Gemini models — native Gemini API)`，按 Enter。

**輸入 API Key：**

畫面會顯示：

```
GOOGLE_API_KEY (or Enter to cancel):
```

貼上你在 Part 1 申請的 Gemini API Key，按 Enter。

> 💡 終端機貼上的快捷鍵：Mac 用 `Cmd + V`，Linux / WSL 用 `Ctrl + Shift + V`。

**設定 Base URL：**

```
Base URL [https://generativelanguage.googleapis.com/v1beta]:
```

直接按 Enter，使用預設值。

**選擇模型：**

選擇 **gemini-2.5-flash-lite**（免費額度最高、速度快、穩定性好）。

> 💡 如果之後想換模型，隨時可以用 `hermes model` 指令切換。

**串接 Telegram：**

設定精靈會問是否串接訊息平台，選擇 **Set up messaging now**。

在平台列表中，**按空格鍵**勾選 Telegram，然後按 Enter。

畫面會顯示：

```
Telegram bot token:
```

貼上你在 Part 2 取得的 Bot Token，按 Enter。（同樣用 `Cmd + V` 或 `Ctrl + Shift + V`）

**設定使用者權限：**

```
Allowed user IDs (comma-separated, leave empty for open access):
```

直接按 Enter（留空表示所有人都能使用，上課方便測試）。

**設定 Home Channel：**

```
Home channel ID (leave empty to set later):
```

直接按 Enter，之後再設定。

**重啟 Gateway：**

```
Restart the gateway to pick up changes? [Y/n]:
```

按 `Y`。

### 3.3 允許所有使用者存取

Hermes 預設會擋住未授權的使用者。為了上課方便，開放所有人存取：

```bash
hermes config set GATEWAY_ALLOW_ALL_USERS true
```

然後重啟 Gateway：

```bash
pkill -f hermes
rm -f ~/.hermes/gateway.pid
hermes gateway start
```

---

## Part 4：測試你的 AI Agent

### 4.1 基本對話

打開 Telegram，在搜尋欄輸入你在 Part 2 設定的 Bot 使用者名稱（就是那個以 `bot` 結尾的名稱，例如 `my_ai_agent_2026_bot`），點進去傳送：

```
你好
```

如果 Bot 回覆了，恭喜！你的 AI Agent 已經上線了 🎉

### 4.2 執行終端機指令

這是 AI Agent 跟一般 LLM 最大的差別。傳送：

```
告訴我現在這台電腦的記憶體使用量
```

Bot 會直接在你的電腦上執行指令，然後把結果傳回來——不是告訴你怎麼查，而是**直接幫你查完了**。

### 4.3 操作檔案

傳送：

```
列出我桌面上的檔案
```

Bot 會使用 terminal 工具，執行 `ls` 指令，然後把你桌面上的檔案列出來。

> 💡 一般 LLM App 做不到這一點——它們無法存取你的電腦。

### 4.4 想想看：還能做什麼？

試試看這些指令，體驗 AI Agent 的能力：

- `幫我建立一個叫 test.txt 的檔案，裡面寫上今天的日期`
- `幫我查今天台北的天氣`
- `列出最近修改過的五個檔案`

---

## Part 5：日常使用

### 啟動 Agent

每次要使用時，在終端機輸入：

```bash
hermes gateway start
```

### 停止 Agent

不用時停掉，節省資源：

```bash
hermes gateway stop
```

### 切換模型

遇到 Rate Limit 錯誤時，可以換一個模型試試：

```bash
hermes model
```

會開啟互動介面讓你選擇不同的 Provider 和模型。

### 重設對話

在 Telegram 中輸入：

```
/reset
```

### 檢查問題

```bash
hermes doctor
```

會自動掃描環境，列出需要修復的問題。

---

## 常見問題

### Bot 沒有回應

**可能原因 1：Gateway 沒有在跑**

```bash
hermes gateway status
```

如果沒在跑，啟動它：

```bash
hermes gateway start
```

**可能原因 2：多個 Gateway 衝突**

```bash
pkill -f hermes
rm -f ~/.hermes/gateway.pid
hermes gateway start
```

**可能原因 3：Telegram 連線問題**

檢查 error log：

```bash
cat ~/.hermes/logs/gateway.error.log | tail -10
```

### 看到 Rate Limit 錯誤

```
HTTP 429 (RESOURCE_EXHAUSTED): You exceeded your current quota
```

這是 Gemini 免費方案的限制（每分鐘約 20 次請求）。

**解決方式**：
- 等 1-2 分鐘再試，每次提問之間不要太快連續傳訊息
- 或切換到其他模型：執行 `hermes model`，選擇 `gemini-2.5-flash` 或 `gemini-2.5-flash-lite`

> 💡 Hermes 每次互動會送出多次 API 呼叫（你的訊息、工具呼叫、結果回傳等），所以一個簡單的問答可能就用掉 4-5 次請求。

### 看到 Unauthorized 錯誤

```
Unauthorized user: XXXXXXX on telegram
```

執行以下指令允許所有使用者存取：

```bash
hermes config set GATEWAY_ALLOW_ALL_USERS true
pkill -f hermes
rm -f ~/.hermes/gateway.pid
hermes gateway start
```

### Bot 說「我沒有 terminal 工具」

這通常是 Gemini 3 系列模型的相容性問題。請使用以下模型：

- ✅ `gemini-2.5-flash-lite`（推薦）
- ✅ `gemini-2.5-flash`
- ❌ `gemini-3-flash-preview`（thought_signature 問題）
- ❌ `gemini-3-pro-preview`（thought_signature 問題）

切換模型：

```bash
hermes config set model.default gemini-2.5-flash-lite
pkill -f hermes
rm -f ~/.hermes/gateway.pid
hermes gateway start
```

執行完後**等約 1 分鐘**讓 Gateway 完全啟動，再回到 Telegram 輸入 `/reset` 重設對話，然後重新傳訊息測試。

---

## 模型選擇指南

| 模型 | 費用 | 特點 | 建議用途 |
|---|---|---|---|
| gemini-2.5-flash-lite | 免費 | 速度快、rate limit 較寬鬆 | 日常使用（推薦） |
| gemini-2.5-flash | 免費 | 能力較強、rate limit 較嚴 | 需要更好推理能力時 |
| gemini-2.5-pro | 免費（有限） | 最強推理能力 | 複雜任務 |

> 💡 如果你的電腦有 16GB 以上記憶體，也可以用 [Ollama](https://ollama.com/) 跑本地模型（例如 `qwen3:8b`），完全離線、不花 API 費用，但需要更多設定。

---

## 加分作業

完成基本設定後，挑戰以下兩個任務，二選一或全做。

### 任務一：自訂 Agent 的角色（System Prompt）

Hermes 的個性存在 `~/.hermes/SOUL.md` 這個檔案，直接編輯它就能改變 Agent 的行為規則。

**步驟：**

1. 在終端機開啟檔案：

```bash
nano ~/.hermes/SOUL.md
```

2. 在最上方加入你的規則，例如：

```
你是台大語言學系的助教，叫做小幫。每次回覆都要用繁體中文，語氣要親切。
```

3. 按 `Ctrl + X`，再按 `Y`，再按 Enter 儲存。

4. 重啟 Gateway：

```bash
pkill -f hermes
rm -f ~/.hermes/gateway.pid
hermes gateway start
```

5. 到 Telegram 輸入 `/reset`，然後傳一則訊息，確認 Agent 的回應符合你設定的角色。

**成功標準**：Bot 的回應反映出你設定的個性或規則。

---

### 任務二：Tool Calling

給 Agent 一個需要用工具才能完成的任務，觀察它如何自動選擇工具。

傳送以下其中一個指令：

```
幫我對這段文字做中文斷詞：「生成式人工智慧正在改變人文學科的研究方式」
```

或：

```
查今天台北的天氣
```

**成功標準**：Bot 自動呼叫工具完成任務，回傳實際結果，而不只是用文字說明怎麼做。

---

## 延伸閱讀

- [Hermes Agent 官方文件](https://hermes-agent.nousresearch.com/docs/)
- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
- [Google AI Studio](https://aistudio.google.com/)
- [Ollama 官網](https://ollama.com/)（本地模型方案）

---

## 附錄：Windows 使用者完整指南

Hermes Agent **不支援原生 Windows**，必須透過 WSL2（Windows Subsystem for Linux）執行。WSL2 會在你的 Windows 裡跑一個輕量的 Linux 環境，Hermes 就在裡面運作。

不用擔心，設定並不困難，以下是完整步驟。

### 步驟 1：安裝 WSL2

1. **以系統管理員身份**開啟 PowerShell（在開始選單搜尋「PowerShell」，右鍵點選「以系統管理員身份執行」）
2. 執行以下指令：

```powershell
wsl --install
```

3. 等待安裝完成，然後**重新開機**
4. 重開機後，Windows 會自動開啟 Ubuntu 視窗，要求你設定：
   - **使用者名稱**（小寫英文，例如 `student`）
   - **密碼**（輸入時不會顯示，這是正常的，打完按 Enter）

> ⚠️ 請記住這組密碼，之後安裝套件時會用到。

### 步驟 2：更新系統套件

在 Ubuntu（WSL）終端裡執行：

```bash
sudo apt update && sudo apt upgrade -y
```

系統會要求你輸入剛才設定的密碼。

### 步驟 3：確認 Python 版本

```bash
python3 --version
```

需要 3.11 以上。WSL2 的 Ubuntu 通常自帶符合條件的版本。如果版本太舊：

```bash
sudo apt install python3.12 -y
```

### 步驟 4：安裝 Hermes Agent

跟 Mac / Linux 完全一樣的指令：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安裝完成後，按照本教學 **Part 3** 的設定精靈步驟操作即可。

### 步驟 5：啟動 Gateway

```bash
hermes gateway start
```

然後到 Telegram 測試你的 Bot。

### Windows 使用者的注意事項

**每次重開機後需要重新啟動：**

WSL2 不會自動啟動 Hermes Gateway。每次開機後需要：

1. 開啟 Ubuntu（WSL）終端
2. 執行 `hermes gateway start`

**檔案路徑不同：**

在 WSL2 裡，你的 Windows 檔案在 `/mnt/c/` 底下。不確定自己的使用者名稱，可以執行：

```bash
echo $USER
```

例如使用者名稱是 `student`，路徑就是：

- Windows 桌面：`/mnt/c/Users/student/Desktop`
- Windows 文件：`/mnt/c/Users/student/Documents`

所以如果你要 Bot 列出 Windows 桌面的檔案，要這樣說：

```
列出 /mnt/c/Users/你的使用者名稱/Desktop 裡的檔案
```

**開啟 WSL 終端的方式：**

- 在開始選單搜尋「Ubuntu」
- 或開啟 PowerShell / CMD，輸入 `wsl`

### 常見的 Windows 專屬問題

**「bad interpreter」錯誤：**

表示你在原生 Windows 的 PowerShell 或 CMD 裡執行了 Hermes 指令。請確認你是在 **WSL2 的 Ubuntu 終端**裡操作。

**WSL2 沒有啟動：**

如果 `wsl --install` 失敗，可能需要先在 BIOS 中啟用虛擬化（Virtualization）。重新開機時進入 BIOS，找到 Intel VT-x 或 AMD-V 選項並啟用。

**hermes 指令找不到：**

如果安裝完後 `hermes` 指令無法使用，執行：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

要讓這個設定永久生效：

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

*本教學基於 Hermes Agent v0.10.0，使用 Gemini 2.5 Flash Lite 模型。*
*最後更新：2026 年 4 月*
