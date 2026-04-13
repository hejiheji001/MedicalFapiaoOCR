# MedicalFapiaoOCR

从就医记录 PDF 中自动提取医疗费发票信息，按发票号码汇总金额与页码，生成 Excel 汇总表。

---

## 零基础一键使用（通过 AI 助手）

> 不需要懂编程。只需要有 [OpenCode](https://github.com/opencode-ai/opencode) 或其他支持 Skills 的 AI 编程助手。

### 第一步：安装 Skill

在终端（命令行）中粘贴以下命令并回车：

```bash
npx skills add hejiheji001/MedicalFapiaoOCR -g -y
```

看到 `Done!` 就说明安装成功了。这是一次性操作，以后不需要再装。

### 第二步：安装 Python 依赖

如果你的电脑还没装过 Python 相关工具，请让 AI 帮你执行：

```
帮我安装 MedicalFapiaoOCR 的 Python 依赖
```

AI 会自动找到 `requirements.txt` 并运行 `pip install`。

### 第三步：把 PDF 交给 AI

打开 AI 助手，直接说：

```
帮我汇总这个PDF里的医疗费发票：C:\Users\你的用户名\Desktop\就医记录.pdf
```

把路径换成你自己的 PDF 文件位置就行。AI 会自动调用本工具，几分钟后在同目录下生成 `医疗费汇总.xlsx`。

### 常见问题

| 问题 | 解决方法 |
|------|----------|
| 提示找不到 `npx` | 先安装 [Node.js](https://nodejs.org/)（选 LTS 版本，一路下一步） |
| 提示找不到 `python` | 先安装 [Python](https://www.python.org/downloads/)（安装时勾选 "Add to PATH"） |
| OCR 速度很慢 | 正常，扫描页需要逐页识别，50 页大约 5 分钟 |
| 输出的金额不对 | 可能是扫描件质量太低，可以尝试用 `--dpi 400` 提高精度 |

---

## 功能

- **混合提取**：电子票据直接读文本层（PyMuPDF），扫描件走 OCR（RapidOCR）
- **并行处理**：多进程 OCR，每个 worker 引擎只初始化一次
- **智能分组**：以发票号码为唯一标识，自动关联票据页与明细页
- **页码容错**：处理人工拼接 PDF 导致的不连续页码（如 `24-25,34-35`）
- **Excel 输出**：3 列对布局，标题行显示总金额

## 命令行使用

```bash
pip install -r requirements.txt

# 基本用法
python run.py 就医记录.pdf

# 指定输出目录和并行度
python run.py 就医记录.pdf -o ./output -w 4

# 保存原始 OCR 数据
python run.py 就医记录.pdf --save-raw
```

### 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pdf_path` | 必填 | 输入 PDF 文件路径 |
| `-o, --output` | PDF 所在目录 | 输出目录 |
| `-w, --workers` | 4 | 并行 OCR worker 数 |
| `--dpi` | 300 | 扫描页面渲染 DPI |
| `--save-raw` | - | 保存 raw_data.json |

### 输出

- `医疗费汇总.xlsx` — 汇总表
- `raw_data.json` — 每页提取的原始数据（可选）

## 依赖

- Python 3.9+
- PyMuPDF, rapidocr-onnxruntime, Pillow, numpy, openpyxl

## AI Agent 集成指南

本工具同时是一个 [Agent Skill](https://skills.sh/)。安装后，AI 助手会自动获得以下能力：

**AI 可以理解的指令示例：**

```
帮我汇总这个PDF里的医疗费发票：/path/to/就医记录.pdf
把这个就医记录做成Excel汇总表
提取这个PDF里所有发票的金额和页码
```

**AI 执行时会：**
1. 读取 SKILL.md 了解工具的完整能力
2. 调用 `run.py` 处理 PDF
3. 返回结果摘要和 Excel 文件路径

**Skill 安装命令：**
```bash
npx skills add hejiheji001/MedicalFapiaoOCR -g -y
```

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
