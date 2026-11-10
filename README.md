# Raspberry Pi 4 / 5 自动化内核升级与引导说明 (v7.4)

本项目通过 GitHub Actions 实现树莓派内核（升级至 `rpi-7.4.y` 长期/候选分支）的跨平台交叉编译、带 `99:` Epoch 版本压制的 `.deb` 打包，以及自动化引导配置，彻底解决官方 `apt` 自动更新覆盖自定义内核及新系统安装卡住的问题。

---
## 一、 系统修改了哪些文件？

在树莓派上安装本项目编译产物时，系统和相关配置文件会发生以下变更：

1. **`/boot/firmware/vmlinuz-*` (或 `/boot/vmlinuz-*`)**
   - **变更说明**：放置了最新交叉编译生成的自研 Linux 内核镜像文件。
2. **`/boot/firmware/config.txt`**
   - **变更说明**：自动清除了原有的 `kernel=` 和 `initramfs` 配置项，并动态写入了当前最新的内核及对应的 `initrd` 引导指向：
     ```text
     kernel=vmlinuz-7.4.0-rc1-v8+
     initramfs initrd.img-7.4.0-rc1-v8+ followkernel
     ```
3. **`/etc/initramfs-tools/initramfs.conf`**
   - **变更说明**：自动将 `MODULES=...` 修改为了 `MODULES=most`。
   - **作用**：确保在全新系统或无预设驱动的环境中生成 `initramfs` 初始内存盘时，能够把全部必要设备驱动打包进去，避免触发 `mkinitramfs: failed to determine device for /` 的安装中断报错。
4. **`dpkg` 包管理优先级（Epoch 压制）**
   - **变更说明**：生成的 `.deb` 带有 `99:` 前缀（例如 `99:7.4.0-rc1-v8+-...`）。
   - **作用**：在 Debian/Ubuntu 体系中，拥有绝对优先级的 Epoch 能够压制官方源的所有升级提示，防止日常 `sudo apt upgrade` 意外覆盖自定义内核。

---

## 二、 压缩包解压后包含什么？

每次 GitHub Actions 构建完成后，在 Release 页面下载对应的 `.tar.gz` 压缩包（如 `kernel-pi4-deb-v7.4.tar.gz`），解压后会包含以下几个关键 `.deb` 文件：
* `linux-image-7.4.0-rc1-v8+_..._arm64.deb` （内核镜像包）
* `linux-headers-7.4.0-rc1-v8+_..._arm64.deb` （内核头文件包，可选安装）
* `linux-libc-dev_..._arm64.deb` （C 库开发头文件包，可选安装）
* `rpi-boot-configurator-pi4_..._all.deb` （**核心自动化引导配置工具包**）

---
第二步：安装引导配置工具
建议优先或同时安装自动化引导配置包。它会在安装钩子（postinst）中自动将 initramfs.conf 调整为 most 模式：

sudo dpkg -i rpi-boot-configurator-*.deb
第三步：安装内核及相关 deb 包
随后安装核心内核镜像及头文件包：
sudo dpkg -i linux-image-*.deb linux-headers-*.deb linux-libc-dev_*.deb
(注：如果在安装过程中遇到任何未完成配置的提示，可随时执行 sudo dpkg --configure -a 进行自愈收尾)

第四步：重启树莓派
安装完成后，直接重启系统使新内核生效：
sudo reboot
第五步：验证是否成功
重启完成后通过 SSH 重新连接树莓派，输入以下命令检查当前运行的内核版本：
uname -a
预期应返回类似 Linux pi4b-4g 7.4.0-rc1-v8+ ... aarch64 的版本信息，即代表 7.4 内核与自动化引导已完美运行！
