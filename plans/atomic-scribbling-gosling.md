# Plan: Jianpu PDF → Keyboard Score Converter

## Context

用户需要一个 Python 程序，输入 PDF 简谱图片（含主旋律和伴奏），自动生成对应的键盘谱（用字母表示的谱子）。

**键位映射规则：**
- 中音 1-7 → ASDFGHJ
- 高音 1-7 → QWERTYU
- 低音 1-7 → ZXCVBNM
- 超出范围的音（倍高/倍低）：通过八度等价折叠到可用范围内

## Architecture

单文件程序 `jianpu2keyboard.py`，分为 5 个模块化部分：

### 1. PDF 处理模块
- 使用 PyMuPDF (fitz) 将 PDF 每页渲染为高清图像
- 支持多页 PDF，逐页处理

### 2. 图像预处理模块
- 灰度化 + 二值化（Otsu 自适应阈值）
- 降噪（ morphology open/close ）
- 水平投影分析 → 检测简谱的每一"行"

### 3. 简谱识别模块 (核心)
采用 **混合策略**，按优先级回退：

**方案 A（推荐，准确率最高）：PaddleOCR**
- 对每行进行 OCR，检测数字 0-7 及其位置
- 通过数字上/下方的像素密度检测八度点（高音点/低音点）
- 解析下划线（减时线）、后划线（增时线）、附点等节奏符号

**方案 B（轻量备选）：Tesseract + OpenCV 模板匹配**
- pytesseract 检测数字
- OpenCV 轮廓分析检测八度点和节奏符号

**方案 C（手动回退）：纯 OpenCV 模板匹配**
- 预生成数字 0-7 的模板，使用 matchTemplate 匹配
- 仅依赖 numpy + opencv，无需额外 OCR 引擎

识别输出为结构化的音符列表：`[(音高1-7, 八度层级, 时值), ...]`

### 4. 音符映射模块
```python
KEYBOARD_MAP = {
    # (octave, pitch) → key
    ( 1, 1): 'Q', ( 1, 2): 'W', ( 1, 3): 'E', ( 1, 4): 'R', ( 1, 5): 'T', ( 1, 6): 'Y', ( 1, 7): 'U',
    ( 0, 1): 'A', ( 0, 2): 'S', ( 0, 3): 'D', ( 0, 4): 'F', ( 0, 5): 'G', ( 0, 6): 'H', ( 0, 7): 'J',
    (-1, 1): 'Z', (-1, 2): 'X', (-1, 3): 'C', (-1, 4): 'V', (-1, 5): 'B', (-1, 6): 'N', (-1, 7): 'M',
}
```
- 八度折叠：octave > 1 → 折叠到 1（高音）；octave < -1 → 折叠到 -1（低音）
- 主旋律和伴奏分行输出，伴奏用括号或缩进标注

### 5. 输出格式化模块
- 每行键盘谱按小节分组显示
- 支持纯文本和 JSON 两种输出格式
- 可选输出时值信息（用下划线/空格表示节奏）

## Files to Create

| 文件 | 说明 |
|------|------|
| `C:\Users\Blackmai\.local\bin\jianpu2keyboard.py` | 主程序（单文件，约 600-800 行） |
| `C:\Users\Blackmai\.local\bin\requirements_jianpu.txt` | 依赖列表 |

## Dependencies

```
PyMuPDF>=1.23.0        # PDF 渲染
opencv-python>=4.8.0   # 图像处理
numpy>=1.24.0          # 数组运算
paddlepaddle>=3.0.0    # OCR 引擎（方案 A）
paddleocr>=2.7.0       # OCR 封装（方案 A）
pytesseract>=0.3.10    # OCR 备选（方案 B）
Pillow>=10.0.0         # 图像 I/O
```

## Verification

1. 准备一张简谱 PDF 测试文件
2. 运行 `python jianpu2keyboard.py test.pdf -o output.txt`
3. 检查输出的键盘字母序列是否正确对应简谱中的数字和八度
4. 测试倍高音/倍低音的八度折叠是否正确
5. 测试多页 PDF 的处理
