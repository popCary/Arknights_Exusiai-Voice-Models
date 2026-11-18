# Exusiai Voice Models（能天使 AI 聲音模型）

《明日方舟》能天使的 AI 聲音轉換模型，包含 RVC v2 與 So-VITS-SVC 4.1 兩種架構。

> ⚠️ **使用前請閱讀文末「聲明」。** 本模型僅供個人學習與娛樂，禁止商用、禁止冒充聲優本人或官方。

## 模型一覽

| 下載檔 | 架構 | 訓練語言 | 素材長度 | 推薦用途 |
|---|---|---|---|---|
| `Exusiai-SoVITS-4.1.zip` | So-VITS-SVC 4.1（44.1 kHz） | 日文 | ~18.5 分 | 翻唱（推薦，音色最像） |
| `Exusiai-RVC-v2.zip` → `exusiai_ja` | RVC v2（48 kHz） | 日文 | ~8.5 分 | 日文歌、即時變聲 |
| `Exusiai-RVC-v2.zip` → `exusiai_zh` | RVC v2（48 kHz） | 中文 | ~5 分 | 中文歌 |

## RVC v2 使用方式

1. `*.pth` 放進 RVC 的 `assets/weights/`
2. `*.index` 放進 `assets/indices/`（或推論時手動選擇索引路徑）
3. WebUI 按「刷新音色列表」後選擇模型

| 參數 | 建議值 |
|---|---|
| 音高提取 | rmvpe |
| 檢索特徵占比（index rate） | `exusiai_ja`：0.75–0.85；`exusiai_zh`：0.6–0.85 |
| 保護清輔音（protect） | 0.33 |
| 響度包絡融合（rms mix rate） | 0.25 |

> 附帶的 `.index` 是精確搜尋（`IndexFlatL2`），不是 RVC 預設的 IVF（nprobe=1），檢索比例拉高也比較不會咬字糊。

## So-VITS-SVC 4.1 使用方式

1. 解壓後把整個 `exusiai_s8000/` 資料夾放進 So-VITS-SVC 的 `trained/`
   （資料夾內需剛好一個 `.pth` 與一個 `config.json`；`feature_and_index.pkl` 可留在同資料夾）
2. WebUI → 本地模型 → 刷新列表 → 選 `exusiai_s8000`
3. 「聚類模型或特徵檢索檔案」上傳 `feature_and_index.pkl`，按「加載模型」

| 參數 | 建議值 |
|---|---|
| 音色 | `exusiai` |
| F0 預測器 | rmvpe |
| 自動 f0 預測 | **關閉**（唱歌開啟會嚴重跑調） |
| 聚類 / 特徵檢索混合比例 | 0.5（咬字不清時降到 0；超過 0.6 不會更像，咬字反而變差） |
| noise_scale | 0.4 |

需求：So-VITS-SVC 4.1-Stable，speech encoder `vec768l12`（ContentVec `checkpoint_best_legacy_500.pt`）。

> `feature_and_index.pkl` 是 Python pickle 格式（So-VITS 原生格式），只載入你信任來源的 pickle 檔。

## 訓練細節

**So-VITS-SVC 4.1（`exusiai_s8000`）**

- 內容特徵：ContentVec 768 維第 12 層；音高：RMVPE；聲碼器：NSF-HiFiGAN
- 底模：sovits 768l12 pre-large 320k
- 資料：日文遊戲語音 + 皮膚語音，約 18.5 分鐘（244 段，切成 5–15 秒）
- batch 8，共訓練約 21,700 步，選用第 8000 步；權重為 FP16
- 特徵檢索：FAISS `IndexFlatL2`（精確搜尋）

**RVC v2**

- `exusiai_ja`：日文遊戲語音約 8.5 分鐘（175 段），300 epoch 中選用第 250 epoch
- `exusiai_zh`：中文語音約 5 分鐘（115 段），300 epoch 中選用第 200 epoch
- 底模 `f0G48k` / `f0D48k`，batch 8

**評估**（5 句 Edge-TTS 測試句轉換後量測）

| 模型 | Whisper 錯字率 | 聲紋相似度（Resemblyzer） |
|---|---|---|
| So-VITS `exusiai_s8000`（檢索 0.5） | < 1% | 0.919 |
| RVC `exusiai_ja`（index 0.85） | 0.5% | 0.903 |
| 參考：真人原音 | — | 0.89–0.91 |
| 參考：未轉換的 TTS | 0.6% | 0.806 |

## 聲明

- 能天使角色及聲音之權利屬於鷹角網路（Hypergryph）與日文 CV 石見舞菜香；中文語音之權利屬於原權利人。
- 本模型為非官方的同人作品，與上述權利人無關，亦不代表其立場。
- **僅限個人、非商業用途。** 禁止用於商業、詐騙、冒充聲優或官方發言、製作不實或有害內容。
- 使用本模型產生的作品請標明「AI 聲音轉換」，並自行遵守原曲與各平台的規範。
- 若權利人要求下架，將立即移除。
