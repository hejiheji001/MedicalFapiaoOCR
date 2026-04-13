# MedicalFapiaoOCR

从就医记录 PDF 中自动提取医疗费发票信息，按发票号码汇总金额与页码，生成 Excel 汇总表。

## 功能

- **混合提取**：电子票据直接读文本层（PyMuPDF），扫描件走 OCR（RapidOCR）
- **并行处理**：多进程 OCR，每个 worker 引擎只初始化一次
- **智能分组**：以发票号码为唯一标识，自动关联票据页与明细页
- **页码容错**：处理人工拼接 PDF 导致的不连续页码（如 `24-25,34-35`）
- **Excel 输出**：3 列对布局，标题行显示总金额

## 快速开始

```bash
pip install -r requirements.txt

# 基本用法
python run.py 就医记录.pdf

# 指定输出目录和并行度
python run.py 就医记录.pdf -o ./output -w 4

# 保存原始 OCR 数据
python run.py 就医记录.pdf --save-raw
```

## 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pdf_path` | 必填 | 输入 PDF 文件路径 |
| `-o, --output` | PDF 所在目录 | 输出目录 |
| `-w, --workers` | 4 | 并行 OCR worker 数 |
| `--dpi` | 300 | 扫描页面渲染 DPI |
| `--save-raw` | - | 保存 raw_data.json |

## 输出

- `医疗费汇总.xlsx` — 汇总表
- `raw_data.json` — 每页提取的原始数据（可选）

## 作为 Agent Skill 安装

```bash
npx skills add hejiheji001/MedicalFapiaoOCR -g -y
```

## 依赖

- Python 3.9+
- PyMuPDF, rapidocr-onnxruntime, Pillow, numpy, openpyxl

## 处理流程

```
PDF
 ├─ 有文本层的页面 → PyMuPDF 文本提取
 └─ 扫描页面 → 300 DPI RGB 渲染 → RapidOCR 并行识别
                          ↓
                 按发票号码分组汇总
                 缺失发票号从前序页继承
                          ↓
                 生成页码范围 + Excel
```

## License

MIT
