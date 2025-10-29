# 🪟 WSL 环境完整解决方案

## 🎯 针对 WSL 的特殊优化

您在 **WSL (Windows Subsystem for Linux)** 中开发，USB 设备无法直接访问。我们为此提供了**完整的自动化解决方案**。

---

## ✅ 已完成的优化

### 1. **自动 WSL 环境检测**

`make_flash.sh` 现在会：
- ✅ 自动检测是否在 WSL 环境运行
- ✅ 检查 Rockchip USB 设备是否已绑定
- ✅ 提供友好的错误提示和解决方案
- ✅ 在设备未连接时给出详细的 usbipd 命令

### 2. **WSL USB 设置向导** (`wsl_usb_setup.sh`)

一键式设置工具，提供：
- ✅ 完整的图形化设置指南
- ✅ 自动设备连接检测
- ✅ Windows PowerShell 命令生成
- ✅ 快速参考卡片
- ✅ 自动生成 Windows 端辅助脚本

### 3. **详细使用文档** (`WSL环境使用指南.md`)

包含：
- ✅ 从零开始的完整设置步骤
- ✅ 常见问题和故障排查
- ✅ 最佳实践和优化建议
- ✅ 自动化脚本示例
- ✅ 快速参考命令表

---

## 🚀 WSL 环境使用流程

### 首次设置（一次性）

#### 第 1 步：Windows 端安装

**在 Windows PowerShell (管理员) 中：**

```powershell
# 安装 usbipd-win
winget install --interactive --exact dorssel.usbipd-win
```

#### 第 2 步：WSL 端安装

**在 WSL 终端中：**

```bash
# 安装 USB/IP 工具
sudo apt update
sudo apt install linux-tools-generic hwdata

# 配置 usbip 命令
sudo update-alternatives --install /usr/local/bin/usbip usbip \
    $(ls /usr/lib/linux-tools/*/usbip | tail -n1) 20
```

### 日常使用（每次开发）

#### 简化版（推荐）

```bash
# 1. 设备进入 Maskrom 模式
#    - 关闭电源
#    - 按住 Maskrom 按钮
#    - 插入 USB-C
#    - 松开按钮

# 2. WSL 中运行设置向导
cd /root/code/os_test/StarryOS/arceos/tools/orangepi5
bash wsl_usb_setup.sh

# 向导会显示详细步骤，按照提示操作即可

# 3. 在 Windows PowerShell (管理员) 中绑定设备
usbipd list                        # 找到 BUSID
usbipd attach --wsl --busid X-X    # 绑定到 WSL

# 4. 回到 WSL，开始刷写
sudo bash make_flash.sh target=SD \
    uimg=../../../starry_aarch64-opi5p.uimg \
    rootfs=../../../rootfs-aarch64.img
```

#### 完整版（手动步骤）

```bash
# === WSL 终端 ===

# 1. 运行设置检查
cd /root/code/os_test/StarryOS/arceos/tools/orangepi5
bash wsl_usb_setup.sh check

# === Windows PowerShell (管理员) ===

# 2. 查找设备
usbipd list

# 输出示例：
# BUSID  VID:PID    DEVICE
# 1-4    2207:350b  Rockchip Maskrom Device  ← 记下 BUSID

# 3. 首次需要 bind（只需一次）
usbipd bind --busid 1-4

# 4. 附加到 WSL（每次使用都需要）
usbipd attach --wsl --busid 1-4

# === 回到 WSL 终端 ===

# 5. 验证连接
lsusb | grep Rockchip
sudo rkdeveloptool ld

# 6. 开始刷写
sudo bash make_flash.sh target=SD \
    uimg=../../../starry_aarch64-opi5p.uimg \
    rootfs=../../../rootfs-aarch64.img
```

---

## 📁 WSL 相关文件

| 文件 | 说明 | 位置 |
|------|------|------|
| **wsl_usb_setup.sh** | WSL USB 设置向导 | `arceos/tools/orangepi5/` |
| **WSL环境使用指南.md** | 详细使用文档 | `arceos/tools/orangepi5/` |
| **make_flash.sh** | 已集成 WSL 检测 | `arceos/tools/orangepi5/` |
| **usb_attach.ps1** | Windows 辅助脚本（自动生成） | 运行 wsl_usb_setup.sh 后生成 |

---

## 🎓 智能特性

### 1. 自动环境检测

```bash
# make_flash.sh 会自动检测 WSL 环境
sudo bash make_flash.sh target=SD uimg=xxx rootfs=xxx

# 如果在 WSL 中且设备未绑定，会显示：
# [WARN] 检测到 WSL 环境
# [WARN] USB 设备需要通过 usbipd-win 绑定到 WSL
# [ERROR] 未检测到 Rockchip 设备
# [ERROR] 快速步骤（在 Windows PowerShell 管理员模式）：
# [ERROR]   1. usbipd list
# [ERROR]   2. usbipd bind --busid X-X
# [ERROR]   3. usbipd attach --wsl --busid X-X
```

### 2. 友好的错误提示

如果设备检测失败，脚本会：
- ✅ 识别 WSL 环境
- ✅ 提供具体的 usbipd 命令
- ✅ 指向设置向导
- ✅ 给出快速解决方案

### 3. 设备状态检查

```bash
# 快速检查设备是否就绪
bash wsl_usb_setup.sh check

# 输出：
# ✓ 检测到 Rockchip Maskrom 设备！
# ✓ rkdeveloptool 可以访问设备
# ✓✓✓ 设备就绪，可以开始刷写！
```

---

## 💡 使用技巧

### 技巧 1: 创建快捷命令

在 `~/.bashrc` 中添加：

```bash
# Orange Pi 5 Plus 快捷命令
alias opi-setup='bash ~/code/os_test/StarryOS/arceos/tools/orangepi5/wsl_usb_setup.sh'
alias opi-check='bash ~/code/os_test/StarryOS/arceos/tools/orangepi5/wsl_usb_setup.sh check'
alias opi-flash='cd ~/code/os_test/StarryOS/arceos/tools/orangepi5 && sudo bash make_flash.sh target=SD uimg=../../../starry_aarch64-opi5p.uimg rootfs=../../../rootfs-aarch64.img'
```

使用：
```bash
opi-setup   # 运行设置向导
opi-check   # 快速检查设备
opi-flash   # 一键刷写
```

### 技巧 2: Windows 快捷脚本

运行 `wsl_usb_setup.sh` 会生成 `usb_attach.ps1`：

```bash
# 在 WSL 中生成脚本
bash wsl_usb_setup.sh
```

将生成的 `usb_attach.ps1` 复制到 Windows 桌面，每次使用时双击运行（管理员权限）。

### 技巧 3: 保持 PowerShell 打开

在 PowerShell 中创建函数：

```powershell
# 在 PowerShell Profile 中添加 ($PROFILE)
function Attach-OrangePi {
    $devices = usbipd list | Select-String "2207:350b"
    if ($devices) {
        $busid = ($devices -split '\s+')[0]
        Write-Host "附加设备 $busid 到 WSL..." -ForegroundColor Green
        usbipd attach --wsl --busid $busid
    } else {
        Write-Host "未找到 Rockchip 设备" -ForegroundColor Red
    }
}

Set-Alias aopi Attach-OrangePi
```

使用：只需在 PowerShell 中输入 `aopi`

---

## 🔍 常见问题速查

### Q1: 每次都需要 attach 吗？

**是的**。每次设备断开重连后都需要重新运行：
```powershell
usbipd attach --wsl --busid X-X
```

但 `bind` 只需要执行一次。

### Q2: 如何快速重新绑定？

**方法 1**: 使用自动生成的 `usb_attach.ps1`  
**方法 2**: 创建 PowerShell 别名（见技巧 3）  
**方法 3**: 保持 PowerShell 窗口打开，直接运行 attach 命令

### Q3: 刷写中途断开怎么办？

```bash
# 1. 在 Windows PowerShell 重新 attach
usbipd attach --wsl --busid X-X

# 2. 在 WSL 中验证
sudo rkdeveloptool ld

# 3. 重新运行刷写
sudo bash make_flash.sh target=SD uimg=xxx rootfs=xxx
```

### Q4: 找不到 usbipd 命令？

**原因**: 未安装或 PowerShell 未重启

**解决**:
```powershell
# 安装
winget install --interactive --exact dorssel.usbipd-win

# 重启 PowerShell
```

### Q5: WSL 中看不到设备？

**检查清单**:
```bash
# 1. WSL 中检查 USB 设备
lsusb | grep -i rockchip

# 2. 如果看不到，在 PowerShell 检查
usbipd list

# 3. 重新 attach
usbipd attach --wsl --busid X-X
```

---

## 📊 完整工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│  WSL + Orange Pi 5 Plus 完整工作流程                        │
└─────────────────────────────────────────────────────────────┘

[首次设置]
    ↓
Windows: 安装 usbipd-win
    ↓
WSL: 安装 USB/IP 工具
    ↓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[日常使用循环]
    ↓
1. 设备进入 Maskrom 模式
    ↓
2. Windows PowerShell (管理员):
   usbipd list                    # 查找 BUSID
   usbipd attach --wsl --busid X-X
    ↓
3. WSL 终端:
   bash wsl_usb_setup.sh check    # 验证
    ↓
4. 编译 (如需要):
   make opi5p
    ↓
5. 刷写:
   cd arceos/tools/orangepi5
   sudo bash make_flash.sh target=SD uimg=xxx rootfs=xxx
    ↓
6. 等待完成
    ↓
7. 设备自动重启
    ↓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[需要重新刷写？]
    ↓
返回步骤 1
```

---

## ✅ 检查清单

### 首次设置检查

- [ ] ✅ Windows 已安装 usbipd-win
- [ ] ✅ WSL 已安装 USB/IP 工具 (linux-tools-generic)
- [ ] ✅ 运行过 `wsl_usb_setup.sh` 了解流程
- [ ] ✅ 设备能进入 Maskrom 模式
- [ ] ✅ PowerShell 中能看到设备 (`usbipd list`)

### 每次刷写检查

- [ ] ✅ 设备已进入 Maskrom 模式
- [ ] ✅ PowerShell 中执行了 `usbipd attach --wsl`
- [ ] ✅ WSL 中 `lsusb` 能看到 Rockchip 设备
- [ ] ✅ `sudo rkdeveloptool ld` 显示 Maskrom
- [ ] ✅ 已编译内核 (`make opi5p`)
- [ ] ✅ rootfs-aarch64.img 存在

---

## 🎉 优势总结

### 与普通 Linux 对比

| 特性 | 普通 Linux | WSL + 我们的方案 |
|------|------------|------------------|
| USB 直接访问 | ✅ 直接 | ❌ 需要转发 → ✅ 自动检测 |
| 设备检测 | ✅ 自动 | ⚠️ 手动绑定 → ✅ 友好提示 |
| 错误提示 | ⚠️ 通用 | ✅ WSL 定制提示 |
| 设置向导 | ❌ 无 | ✅ 完整向导 |
| 文档支持 | ⚠️ 基础 | ✅ 详细 WSL 文档 |

### 我们提供的优化

✅ **自动检测** - 脚本自动识别 WSL 环境  
✅ **友好提示** - 给出具体的 usbipd 命令  
✅ **设置向导** - 一键式设置工具  
✅ **详细文档** - 完整的 WSL 使用指南  
✅ **故障排查** - 常见问题和解决方案  
✅ **辅助脚本** - 自动生成 Windows 端脚本  
✅ **快速检查** - 设备状态一键检测  

---

## 📖 相关文档

| 文档 | 说明 | 位置 |
|------|------|------|
| **WSL环境使用指南.md** | 完整 WSL 使用文档 | `arceos/tools/orangepi5/` |
| **完整刷写方案.md** | 完整刷写技术文档 | `arceos/tools/orangepi5/` |
| **快速开始指南.md** | 5 分钟快速入门 | `arceos/tools/orangepi5/` |
| **README.md** | 工具集总览 | `arceos/tools/orangepi5/` |

---

## 🚀 立即开始

```bash
# 1. 运行设置向导
cd /root/code/os_test/StarryOS/arceos/tools/orangepi5
bash wsl_usb_setup.sh

# 2. 按照向导提示操作

# 3. 开始刷写
sudo bash make_flash.sh target=SD \
    uimg=../../../starry_aarch64-opi5p.uimg \
    rootfs=../../../rootfs-aarch64.img
```

---

## 💬 反馈

如果遇到问题：
1. 查看 `WSL环境使用指南.md` 的故障排查章节
2. 运行 `bash wsl_usb_setup.sh` 获取帮助
3. 检查 usbipd-win 是否最新版本

---

**您现在拥有了最完善的 WSL 环境 USB 设备使用方案！** 🎉

---

**文档版本**: 1.0  
**最后更新**: 2025-10-29  
**适用环境**: WSL 2 + Windows 10/11  
**维护者**: StarryOS Team

