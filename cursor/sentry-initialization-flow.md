# Sentry Java SDK 初始化流程时序图

本文档通过时序图的形式详细展示了 Sentry Java SDK 的初始化流程，包括核心组件的创建、配置加载、集成注册等关键步骤。

## 核心初始化流程

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Sentry as Sentry
    participant Lock as AutoClosableReentrantLock
    participant Options as SentryOptions
    participant Scopes as IScopes
    participant Client as SentryClient
    participant Transport as ITransport
    participant Integrations as Integration[]
    participant Storage as IScopesStorage

    User->>Sentry: init(OptionsConfiguration)
    Sentry->>Lock: acquire()
    Lock-->>Sentry: token

    Note over Sentry: 1. 平台检查
    alt Android平台 && 非SentryAndroidOptions
        Sentry-->>User: throw IllegalArgumentException
    end

    Note over Sentry: 2. 预初始化配置
    Sentry->>Sentry: preInitConfigurations(options)
    Sentry->>Options: isEnableExternalConfiguration()
    alt 启用外部配置
        Sentry->>Options: merge(ExternalOptions)
    end
    Sentry->>Options: getDsn()
    alt DSN为空或无效
        Sentry->>Sentry: close()
        Sentry-->>User: return false
    end
    Sentry->>Options: retrieveParsedDsn()

    Note over Sentry: 3. 检查是否应该初始化
    Sentry->>Sentry: shouldInit(options)
    alt 不应该初始化
        Sentry-->>User: 记录警告并返回
    end

    Note over Sentry: 4. 关闭现有Scopes
    Sentry->>Scopes: getCurrentScopes()
    Sentry->>Scopes: close(true)

    Note over Sentry: 5. 创建新的Scopes结构
    Sentry->>Sentry: globalScope.replaceOptions(options)
    Sentry->>Scopes: new Scope(options) [rootScope]
    Sentry->>Scopes: new Scope(options) [rootIsolationScope]
    Sentry->>Scopes: new Scopes(rootScope, rootIsolationScope, globalScope)

    Note over Sentry: 6. 初始化核心组件
    Sentry->>Sentry: initLogger(options)
    Sentry->>Sentry: initForOpenTelemetryMaybe(options)
    Sentry->>Sentry: initScopesStorage(options)
    Sentry->>Storage: close()
    alt OpenTelemetry模式关闭
        Sentry->>Storage: new DefaultScopesStorage()
    else OpenTelemetry模式开启
        Sentry->>Storage: ScopesStorageFactory.create()
    end

    Note over Sentry: 7. 设置Scopes存储
    Sentry->>Storage: set(rootScopes)

    Note over Sentry: 8. 初始化配置
    Sentry->>Sentry: initConfigurations(options)
    Sentry->>Options: getOutboxPath()
    alt outboxPath存在
        Sentry->>Sentry: 创建outbox目录
    end
    Sentry->>Options: getCacheDirPath()
    alt cacheDirPath存在
        Sentry->>Sentry: 创建cache目录
        Sentry->>Options: setEnvelopeDiskCache(EnvelopeCache)
    end
    Sentry->>Options: getProfilingTracesDirPath()
    alt 性能分析启用
        Sentry->>Sentry: 创建profiling目录
        Sentry->>Sentry: 清理旧的trace文件
    end

    Note over Sentry: 9. 配置模块和调试元数据
    Sentry->>Options: setModulesLoader()
    Sentry->>Options: setDebugMetaLoader()
    Sentry->>Options: setThreadChecker()
    Sentry->>Options: addPerformanceCollector()

    Note over Sentry: 10. 创建并绑定客户端
    Sentry->>Client: new SentryClient(options)
    Client->>Transport: transportFactory.create(options)
    Transport-->>Client: transport实例
    Client-->>Sentry: client实例
    Sentry->>Scopes: bindClient(client)

    Note over Sentry: 11. 注册集成
    loop 遍历所有集成
        Sentry->>Integrations: register(ScopesAdapter, options)
        Integrations->>Options: 获取配置
        Integrations->>Scopes: 注册回调和监听器
        Integrations-->>Sentry: 注册完成
    end

    Note over Sentry: 12. 后续处理
    Sentry->>Sentry: notifyOptionsObservers(options)
    Sentry->>Sentry: finalizePreviousSession(options, scopes)
    Sentry->>Sentry: handleAppStartProfilingConfig(options)

    Sentry->>Lock: release()
    Sentry-->>User: 初始化完成
```

## Android 特定初始化流程

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant SentryAndroid as SentryAndroid
    participant Context as Android Context
    participant AndroidOptions as SentryAndroidOptions
    participant AndroidInit as AndroidOptionsInitializer
    participant AppStart as AppStartMetrics
    participant Integrations as Android Integrations

    User->>SentryAndroid: init(context, configuration)
    SentryAndroid->>SentryAndroid: staticLock.acquire()

    Note over SentryAndroid: 1. 检查可用的集成
    SentryAndroid->>SentryAndroid: 检查Timber可用性
    SentryAndroid->>SentryAndroid: 检查Fragment可用性
    SentryAndroid->>SentryAndroid: 检查Replay可用性

    Note over SentryAndroid: 2. 创建Android特定组件
    SentryAndroid->>SentryAndroid: new BuildInfoProvider()
    SentryAndroid->>SentryAndroid: new ActivityFramesTracker()

    Note over SentryAndroid: 3. 加载默认配置和元数据
    SentryAndroid->>AndroidInit: loadDefaultAndMetadataOptions(options, context)
    AndroidInit->>Context: 读取AndroidManifest.xml
    AndroidInit->>AndroidOptions: 设置默认值
    AndroidInit->>AndroidOptions: 应用元数据配置
    AndroidInit-->>SentryAndroid: 配置完成

    Note over SentryAndroid: 4. 安装默认集成
    SentryAndroid->>AndroidInit: installDefaultIntegrations()
    AndroidInit->>AndroidOptions: 添加ActivityLifecycleIntegration
    AndroidInit->>AndroidOptions: 添加AnrIntegration
    AndroidInit->>AndroidOptions: 添加NetworkBreadcrumbsIntegration
    AndroidInit->>AndroidOptions: 添加SystemEventsBreadcrumbsIntegration
    alt Fragment可用
        AndroidInit->>AndroidOptions: 添加FragmentLifecycleIntegration
    end
    alt Timber可用
        AndroidInit->>AndroidOptions: 添加SentryTimberIntegration
    end
    alt Replay可用
        AndroidInit->>AndroidOptions: 添加ReplayIntegration
    end

    Note over SentryAndroid: 5. 执行用户配置回调
    SentryAndroid->>User: configuration.configure(options)
    User-->>SentryAndroid: 配置完成

    Note over SentryAndroid: 6. 设置应用启动指标
    SentryAndroid->>AppStart: getInstance()
    alt 性能V2启用
        SentryAndroid->>AppStart: setStartedAt(Process.getStartUptimeMillis())
    end
    SentryAndroid->>AppStart: registerApplicationForegroundCheck()
    SentryAndroid->>AppStart: setSdkInitStartedAt(sdkInitMillis)

    Note over SentryAndroid: 7. 初始化集成和处理器
    SentryAndroid->>AndroidInit: initializeIntegrationsAndProcessors()
    AndroidInit->>AndroidOptions: 添加DefaultAndroidEventProcessor
    AndroidInit->>AndroidOptions: 添加PerformanceAndroidEventProcessor
    AndroidInit->>AndroidOptions: 添加ScreenshotEventProcessor
    AndroidInit->>AndroidOptions: 添加ViewHierarchyEventProcessor
    AndroidInit->>AndroidOptions: 添加AnrV2EventProcessor

    Note over SentryAndroid: 8. 去重集成
    SentryAndroid->>SentryAndroid: deduplicateIntegrations()

    Note over SentryAndroid: 9. 调用核心Sentry初始化
    SentryAndroid->>Sentry: init(OptionsContainer, configuration, globalHubMode=true)
    Note over Sentry: 执行核心初始化流程...

    Note over SentryAndroid: 10. 启动会话和Replay
    SentryAndroid->>SentryAndroid: 检查前台重要性
    alt 应用在前台
        alt 自动会话跟踪启用
            SentryAndroid->>Scopes: startSession()
        end
        SentryAndroid->>Options: getReplayController().start()
    end

    SentryAndroid-->>User: 初始化完成
```

## 集成注册流程

```mermaid
sequenceDiagram
    participant Sentry as Sentry
    participant Integration as Integration
    participant Scopes as IScopes
    participant Options as SentryOptions
    participant Platform as 平台组件

    Note over Sentry: 遍历所有集成进行注册
    loop 每个集成
        Sentry->>Integration: register(scopes, options)
        
        Note over Integration: 集成特定的注册逻辑
        alt ActivityLifecycleIntegration
            Integration->>Platform: application.registerActivityLifecycleCallbacks()
            Integration->>Options: 配置性能监控
            Integration->>Options: 配置帧率跟踪
        else AnrIntegration
            Integration->>Platform: 启动ANR监控线程
            Integration->>Platform: 设置ANR检测器
        else NetworkBreadcrumbsIntegration
            Integration->>Platform: 注册网络状态监听器
            Integration->>Platform: 设置网络拦截器
        else SystemEventsBreadcrumbsIntegration
            Integration->>Platform: 注册系统事件广播接收器
            Integration->>Platform: 设置Intent过滤器
        else UncaughtExceptionHandlerIntegration
            Integration->>Platform: 设置全局异常处理器
            Integration->>Platform: 保存原有异常处理器
        else ShutdownHookIntegration
            Integration->>Platform: 注册JVM关闭钩子
        end

        Note over Integration: 更新SDK版本信息
        Integration->>Options: addIntegrationToSdkVersion()
        Integration->>Options: addPackageInfo()
        
        Integration-->>Sentry: 注册完成
    end
```

## 客户端创建流程

```mermaid
sequenceDiagram
    participant Sentry as Sentry
    participant Client as SentryClient
    participant Options as SentryOptions
    participant TransportFactory as ITransportFactory
    participant Transport as ITransport
    participant RequestResolver as RequestDetailsResolver

    Sentry->>Client: new SentryClient(options)
    Client->>Options: 验证options非空
    Client->>Client: enabled = true

    Note over Client: 创建传输层
    Client->>Options: getTransportFactory()
    alt TransportFactory是NoOp
        Client->>TransportFactory: new AsyncHttpTransportFactory()
        Client->>Options: setTransportFactory(transportFactory)
    end

    Client->>RequestResolver: new RequestDetailsResolver(options)
    Client->>RequestResolver: resolve()
    RequestResolver-->>Client: RequestDetails

    Client->>TransportFactory: create(options, requestDetails)
    TransportFactory->>Transport: 创建具体传输实现
    Transport-->>TransportFactory: transport实例
    TransportFactory-->>Client: transport实例

    Client-->>Sentry: SentryClient实例
```

## 配置加载流程

```mermaid
sequenceDiagram
    participant Sentry as Sentry
    participant Options as SentryOptions
    participant External as ExternalOptions
    participant Properties as PropertiesProvider
    participant Manifest as AndroidManifest
    participant Files as 配置文件

    Note over Sentry: 预初始化配置检查
    Sentry->>Options: isEnableExternalConfiguration()
    alt 启用外部配置
        Sentry->>Properties: PropertiesProviderFactory.create()
        Properties->>Files: 读取sentry.properties
        Properties->>Files: 读取系统属性
        Properties->>Files: 读取环境变量
        Properties-->>External: 配置数据
        Sentry->>External: from(properties, logger)
        External-->>Options: 外部配置
        Sentry->>Options: merge(externalOptions)
    end

    Note over Sentry: Android特定配置加载
    alt Android平台
        Sentry->>Manifest: 读取meta-data
        Manifest->>Options: 设置DSN
        Manifest->>Options: 设置debug模式
        Manifest->>Options: 设置采样率
        Manifest->>Options: 设置环境信息
        Manifest->>Options: 设置发布版本
    end

    Note over Sentry: DSN验证
    Sentry->>Options: getDsn()
    alt DSN为空或禁用
        Sentry->>Sentry: close()
        Sentry-->>Sentry: return false
    else DSN无效
        Sentry-->>Sentry: throw IllegalArgumentException
    end

    Sentry->>Options: retrieveParsedDsn()
    Options->>Options: 解析和验证DSN格式
    Options-->>Sentry: 解析完成
```

## 关键组件说明

### 1. **Sentry 主类**
- 提供静态API入口
- 管理全局状态和锁
- 协调整个初始化流程

### 2. **SentryOptions**
- 存储所有配置选项
- 支持外部配置合并
- 提供默认值和验证

### 3. **IScopes**
- 管理作用域层次结构
- 提供线程安全的上下文管理
- 支持作用域继承和隔离

### 4. **SentryClient**
- 核心事件处理客户端
- 管理传输层连接
- 处理事件序列化和发送

### 5. **Integration**
- 提供框架和平台集成
- 自动注册监听器和钩子
- 扩展SDK功能

### 6. **Transport**
- 处理网络传输
- 实现重试和限流
- 支持离线缓存

这个初始化流程确保了Sentry SDK能够：
- 正确加载配置
- 安全地初始化组件
- 注册必要的集成
- 建立可靠的传输连接
- 提供完整的错误监控功能 