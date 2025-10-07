# 仰卧起坐计数器 - Android APK打包指南

## 项目概述

本项目包含以下关键文件：

- `situp_counter_kivy.py`：使用Kivy框架重写的仰卧起坐计数器应用程序
- `buildozer.spec`：Buildozer配置文件，包含应用信息和依赖配置
- `test_kivy_app.sh`：用于验证Kivy应用程序运行环境的测试脚本

这个项目使用Python开发了一个仰卧起坐计数器应用，现在我们将把它打包成Android APK文件，使其可以在安卓手机上运行。

## 打包准备工作

### 环境要求
- 建议在Linux或macOS系统上进行打包（Windows用户建议使用WSL）
- 需要安装Python 3.8+、Git、OpenJDK 8
- pip：最新版本
- Android SDK和NDK：通过Buildozer自动管理

### 步骤1：安装Buildozer和必要依赖

在终端中运行以下命令：

#### 在Linux或WSL上安装

```bash
# 更新包列表
sudo apt update
# 安装必要的依赖
sudo apt install -y git zip unzip openjdk-8-jdk python3-pip autoconf libtool pkg-config zlib1g-dev libncurses5-dev libncursesw5-dev libtinfo5 cmake build-essential libffi-dev libssl-dev
# 升级pip
pip3 install --upgrade pip
# 安装Buildozer
pip3 install buildozer
# 安装Cython（Kivy需要）
pip3 install cython
```

#### 在macOS上安装

```bash
# 安装Homebrew（如果尚未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# 安装必要的依赖
brew install git openjdk@8 python3 autoconf libtool pkg-config zlib cmake libffi openssl
# 升级pip
pip3 install --upgrade pip
# 安装Buildozer
pip3 install buildozer
# 安装Cython（Kivy需要）
pip3 install cython
```

### 步骤2：初始化Buildozer（已完成）

已为您创建了`buildozer.spec`配置文件，其中包含了必要的设置：
- 应用名称：仰卧起坐计数器
- 包名：org.example.situpcounter
- 所需依赖：python3, kivy, opencv-python, numpy, mediapipe
- Android权限：CAMERA
- 支持的架构：armeabi-v7a, arm64-v8a

项目中已包含预配置的`buildozer.spec`文件，关键配置如下：

```ini
# 应用程序基本信息
title = 仰卧起坐计数器
package.name = situpcounter
package.domain = org.example
version = 0.1

# 需求列表
requirements = python3,kivy,opencv-python,numpy,mediapipe

# Android配置
android.api = 28
android.minapi = 21
android.ndk = 21.3.6528147
android.sdk = 28
android.src.folder = .

# 权限设置
android.permissions = CAMERA

# 架构设置
android.archs = armeabi-v7a,arm64-v8a

# 自动添加的source.include_exts = py,png,jpg,kv,atlas
```

如需自定义配置，请直接编辑`buildozer.spec`文件。

### 步骤3：创建必要的目录结构

```bash
# 在项目根目录创建assets文件夹（如果需要放置模型文件等）
mkdir -p assets
```

### 步骤4：测试Kivy应用程序

在进行APK打包前，建议先测试Kivy应用程序是否能正常运行：

```bash
# 运行测试脚本
chmod +x test_kivy_app.sh
./test_kivy_app.sh
```

此脚本会检查所有依赖项是否已安装，并尝试启动应用程序。测试脚本以退出码0成功完成，说明我们已经解决了应用程序中的问题。

## 开始打包

在项目根目录下运行以下命令开始打包APK：

```bash
# 调试版本打包
buildozer -v android debug

# 或者直接部署到已连接的设备
buildozer android debug deploy run
```

> **注意**：首次运行此命令会下载和配置Android SDK、NDK等依赖，这可能需要较长时间，请耐心等待。

## 打包过程中可能遇到的问题及解决方案

### 问题1：首次打包时下载依赖时间过长

**解决方案**：这是正常现象，首次打包需要下载Android SDK、NDK等大量资源，请耐心等待。

### 问题2：MediaPipe库无法正常打包

**解决方案**：如果在打包过程中遇到MediaPipe相关的错误，可能需要修改buildozer.spec文件中的依赖配置，或者考虑使用自定义的buildozer recipes。确保buildozer.spec中的requirements包含正确版本的MediaPipe，也可以尝试使用较低版本的MediaPipe以提高兼容性。

### 问题3：OpenCV库相关错误

**解决方案**：确保在buildozer.spec文件中正确包含了opencv-python依赖，有时可能需要指定特定版本。在buildozer.spec中添加以下配置可能会有所帮助：
```ini
source.include_patterns = assets/*,images/*.png,cv2/*,*.so
```

### 问题4：内存不足错误

**解决方案**：打包过程需要大量内存，建议关闭其他占用内存的程序，或增加系统交换空间。确保您的系统至少有4GB RAM和10GB可用磁盘空间。对于WSL用户，可以考虑扩展WSL的虚拟硬盘。

### 问题5：网络问题

由于需要下载大量依赖，网络问题可能导致打包失败。建议使用稳定的网络连接，必要时可以配置代理。

## 生成的APK文件位置

成功打包后，APK文件将位于项目目录下的`bin`文件夹中，文件名类似于：`situpcounter-0.1-debug.apk`

打包完成后，您可以在以下位置找到生成的APK文件：

```
buildozer.spec
|-- bin/
    |-- situpcounter-0.1-debug.apk
```

## 安装APK到手机

有两种方法可以将APK安装到您的Android设备：

1. 通过USB连接设备并启用调试模式，然后使用以下命令：
   ```bash
   adb install ./bin/situpcounter-0.1-debug.apk
   ```

2. 将APK文件复制到您的设备上，然后使用文件管理器手动安装

> 注意：安装前需要在设备上启用"未知来源应用"的安装权限

## 常见问题解答

### Q: 为什么需要使用Kivy框架？
**A**: Kivy是一个跨平台的Python GUI框架，它允许我们使用Python开发可以在Android上运行的应用程序。

### Q: 应用在手机上运行时卡顿怎么办？
**A**: 可以尝试降低摄像头分辨率，减少处理帧率，或者在buildozer.spec中优化NDK配置。

### Q: 如何自定义应用图标和启动画面？
**A**: 修改buildozer.spec文件中的`icon.filename`和`presplash.filename`配置项，然后提供相应的图片文件。

## 应用功能说明

修复后的Kivy版仰卧起坐计数器应用程序具有以下功能：

1. **摄像头实时姿态检测**：使用MediaPipe进行人体姿态估计
2. **仰卧起坐自动计数**：根据身体角度变化自动计数
3. **位置提示**：提供用户正确姿势的视觉指导
4. **重置计数**：通过按钮可以重置计数
5. **安全退出**：确保资源正确释放的退出机制
6. **健壮的错误处理**：包含全面的异常处理，提高应用稳定性

## 重要说明

1. 由于MediaPipe和OpenCV库的复杂性，首次打包可能会遇到各种问题，需要根据具体错误进行调试。
2. 本指南提供的是基础打包流程，实际操作中可能需要根据您的系统环境和配置进行调整。
3. 如果您在打包过程中遇到无法解决的问题，建议查阅Kivy和Buildozer的官方文档或寻求社区支持。

---

希望这份指南能帮助您成功将仰卧起坐计数器应用打包为Android APK！如有任何问题，请随时寻求帮助。
