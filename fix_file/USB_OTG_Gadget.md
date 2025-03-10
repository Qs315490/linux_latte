# USB Gadget
负责与电脑通讯，设置usb otg为设备模式即可连接电脑。

usb 串口、共享网络、adb都是基于此功能实现。

# 开启方法
如果是解锁所有选项的BIOS，设置`USB OTG Support`为`PCI Mode`

原厂BIOS，BIOS的setup变量0x2a2偏移地址。  
值从0x3(AUTO)改为0x1(PCI)

推荐修改工具|使用方法
-|-
[setup_var.efi](https://github.com/datasone/setup_var.efi)|efi shell执行`setup_var.efi Setup:0x2a2=0x1`
grub的setup_var模块|grub shell执行`setup_var 0x2a2 0x1`
[ru.efi](https://ruexe.blogspot.com/)|未验证 ~~alt+=, ctrl+↓两次, 按方向键直到左上角为0x2a2, 直接输入01, ctrl+w保存~~

# USB OTG Support
值|描述
-|-
0x0|Disable
0x1|PCI Mode
0x2|ACPI Mode
0x3|AUTO Mode 与 ACPI Mode 作用相同