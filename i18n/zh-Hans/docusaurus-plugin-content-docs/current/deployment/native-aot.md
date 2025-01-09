---
id: native-aot
title: Native AOT 部署
---

Native AOT（Ahead-of-Time）编译允许您将 Avalonia 应用程序发布为自包含的可执行文件，具有原生性能特性。本指南涵盖了 Avalonia 特定的注意事项和Native AOT 部署的设置。

## 对于 Avalonia 应用程序的优势

Native AOT 编译可为 Avalonia 应用程序提供多项优势：

- 更快的应用程序启动时间，尤其适用于桌面应用程序
- 减少内存占用，适用于资源有限的环境
- 独立部署，无需安装 .NET 运行时
- 通过减少攻击面（无 JIT 编译）提高安全性
- 与裁剪（trimming）结合使用时，发行量更小

## 设置 Avalonia 的 Native AOT

### 1. 项目配置

将以下内容添加到您的 csproj 文件中：

```xml
<PropertyGroup>
    <PublishAot>true</PublishAot>
    <!-- Recommended Avalonia trimming settings for Native AOT -->
    <BuiltInComInteropSupport>false</BuiltInComInteropSupport>
    <TrimMode>link</TrimMode>
</PropertyGroup>
```

### 2. 剪裁配置

Native AOT 需要剪裁。添加这些特定于 Avalonia 的剪裁设置：

```xml
<ItemGroup>
    <!-- Preserve Avalonia types for reflection -->
    <TrimmerRootAssembly Include="Avalonia.Themes.Fluent" />
    <TrimmerRootAssembly Include="Avalonia.Themes.Default" />
</ItemGroup>
```

## Avalonia 特定的注意事项

### XAML 加载
使用Native AOT 时，XAML 在构建时编译到应用程序中。确保您：
- 在 XAML 文件中使用 `x:CompileBindings="True"`
- 避免在运行时动态加载 XAML
- 尽可能使用静态资源引用而不是动态资源

### 资产和资源
- 将所有资产捆绑为嵌入资源（embedded resources）
- 对资产使用 `AvaloniaResource` 构建操作
- 避免从外部源动态加载资产

### ViewModel 和依赖注入
- 在启动时注册您的 ViewModel
- 使用编译时 DI 配置
- 避免基于反射的服务定位

## 发布 Avalonia 原生 AOT 应用程序

### Windows
```bash
dotnet publish -r win-x64 -c Release
```

### Linux
```bash
dotnet publish -r linux-x64 -c Release
```

### macOS
基于 Intel 的 macOS
```bash
dotnet publish -r osx-x64 -c Release
```
基于 Apple silicon 的 macOS
```bash
dotnet publish -r osx-arm64 -c Release
```

:::tip
然后您可以使用 Apple 的 [lipo 工具](https://developer.apple.com/documentation/apple-silicon/building-a-universal-macos-binary) 将 Intel 和 Apple Silicon 二进制文件合并，允许您发布通用二进制文件。
:::

## 常见问题排查

### 1. 缺少 XAML 控件
如果控件在运行时缺失：
```xml
<ItemGroup>
    <!-- Add specific Avalonia controls you're using -->
    <TrimmerRootAssembly Include="Avalonia.Controls" />
</ItemGroup>
```
### 2. 反射相关错误
对于使用反射的 ViewModel 或服务：
```xml
<ItemGroup>
    <TrimmerRootDescriptor Include="TrimmerRoots.xml" />
</ItemGroup>
```

创建一个 `TrimmerRoots.xml` 文件：
```xml
<linker>
    <assembly fullname="YourApplication">
        <type fullname="YourApplication.ViewModels*" preserve="all"/>
    </assembly>
</linker>
```

## 已知限制

使用 Avalonia 的 Native AOT 时，请注意以下限制：
- 动态控件创建必须在剪裁设置中配置
- 某些第三方 Avalonia 控件可能不兼容 AOT
- 平台特定功能需要显式配置
- 设计时（design-time）工具中的实时预览可能有限

## 平台支持

| 平台 | 状态 |
|----------|--------|
| Windows x64 | ✅ 支持 | 
| Windows Arm64 | ✅ 支持 | 
| Linux x64 | ✅ 支持 | |
| Linux Arm64 | ✅ 支持 | 
| macOS x64 | ✅ 支持 | |
| macOS Arm64 | ✅ 支持 | 
| Browser | ❌ 不支持 |

## 最佳实践

1. **应用程序结构**
   - 一致地使用 MVVM 模式
   - 最小化反射使用
   - 优先使用编译时配置

2. **资源管理**
   - 尽可能使用静态资源
   - 捆绑所有必需的资产
   - 在 IDisposable 中实现适当的清理

3. **性能优化**
   - 启用绑定编译
   - 使用编译绑定（compiled bindings）
   - 对大型集合实现适当的虚拟化

## 其他资源

- [Avalonia Native AOT 示例应用程序](https://github.com/AvaloniaUI/Avalonia.Samples)