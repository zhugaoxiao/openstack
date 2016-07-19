Devstack 从入门到放弃
# 说明
* 不能使用root用户。
* 默认获取Devstack进行安装，安装的是master分支的代码，但是在实际开发中(比如我们做产品的时候)，都是基于某个stable分支进行，所以一般情况在clone devstack的时候需要指定stable分支。

# 环境准备
* VM准备
安装一台CentOS 7的虚拟机，内存4GB+，CPU 2+，网络NAT网卡1G+
* 环境配置
1、 主机名设置
```
# hostnamectl set-hostname devstack --static
```
2、 关闭selinux
```
# /usr/sbin/sestatus -v
# /usr/sbin/setenforce 0
# sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
```
3、关闭防火墙、网络管理
```
# systemctl stop NetworkManager
# systemctl disable NetworkManager
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```
4、修改系统镜像源
```
# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# yum makecache
```
5、 相关包安装
```
# yum install -y git yum-utils python-testrepository scsi-target-utils net-tools
# yum -y update
# systemctl reboot
```

# devstack安装
* 设置代理
需要配置一下代理，不然部分资源无法下载。
```
# export  http_proxy=192.168.56.1:17000
# export  https_proxy=192.168.56.1:17000
#排除不需要代理的IP地址(可选)
# export no_proxy="127.0.0.1,192.168.56.1,mirrors.aliyun.com, mirrors.aliyuncs.com "
```
下载devstack：
```
# cd /home
# git clone https://github.com/openstack-dev/devstack.git -b stable/mitaka
# git clone https://github.com/openstack-dev/devstack.git -b stable/kilo
```
需要创建stack用户运行
```
# cd /home/devstack/tools/
# bash ./create-stack-user.sh
```
修改devstack目录权限，让stack用户可以运行
```
# chown -R stack:stack /home/devstack
# chmod 777 /opt/stack -R
```
切换到stack用户下
```
# su stack
```
修改pypi源

```
$ mkdir ~/.pip
cat << EOF > ~/.pip/pip.conf
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
EOF

$ cat << EOF > ~/.pydistutils.cfg
[easy_install]
index-url = http://mirrors.aliyun.com/pypi/simple/
EOF
```
更换git源协议
在执行devstack的过程中，要去clone官方的源码，默认用的是git://协议，如果你配置了https的代理，可以更换为https://协议。
```
$ sed -i '/git:/s/git:/https:/' /home/devstack/stackrc
```
开始安装
```
$ cd /home/devstack
$ ./stack.sh
```
安装优化，提前下载
```
$ cd /opt/stack
$ for i in nova cinder glance heat horizon keystone neutron swift neutron-lbaas neutron-fwaas requirements \
heat-cfntools heat-templates tempest \
os-apply-config os-collect-config os-refresh-config \
python-heatclient python-neutronclient python-novaclient python-openstackclient python-troveclient \
dib-utils diskimage-builder tripleo-image-elements trove trove-dashboard
do
git clone https://github.com/openstack/$i
done
```
使用trystack镜像源
```
$ sed -i '/s/git\:\/\/git.openstack.org/http\:\/\/git.trystack.cn/' stackrc
```

get-pip.py优化

手动下载文件检测get-pip.py的方式，这里面有两种方式一种是手动下载get-pip.py之后，注释代码。
```
curl -f  https://bootstrap.pypa.io/get-pip.py /home/vagrant/devstack/files/get-pip.py
```
修改install_pip.sh中PIP_GET_PIP_URL的地址。
```
$ vi devstack/tools/install_pip.sh
PIP_GET_PIP_URL=https://bootstrap.pypa.io/get-pip.py
```
OFFLINE模式下安装Devstack
在Devstack中提供了一种OFFLINE的方式，这种方式的含义就是，当你第一次完成安装后，所有需要的内容已经下载到本地，再次运行就没有必要访问网络了(前提是你不想升级)，所以可以将安装模式设置为OFFLINE，避免网络的访问，方法为：
```
$ vi devstack/localrc
OFFLINE=True
```
使用
其实使用OFFLINE模式，可以在离线状态下无数次重新运行devstack，但是如果不是为了重新配置，我们并没有需要每次重新运行stack.sh。在Devstack中提供了另外一个脚本叫做rejoin-stack.sh，原理很简单就是把所有的进程重新组合进screen，所以我们借助这个脚本完全可以不重新执行stack.sh，快速恢复环境。但是当虚拟机重启后，cinder使用的卷组并不会自动重建，所以在运行rejoin之前，需要将恢复卷组的工作，放入开机启动的脚本中。
```
# cat /etc/init.d/cinder-setup-backing-file
losetup /dev/loop1 /opt/stack/data/stack-volumes-default-backing-file
losetup /dev/loop2 /opt/stack/data/stack-volumes-lvmdriver-1-backing-file
exit 0
```
Run as root
```
# chmod 755 /etc/init.d/cinder-setup-backing-file
# ln -s /etc/init.d/cinder-setup-backing-file /etc/rc2.d/S10cinder-setup-backing-file
```
Run as normal user
```
$ cd $HOME/devstack
$ ./rejoin-stack.sh
```
