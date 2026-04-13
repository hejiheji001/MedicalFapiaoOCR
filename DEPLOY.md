# DEPLOY.md — 部署与故障排查指南

> 本文档面向 AI Agent 阅读。当用户要求安装或使用 MedicalFapiaoOCR 时，请严格按照以下步骤执行。
> 遇到任何错误，在本文档"故障排查"部分查找对应的解决方案后自行处理，无需询问用户。

---

## 一、环境要求检查

按顺序检查以下环境，**全部通过后**再进入安装步骤。

### 1. Node.js

```bash
node --version
```

- 通过条件：输出 `v18.x.x` 或更高版本
- 未通过：见故障排查 → [E-001]

### 2. Python

```bash
python --version
```

- 通过条件：输出 `Python 3.9.x` 或更高版本
- 未通过：见故障排查 → [E-002]
- 注意：某些系统上命令是 `python3`，两个都试

### 3. pip

```bash
pip --version
```

- 通过条件：有输出且对应 Python 3.9+
- 未通过：见故障排查 → [E-003]

---

## 二、安装步骤

按顺序执行，每一步确认成功后再继续下一步。

### Step 1：安装 Skill

```bash
npx skills add hejiheji001/MedicalFapiaoOCR -g -y
```

- 成功标志：输出包含 `Installed 1 skill` 和 `Done!`
- 失败：见故障排查 → [E-004]

### Step 2：定位 Skill 目录

Skill 安装后的路径因操作系统而异：

| 系统 | 路径 |
|------|------|
| Windows | `%USERPROFILE%\.agents\skills\medical-fapiao-ocr\` |
| macOS / Linux | `~/.agents/skills/medical-fapiao-ocr/` |

验证：确认该目录下存在 `run.py` 和 `requirements.txt`。

### Step 3：安装 Python 依赖

```bash
pip install -r <skill目录>/requirements.txt
```

- 成功标志：所有包安装完成，无报错
- 失败：见故障排查 → [E-005]

### Step 4：验证安装

```bash
python <skill目录>/run.py --help
```

- 成功标志：输出参数帮助信息
- 失败：见故障排查 → [E-006]

---

## 三、使用方法

### 基本调用

```bash
python <skill目录>/run.py "<PDF文件路径>"
```

Excel 输出到 PDF 所在目录。

### 指定输出目录

```bash
python <skill目录>/run.py "<PDF文件路径>" -o "<输出目录>"
```

### 调参

| 参数 | 默认值 | 用途 |
|------|--------|------|
| `-w, --workers` | 4 | OCR 并行数。内存不足时降到 2 |
| `--dpi` | 300 | 扫描精度。识别不准时提高到 400 |
| `--save-raw` | 关 | 保存每页原始数据到 raw_data.json |

### 正常输出示例

```
DONE — 261104.53 元 (27 invoices)
```

---

## 四、故障排查

遇到错误时，按错误代码查找解决方案，**自行执行修复后重试**。

### [E-001] Node.js 未安装

**症状**：`node` 命令不存在

**修复**（按操作系统选择）：

```bash
# Windows (winget)
winget install OpenJS.NodeJS.LTS

# Windows (choco)
choco install nodejs-lts -y

# macOS
brew install node

# Ubuntu / Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

修复后重新验证 `node --version`。

### [E-002] Python 未安装或版本过低

**症状**：`python` 命令不存在，或版本低于 3.9

**修复**：

```bash
# Windows (winget)
winget install Python.Python.3.12

# macOS
brew install python@3.12

# Ubuntu / Debian
sudo apt-get install -y python3 python3-pip
```

Windows 注意：安装时必须勾选 "Add Python to PATH"。如果已经装了但命令找不到，尝试 `python3` 或检查 PATH 环境变量。

### [E-003] pip 不可用

**症状**：`pip` 命令不存在

**修复**：

```bash
python -m ensurepip --upgrade
python -m pip install --upgrade pip
```

如果 `python` 不行就换 `python3`。

### [E-004] Skill 安装失败

**症状**：`npx skills add` 报错

**可能原因与修复**：

| 原因 | 修复 |
|------|------|
| 网络问题 | 重试，或设置代理 `npm config set proxy http://...` |
| npx 缓存损坏 | `npx clear-npx-cache` 后重试 |
| 权限不足 | Windows：以管理员运行终端；macOS/Linux：加 `sudo` |

### [E-005] Python 依赖安装失败

**症状**：`pip install -r requirements.txt` 报错

**常见情况**：

| 错误信息 | 修复 |
|----------|------|
| `error: Microsoft Visual C++ 14.0 or greater is required` | 安装 [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)，勾选 "C++ build tools" |
| `Could not find a version that satisfies` | `pip install --upgrade pip` 后重试 |
| `Permission denied` | 加 `--user` 参数：`pip install --user -r requirements.txt` |
| 网络超时 | 换源：`pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple` |

### [E-006] run.py 无法执行

**症状**：`python run.py --help` 报 ImportError

**修复**：依赖没装全。重新执行 Step 3。如果仍然失败，逐个安装：

```bash
pip install PyMuPDF rapidocr-onnxruntime Pillow numpy openpyxl
```

### [E-007] OCR 运行时内存耗尽 / 进程崩溃

**症状**：运行中途卡死或报 `MemoryError`

**修复**：降低并行数

```bash
python <skill目录>/run.py "<PDF路径>" -w 2
```

如果 `-w 2` 仍然崩溃，用 `-w 1` 单线程运行。

### [E-008] UnicodeEncodeError (Windows)

**症状**：`UnicodeEncodeError: 'charmap' codec can't encode characters`

**修复**：设置环境变量后运行

```bash
# PowerShell
$env:PYTHONIOENCODING="utf-8"; python <skill目录>/run.py "<PDF路径>"

# CMD
set PYTHONIOENCODING=utf-8 && python <skill目录>/run.py "<PDF路径>"
```

### [E-009] OCR 识别不准确 / 金额错误

**症状**：输出的金额与实际不符

**修复**：提高渲染精度

```bash
python <skill目录>/run.py "<PDF路径>" --dpi 400
```

如果仍不准确，使用 `--save-raw` 导出原始数据，逐页检查 `raw_data.json` 中的识别结果，手动核对有问题的页面。

### [E-010] 输出 0 张发票

**症状**：`DONE — 0.00 元 (0 invoices)`

**可能原因**：

1. PDF 不包含中文医疗收费票据格式 → 本工具不适用
2. PDF 加密或有权限限制 → 先解密 PDF
3. 全部页面为扫描件且 OCR 全部失败 → 用 `--save-raw` 检查，确认 `--dpi 300` 以上

---

## 五、给 AI Agent 的执行清单

当用户说"帮我安装"或"帮我部署"时，执行以下清单：

```
1. 检查 node --version        → 不通过则修复 [E-001]
2. 检查 python --version      → 不通过则修复 [E-002]
3. 检查 pip --version         → 不通过则修复 [E-003]
4. npx skills add hejiheji001/MedicalFapiaoOCR -g -y    → 失败则查 [E-004]
5. pip install -r <skill目录>/requirements.txt           → 失败则查 [E-005]
6. python <skill目录>/run.py --help                      → 失败则查 [E-006]
7. 告知用户："安装完成，以后直接告诉我 PDF 路径即可。"
```

当用户说"帮我汇总发票"时：

```
1. 确认 PDF 路径存在
2. python <skill目录>/run.py "<PDF路径>"
3. 如果报错，按错误代码查本文档修复后重试
4. 成功后告知用户：发票数量、总金额、Excel 文件位置
```
