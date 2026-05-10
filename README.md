# 手写模拟器 Pro

<p align="center">
  <b>纯前端手写模拟器</b> | <b>零后端依赖</b> | <b>打开浏览器就能用</b>
</p>

<p align="center">
  <a href="#在线体验">在线体验</a> •
  <a href="#功能特性">功能特性</a> •
  <a href="#快速开始">快速开始</a> •
  <a href="#技术架构">技术架构</a> •
  <a href="#自定义扩展">自定义扩展</a>
</p>

---

## 在线体验

👉 **[点击使用](https://water-lrx.github.io/handwriting-simulator)**

无需注册、无需安装，打开链接直接输入文字，即可生成逼真的手写稿。

---

## 功能特性

| 类别 | 功能 |
|------|------|
| **字体** | 4 款内置手写字体：云烟体、华阳手写体、李国夫手写体、青叶手写体 |
| **纸张** | 7 种纸张背景：A4 本子、A4 纯白、条纸、格子纸、牛皮纸等 |
| **预设** | 6 种场景一键切换：工整、自然、潦草、作业、信纸、笔记 |
| **墨迹** | 颜色浓淡变化、文字粗细、随机偏移、旋转、大小变化，模拟真实手写痕迹 |
| **预览** | 输入文字即时看到手写效果，实时渲染不卡顿 |
| **导出** | PNG 单页高清图 / PDF 多页文档 |
| **配置** | 偏好设置自动保存到浏览器本地存储 |

---

## 快速开始

### 方式一：在线使用（推荐）

直接访问 [https://water-lrx.github.io/handwriting-simulator](https://water-lrx.github.io/handwriting-simulator)，输入文字即可生成手写稿。

### 方式二：本地打开

```bash
git clone https://github.com/water-lrx/handwriting-simulator.git
cd handwriting-simulator
# 方式 A：直接双击 index.html
# 方式 B：本地服务器（字体加载更稳定）
npx serve .
# 或
python -m http.server 8080
```

### 方式三：桌面端（Python）

如果你需要更高性能的本地渲染，或处理超长文本，可以使用 Python 桌面端：

```bash
cd desktop-version  # 项目根目录
cd gui_tkinter      # 或 cd gui（PyQt6 版本）
python main_tkinter.py
```

桌面端特性：
- 多线程后台渲染，界面永不卡顿
- 预览与导出分离（预览用低 DPI，导出用高 DPI）
- 支持自定义字体和背景批量导入

### 方式四：后端 API（Docker）

需要集成到其他系统？我们提供了基于 [handright](https://github.com/Gsllchb/Handright) 的高性能后端 API：

```bash
cd handright-backend
docker-compose up -d
# 访问 http://localhost:8000/docs 查看 API 文档
```

---

## 技术架构

本项目采用**三层架构**设计，覆盖从个人本地使用到线上分享的全场景需求：

```
┌─────────────────────────────────────────────────────────────┐
│                        手写模拟器 Pro                         │
├─────────────────┬─────────────────┬─────────────────────────┤
│   网页端         │   桌面端         │      后端 API           │
│  (GitHub Pages) │  (Python + PIL) │   (FastAPI + handright) │
├─────────────────┼─────────────────┼─────────────────────────┤
│ HTML5 Canvas 2D │ PIL/Pillow      │ handright 引擎          │
│ pdf-lib.js      │ numpy           │ PyMuPDF                 │
│ FontFace API    │ PyQt6 / tkinter │ Docker                  │
├─────────────────┼─────────────────┼─────────────────────────┤
│ 零后端 · 即开即用 │ 高性能本地渲染   │ 两层扰动 · 更自然       │
│ 适合分享给小白的 │ 适合大批量处理   │ 适合系统集成            │
└─────────────────┴─────────────────┴─────────────────────────┘
```

### 网页端渲染管线

```
用户输入文字
    │
    ▼
FontFace 字体加载 ──→ document.fonts.ready 等待就绪
    │
    ▼
Canvas measureText() ──→ 自动换行计算
    │
    ▼
requestAnimationFrame 帧渲染 ──→ 逐行绘制字符
    │                                    │
    │    ┌───────────────────────────────┘
    │    ▼
    │  每字符：随机颜色 + 描边粗细 + 偏移 + 旋转 + 缩放
    │    │
    ▼    ▼
Canvas 2D 合成 ──→ PNG / PDF 导出
```

**关键优化点：**

1. **渲染与 UI 分离**：预览使用 `requestAnimationFrame` 逐帧渲染（每帧 3 行），避免同步绘制阻塞 UI 线程
2. **世代计数器**：`renderGeneration` 变量解决快速切换时的帧重叠问题——旧帧检测到世代变化自动终止
3. **字体加载同步**：通过 `document.fonts.ready` 确保字体完全加载后再进行文字测量和绘制，避免字体回退
4. **种子随机**：`seededRandom(LCG)` 保证同一设置下输出确定，便于微调参数后重新生成

### 桌面端渲染管线

```
用户输入
    │
    ▼
250ms 防抖定时器 ──→ 取消旧渲染任务
    │
    ▼
QThreadPool / threading ──→ 后台线程渲染
    │
    ▼
预览：低 DPI (120) + 尺寸限制 ──→ 快速反馈
导出：高 DPI (200~600) + 完整尺寸 ──→ 高质量输出
```

**关键优化点：**

1. **三级缓存**：
   - `_FontCache`：`(path, size)` → `ImageFont.FreeTypeFont`，避免重复加载 TTF
   - `_BgCache`：`(path, size)` → `Image.Image`，背景图按尺寸缓存
   - `_char_cache`：`(char, font, color, stroke)` → `Image.Image`，单字符透明图层缓存
2. **换行快速路径**：先用 `font_size * 0.6` 估算宽度，仅当估算值接近边界时才调用昂贵的 `font.getbbox()`
3. **预览与导出参数分离**：预览时自动降低 DPI 和画布尺寸，导出时使用用户设定的完整参数

---

## 自定义扩展

### 添加自定义字体

1. 将 `.ttf` 字体文件放入 `字体/` 文件夹
2. 在 `index.html` 的 `<style>` 区域添加：
   ```css
   @font-face { font-family: 'F5'; src: url('字体/你的字体.ttf') format('truetype'); }
   ```
3. 在 `<script>` 的 `BUILT_IN_FONTS` 数组中添加：
   ```js
   { name: '你的字体名', family: 'F5' }
   ```

### 添加自定义背景

1. 将 `.jpg/.png` 图片放入 `背景/` 文件夹
2. 在 `BUILT_IN_BACKGROUNDS` 数组中添加路径

### 调整预设参数

在 `PRESETS` 对象中新增或修改预设：

```js
const PRESETS = {
  你的预设: { fontSize: 40, lineSpacing: 1.8, fontWeight: 1.5, /* ... */ },
};
```

---

## 部署方式

### GitHub Pages（免费）

1. Fork 或创建新仓库，上传 `deploy-ready/` 内所有文件
2. 进入仓库 **Settings → Pages**
3. **Source** 选择 **GitHub Actions**
4. 等待部署完成，访问 `https://你的用户名.github.io/仓库名`

> 项目已内置 `.github/workflows/deploy.yml`，push 到 `main` 分支即自动部署。

### Vercel（免费）

1. 访问 [vercel.com](https://vercel.com) 用 GitHub 账号登录
2. 点击 "Add New Project"
3. 导入本仓库
4. Framework Preset 选 **Other**，Build Command 留空，Output Directory 填 `.`
5. 点击 Deploy

### Netlify（免费）

1. 访问 [netlify.com](https://netlify.com) 用 GitHub 账号登录
2. 点击 "Add new site" → "Import an existing project"
3. 选择本仓库，直接部署

---

## 项目结构

```
handwriter/
├── core/
│   └── renderer.py              # PIL 渲染引擎核心
├── gui/
│   └── main_window.py           # PyQt6 桌面 GUI
├── gui_tkinter/
│   └── main_window.py           # tkinter 桌面 GUI（零额外依赖）
├── main.py                      # PyQt6 入口
├── main_tkinter.py             # tkinter 入口
├── requirements.txt            # Python 依赖
├── web/
│   └── index.html              # 网页版源码（开发版）
└── release/
    ├── deploy-ready/           # GitHub Pages 部署包
    │   ├── index.html          # 网页版（生产构建）
    │   ├── 字体/               # 4 款手写字体
    │   ├── 背景/               # 7 种纸张背景
    │   ├── README.md           # ← 本文档
    │   └── .github/workflows/
    │       └── deploy.yml      # 自动部署配置
    ├── handright-backend/      # FastAPI 后端
    │   ├── app/main.py         # API 服务
    │   ├── Dockerfile
    │   └── docker-compose.yml
    └── 部署指南.md
```

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 网页渲染 | HTML5 Canvas 2D, `requestAnimationFrame`, `FontFace` API |
| 网页导出 | [pdf-lib](https://pdf-lib.js.org/) 浏览器端 PDF 生成 |
| 桌面渲染 | PIL/Pillow, numpy |
| 桌面 GUI | PyQt6（完整功能）/ tkinter（零依赖） |
| 后端引擎 | [handright](https://github.com/Gsllchb/Handright)（两层扰动） |
| 后端框架 | FastAPI, PyMuPDF |
| 部署 | GitHub Actions → GitHub Pages |

---

## License

MIT License — 可自由用于个人或商业项目。
