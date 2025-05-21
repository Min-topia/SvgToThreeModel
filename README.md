# SVG to 2.5D Converter

![Project Preview](/public/preview.png) <!-- 添加项目预览图 -->

## 简介

SVG to 2.5D Converter 是一个基于 Vue 3 和 Three.js 的工具，能够将 SVG 平面图转换为具有高度的 2.5D 墙体模型，并保留原始文本元素。该项目特别适用于建筑平面图的可视化展示。

## 功能特性

- 🏗️ 将 SVG 路径转换为具有可调节高度的 3D 墙体
- 🎨 实时调整墙体高度和颜色
- ✏️ 保留 SVG 中的文本元素并支持多种控制选项
- 🖱️ 交互式 3D 场景，支持旋转、缩放和平移
- ⚙️ 直观的 GUI 控制面板
- 📱 响应式设计，适配不同屏幕尺寸

## 技术栈

- [Vue 3](https://vuejs.org/) - 前端框架
- [Three.js](https://threejs.org/) - 3D 渲染引擎
- [Troika Three Text](https://github.com/protectwise/troika/tree/main/packages/troika-three-text) - Three.js 文本渲染
- [dat.GUI](https://github.com/dataarts/dat.gui) - 轻量级 GUI 控制面板
- [Vite](https://vitejs.dev/) - 前端构建工具

## 安装与使用

### 项目安装

```bash
# 克隆仓库
git clone https://github.com/Min-topia/SvgToThreeModel.git
cd SvgToThreeModel

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### 构建生产版本

```bash
npm run build
```

### 预览生产版本

```bash
npm run preview
```

## 使用方法

1. 将您的 SVG 平面图文件命名为 `floorplan.svg` 并放置在 `public` 目录下
2. 启动项目后，系统会自动加载并解析 SVG 文件
3. 使用右侧控制面板调整参数：
   - **墙体设置**：调整高度和颜色
   - **文本设置**：控制文本大小、位置、颜色等属性

## 项目结构

```
handlesvgto2-5d/
├── public/                  # 静态资源
│   └── floorplan.svg        # SVG 平面图文件
├── src/
│   ├── App.vue              # 主组件
│   └── main.js              # 应用入口
├── package.json             # 项目配置
└── README.md                # 项目文档
```

## 自定义配置

您可以通过修改 `src/App.vue` 中的以下配置来自定义应用行为：

```javascript
const config = {
  wallHeight: 50, // 默认墙体高度
  wallColor: "#888888", // 默认墙体颜色
  textScale: 0.5, // 默认文本缩放
  // 更多配置...
};
```

## 贡献指南

欢迎贡献！请遵循以下步骤：

1. Fork 项目仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 许可证

本项目采用 [MIT License](LICENSE)。

## 致谢

- Three.js 团队提供的强大 3D 渲染库
- Vue 团队构建的优秀前端框架
- Troika 团队提供的 Three.js 文本解决方案

---

**提示**：要获得最佳体验，请确保您的 SVG 文件使用路径(`<path>`)元素定义墙体，文本使用`<text>`元素定义。
