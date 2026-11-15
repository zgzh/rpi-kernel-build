# Raspberry Pi Kernel Build

本项目通过 GitHub Actions 实现树莓派官方内核的跨平台交叉编译、带 99: Epoch 版本压制的 .deb 打包，以及自动化引导与 initramfs 防护配置。

## 一、 系统修改与防护机制
1. 内核镜像文件: /boot/firmware/vmlinuz-* (放置自研交叉编译内核镜像)
2. 引导配置文件: /boot/firmware/config.txt (自动写入 kernel 及 initrd 引导项)
3. Initramfs 防护: /etc/initramfs-tools/conf.d/99-rpi-modules.conf (锁定 MODULES=most，彻底防止 initramfs 报错)
4. Dpkg 包管理优先级: 带有 99: Epoch 前缀，压制官方源升级

## 二、 Release 页面独立 .deb 包
- linux-image-*-v8+_arm64.deb (核心内核镜像，必需)
- rpi-boot-configurator-pi4/pi5 (自动化引导配置与防护工具，必需)
- linux-headers-*-v8+_arm64.deb (内核头文件，可选)

## 三、 安装与验证步骤
第一步，优先安装引导配置包：
sudo dpkg -i rpi-boot-configurator-*.deb

第二步，安装内核镜像及头文件：
sudo dpkg -i linux-image-*.deb linux-headers-*.deb

第三步，重启树莓派：
sudo reboot

第四步，验证内核与 BTF 状态：
uname -a &&
ls -lah /sys/kernel/btf/vmlinux
