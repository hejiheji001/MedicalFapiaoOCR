---
name: medical-fapiao-ocr
description: 从医疗费用PDF中自动提取发票信息（发票号码、金额、页码），生成Excel汇总表。支持电子票据文本提取和扫描件OCR识别，并行处理，自动分组。
---

# Medical Fapiao OCR - 医疗费发票OCR汇总工具

从就医记录PDF中提取所有医疗费发票，按发票号码汇总金额和页码，输出Excel汇总表。

## 适用场景

- 医疗费用报销汇总
- 就医记录PDF中包含多张收费票据（住院/门诊）和收费明细
- PDF可能是裁切过的（不从第1页开始），输出使用PDF实际页码
- 票据和明细可能不连续排列（人工拼接PDF导致）

## 工作流程

### Step 1: 文本提取（有文本层的页面）

使用 PyMuPDF 直接提取嵌入文本的页面（电子票据），识别：
- 票据号码（`票据号码：XXXXXXXXXX`）
- 金额（`（小写）XXX.XX`）
- 页面类型（RECEIPT=收费票据 / DETAIL=收费明细）

### Step 2: 并行OCR（扫描页面）

对纯扫描图片页面，使用 RapidOCR 进行中文OCR：
- 300 DPI RGB模式提取页面图片
- ProcessPoolExecutor 并行处理（默认4 workers，避免资源耗尽）
- 每个worker内OCR引擎只初始化一次（batch模式）

### Step 3: 合并分组

- 按PDF实际页码排序所有结果
- 修复缺失的发票号码（明细页从前序页面继承）
- 按发票号码分组，汇总金额和页码

### Step 4: 生成页码范围

- 单页 → `16`
- 连续页 → `1-15`
- 不连续页（人工拼接错误） → `24-25,34-35`

### Step 5: 输出Excel

3列对布局（金额 | 页码 × 3组），标题行显示总金额，格式参照模板。

## 使用方法

```bash
# 基本用法
python run.py <PDF路径>

# 指定输出目录
python run.py <PDF路径> -o <输出目录>

# 调整并行度
python run.py <PDF路径> -w 6

# 完整参数
python run.py 就医记录.pdf -o ./output -w 4 --dpi 300
```

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pdf_path` | (必填) | 输入PDF文件路径 |
| `-o, --output` | PDF所在目录 | 输出目录 |
| `-w, --workers` | 4 | 并行OCR worker数（建议不超过CPU物理核心数的一半） |
| `--dpi` | 300 | 扫描页面提取DPI |
| `--save-raw` | False | 保存原始OCR数据到raw_data.json |

### 输出文件

- `医疗费汇总.xlsx` — Excel汇总表
- `raw_data.json` — (可选) 每页的原始提取数据

## 依赖

```
PyMuPDF>=1.23.0
rapidocr-onnxruntime>=1.3.0
Pillow>=10.0.0
numpy>=1.24.0
openpyxl>=3.1.0
```

安装：`pip install -r requirements.txt`

## 发票识别规则

### 票据页 (RECEIPT)
- 关键词：`收费票据`
- 金额提取：`（小写）XXX.XX` 或 `(小写) XXX.XX`
- 发票号码：`票据号码：XXXXXXXXXX`

### 明细页 (DETAIL)
- 关键词：`收费明细`、`明细`（排除`含明细`噪声）
- 或包含 `小计`/`合计`
- 发票号码：`所属电子票据号码：XXXXXXXXXX` 或从前序页面继承

### 发票号码关联
- OCR结果中 `所属` 或 `所` 后跟8位以上数字 → 关联发票号
- 明细页/未知页缺失发票号时，向前查找最近的已知发票号

## 已知限制

- OCR对低质量扫描件可能识别不准确（建议300 DPI以上）
- `MAX_WORKERS` 设置过高可能导致内存耗尽（14核曾导致崩溃，4核稳定）
- 仅支持中文医疗收费票据格式（贵州省/浙江省门诊/住院票据）
