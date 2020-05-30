# DEbian8.0环境适配
## 关于找不到/mnt/hgfs文件夹
* 确保安装好vmware-tools后，设置完共享文件夹， 命令行`vmware-hgfsclient`后，如果终端显示了你创建的共享文件夹则证明设置完成；
* 前提准备好后安装相关软件：`apt-get install open-vm-dkms`当然一般肯定是提示找不到这个软件包的（如果成功了，则：`mkdir /mnt/hgfs`后`mount -t vmhgfs .host:/ /mnt/hgfs`），建议自行换源或网上自下软件包，或者get另一软件：`apt-get install open-vm-tools-dkms`
* 安装完**open-vm-tools-dkms**后，手动创建**hgfs**文件夹：`mkdir /mnt/hgfs`，尔后进行挂载：`vmhgfs-fuse .host:/ /mnt/hgfs`，报错？那你肯定没安装**fuse**这个🖊：`apt-get install fuse`
* 这样就手动挂载完毕了，但这有个问题，重启就没了，所以为了一劳永逸，可以修改*/etc/fstab*文件：
    * 如果之前用的是**open-vm-kms**,那么在*fstab*文件最后加入：`.host:/sharefolder /mnt/hgfs vmhgfs defaults, nonempty 0 0`，重启（其中，**sharefolder**为共享文件夹名字，当然你可以直接`.host:/`默认挂载全部共享文件夹）；
    * 如果用的是**open-vm-tools-dkms**，那么在其最后加入： `.host:/sharefolder /mnt/hgfs fuse.vmhgfs-fuse allow_other, defaults, nonempty 0 0`，重启，完毕；

## 关于windows共享给虚拟机linux的文件，行尾存在^M问题：
 * 在VIM命令模式下输入：`:%s/^M$//g`，**注意**：^M要用 *ctrl + v, vtrl + m*来输入；
