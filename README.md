Smart AI Image Editing Agent (智慧多模態修圖代理人)
這是一個基於 多模態代理 (Multimodal AI Agent) 架構的自動化圖像編輯系統，現在更加入了 Gradio Web UI 視覺化介面。

它不僅僅是文生圖，更是一個具備「視覺理解」、「決策能力」與「排版能力」的 AI 系統。它能聽懂你的中文指令，自動判斷是否需要尋找素材、偵測修圖區域，甚至幫你製作帶有文字的梗圖。

核心特色 (Key Features)
直覺式 Web UI 操作 (New!)

提供 文字轉圖片、圖片轉圖片 (AI 修圖)、梗圖嵌字 三大功能分頁。

視覺化調整 LoRA 風格、生成數量、裁切比例與重繪強度。

智慧決策中樞 (GPT-4o Brain)

理解模糊的中文指令（如：「把頭換成鋼鐵人」、「變身成麥塊風格」）。

自動選擇合適的 LoRA 模型與權重。

判斷是否啟動「素材獵人」模式（針對特定角色或物件）。

強力視覺偵測 (Florence-2 + SAM)

使用 Florence-2 進行精準的語意定位（Phrase Grounding）。

整合 SAM (Segment Anything) 進行像素級精細切割。

Box-to-Mask 機制：針對換頭任務強制使用邊界框，確保完整覆蓋，不會只切到臉部五官。

專業級重繪與合成 (SDXL + IP-Adapter)

風格轉換：動態掛載 LoRA 模型（如：盲盒風、像素風）。

素材融合 (Material Hunter)：若指令包含特定物件（如「鋼鐵人頭盔」），系統會先生成去背素材，再透過 IP-Adapter 進行風格參考與光影融合。

梗圖文字排版

內建智慧嵌字功能，自動計算文字寬度、換行與置中，支援上方或下方配字。

系統架構 (Architecture)
系統採用模組化「接力賽」設計，由 run.py 統一調度：
```text
graph TD
    User[用戶/Web UI] -->|中文指令| Brain[階段 0: AI 大腦 (GPT-4o)]
    Brain -->|產出 brain_decision.json| Check{需要素材?}
    
    Check -- Yes --> Hunter[階段 1.5: 素材獵人 (SDXL)]
    Hunter -->|生成去背素材| Detect
    
    Check -- No --> Detect[階段 1 & 2: 偵測與遮罩 (Florence-2 + SAM)]
    Detect -->|生成 mask| Artist
    
    Artist[階段 3: 藝術家 (Fooocus Inpaint + IP-Adapter)]
    Artist -->|合成與重繪| Result[最終成品]
    Result -->|顯示| GradioUI
```
安裝指南 (Installation)
本專案依賴多個 AI 模型，請依照以下步驟建立環境。

1. 建立虛擬環境
建議使用 Anaconda：
```text
conda create -n agent_env python=3.10
conda activate agent_env
```
2. 安裝 PyTorch (GPU 版)
請依照你的 CUDA 版本調整，推薦指令：


```text
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```
3. 安裝專案依賴
```text
# 確保包含 gradio, openai, diffusers, transformers, opencv-python, pillow 等
pip install -r requirements.txt
```
(如果尚未建立 requirements.txt，請參考專案中的 import 手動安裝)

4. 設定 API Key
在專案根目錄建立 .env 檔案，填入 OpenAI 金鑰：

Ini, TOML

OPENAI_API_KEY=sk-proj-你的OpenAI金鑰...
模型準備 (Model Setup)
請確保你的資料夾結構如下，缺一不可：
```text

C:\generate AI\
│
├── models\
│   ├── loras\                      # (存放風格 LoRA)
│   │   ├── minecraft.safetensors
│   │   ├── cute_blindbox_sdxl.safetensors
│   │   └── ...
│   │
│   ├── ip_adapter\                 # (IP-Adapter 模型)
│   │   ├── ip-adapter_sdxl.bin     # 下載自 h94/IP-Adapter
│   │   └── image_encoder\          # 下載自 h94/IP-Adapter
│   │       ├── config.json
│   │       └── model.safetensors
│   │
│   └── juggernatuXL_v8_diffusers\  # (主繪圖模型 Checkpoint)
│       └── juggernautXL_v8Rundiffusion.safetensors
│
├── images\                         # (自動生成的圖片暫存區)
├── app_gradio.py                   # (啟動程式)
└── ... (其他 python 腳本)
```
模型下載來源推薦：

Checkpoint: Juggernaut XL (Civitai) (或其他 SDXL Inpaint 模型)

IP-Adapter: HuggingFace h94/IP-Adapter (下載 sdxl_models/ip-adapter_sdxl.bin 以及 sdxl_models/image_encoder/)

LoRA: Civitai (請找 SDXL 專用 LoRA)

設定檔 (Configuration)
lora_library.json
請在此檔案定義你的 LoRA 模型與觸發詞。這讓 AI 大腦知道有哪些風格可用。
```text
JSON

{
    "None": { "filename": null, "trigger": "" },
    "Minecraft": {
        "filename": "minecraft.safetensors",
        "trigger": "minecraft, voxel, pixel art, blocky"
    },
    "Blindbox": {
        "filename": "cute_blindbox_sdxl.safetensors",
        "trigger": "blindbox, chibi, 3d render, cute"
    }
}
```
使用方法 (Usage)
方式一：啟動 Web UI (推薦)
這是最直覺的使用方式，擁有完整的操作介面。

```text

python app_gradio.py
```
執行後，請在瀏覽器打開終端機顯示的網址 (通常是 http://127.0.0.1:7860)。

Tab 1: 文字轉圖片 - 快速生成素材或底圖。

Tab 2: 圖片轉圖片 - 上傳圖片，輸入中文指令（如「幫他戴上墨鏡」），選擇 LoRA 風格。

Tab 3: 圖片嵌字 - 上傳圖片，輸入想要生成的梗圖文字，自動排版合成。

方式二：命令行模式 (進階除錯用)
如果你想單獨測試後端流程，可以直接執行主程式：

```text
python run.py
系統會提示你輸入指令，並依序執行各階段腳本。

檔案說明 (File Description)
app_gradio.py: [入口] Gradio Web 介面主程式，負責處理 UI 邏輯並呼叫後端。

run.py: [總指揮] 負責串接所有階段的 Pipeline。

run_stage0_llm.py: [大腦] 呼叫 GPT-4o 解析意圖、選擇 LoRA、撰寫 Prompt，並決定是否需要素材。

run_stage1_5_material.py: [獵人] 若大腦決定需要素材，此腳本負責生成去背的參考圖 (如頭盔、特定道具)。

run_florence_plus_sam.py: [眼睛] 負責偵測目標物體並生成遮罩 (Mask)。

run_stage3_inpaint.py: [畫家] 使用 Fooocus SDXL + IP-Adapter 進行最終的局部重繪。

insert_word.py: [排版] 負責處理圖片上的文字排版與繪製。

