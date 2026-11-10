# Raspberry Pi 4 / 5 自动化内核升级与引导说明 (v7.4)

本项目通过 **GitHub Actions** 实现树莓派内核（升级至 `rpi-7.4.y` 长期/候选分支）的跨平台交叉编译、带 `99:` Epoch 版本压制的 `.deb` 打包，以及自动化引导配置，彻底解决官方 `apt` 自动更新覆盖自定义内核及新系统安装卡住的问题。

---

## 一、 系统修改了哪些文件？

在树莓派上安装本项目编译产物时，系统和相关配置文件会发生以下变更：

| 变更文件 / 项目 | 变更类型 | 变更说明与作用 |
| :--- | :--- | :--- |
| `/boot/firmware/vmlinuz-*` (或 `/boot/vmlinuz-*`) | 文件更新 | 放置最新交叉编译生成的自研 Linux 内核镜像文件。 |
| `/boot/firmware/config.txt` | 配置重构 | 自动清除原有的 `kernel=` 和 `initramfs` 配置项，动态写入当前最新内核及对应 `initrd` 引导指向：<br>`kernel=vmlinuz-7.4.0-rc1-v8+`<br>`initramfs initrd.img-7.4.0-rc1-v8+ followkernel` |
| `/etc/initramfs-tools/initramfs.conf` | 参数修改 | 自动将 `MODULES=...` 修改为 `MODULES=most`。<br>**作用**：确保在全新系统或无预设驱动的环境中生成 `initramfs` 初始内存盘时，能够把全部必要设备驱动打包进去，避免触发 `mkinitramfs: failed to determine device for /` 的安装中断报错。 |
| **`dpkg` 包管理优先级** | Epoch 压制 | 生成的 `.deb` 带有 `99:` 前缀（例如 `99:7.4.0-rc1-v8+-...`）。<br>**作用**：在 Debian/Ubuntu 体系中，拥有绝对优先级的 Epoch 能够压制官方源的所有升级提示，防止日常 `sudo apt upgrade` 意外覆盖自定义内核。 |

---

## 二、 压缩包解压后包含什么？

每次 GitHub Actions 构建完成后，在 Release 页面下载对应的 `.tar.gz` 压缩包（如 `kernel-pi4-deb-v7.4.tar.gz`），解压后会包含以下几个关键 `.deb` 文件：
* `linux-image-7.4.0-rc1-v8+_..._arm64.deb` （内核镜像包）
* `linux-headers-7.4.0-rc1-v8+_..._arm64.deb` （内核头文件包，可选安装）
* `linux-libc-dev_..._arm64.deb` （C 库开发头文件包，可选安装）
* `rpi-boot-configurator-pi4_..._all.deb` （核心自动化引导配置工具包）

---

## 三、 解压后全新安装步骤

无论是在全新刷写的树莓派系统上，还是在日常维护升级中，推荐按照以下顺序进行安装：

### 第一步：解压下载的资产
将从 GitHub Releases 下载的压缩包上传至树莓派并解压：
```bash
tar -zxvf kernel-pi4-deb-v7.4.tar.gz && cd deb_output/
```

### 第二步：安装引导配置工具
建议优先或同时安装自动化引导配置包。它会在安装钩子（`postinst`）中自动将 `initramfs.conf` 调整为 `most` 模式：
```bash
sudo dpkg -i rpi-boot-configurator-*.deb
```

### 第三步：安装内核及相关 deb 包
随后安装核心内核镜像及头文件包：
```bash
sudo dpkg -i linux-image-*.deb linux-headers-*.deb linux-libc-dev_*.deb
```
*(注：如果在安装过程中遇到任何未完成配置的提示，可随时执行 `sudo dpkg --configure -a` 进行自愈收尾)* 

### 第四步：重启树莓派
安装完成后，直接重启系统使新内核生效：
```bash
sudo reboot
```

### 第五步：验证是否成功
重启完成后通过 SSH 重新连接树莓派，输入以下命令检查当前运行的内核版本：
```bash
uname -a
```
预期应返回类似 `Linux pi4b-4g 7.4.0-rc1-v8+ ... aarch64` 的版本信息，即代表 7.4 内核与自动化引导已完美运行！
