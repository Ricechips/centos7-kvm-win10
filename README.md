# 工作要求，搞了一个礼拜，结合了各大blog文章、各大网站教程，终于把透传完成了，现在总结一下流程8.

## 环境
> `host机:centos7.7` kvm虚拟化 `虚机：win10`

## u盘安装centos
> bios设置u盘启动  
> 进入安装界面按TAB修改路径  
> 将vmlinuz initrd ... quiet 改为 vmlinuz initrd=initrd.img linux dd quiet 找到u盘设备名 (我的是sda1)  
> ctrl+alt+del重启 再次配置 vmlinuz ... stage2=hd:/dev/sda1 quiet   
> 后面正常安装  

## kvm安装
> *yum install -y qemu-kvm libvirt virt-install bridge-utils* （依次是 用户态 命令行 工具 桥接设备)  
> 查询kvm模块 *lsmod | grep kvm*  
> 启动libvirtd<br> *systemctl enable libvirtd <br>  systemctl start libvirtd <br>   systemctl status libvirtd*  
> 安装virt-manager <br>*yum -y install virt-manager* (这个是虚拟机图形化管理界面)  

## 显卡透传
> - 配置iommu *vim /etc/default/grub* 将*intel_iommu=on*放在*GRUB_CMDLINE_LINUX=“ ”*里面  
> 更新grub *grub2-mkconfig -o /boot/grub2/grub.cfg*  之后`reboot`  
> 用*dmesg | grep -e DMAR -e IOMMU*看是否有IOMMU enabled输出  
> - lspci 找到要透传的显卡，记下ID 如：83:00:0 AMD 83:00:1 Audio  
> *lspci -vv -s 83:00:0 | grep driver*可查其驱动  
> 查询到之后将驱动禁用 *vim /etc/modprobe.d/blacklist.conf* `blacklist radeon` `blacklist snd_hda_intel`  
> - 加载vfio驱动<br> *modprobe vfio*<br> *modprobe vfio-pci*  
> - 从`host机`卸载A卡 <br>*virsh nodedev-detach pci_0000_83_00_0* <br>*virsh nodedev-detach pci_0000_83_00_1*  
> 此时再查询驱动，得到*kernel driver in use: vfio-pci*

## 安装OVMF
> - *wget http://www.kraxel.org/repos/firmware.repo -o /etc/yum.repos.d/firmware.repo*配置yum源  
> *yum install edk2.git-ovmf-x64*  
> - 配置libvirtd<br> *vim /etc/libvirt/qemu.conf* <br>`nvram = [ "/usr/share/edk2.git/ovmf-x64/OVMF_CODE-pure-efi.fd:/usr/share/edk2.git/ovmf-x64/OVMF_VARS-pure-efi.fd", ]`  
> - 重启libvirtd *systemctl restart libvirtd*

## 安装win10
> virt-manager 打开图形管理界面  
> 在配置中以UEFI启动  
> CDROM添入win10.iso  
> 这里的引导项有：`1.CDROM(win10.iso) 2.CDROM(virtio.iso) 3.VirtIO磁盘(100G)`  
> 添加PCI设备(83:00:0和83:00:1)  
> 之后正常装系统  
> ps:virtio是对此win虚机进行优化，性能实测有质的提升 可在[此处](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)下载 对了，网卡也可以改成virtio,具体路径在`NetKVM\w7\amd64`

## usb透传
> lsusb查询项透传的usb设备  
> 在manager上面直接添加(修改配置文件蛮麻烦的，图形化轻松搞定)

### 一些心得
这东西做出来也算是磕破了头吧，查了不知道多少的资料，每一篇都有相似的操作，但又不尽相同，如果按某一篇en头做，基本是失败的，我的做法是结合每篇的精华操作，把流程搞清了，每一步做的意义是什么，明白了之后再具体实现其步骤，有针对性找操作流程，当然，同时不能忘记对硬件的考虑，如`硬盘接口脱落，内存条插了总内存反而变少了，主板太热了，无线鼠标会被金属板隔离传输导致卡顿.....`行吧，下个任务见。
