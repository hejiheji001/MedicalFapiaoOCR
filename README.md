# MedicalFapiaoOCR

从就医记录 PDF 中自动提取医疗费发票信息，按发票号码汇总金额与页码，生成 Excel 汇总表。

---

## 零基础使用（全程对话，无需碰命令行）

> 你不需要懂编程，不需要打开终端，不需要输入任何命令。
> 只需要一个 AI 编程助手（如 [OpenCode](https://github.com/opencode-ai/opencode)、Claude Code、Cursor 等），全程用自然语言对话即可。

### 你只需要做一件事：把下面这段话复制给 AI

**首次使用**（只需说一次，AI 会帮你全部装好）：

```
帮我安装一个医疗费发票汇总工具。
请执行：npx skills add hejiheji001/MedicalFapiaoOCR -g -y
然后安装它的 Python 依赖：pip install -r requirements.txt
（requirements.txt 在 skill 安装目录里，通常是 ~/.agents/skills/medical-fapiao-ocr/）
```

**之后每次使用**，只需要告诉 AI 你的 PDF 在哪：

```
帮我汇总这个 PDF 里的医疗费发票：C:\Users\你的用户名\Desktop\就医记录.pdf
```

AI 会自动完成所有工作，几分钟后告诉你结果，并在 PDF 同目录下生成 `医疗费汇总.xlsx`。

### 你还可以这样说

```
把桌面上的就医记录做成 Excel 汇总表
提取这个 PDF 里所有发票的金额和页码
医疗费报销汇总，文件在 D:\报销材料\就医记录.pdf
```

### 常见问题

| 问题 | 告诉 AI |
|------|---------|
| AI 说找不到 npx | "帮我安装 Node.js" |
| AI 说找不到 python | "帮我安装 Python" |
| OCR 速度很慢 | 正常现象，50 页扫描件大约需要 5 分钟 |
| 金额识别有误 | "用 --dpi 400 重新跑一次" |

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
