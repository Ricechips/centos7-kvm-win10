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
> 启动libvirtd *systemctl enable libvirtd   systemctl start libvirtd    systemctl status libvirtd*  
> 安装virt-manager *yum -y install virt-manager* (这个是虚拟机图形化管理界面)  

## 显卡透传
>
>
>
>
