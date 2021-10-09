+ dolphin无法打开终端

sudo pacman -S kinit

+ ERROR: ‘/dev/disk/by-label/’ device did not show up after 30 seconds

拔掉装机U盘，重新插入，输入exit

所有设备最好插到后置接口，前置接口很有可能失效

+ 安装新的显卡驱动后开机卡在clean
ctrl + alt + f3
通过mhdw来卸载旧的显卡驱动之后重启
```bash
sudo mhdw -r pci video-vesa
```

