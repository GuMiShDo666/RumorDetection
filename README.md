## 项目概述

本项目 **RumorDetection** 基于中文 BERT 模型与 GPT-4o-mini 混合推理，实现谣言检测与结构化报告生成。支持**文本检测**与**图片检测**两种输入形式：

**本地流程：**

1. 用预训练 BERT 对输入文本进行初步二分类（“谣言”/“非谣言”） (\[GitHub]\[1])
2. 若置信度落于灰区，则调用 OpenAI GPT-4o-mini 生成基于 JSON Schema 的详细检测报告 (\[GitHub]\[1])
3. 图片检测：通过 OCR 引擎（如 Tesseract 或自定义模块）从图片中提取文字，再进入上述文本检测流程，实现“图片→OCR→文本清洗→BERT/GPT 预测”的一条龙服务

同时提供：

* 文本数据预处理脚本，将 Word 文档转为 CSV 并切分为 train/valid/test (\[GitHub]\[2])
* 训练脚本，实现端到端 BERT 模型训练与最佳模型保存 (\[GitHub]\[3])
* Flask Web 服务，暴露 RESTful API，支持前端交互 (\[GitHub]\[4])
* **图片检测 API**：可通过 `/api/image_detect` 端点上传图片，返回 OCR 文本及谣言检测结果和报告

---

## 环境与依赖

建议使用 Python 3.8+，创建虚拟环境后安装下列库：

```bash
pip install \
  torch \
  transformers \
  pandas \
  numpy \
  scikit-learn \
  openai \
  flask \
  flask-cors \
  python-docx \
  pillow \
  pytesseract \
  seaborn \
  matplotlib
```

* **Pillow & pytesseract**：处理图片输入并执行 OCR 文字提取
* **PyTorch & Transformers**：BERT 模型加载与微调 (\[GitHub]\[5], \[GitHub]\[6])
* **pandas & numpy & scikit-learn**：数据处理与评价指标计算 (\[GitHub]\[5], \[GitHub]\[7])
* **openai**：GPT-4o-mini 调用 (\[GitHub]\[1])
* **flask & flask-cors**：提供 HTTP API (\[GitHub]\[4])
* **python-docx**：将 .docx 文档转为 CSV (\[GitHub]\[2])
* **seaborn & matplotlib**：混淆矩阵可视化（可选） (\[GitHub]\[7])

---

## 快速开始

1. **克隆项目**

   ```bash
   git clone git@github.com:JieJieSAM/RumorDetection.git
   cd RumorDetection
   ```

2. **准备数据**

   * 将你的 Word 文档路径在 `docx_to_csv_and_split.py` 中修改为实际路径 (\[GitHub]\[2])
   * 运行：

     ```bash
     python docx_to_csv_and_split.py
     ```
   * 数据集生成于 `data/` 下，包含 `train.csv`、`valid.csv`、`test.csv` (\[GitHub]\[8])

3. **训练模型**

   ```bash
   python train.py
   ```

   * 最佳模型保存在 `models/best_model.pth` (\[GitHub]\[8])

4. **本地文本检测示例**

   ```bash
   python inference.py
   ```

   * 输入任意中文文本，返回 `(label, confidence, report)` (\[GitHub]\[1])

5. **本地图片检测示例**

   ```bash
   python inference.py --image /path/to/image.png
   ```

   * 使用自定义脚本或直接调用 `/api/image_detect`，通过 OCR 提取文字并执行谣言检测

6. **启动 Web 服务**

   ```bash
   python app.py
   ```

   * 访问 `http://localhost:5000/` 查看前端页面 (\[GitHub]\[4])
   * POST `/api/predict` 接口说明：

     ```json
     {
       "text": "待检测文本"
     }
     ```

     返回：

     ```json
     {
       "label": "谣言|非谣言|不确定",
       "confidence": 0.85,
       "report": { …JSON 结构化报告… }
     }
     ```
   * POST `/api/image_detect` 接口说明：

     ```bash
     curl -X POST -F "file=@/path/to/image.png" http://localhost:5000/api/image_detect
     ```

     返回：

     ```json
     {
       "label": "谣言|非谣言|unknown",
       "confidence": 0.78,
       "report": { … },
       "ocr_text": "提取的文本内容"
     }
     ```

---

## 配置说明

所有路径及超参数在 `config.py` 中统一管理：

```python
TRAIN_CSV = 'data/train.csv'
VALID_CSV = 'data/valid.csv'
TEST_CSV  = 'data/test.csv'

PRE_TRAINED_MODEL_NAME = 'bert-base-chinese'
BATCH_SIZE = 16
EPOCHS     = 5
LEARNING_RATE = 2e-5
MAX_LEN    = 128

MODEL_SAVE_PATH = 'models/best_model.pth'
LABEL_NAMES = ['非谣言', '谣言']
```

可根据需要调整 (\[GitHub]\[8])。

---

## 目录结构

```
RumorDetection/
├─ app.py                       # Flask 服务入口 :contentReference[oaicite:17]{index=17}
├─ inference.py                 # 混合推理脚本 :contentReference[oaicite:18]{index=18}
├─ train.py                     # 模型训练脚本 :contentReference[oaicite:19]{index=19}
├─ data_loader.py               # PyTorch Dataset & DataLoader :contentReference[oaicite:20]{index=20}
├─ docx_to_csv_and_split.py     # 文档转 CSV & 划分 :contentReference[oaicite:21]{index=21}
├─ ocr_engine.py                # OCR 文本提取模块
├─ utils.py                     # 工具函数 :contentReference[oaicite:23]{index=23}
├─ config.py                    # 全局配置 :contentReference[oaicite:22]{index=22}
├─ model/
│   └─ BertRumorDetector.py     # BERT 模型定义 :contentReference[oaicite:24]{index=24}
├─ static/                      # 静态资源目录（前端页面、图片等）
├─ templates/index.html         # 前端页面 :contentReference[oaicite:25]{index=25}
└─ data/                        # 数据目录：train/valid/test CSV
```

---

## 许可与贡献

欢迎 Fork & PR，如有问题请在 Issues 中提出。
