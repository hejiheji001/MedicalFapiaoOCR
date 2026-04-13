# MedicalFapiaoOCR

从就医记录 PDF 中自动提取医疗费发票信息，按发票号码汇总金额与页码，生成 Excel 汇总表。

---

## 安装

把下面这段话粘贴给你的 AI 助手（[OpenCode](https://github.com/opencode-ai/opencode)、Claude Code、Cursor 等），它会帮你完成一切：

```
安装医疗费发票汇总工具 MedicalFapiaoOCR，按照这个文档完成所有步骤：
https://raw.githubusercontent.com/hejiheji001/MedicalFapiaoOCR/refs/heads/main/DEPLOY.md
```

## 使用

告诉 AI 你的 PDF 在哪就行：

```
帮我汇总这个 PDF 里的医疗费发票：C:\Users\你的用户名\Desktop\就医记录.pdf
```

AI 会自动完成所有工作，几分钟后告诉你结果，并在 PDF 同目录下生成 `医疗费汇总.xlsx`。

你还可以这样说：

```
把桌面上的就医记录做成 Excel 汇总表
提取这个 PDF 里所有发票的金额和页码
医疗费报销汇总，文件在 D:\报销材料\就医记录.pdf
```

---

## 功能

- **混合提取**：电子票据直接读文本层（PyMuPDF），扫描件走 OCR（RapidOCR）
- **并行处理**：多进程 OCR，每个 worker 引擎只初始化一次
- **智能分组**：以发票号码为唯一标识，自动关联票据页与明细页
- **页码容错**：处理人工拼接 PDF 导致的不连续页码（如 `24-25,34-35`）
- **Excel 输出**：3 列对布局，标题行显示总金额

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

## 手动使用（面向开发者）

```bash
pip install -r requirements.txt
python run.py 就医记录.pdf
python run.py 就医记录.pdf -o ./output -w 4 --dpi 300 --save-raw
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pdf_path` | 必填 | 输入 PDF 文件路径 |
| `-o, --output` | PDF 所在目录 | 输出目录 |
| `-w, --workers` | 4 | 并行 OCR worker 数 |
| `--dpi` | 300 | 扫描页面渲染 DPI |
| `--save-raw` | - | 保存 raw_data.json |

## License

MIT
