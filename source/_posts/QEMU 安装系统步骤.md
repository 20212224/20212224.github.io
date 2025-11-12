---
title: QEMU 安装系统记录 (windows)
date: 2025-07-10 16:41:35
tags: virtual
---

# QEMU 安装系统记录 (windows)
1. 下载安装[QEMU](https://www.qemu.org/)
2. 安装虚拟机(win)
    - 创建虚拟硬盘
      ```
      qemu-img create -f qcow2 ".\windows11.qcow2" 60G
      ```
    - 安装系统(需要VirtIO的话这里可以加上,不然后面有点麻烦) (安装win11 需要改注册表绕过检查, 绕过账号登录)
      ```
      qemu-system-x86_64 `
      -m 8G ` # 内存
      -smp 4 ` # 核心
      -accel whpx ` # 加速
      -boot d ` # 从CD/DVD启动
      -cdrom ".\ISOs\Win11_23H2_Chinese_Simplified_x64v2.iso" ` # 挂载CD/DVD
      -drive file=".\windows11.qcow2",format=qcow2 ` # 挂载虚拟硬盘 需要可以加上if=virtio
      -netdev user,id=net0 ` # 指定网络模式和设备id net0
      -device e1000,netdev=net0 ` # 添加虚拟网卡绑定到网络后端  net0
      -usb -device usb-tablet # 添加虚拟USB平板输入设备, 改善鼠标精度

      # virtio
      qemu-system-x86_64 `
      -m 8G `
      -smp 4 `
      -accel whpx `
      -boot d `
      -cdrom ".\ISOs\Win11_23H2_Chinese_Simplified_x64v2.iso" `
      -drive file=".\ISOs\virtio-win-0.1.271.iso",media=cdrom,readonly=on `
      -drive file=".\windows11.qcow2",format=qcow2,if=virtio `
      -netdev user,id=net0 `
      -device virtio-net-pci,netdev=net0 `
      -usb -device usb-tablet `
      -vga virtio `
      -display sdl,gl=on
      ```
    - 启动命令
      ```
      # 根据是否使用virtio 添加替换部分配置
      qemu-system-x86_64 `
      -m 8G `
      -smp 4 `
      -accel whpx `
      -drive file=".\windows11.qcow2",format=qcow2 `
      -netdev user,id=net0 `
      -device e1000,netdev=net0 `
      -usb -device usb-tablet
      ```
  3. 安装 [VirtIO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/)(已安装的系统)
      ```
      qemu-system-x86_64 `
      -m 8G `
      -smp 4 `
      -accel whpx `
      -drive file=".\ISOs\virtio-win-0.1.271.iso",media=cdrom,readonly=on ` # 挂载为只读光驱
      -drive file=".\windows11.qcow2",if=virtio,format=qcow2 ` # 使用 VirtIO 总线接口 这里先不写 if=virtio
      -netdev user,id=net0 `
      -device virtio-net-pci,netdev=net0 `  # 将virtio网络设备连接到虚拟机
      -usb -device usb-tablet `
      -vga virtio ` # 指定virtio虚拟显卡
      -display sdl,gl=on # 指定sdl创建显示窗口, 尝试opengl加速
      ```
      - 测试发现不需要安全模式也可以,只要挂载一个使用`virtio`的硬盘,然后再系统安装驱动就可以了
      - 进入系统后找到挂载的virtio驱动盘,进入点击安装对应版本驱动(不知道为什么硬盘这个驱动反正就是安装不成功,有个感叹号,如果可以还是安装系统时就加载这个驱动,不然要按照下面的方法有点麻烦)

      - 前面的问题`if=virtio`,写这个参数对于已经安装的系统会找不到启动盘 `INACCESSIBLE BOOT DEVICE`, 需要VM开启安全模式,然后挂载一个空硬盘`-drive file=".\empty.qcow2,if=virtio",if=virtio` 再启动, 进入系统后virtio硬盘需要的驱动会安装好, 然后退出安全模式->关机-> 去掉空硬盘挂载,在系统盘补上`if=virtio`->正常命令开机ok了[问题地址](https://superuser.com/questions/1057959/windows-10-in-kvm-change-boot-disk-to-virtio#)
        + `bcdedit /set "{current}" safeboot minimal` 启动安全模式
        + `bcdedit /deletevalue "{current}" safeboot` 关闭安全模式

  4. 桥接网络
      - 下载安装`tap-windows`驱动[https://build.openvpn.net/downloads/releases/](https://build.openvpn.net/downloads/releases/)
      - 安装完成后,在网络适配器了会多一个tap设备->修改名字为tap0方便后面使用->选择这个设备和物理网卡,右键桥接,如果失败就从网桥的属性重新勾选两个设备
      - 在启动命令中加上 `-netdev tap,id=net0,ifname=tap0`
          + `tap` 指定为tap模式
          + `id=net0` 网络后端id
          + `ifname=tap0` 绑定到宿主机的tap0


