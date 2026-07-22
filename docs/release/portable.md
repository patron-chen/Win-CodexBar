# Windows Portable 发布包

项目已经提供正式发布脚本 `scripts\windows-release-build.ps1`。该脚本通过干净的托管 checkout 构建 Tauri 桌面程序和 CLI，验证可执行文件，生成 Inno Setup 安装包，并输出项目规定的单文件 portable EXE 及 SHA-256 sidecar。

Portable 发布物不是 ZIP，也不需要旁置 `WebView2Loader.dll`：

```text
CodexBar-<version>-portable.exe
CodexBar-<version>-portable.exe.sha256
```

## 前置条件

- Windows PowerShell 5.1 或 PowerShell 7。
- Git、Node.js、pnpm、Rustup/Cargo。
- Visual Studio C++ Build Tools；建议从 x64 Developer PowerShell 运行。
- Inno Setup 6（`ISCC.exe`）。正式脚本会同时生成安装包，因此只取 portable 文件时也需要 Inno Setup。
- 能访问 Git remote、crates.io、Microsoft WebView2 和 VC++ Runtime 下载地址。

推荐先确认：

```powershell
git --version
pnpm --version
cargo --version
rustup show
Get-Command ISCC.exe
```

## 一键生成

在仓库根目录运行，使用实际发布标签替换示例版本：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\windows-release-build.ps1 -Ref v0.45.2
```

需要同时验证安装、卸载流程时：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\windows-release-build.ps1 -Ref v0.45.2 -SmokeInstall
```

首次执行会下载依赖并完整编译，后续构建复用 `C:\code\Win-CodexBar-release\cache`。

## 输入代码规则

- `-Ref` 必须是托管 checkout 能解析的远程标签、分支或提交，正式发布推荐使用标签。
- 脚本不会打包当前工作区的未提交改动。需要先提交、推送并创建对应标签，再传给 `-Ref`。
- 默认托管目录为 `C:\code\Win-CodexBar-release\source`。脚本会在该目录执行强制 checkout、reset 和 clean；不要把 `-WorkRoot` 指向当前开发仓库或包含重要文件的目录。
- `-WarmCacheOnly` 只预热编译缓存，不生成 portable 发布物。

## 输出位置

默认发布资产：

```text
C:\code\Win-CodexBar-release\assets\CodexBar-<version>-portable.exe
C:\code\Win-CodexBar-release\assets\CodexBar-<version>-portable.exe.sha256
C:\code\Win-CodexBar-release\assets\CodexBar-<version>-Setup.exe
C:\code\Win-CodexBar-release\assets\CodexBar-<version>-Setup.exe.sha256
```

构建日志：

```text
C:\code\Win-CodexBar-release\assets\tauri-build.log
C:\code\Win-CodexBar-release\assets\tauri-build.err.log
```

## 校验发布物

```powershell
Get-FileHash C:\code\Win-CodexBar-release\assets\CodexBar-0.45.2-portable.exe -Algorithm SHA256
Get-Content C:\code\Win-CodexBar-release\assets\CodexBar-0.45.2-portable.exe.sha256
cmd.exe /c scripts\ci\assert-release-assets.cmd
```

`Get-FileHash` 的值应与 sidecar 第一列一致；资产检查脚本应找到 installer、portable 及两个 SHA-256 文件。

## 常用选项

```powershell
# 使用其他专用缓存/输出根目录
.\scripts\windows-release-build.ps1 -Ref v0.45.2 -WorkRoot D:\CodexBar-release

# 强制重新下载并校验 Microsoft 安装依赖
.\scripts\windows-release-build.ps1 -Ref v0.45.2 -RefreshInstallerDependencies

# 构建后上传到已存在的 GitHub Release（需要 gh 已登录）
.\scripts\windows-release-build.ps1 -Ref v0.45.2 -UploadRelease v0.45.2
```

自定义 `-WorkRoot` 后，资产路径和手工校验命令也要改为该目录下的 `assets`。