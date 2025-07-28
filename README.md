# PDF内容提取工具

该工具实现了半自动方案：人工选区 + 自动内容识别，用于从PDF文件中提取文章内容（包括标题、副标题、正文等）。

## 项目结构

- `main.py`: 主程序入口
- `main_web.py`: Web后端主程序
- `gui_selector.py`: GUI界面模块，用于PDF预览和人工选区
- `content_analyzer.py`: 内容分析器模块，用于自动识别标题、副标题和正文
- `requirements.txt`: 项目依赖文件
- `requirements-web.txt`: Web版本依赖文件
- `frontend/`: Web前端文件目录

## 安装依赖

```bash
pip install -r requirements.txt
```

## 使用方法

### 桌面版本

1. 运行主程序：
   ```bash
   python main.py
   ```
   或者在Windows系统中双击运行 `run.bat` 文件

2. 在GUI界面中打开PDF文件

3. 使用鼠标在PDF页面上选择需要提取的区域

4. 点击"提取选定区域内容"按钮

5. 程序将自动识别选区内容中的标题、副标题和正文

### Web版本

1. 安装Web版本依赖：
   ```bash
   pip install -r requirements-web.txt
   ```

2. 运行Web后端程序：
   ```bash
   python main_web.py
   ```

3. 在浏览器中访问 `http://localhost:8000` 

4. 上传PDF文件并在页面上选择需要提取的区域

5. 点击"提取选定区域内容"按钮

6. 程序将自动识别选区内容中的标题、副标题和正文

## 技术方案

### 半自动方案

1. **人工选区**：用户通过GUI界面用鼠标选择PDF中需要提取的区域
2. **自动内容识别**：程序自动分析选区内容，识别其中的标题、副标题和正文

### 核心技术

- **PDF处理**：使用PyMuPDF库提取PDF内容
- **GUI界面**：使用PyQt5实现PDF预览和区域选择功能
- **内容识别**：使用正则表达式和启发式规则识别标题、副标题和正文

## 后续优化建议

1. 添加OCR功能以处理扫描版PDF
2. 实现更智能的内容识别算法（如机器学习模型）
3. 支持批量处理多个PDF文件
4. 添加导出功能（如导出为Word、Markdown格式）