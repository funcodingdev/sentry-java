# Sentry Java SDK 项目概览

## 项目简介
这是 Sentry Java SDK 的官方仓库，包含了 Java 和 Android 平台的错误监控和性能监控功能。

## 项目结构文件
- `project_structure.txt` - 完整的项目文件结构 (3306 行)
- `project_structure_detailed.txt` - 包含文件大小和修改时间的详细结构
- `project_directories.txt` - 仅包含目录结构的简化版本 (1124 行)

## 📚 重要文档

### 🚀 Android 性能监控核心文档
- **`aidocs/sentry-android-performance-monitoring.md`** - **Android 性能监控完整指南** ⭐
  - 整合了启动监控、UI 卡顿监控、网络监控和性能分析
  - 包含实用的配置示例和最佳实践
  - 专为 Android 开发者和性能工程师设计

- **`aidocs/sentry-trace-operations-guide.md`** - **Trace 操作字段监控机制详解** ⭐
  - 详细说明 Sentry 管理后台中各种 Trace 数据字段
  - 包含 ui.load.initial_display、ui.load.full_display 等关键指标
  - 涵盖 http.client/server、db.query、file.read/write、graphql.operation 等操作类型
  - 解释监控时机、触发条件和实现原理
  - 包含性能指标 (Measurements) 完整说明和最佳实践
  - 适用于所有需要理解监控数据的开发者和运维人员

### 📖 详细技术文档
- **`aidocs/README.md`** - 文档导航和使用指南
- **`aidocs/sentry-startup-monitoring.md`** - 启动性能监控详细分析
- **`aidocs/sentry-ui-jank-monitoring.md`** - UI 卡顿监控机制
- **`aidocs/sentry-network-monitoring.md`** - 网络性能监控
- **`aidocs/sentry-profiling-analysis.md`** - 性能分析和方法跟踪

## 主要模块

### 核心模块
- **sentry** - 核心 Java SDK，包含基础功能
- **sentry-android-core** - Android 核心功能
- **sentry-android** - Android SDK 主模块
- **sentry-android-fragment** - Android Fragment 集成

### 集成测试模块
- **sentry-android-integration-tests** - Android 集成测试
  - `sentry-uitest-android` - UI 测试
  - `sentry-uitest-android-benchmark` - 性能基准测试
  - `sentry-uitest-android-critical` - 关键功能测试
  - `test-app-plain` - 普通测试应用

### 其他重要模块
- **sentry-apollo** - Apollo GraphQL 集成
- **sentry-apollo3** - Apollo 3 集成
- **sentry-bom** - Bill of Materials
- **sentry-compose** - Jetpack Compose 集成
- **sentry-jdbc** - JDBC 集成
- **sentry-jul** - Java Util Logging 集成
- **sentry-kotlin-extensions** - Kotlin 扩展
- **sentry-log4j2** - Log4j2 集成
- **sentry-logback** - Logback 集成
- **sentry-okhttp** - OkHttp 集成
- **sentry-opentelemetry** - OpenTelemetry 集成
- **sentry-servlet** - Servlet 集成
- **sentry-spring** - Spring 集成
- **sentry-spring-boot** - Spring Boot 集成
- **sentry-spring-jakarta** - Spring Jakarta 集成

## 构建和配置
- **buildSrc** - Gradle 构建脚本
- **gradle** - Gradle 配置
- **scripts** - 构建和维护脚本

## 文档和工具
- **aidocs** - AI 文档和架构图
- **docs** - 项目文档
- **.github** - GitHub 工作流和模板

## Android 性能监控核心功能

### 🚀 启动性能监控
- **冷启动监控**: 进程创建到首帧绘制的完整流程
- **热启动监控**: Activity 重启的性能分析
- **关键指标**: TTID (Time To Initial Display)、TTFD (Time To Full Display)
- **自动检测**: 启动类型自动识别和分类

### 🎨 UI 性能监控
- **帧率监控**: 实时帧率检测和分析
- **卡顿检测**: 慢帧和冻结帧自动识别
- **双重策略**: Performance V1 (Activity级) + V2 (Span级)
- **多设备支持**: 60fps/90fps/120fps 设备适配

### 🌐 网络性能监控
- **HTTP 监控**: OkHttp 拦截器自动集成
- **性能指标**: DNS、连接、SSL、传输时间
- **错误捕获**: 4xx/5xx 状态码自动捕获
- **分布式追踪**: 跨服务请求链路追踪

### 📊 性能分析 (Profiling)
- **方法跟踪**: Android Debug API 方法调用分析
- **资源监控**: CPU、内存使用情况
- **瓶颈识别**: 性能热点自动识别
- **智能采样**: 可配置的采样策略

## 主要功能领域

### 错误监控
- 异常捕获和报告
- 崩溃恢复
- ANR (Application Not Responding) 检测
- 原生崩溃处理

### 性能监控
- 启动时间监控
- UI 卡顿监控
- 网络请求监控
- 自定义性能指标

### 会话管理
- 用户会话跟踪
- 会话状态管理

### 重放功能
- 用户交互重放
- 错误重现

## 技术栈
- Java/Kotlin
- Android SDK
- Gradle 构建系统
- 多种第三方库集成 (Spring, OkHttp, Apollo 等)

## 快速开始 (Android 性能监控)

```kotlin
SentryAndroid.init(this) { options ->
    options.dsn = "YOUR_DSN"
    
    // 启用 Android 性能监控
    options.isEnableAppStartTracking = true      // 启动监控
    options.isEnableFramesTracking = true        // UI 监控
    options.isEnablePerformanceV2 = true         // 精确监控
    options.captureFailedRequests = true         // 网络监控
    
    // 设置采样率
    options.tracesSampleRate = 0.1               // 生产环境 10%
    options.profilesSampleRate = 0.1             // 生产环境 10%
}
```

## 文件统计
- 总文件数: 3306+ 个文件
- 主要目录数: 1124+ 个目录
- 支持多种集成和框架

## 推荐阅读路径

### 对于 Android 开发者
1. **首先阅读**: `aidocs/sentry-android-performance-monitoring.md` - Android 性能监控完整指南
2. 了解启动性能优化策略
3. 掌握 UI 卡顿检测和优化
4. 学习网络性能监控配置

### 对于性能工程师
1. 重点关注性能分析 (Profiling) 部分
2. 学习各种性能指标的含义和优化策略
3. 配置智能采样和监控策略

这个项目结构展示了一个成熟的、功能丰富的监控 SDK，特别是在 Android 平台的性能监控方面提供了全面的解决方案。 