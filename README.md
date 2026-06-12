# tauri-vue-template 

一个极致精修、追求原生桌面交互质感的 **Tauri v2 + Vue 3 + TypeScript** 跨端桌面应用通用脚手架（Template）。它专为“无边框（Frameless）”客户端设计，像素级复刻了 macOS 窗口的核心动效与视觉美学。

---

## ✨ 核心打磨特性

* 📐 **几何像素级红绿灯**：放弃了在不同系统字体下容易错位、粗细不均的文本字符（如 `×` `+`），采用纯 CSS 几何线条像素级绘制，确保鼠标悬浮时图标绝对居中、丝滑亮起。
* 🫧 **Mac 原生视网膜质感**：全局注入系统级中文字体边缘抗锯齿平滑优化（`antialiased`）、`12px` 优雅大圆角裁剪，以及全套视网膜级半透描边与窗口下沉阴影。
* 🛡️ **无缝拖拽与防挡点击**：完美集成 `data-tauri-drag-region` 隐形窗体拖拽杆。通过细粒度的 `z-index` 堆叠上下文优化，彻底解决了原生拖拽区域容易遮挡控制按钮点击的行业通病。
* 🔔 **仿原生毛玻璃 Toast**：内置一套基于前端状态调度的独立 Toast 通知组件，完美支持 `backdrop-filter` 视网膜模糊，自适应深浅色主题。
* 🌓 **全局皮肤变量引擎**：精心调配的跨项目深浅色（Dark/Light）调色盘，支持跟随系统或手动一键平滑切换，换肤只需修改顶层 CSS 变量。
* 🚫 **去除默认 Tooltip 干扰**：移除了红绿灯控制按钮在网页端难看的 HTML 默认纯文本气泡，保证最纯粹的原生客户端沉浸感。

---


## 📦 内置核心 UI 组件使用指南

为了保持极简与纯粹，本脚手架没有引入庞大的第三方 UI 库，而是手写了两个桌面端最常用、且像素级复刻 macOS 动效的基础组件。

### 1. 全局轻提示 (Toast)

采用极其优雅的**命令式调用**，无需在每个页面的 HTML 模板里写标签，自带毛玻璃果冻回弹动效。

**前提条件**：确保 `<ToastContainer />` 已在根组件（通常是 `App.vue`）的 `<template>` 中挂载过一次。

**使用方法**：在任何 `.vue` 或 `.ts` 文件中，只需引入暴露出的 `showToast` 方法即可调用。

```vue
<script setup lang="ts">

import { showToast } from '../components/toast.vue'; // 路径根据实际项目层级调整
const handleSave = () => {
 // 执行你的业务逻辑...
 // 一行代码调用提示
 showToast('保存成功！');
};
</script>
```

### 2. 原生质感确认弹窗 (ConfirmModal)
采用 Vue 标准的 v-model:visible 控制显隐。完美兼容暗黑模式，并自带防误触逻辑与危险操作警示色。

**使用方法**：引入组件，绑定 visible 状态，监听 @confirm 事件。

```vue
<script setup lang="ts">
import { ref } from 'vue';
import ConfirmModal from '../components/confirmModal.vue';

// 1. 定义弹窗显示状态
const showDeleteModal = ref(false);

// 2. 定义确认后执行的函数
const executeDelete = () => {
  // 执行你的删除逻辑...
  showToast('文件已被永久删除');
};
</script>

<template>
  <!-- 触发按钮 -->
  <button @click="showDeleteModal = true">删除文件</button>

  <!-- 弹窗组件 -->
  <ConfirmModal 
    v-model:visible="showDeleteModal"
    title="删除确认"
    message="确定要永久删除该文件吗？此操作无法撤销。"
    confirmText="删除" 
    cancelText="保留" 
    :isDanger="true" 
    @confirm="executeDelete"
  />
</template>

```

**Props 属性说明**：

- `title` (String): 弹窗标题，默认 “提示”
- `message` (String): 详细的提示信息，支持换行符 `\n`
- `confirmText` / `cancelText` (String): 自定义按钮文字
- `isDanger` (Boolean): 设为 `true` 时，确认按钮会变成系统级警告红（类似 iOS/macOS 的破坏性操作高亮色）

---

## 🛠️ 底层开发环境准备

无论是你还是其他开发者，在运行或编译此 Tauri 项目之前，请确保本地电脑已配置好以下底层系统环境：

### 1. 基础环境
* **Node.js**：建议安装最新的 LTS 版本（>= 18.0.0）。

### 2. 系统平台环境依赖（关键）
* 安装 Microsoft Visual Studio 运行库及 **C++ 生成工具 (MSVC)**。
* 安装 Rust 编译链：打开 PowerShell 运行 `iwr -useb https://tap.rustup.rs | iex`。

---

## 🚀 开发者复用与高效流转指南

为了让复用此脚手架的流程达到极致的丝滑（无需手动复制、粘贴、删 Git、改配置），推荐将此模板固化为你系统全局的快捷命令。

在 PowerShell 中运行 `notepad $PROFILE`（若提示新建文件则点击确定），在文件末尾粘贴以下全自动初始化函数：

```powershell
function create-mac-app {
    if ($args.Count -eq 0) {
        Write-Host "错误：请提供项目名称！例如：create-mac-app my-new-project" -ForegroundColor Red
        return
    }
    
    # -------------------------------------------------------------------------
    # 【命名规范转换区】规范化处理核心逻辑
    # -------------------------------------------------------------------------
    # $newName: 保持原始输入（如 LumeReader），用于文件夹、前端包名、客户端 productName
    $newName = $args[0]
    
    # $rustPackageName: 强制全小写（如 lumereader）
    # 规范：Rust 的 [package] name 不允许包含大写字母，否则 cargo 编译会直接报错
    $rustPackageName = $newName.ToLower()
    
    # $snakeName: 将中划线转换为下划线（如 my-app -> my_app）
    $snakeName = $newName -replace '-', '_'
    
    # $rustLibName: 转换为小写下划线（Snake Case，如 LumeReader -> lumereader）
    # 规范：Rust 的 [lib] name 必须遵循小写下划线规范，且后续用于 main.rs 的模块导入
    $rustLibName = $snakeName.ToLower()
    # -------------------------------------------------------------------------
    
    Write-Host ""
    Write-Host "====== Tauri 应用初始化向导 ======" -ForegroundColor Cyan
    
    # 1. 交互式获取配置
    $inputTitle = Read-Host "请输入应用显示名称 (用于任务栏/Dock等，直接回车默认: $newName)"
    $appTitle = if ([string]::IsNullOrWhiteSpace($inputTitle)) { $newName } else { $inputTitle }

    $inputBundle = Read-Host "请输入应用包名/唯一标识符 (直接回车默认: com.momo.$newName)"
    $bundleId = if ([string]::IsNullOrWhiteSpace($inputBundle)) { "com.momo.$newName" } else { $inputBundle }
    
    # 2. 确认配置信息
    Write-Host "----------------------------------------" -ForegroundColor Gray
    Write-Host "配置已锁定（已激活 Rust 命名规范保护）：" -ForegroundColor Gray
    Write-Host "  -> 文件夹 & 客户端显示名:  $newName" -ForegroundColor Yellow
    Write-Host "  -> Rust Cargo 包名 (全小写): $rustPackageName" -ForegroundColor Yellow
    Write-Host "  -> Rust Lib 库名 (下划线):   ${rustLibName}_lib" -ForegroundColor Yellow
    Write-Host "  -> 应用显示名称 (Title) :    $appTitle" -ForegroundColor Yellow
    Write-Host "  -> 唯一标识符 (Bundle ID):   $bundleId" -ForegroundColor Yellow
    Write-Host "----------------------------------------" -ForegroundColor Gray
    Write-Host ""

    # [1/4] 拉取模板
    Write-Host "[1/4] 正在从 GitHub 拉取项目模板..." -ForegroundColor Cyan
    npx degit momo-12138/tauri-vue-template $newName
    
    if (Test-Path $newName) {
        # 切换到项目根目录
        $projectPath = (Get-Item $newName).FullName
        Set-Location $projectPath
        
        # 【重要安全规范】：统一使用无 BOM 的 UTF-8 编码
        # 原因：带 BOM 的 UTF-8 会导致 Rust 的 toml 解析器和 tauri-cli 报错崩溃
        $utf8NoBom = New-Object System.Text.UTF8Encoding($false)
        
        # [2/4] 更新 package.json
        Write-Host "[2/4] 正在配置 package.json..." -ForegroundColor Cyan
        $pkgPath = Join-Path $projectPath "package.json"
        if (Test-Path $pkgPath) {
            # 前端 package 允许保持原始大小写
            $pkgContent = (Get-Content $pkgPath -Raw -Encoding utf8) -replace 'tauri-vue-template', $newName
            [System.IO.File]::WriteAllText($pkgPath, $pkgContent, $utf8NoBom)
        }
        
        # [3/4] 更新 tauri.conf.json 里的标识符、标题和产品名称
        Write-Host "[3/4] 正在配置 tauri.conf.json..." -ForegroundColor Cyan
        $jsonPath = Join-Path $projectPath "src-tauri/tauri.conf.json"
        if (Test-Path $jsonPath) {
            # 保持 $newName 和 $appTitle 的大小写
            $jsonContent = (Get-Content $jsonPath -Raw -Encoding utf8) `
                -replace '"identifier":\s*".*?"', "`"identifier`": `"$bundleId`"" `
                -replace '"title":\s*".*?"', "`"title`": `"$appTitle`"" `
                -replace '"productName":\s*".*?"', "`"productName`": `"$newName`""
            [System.IO.File]::WriteAllText($jsonPath, $jsonContent, $utf8NoBom)
        }
        
        # [4/4] 更新 Cargo.toml 和 src/main.rs (Rust 后端配置)
        Write-Host "[4/4] 正在配置 Rust 后端核心文件..." -ForegroundColor Cyan
        
        # 4.1 修改 Cargo.toml 
        # 规范应用：[package].name 使用全小写($rustPackageName)，[lib].name 使用全小写下划线+后缀(${rustLibName}_lib)
        $tomlPath = Join-Path $projectPath "src-tauri/Cargo.toml"
        if (Test-Path $tomlPath) {
            $tomlContent = (Get-Content $tomlPath -Raw -Encoding utf8) `
                -replace 'name\s*=\s*"tauri-vue-template"', "name = `"$rustPackageName`"" `
                -replace 'name\s*=\s*"tauri-vue-template_lib"', "name = `"${rustLibName}_lib`"" `
                -replace 'name\s*=\s*"tauri_vue_template_lib"', "name = `"${rustLibName}_lib`""
            [System.IO.File]::WriteAllText($tomlPath, $tomlContent, $utf8NoBom)
            Write-Host "  -> Cargo.toml 配置完成" -ForegroundColor Gray
        } else {
            Write-Host "  -> 未检测到 Cargo.toml，跳过。" -ForegroundColor Yellow
        }

        # 4.2 修改 src/main.rs 
        # 规范应用：此处的静态调用必须与 Cargo.toml 中的 [lib].name 严格保持 100% 同步小写下划线
        $mainRsPath = Join-Path $projectPath "src-tauri/src/main.rs"
        if (Test-Path $mainRsPath) {
            $mainContent = (Get-Content $mainRsPath -Raw -Encoding utf8) `
                -replace 'tauri-vue-template_lib', "${rustLibName}_lib" `
                -replace 'tauri_vue_template_lib', "${rustLibName}_lib"
            [System.IO.File]::WriteAllText($mainRsPath, $mainContent, $utf8NoBom)
            Write-Host "  -> src/main.rs 配置完成" -ForegroundColor Gray
        } else {
            Write-Host "  -> 未检测到 src/main.rs，跳过。" -ForegroundColor Yellow
        }
        
        Write-Host ""
        Write-Host "恭喜！项目 '$newName' 初始化成功！" -ForegroundColor Green
        Write-Host "当前工作目录: $(Get-Location)" -ForegroundColor Green
        Write-Host "提示：你可以运行 'npm install' 或 'pnpm install' 开始安装依赖。" -ForegroundColor Gray
    } else {
        Write-Host "错误：模板拉取失败，请检查网络或 degit 配置。" -ForegroundColor Red
    }
}
```

保存关闭后，在终端运行 `. $PROFILE` 使其立即生效。

## 🏁 日常黄金开发流

当上述快捷命令配置完成后，日后开启任何一个全新的应用创意，只需在终端无脑敲入三行命令：

##### 1.一键拉取干净源码，全自动剥离旧 Git 记录，并全自动完成标识符与包名重设，之后自动直达新项目根目录：

   ```bash
   create-mac-app my-cool-app
   ```
##### 2.安装依赖：

   ```bash
   npm install
   ```

##### 3.启动新项目：

```bash
npm run tauri dev
```
