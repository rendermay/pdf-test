# PDF内容提取工具Web化技术方案

## 概述

本文档描述了如何将现有的基于 PyQt5 的桌面应用重构为基于 Web 的应用，以实现图形用户界面和人工选区功能。Web 化将使工具更易于部署和使用，用户可以通过浏览器访问而无需安装桌面应用。

## 技术架构

### 前端技术
- **HTML5/CSS3/JavaScript**: 基础前端技术
- **Canvas API**: 用于显示PDF页面和实现人工选区功能
- **PDF.js**: Mozilla 开发的用于在浏览器中渲染PDF的库
- **Bootstrap 5**: 用于响应式设计和UI组件

### 后端技术
- **FastAPI**: 现代、快速（高性能）的Web框架，用于构建API
- **PyMuPDF (fitz)**: 用于PDF处理
- **Uvicorn**: 用于运行FastAPI应用的ASGI服务器

### 通信协议
- **WebSocket**: 用于实时通信（如PDF页面渲染进度）
- **RESTful API**: 用于文件上传、内容提取等操作

## 功能模块设计

### 1. PDF上传模块
- 用户通过Web界面上传PDF文件
- 后端接收并存储上传的文件
- 返回文件ID用于后续操作

### 2. PDF渲染模块
- 使用PDF.js在前端Canvas中渲染PDF页面
- 支持页面导航（上一页/下一页）
- 支持缩放功能

### 3. 人工选区模块
- 在前端Canvas上实现鼠标拖拽选择区域
- 实时显示选择区域的边框
- 将选择区域坐标发送到后端

### 4. 内容提取模块
- 后端接收选择区域坐标
- 使用PyMuPDF提取指定区域的文本内容
- 返回提取的文本内容

### 5. 内容分析模块
- 复用现有的内容分析器（content_analyzer.py）
- 识别标题、副标题和正文
- 返回结构化的内容分析结果

## 项目结构

```
PDF-test/
├── backend/
│   ├── main.py              # FastAPI应用入口
│   ├── pdf_processor.py     # PDF处理模块
│   └── content_analyzer.py  # 内容分析器（复用现有）
├── frontend/
│   ├── index.html           # 主页面
│   ├── style.css            # 样式文件
│   ├── script.js            # 前端逻辑
│   └── pdf.js               # PDF.js库
├── uploads/                 # 上传文件存储目录
├── requirements.txt         # 项目依赖
└── README.md
```

## 核心实现细节

### 前端实现

1. **PDF渲染**:
   ```javascript
   // 使用PDF.js渲染PDF页面
   const pdfjsLib = window['pdfjs-dist/build/pdf'];
   pdfjsLib.getDocument('uploads/sample.pdf').promise.then(pdf => {
       // 渲染第一页
       pdf.getPage(1).then(page => {
           const canvas = document.getElementById('pdf-canvas');
           const context = canvas.getContext('2d');
           const viewport = page.getViewport({ scale: 1.5 });
           
           canvas.height = viewport.height;
           canvas.width = viewport.width;
           
           const renderContext = {
               canvasContext: context,
               viewport: viewport
           };
           page.render(renderContext);
       });
   });
   ```

2. **人工选区**:
   ```javascript
   // Canvas鼠标事件处理
   let isDrawing = false;
   let startX, startY;
   
   canvas.addEventListener('mousedown', e => {
       isDrawing = true;
       startX = e.offsetX;
       startY = e.offsetY;
   });
   
   canvas.addEventListener('mousemove', e => {
       if (!isDrawing) return;
       
       // 清除之前的绘制内容
       context.clearRect(0, 0, canvas.width, canvas.height);
       
       // 重新渲染PDF页面
       page.render(renderContext);
       
       // 绘制选择区域
       const width = e.offsetX - startX;
       const height = e.offsetY - startY;
       context.strokeStyle = 'red';
       context.lineWidth = 2;
       context.strokeRect(startX, startY, width, height);
   });
   
   canvas.addEventListener('mouseup', e => {
       isDrawing = false;
       // 发送选区坐标到后端
       const endX = e.offsetX;
       const endY = e.offsetY;
       sendSelection(startX, startY, endX, endY);
   });
   ```

### 后端实现

1. **FastAPI应用**:
   ```python
   from fastapi import FastAPI, File, UploadFile
   from fastapi.staticfiles import StaticFiles
   import fitz
   
   app = FastAPI()
   
   # 挂载静态文件目录
   app.mount("/frontend", StaticFiles(directory="frontend"), name="frontend")
   app.mount("/uploads", StaticFiles(directory="uploads"), name="uploads")
   
   @app.post("/upload/")
   async def upload_pdf(file: UploadFile = File(...)):
       # 保存上传的文件
       with open(f"uploads/{file.filename}", "wb") as buffer:
           buffer.write(await file.read())
       return {"filename": file.filename}
   
   @app.post("/extract/")
   async def extract_content(data: dict):
       # 提取指定区域的内容
       pdf_path = data["pdf_path"]
       page_num = data["page_num"]
       coords = data["coords"]  # [x0, y0, x1, y1]
       
       doc = fitz.open(pdf_path)
       page = doc[page_num]
       rect = fitz.Rect(coords)
       text = page.get_text(clip=rect)
       
       return {"content": text}
   ```

2. **内容分析**:
   ```python
   # 复用现有的内容分析器
   from content_analyzer import analyze_content, identify_document_structure
   
   @app.post("/analyze/")
   async def analyze_text(data: dict):
       text = data["text"]
       
       # 分析内容
       analysis_result = analyze_content(text)
       structure = identify_document_structure(text)
       
       return {
           "analysis": analysis_result,
           "structure": structure
       }
   ```

## 部署方案

### 开发环境
1. 安装依赖：
   ```bash
   pip install -r requirements.txt
   ```
2. 启动应用：
   ```bash
   uvicorn backend.main:app --reload
   ```
3. 访问应用：http://localhost:8000/frontend/index.html