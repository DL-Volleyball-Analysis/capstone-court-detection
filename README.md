# Volleyball Court Detection | 排球場地與落點偵測

A complete volleyball ball landing detection pipeline with court boundary detection and IN/OUT classification.

完整的排球落地點偵測流程，包含場地邊界檢測和 IN/OUT 判定。

---

## Features | 功能特色

| Feature | Description |
|---------|-------------|
| Ball Tracking | VballNet ONNX model for ball trajectory detection |
| Court Detection | Interactive 4-point court boundary annotation |
| Landing Detection | Automatic IN/OUT classification with HUD overlay |
| Video Output | Visualization with landing markers and history panel |

| 功能 | 說明 |
|------|------|
| 球體追蹤 | VballNet ONNX 模型進行球體軌跡偵測 |
| 場地檢測 | 互動式四點場地邊界標註 |
| 落點偵測 | 自動 IN/OUT 分類並顯示 HUD 面板 |
| 影片輸出 | 落點標記和歷史記錄面板的視覺化 |

---

## Project Structure | 專案結構

```
volleyball-court-detection/
├── models/                          # ONNX model files
│   ├── VballNetV1_seq9_grayscale_148_h288_w512.onnx
│   └── ...
├── src/
│   ├── inference_onnx_seq9_gray_v2.py   # Ball detection inference
│   ├── click_court.py                   # Court boundary annotation tool
│   └── postprocess_inout_patched_v3.py  # Landing detection + IN/OUT
├── output/                          # Output directory (generated)
├── requirements.txt
├── pyproject.toml
└── README.md
```

---

## Installation | 安裝

### Prerequisites | 前置需求

- Python 3.10+
- pip

### Install Dependencies | 安裝依賴

```bash
pip install -r requirements.txt
```

Or install individually:

```bash
pip install onnxruntime opencv-python pandas numpy scipy tqdm
```

---

## Usage | 使用方法

### Step 1: Ball Detection | 第一步：球體偵測

Run the ONNX inference script to detect ball positions from video:

執行 ONNX 推論腳本從影片中偵測球的位置：

```bash
python src/inference_onnx_seq9_gray_v2.py \
  --video_path input_video.mp4 \
  --model_path models/VballNetV1_seq9_grayscale_148_h288_w512.onnx \
  --output_dir output \
  --track_length 10 \
  --visualize
```

**Output | 輸出：**
- `output/{video_name}_predict_ball.csv` - Ball coordinates per frame | 每幀球座標
- `output/{video_name}_predict.mp4` - Visualization video (if --visualize) | 視覺化影片

### Step 2: Court Annotation | 第二步：場地標註

If `court.json` does not exist, the script will automatically launch the annotation tool.

如果 `court.json` 不存在，腳本會自動啟動標註工具。

Manual launch | 手動啟動：

```bash
python src/click_court.py \
  --video input_video.mp4 \
  --out output/court.json
```

**Instructions | 操作說明：**
- Click 4 corners in order: Top-Left, Top-Right, Bottom-Right, Bottom-Left
- 依序點擊四個角：左上、右上、右下、左下
- Press `S` to save, `Z` to undo, `C` to clear, `G` to toggle grid
- 按 `S` 儲存、`Z` 復原、`C` 清空、`G` 切換網格

### Step 3: Landing Detection | 第三步：落點偵測

Run the landing detection with IN/OUT classification:

執行落點偵測與 IN/OUT 分類：

```bash
python src/postprocess_inout_patched_v3.py \
  --csv_path output/{video_name}_predict_ball.csv \
  --court_json output/court.json \
  --out_csv output/landings_inout.csv \
  --video_path input_video.mp4 \
  --out_video output/landings_overlay.mp4 \
  --curve_filter_on \
  --min-x-drift-px 250 \
  --hud \
  --verbose
```

**Output | 輸出：**
- `output/landings_inout.csv` - Landing coordinates with IN/OUT labels | 落點座標與 IN/OUT 標籤
- `output/landings_overlay.mp4` - Video with HUD overlay | 帶有 HUD 面板的影片

---

## Model Information | 模型資訊

| Model | Input | Output | Description |
|-------|-------|--------|-------------|
| VballNetV1_seq9_grayscale | 9 grayscale frames (1, 9, 288, 512) | Heatmap | Ball position detection |

The model uses a 9-frame grayscale sequence to capture ball motion patterns for improved detection accuracy.

模型使用 9 幀灰階序列來捕捉球的運動模式，以提高檢測準確度。

---

## Command Line Options | 命令列選項

### inference_onnx_seq9_gray_v2.py

| Option | Description |
|--------|-------------|
| `--video_path` | Path to input video file |
| `--model_path` | Path to ONNX model file |
| `--output_dir` | Directory for output files |
| `--track_length` | Length of ball track visualization |
| `--visualize` | Enable real-time visualization |
| `--only_csv` | Save only CSV, skip video output |

### postprocess_inout_patched_v3.py

| Option | Description |
|--------|-------------|
| `--csv_path` | Path to ball detection CSV |
| `--court_json` | Path to court boundary JSON |
| `--out_csv` | Output CSV for landing events |
| `--video_path` | Input video for overlay |
| `--out_video` | Output video with overlays |
| `--curve_filter_on` | Enable curve-based filtering |
| `--min-x-drift-px` | Minimum x-movement threshold (pixels) |
| `--hud` | Display history HUD panel |
| `--verbose` | Show detailed debug information |

---

## Output Format | 輸出格式

### landings_inout.csv

| Column | Description |
|--------|-------------|
| frame | Frame number of landing event |
| x | X coordinate of landing position |
| y | Y coordinate of landing position |
| in_out | Classification: "in", "out", or "unknown" |
| time_sec | Timestamp in seconds |

---

## Tips | 小技巧

- Adjust `--min-x-drift-px` (150-300 recommended) for stricter landing event filtering
- 調整 `--min-x-drift-px`（建議 150-300）以更嚴格過濾落點事件
- Use `--hud` to display landing history panel on the right side of the video
- 使用 `--hud` 在影片右側顯示落點歷史記錄面板
- Enable `--verbose` for debugging detailed information
- 開啟 `--verbose` 以顯示詳細除錯資訊

---

## Related Projects | 相關專案

- [volleyball_analysis_webapp](https://github.com/DL-Volleyball-Analysis/volleyball_analysis_webapp) - Main web application
- [action-recognition-yolov11](https://github.com/DL-Volleyball-Analysis/action-recognition-yolov11) - Action recognition training

---

## License | 授權

This project is part of the DL-Volleyball-Analysis capstone project.

本專案為 DL-Volleyball-Analysis 專題研究的一部分。

---

*National Taiwan Ocean University - Department of Computer Science*  
*國立臺灣海洋大學 資訊工程學系*
