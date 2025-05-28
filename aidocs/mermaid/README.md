# Sentry Java SDK Mermaid 时序图文件

本目录包含从 aidocs 目录中的 Markdown 文档提取的所有 Mermaid 时序图文件。这些文件可以直接用于 [Mermaid CLI](https://github.com/mermaid-js/mermaid-cli) 生成图像。

## 📁 目录结构

```
mermaid/
├── README.md                    # 本说明文档
├── svg/                        # 生成的 SVG 图像文件
│   ├── sentry-*.svg            # 12 个 SVG 时序图文件
└── *.mmd                       # 12 个 Mermaid 源文件
```

## 📁 文件列表

### 🚀 初始化流程 (sentry-initialization-flow.md)

| 文件名 | 描述 | 来源 | SVG 文件 |
|--------|------|------|----------|
| `sentry-initialization-flow-core-initialization.mmd` | 核心初始化流程 | 核心初始化流程时序图 | ✅ 已生成 |
| `sentry-initialization-flow-android-initialization.mmd` | Android 特定初始化流程 | Android 特定初始化流程时序图 | ✅ 已生成 |
| `sentry-initialization-flow-integration-registration.mmd` | 集成注册流程 | 集成注册流程时序图 | ✅ 已生成 |
| `sentry-initialization-flow-client-creation.mmd` | 客户端创建流程 | 客户端创建流程时序图 | ✅ 已生成 |
| `sentry-initialization-flow-configuration-loading.mmd` | 配置加载流程 | 配置加载流程时序图 | ✅ 已生成 |

### 📱 快速参考 (sentry-init-quick-reference.md)

| 文件名 | 描述 | 来源 | SVG 文件 |
|--------|------|------|----------|
| `sentry-init-quick-reference-android-flow.mmd` | Android 特定流程 | Android 特定流程时序图 | ✅ 已生成 |

### 🚀 启动监控 (sentry-startup-monitoring.md)

| 文件名 | 描述 | 来源 | SVG 文件 |
|--------|------|------|----------|
| `sentry-startup-monitoring-time-measurement.mmd` | 时间测量流程 | 时间测量流程时序图 | ✅ 已生成 |

### 💥 崩溃监控 (sentry-crash-monitoring.md)

| 文件名 | 描述 | 来源 | SVG 文件 |
|--------|------|------|----------|
| `sentry-crash-monitoring-exception-capture.mmd` | 异常捕获流程 | 异常捕获流程时序图 | ✅ 已生成 |
| `sentry-crash-monitoring-anr-detection.mmd` | ANR 检测流程 | ANR 检测流程时序图 | ✅ 已生成 |
| `sentry-crash-monitoring-native-crash.mmd` | Native 崩溃处理流程 | Native 崩溃处理流程时序图 | ✅ 已生成 |
| `sentry-crash-monitoring-startup-crash.mmd` | 启动崩溃处理流程 | 启动崩溃处理流程时序图 | ✅ 已生成 |
| `sentry-crash-monitoring-crash-recovery.mmd` | 崩溃恢复流程 | 崩溃恢复流程时序图 | ✅ 已生成 |

## 🛠️ 使用方法

### 直接使用 SVG 文件

所有时序图已经预先生成为 SVG 格式，可以直接使用：

```bash
# 查看所有生成的 SVG 文件
ls -la svg/

# 在浏览器中打开 SVG 文件
open svg/sentry-initialization-flow-core-initialization.svg
```

### 安装 Mermaid CLI

如果需要重新生成或自定义输出格式：

```bash
npm install -g @mermaid-js/mermaid-cli
```

### 重新生成 SVG 图像

```bash
# 重新生成所有 SVG 文件
for file in *.mmd; do
    mmdc -i "$file" -o "svg/${file%.mmd}.svg"
done

# 生成单个文件
mmdc -i sentry-initialization-flow-core-initialization.mmd -o svg/core-initialization.svg
```

### 生成 PNG 图像

```bash
# 生成 PNG 格式
mmdc -i sentry-initialization-flow-core-initialization.mmd -o svg/core-initialization.png

# 使用深色主题
mmdc -i sentry-initialization-flow-core-initialization.mmd -o svg/core-initialization.png -t dark

# 设置背景透明
mmdc -i sentry-initialization-flow-core-initialization.mmd -o svg/core-initialization.png -b transparent
```

### 生成 PDF 文档

```bash
mmdc -i sentry-initialization-flow-core-initialization.mmd -o svg/core-initialization.pdf
```

## 📊 统计信息

- **总文件数**: 12 个 MMD 文件 + 12 个 SVG 文件
- **来源文档**: 4 个 Markdown 文件
- **时序图类型**: sequenceDiagram
- **MMD 总行数**: 约 500+ 行
- **SVG 文件大小**: 22KB - 53KB

## 🎨 SVG 文件特点

- **矢量格式**: 可无损缩放，适合各种尺寸显示
- **Web 友好**: 可直接在浏览器中查看，支持嵌入网页
- **高质量**: 清晰的文字和线条，适合文档和演示
- **兼容性**: 支持大多数图像查看器和编辑软件

## 🔗 相关链接

- [Mermaid 官方文档](https://mermaid-js.github.io/mermaid/)
- [Mermaid CLI GitHub](https://github.com/mermaid-js/mermaid-cli)
- [时序图语法参考](https://mermaid-js.github.io/mermaid/#/sequenceDiagram)

## 📝 注意事项

1. **中文字符**: 部分文件包含中文字符，SVG 文件已正确渲染
2. **复杂度**: 一些时序图较为复杂，SVG 文件保持了完整的细节
3. **更新**: 当源 MMD 文件更新时，需要重新生成 SVG 文件
4. **文件大小**: SVG 文件相对较大，但保证了高质量的渲染效果

## 🚀 快速开始

1. **查看时序图**: 直接打开 `svg/` 目录中的 SVG 文件
2. **嵌入文档**: 将 SVG 文件引用到你的文档中
3. **自定义生成**: 修改 MMD 文件后重新生成 SVG

---

💡 **提示**: SVG 文件可以直接在 GitHub、GitLab 等平台中显示，也可以嵌入到 HTML、Markdown 文档中使用。 