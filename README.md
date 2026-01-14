# 🤖 AI 对话浏览器代理（AI Chat Browser Agent）

<div align="center">

**下一代智能浏览器自动化解决方案**

[![Vue 3](https://img.shields.io/badge/Vue-3.x-42b883?style=flat-square&logo=vue.js)](https://vuejs.org/)
[![Vite](https://img.shields.io/badge/Vite-5.x-646cff?style=flat-square&logo=vite)](https://vitejs.dev/)
[![TailwindCSS](https://img.shields.io/badge/Tailwind-3.x-38bdf8?style=flat-square&logo=tailwind-css)](https://tailwindcss.com/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
![Storage](https://img.shields.io/badge/storage-EdgeKV-cyan.svg)


[English](README.en.md) | 简体中文

</div>

---

## 🌟 项目亮点

**AI 对话浏览器代理**是一款革命性的智能浏览器自动化平台，融合了前沿的计算机视觉技术与大语言模型能力。通过自然语言交互，用户仅需用日常对话即可驱动浏览器完成复杂的自动化任务，真正实现"所说即所得"的智能化体验。

### ✨ 核心特性

🧠 **AI 视觉理解**  
基于 Qwen3 视觉大模型，深度理解网页结构和视觉元素，无需传统的 CSS 选择器或 XPath

💬 **自然语言驱动**  
用人类语言描述任务，AI 自动解析意图并执行相应操作，零编程门槛

🎯 **智能决策引擎**  
自主分析页面状态，动态调整执行策略，应对复杂交互场景

⚡ **高性能架构**  
采用 Vite + Vue 3 现代化技术栈，极速开发体验与运行性能

📊 **可视化管理**  
直观的测试用例管理与执行历史追踪，全流程透明可控

🔄 **持续学习能力**  
从历史执行中学习优化，不断提升任务成功率

---

## 🏗️ 架构设计

### 技术栈

<table>
<tr>
<td align="center"><b>前端框架</b></td>
<td>Vue 3 (Composition API) - 渐进式 JavaScript 框架</td>
</tr>
<tr>
<td align="center"><b>构建工具</b></td>
<td>Vite 5 - 新一代前端构建工具</td>
</tr>
<tr>
<td align="center"><b>样式方案</b></td>
<td>TailwindCSS 3 - 原子化 CSS 框架</td>
</tr>
<tr>
<td align="center"><b>路由管理</b></td>
<td>Vue Router 4 - 官方路由解决方案</td>
</tr>
<tr>
<td align="center"><b>HTTP 客户端</b></td>
<td>Axios - 强大的请求库</td>
</tr>
<tr>
<td align="center"><b>AI 引擎</b></td>
<td>Qwen3 Vision Model - 阿里通义千问视觉大模型</td>
</tr>
</table>

### 项目结构

```
frontend/
├── src/
│   ├── components/              # 🧩 可复用组件库
│   ├── pages/                   # 📄 页面级组件
│   │   ├── HomePage.vue         #    🏠 仪表盘首页
│   │   ├── TestCasesPage.vue    #    📝 测试用例管理中心
│   │   └── ExecutionsPage.vue   #    📊 执行历史分析
│   ├── styles/                  # 🎨 样式系统
│   │   └── index. css            #    全局样式 + Tailwind 配置
│   ├── utils/                   # 🛠️ 工具函数集
│   │   └── api.js               #    API 调用封装层
│   ├── App. vue                  # 🚀 应用根组件
│   └── main.js                  # ⚙️ 应用入口
├── index.html                   # 📋 HTML 模板
├── vite.config.js               # ⚡ Vite 构建配置
├── tailwind.config.js           # 🎨 Tailwind 主题配置
├── postcss.config.js            # 📦 PostCSS 配置
└── package.json                 # 📚 项目依赖清单
```

---

## 🚀 快速开始

### 前置要求

- Node.js >= 16.x
- npm >= 8.x 或 pnpm >= 7.x

### 安装部署

```bash
# 克隆项目
git clone https://github.com/hobk/ai-brower-agent.git
cd ai-brower-agent/frontend

# 安装依赖
npm install
# 或使用 pnpm (推荐)
pnpm install
```

### 开发模式

```bash
npm run dev
```

访问 `http://localhost:5173` 即可开始体验 🎉

### 生产构建

```bash
npm run build
```

构建产物输出至 `dist/` 目录，可直接部署到任意静态服务器

---

## 📦 功能模块

### 🏠 智能仪表盘
- 实时统计数据可视化
- 任务执行趋势分析
- 快捷操作入口
- 系统健康度监控

### 📝 测试用例中心
- 可视化用例编辑器
- 拖拽式步骤编排
- 用例版本管理
- 批量导入/导出

### 🚀 执行历史追踪
- 详���执行日志记录
- 可视化执行流程回放
- 错误诊断与分析
- 性能指标统计

---

## 🔮 未来规划

- [ ] 支持更多 AI 视觉模型接入
- [ ] 多浏览器内核适配（Chrome、Firefox、Edge）
- [ ] 分布式任务调度系统
- [ ] 实时协作编辑功能
- [ ] 插件市场与生态建设
- [ ] 云端部署一键启动

---

## 🤝 贡献指南

我们欢迎所有形式的贡献！无论是新功能、Bug 修复、文档改进还是设计建议。

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

---

## 📄 开源协议

本项目采用 [MIT License](LICENSE) 开源协议

---

## 💖 致谢

感谢以下开源项目为本项目提供的灵感与技术支持：

- [Vue.js](https://vuejs.org/) - 渐进式 JavaScript 框架
- [Vite](https://vitejs.dev/) - 下一代前端工具链
- [TailwindCSS](https://tailwindcss.com/) - 原子化 CSS 框架
- [Qwen](https://github.com/QwenLM/Qwen) - 阿里云通义千问大模型

---

<div align="center">

**如果这个项目对您有帮助，请给我们一个 ⭐️ Star！**

Made with ❤️ by [hobk](https://github.com/hobk)

</div>



> 本项目由阿里云ESA提供加速、计算和保护 ![img](https://img.alicdn.com/imgextra/i3/O1CN01H1UU3i1Cti9lYtFrs_!!6000000000139-2-tps-7534-844.png)

