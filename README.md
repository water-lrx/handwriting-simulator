# 手写模拟器 Pro — 在线版

纯前端手写模拟器，无需服务器，打开浏览器就能用。将文字转换为逼真的手写稿，支持导出 PNG/PDF。

## 在线体验

👉 **[点击使用](https://你的用户名.github.io/handwriting-simulator)** （部署后替换此链接）

## 功能特性

- **4 款手写字体**：云烟体、华阳手写体、李国夫手写体、青叶手写体
- **7 种纸张背景**：A4本子、A4纯白、条纸、格子纸、牛皮纸等
- **6 种场景预设**：工整、自然、潦草、作业、信纸、笔记
- **墨迹效果**：颜色浓淡变化、随机偏移、旋转、大小变化
- **实时预览**：输入文字即时看到手写效果
- **导出格式**：PNG 单页 / PDF 多页
- **配置保存**：保存偏好设置到浏览器

## 本地使用

```bash
# 方式一：直接打开
double-click index.html

# 方式二：本地服务器（推荐，字体加载更稳定）
npx serve .
# 或
python -m http.server 8080
```

## 部署方式

### 方式一：GitHub Pages（免费）

1. Fork 或创建新仓库，上传本项目文件
2. 进入仓库 Settings → Pages
3. Source 选择 **GitHub Actions**
4. 等待部署完成，访问 `https://你的用户名.github.io/仓库名`

### 方式二：Vercel（免费，推荐）

1. 访问 [vercel.com](https://vercel.com) 用 GitHub 账号登录
2. 点击 "Add New Project"
3. 导入本仓库
4. Framework Preset 选 **Other**，Build Command 留空，Output Directory 填 `.`
5. 点击 Deploy，1 分钟后即可在线访问

### 方式三：Netlify（免费）

1. 访问 [netlify.com](https://netlify.com) 用 GitHub 账号登录
2. 点击 "Add new site" → "Import an existing project"
3. 选择本仓库，直接部署

## 添加自定义字体

1. 将 `.ttf` 字体文件放入 `字体/` 文件夹
2. 在 `index.html` 的 `<style>` 区域添加：
   ```css
   @font-face { font-family: 'F5'; src: url('字体/你的字体.ttf') format('truetype'); }
   ```
3. 在 `<script>` 的 `BUILT_IN_FONTS` 数组中添加：
   ```js
   { name: '你的字体名', family: 'F5' }
   ```

## 添加自定义背景

1. 将 `.jpg/.png` 图片放入 `背景/` 文件夹
2. 在 `BUILT_IN_BACKGROUNDS` 数组中添加路径

## 技术栈

- 纯 HTML5 + CSS3 + JavaScript（零框架依赖）
- Canvas 2D 渲染引擎
- [pdf-lib](https://pdf-lib.js.org/) 浏览器端 PDF 生成
- GitHub Actions 自动部署

## License

MIT
