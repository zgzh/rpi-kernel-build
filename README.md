# Raspberry Pi Kernel Build

## 1. 核心内核文件
- 路径: /boot/firmware/vmlinuz-*
- 说明: 自研交叉编译 Linux 内核镜像。

## 2. 引导配置
- 路径: /boot/firmware/config.txt
- 说明: 自动写入 kernel 及 initrd 引导项。

## 3. 防护配置
- 路径: /etc/initramfs-tools/conf.d/99-rpi-modules.conf
- 说明: 锁定 MODULES=most，防止 initramfs 报错。

## 4. 安装步骤
第一步，安装引导配置包：
sudo dpkg -i rpi-boot-configurator-*.deb

第二步，安装内核镜像：
sudo dpkg -i linux-image-*.deb linux-headers-*.deb

第三步，重启系统：
sudo reboot
