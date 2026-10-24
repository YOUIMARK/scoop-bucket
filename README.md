# scoop-bucket

个人 Scoop bucket（bucket 名称 `cus`）。

## 使用方法

```powershell
scoop bucket add cus https://github.com/YOUIMARK/scoop-bucket
scoop install wslcontainerdesktop
```

## 软件列表

| 清单 | 说明 | 备注 |
| --- | --- | --- |
| `wslcontainerdesktop` | [WSL Container Desktop](https://github.com/mhackermsft/wslcontainerdesktop) —— 图形化管理 WSL 容器（WSL 原生容器与外部 Docker 引擎） | 上游只发 MSIX，**安装与升级需要管理员权限** |

### wslcontainerdesktop

- **为什么需要管理员**：上游用自签证书签名 MSIX，安装必须先 `Import-Certificate` 到 `Cert:\LocalMachine\TrustedPeople`，再 `Add-AppxPackage` 侧载。请以管理员身份打开 PowerShell 或 Windows 终端后执行安装/升级。
- **证书副作用**：该发布者证书（`CN=Michael Hacker`）会进入本机「受信任人」存储，意味着同一证书签名的其他包也可无提示侧载；`scoop uninstall` 会移除它。
- **数据放哪**：应用把配置写在 `%LOCALAPPDATA%\WslContainerDesktop`，打包后实际落在 `%LOCALAPPDATA%\Packages\<包族名>\LocalCache\Local\WslContainerDesktop`。安装时会把该目录用 junction 指向 Scoop 的 persist 目录（默认 `C:\Scoop\persist\wslcontainerdesktop\data`），卸载后数据仍在。
- **改数据位置**：在安装/升级**之前**设置环境变量。

  ```powershell
  $env:SCOOP_WSLCD_PERSIST_DATA = '0'          # 关闭重定向，数据留在 MSIX 容器内
  $env:SCOOP_WSLCD_PERSIST_DATA = 'D:\wlscd'   # 改用自定义目录
  ```

- **下载缓存**：安装包约 111 MB，缓存在 `C:\Scoop\cache`；`scoop cache rm wslcontainerdesktop` 清缓存，`scoop cleanup wslcontainerdesktop` 清旧版本目录。
- **启动**：开始菜单里的 *WSL Container Desktop*（MSIX 自带条目），或命令行执行 `wslcd`（Scoop 生成的 shim）。
- **升级**：`scoop update wslcontainerdesktop`。安装脚本会重建 `launch.cmd` 与数据 junction，`persist` 中的数据不受影响。
- **卸载**：`scoop uninstall wslcontainerdesktop`。移除应用包与 shim；以管理员身份运行时还会撤销导入的证书。`persist\wslcontainerdesktop\data` 会保留。
- **限制**：仅提供 x64；Windows 可能在包更新或清理时重建 `LocalCache`，此时重跑 `scoop update wslcontainerdesktop` 即可恢复数据链接。

## 维护

```powershell
.\bin\checkver.ps1 wslcontainerdesktop      # 检查上游新版本
.\bin\checkhashes.ps1 wslcontainerdesktop   # 更新 hash
.\bin\formatjson.ps1                        # 统一清单格式
.\bin\test.ps1                              # 桶测试（需要 Pester / BuildHelpers）
```
