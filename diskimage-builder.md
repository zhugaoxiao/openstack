# diskimage-builder那点事

diskimage-builder 是一个用于为云环境构建操作系统镜像的工具，是TripleO项目的子项目。它支持当前主流的Linux版本并且可以生成各种通用格式（qcow2,vhd,raw等），裸机文件系统镜像和ram-disk镜像。

支持下列系统类型，暂时不支持Windows系列，Windows系列可以使用另外的工具：
- fedora
- debian
- ubuntu
- centos
- rhel
- gentoo

DiskImage-Builder是通过chroot到一个创建的临时目录中(默认在/tmp/image.*中可以看到)，同时绑定系统的/proc, /sys, 和/dev 目录来配置(硬件资源)环境的。当提供好的脚本执行完成后，将该tmp目录内容打入到镜像文件中即完成了所有的制作过程。

DiskImage-Builder提供了一个完整的执行环境,那么要定制一个满足自己需求的镜像,只需要按照其提供的格式完成几个element, 接着就可以完成定制镜像了。

## element

element是一堆符合特定名称的文件(主要为脚本)/文件夹的集合。其主要包含以下的元素:

* 用于执行脚本, 命名及存放的路径均有特殊含义(这些脚本中描述了在制作镜像的过程中需要执行哪些操作,比如安装好apache,创建用户等.)
* 依赖描述
* 描述文件, 不是强制的，但提供这个就相当于有一个良好的注释, 便于他人阅读和使用

## element的执行脚本

DiskImage-Builder默认提供了一些基础的element，可以在源码目录中diskimage-builder/elements中看到。其中执行的脚本，是按照目录划分的，按照顺序执行。每一个目录操作都有几个属性：

* 其执行的环境,是否在chroot中
* 输入变量
* 输出变量
以下列出了其执行的目录, 执行按先后顺率来排列


操作|	执行目录|	接受变量|	输出变量
---|---|---|---|---
root.d	    | outside chroot | $ARCH=i386,amd64,armhf $TARGET_ROOT={path} |	-|
extra-data.d|	outside chroot |	$TMP_HOOKS_PATH	|-
pre-install.d|	in chroot    |	-|	-|
install.d    |	in chroot    |	-|	-|
post-install.d | in chroot   |	-|	-|
block-device.d | outside chroot |	$IMAGE_BLOCK_DEVICE={path} $TARGET_ROOT={path}	|$IMAGE_BLOCK_DEVICE={path}
finalise.d	  | in chroot    |	-|	-|
cleanup.d     |	outside chroot |	$ARCH=i386,amd64,armhf $TARGET_ROOT={path}|	-|


## element之间的依赖

element的依赖主要由两个文件来描述

* element-deps:

描述该elements所依赖的element, 那么在执行该elment之前会先执行其依赖的element。如默认的element: ubuntu
```
cache-url cloud-init-datasources dib-run-parts dkms dpkg
```
也就是说在执行elementubuntu之前会先去执行如dkms,dpkg,cache-url等所依赖的element。

* element-provides:

描述该elements额外提供哪些element的功能, 也就说若执行该element,那么其额外提供的element就不会在执行。依然拿ubuntu来举例：
```
operating-system
```
这个就说明,若选择了ubuntu，则不会执行名为operating-system的element了。

## 安装

环境要求
4 GB MEM，/tmp > 2GB
* pip安装
```
# yum install -y python-pip
# pip install diskimage-builder pyyaml
```
* yum安装
```
# yum install -y diskimage-builder PyYAML
```
* 源码安装
```
# yum install -y git PyYAML
# git clone https://github.com/openstack/diskimage-builder.git
# git clone https://github.com/openstack/dib-utils.git
# cat <EOF> ~/.profilefile
PATH=$PATH:$(pwd)/diskimage-builder/bin:$(pwd)/dib-utils/bin
EOF
```

## 使用

### 构建镜像

* 常用参数

-a i386,amd64,armhf
-t qcow2,tar,vhd,docker,aci,raw
-u 不压缩
-c 清空环境变量
-n 跳过默认 'base' element
-p 指定相关包


* rhel系列

```
# export DIB_DISTRIBUTION_MIRROR=http://mirrors.aliyun.com/centos
# export DIB_CLOUD_IMAGES=http://cloud.centos.org/centos/7/images
# export DIB_EPEL_MIRROR=http://mirrors.aliyun.com/epel
# DIB_DEBUG_TRACE=0

# disk-image-create -a amd64 -o centos7 vm centos7
# disk-image-create -a amd64 -o centos vm centos

# disk-image-create -a amd64 -o rhel-baremetal base rhel baremetal

# disk-image-create -a amd64 -o fedora vm fedora  
# disk-image-create -a amd64 -o fedora baremetal fedora local-conf dhcp-all-interfaces
```

* ubuntu

```
# export DIB_RELEASE=trusty
# export DIB_DISTRIBUTION_MIRROR=http://cn.archive.ubuntu.com/ubuntu
# export DIB_LOCAL_IMAGE=/root/image/centos7-custom.qcow2
# export DIB_CLOUD_INIT_DATASOURCES="Ec2, ConfigDrive, OpenStack"
# export DIB_CLOUD_IMAGES=http://cloud-images.ubuntu.com

# disk-image-create -a amd64 -o ubuntu vm ubuntu
# disk-image-create -a amd64 -o ubuntu-baremetal vm base ubuntu baremetal
# disk-image-create -a amd64 -o ubuntu-nocloud vm base ubuntu cloud-init-nocloud
# disk-image-create -a amd64 -o ubuntu-mysql -p mysql-server,tmux vm ubuntu
```


### 构建pxe镜像

构建pxe镜像，用于ironic，会包括initramfs和kernel两个文件。其实这个命令就是disk-image-create

```
# ramdisk-image-create -a amd64 pxe-deploy

# ramdisk-image-create -a amd64 -o deploy-baremetal ubuntu deploy-baremetal

# ramdisk-image-create -a amd64 -o deploy-ironic.ramdisk ubuntu deploy-ironic
```

### 构建自定义镜像

* nginx
创建 nginx element

```
# mkdir –p ~/elements/nginx
# cd ~/elements/nginx
# mkdir install.d
# touch 15-nginx
# chmod a+x 15-nginx

# echo "
!/bin/bash
set -eux
install-packages nginx" >15-nginx

# disk-image-create -a amd64 -o base-nginx vm base baremetal nginx
```

* mongoDB
创建 mongodb element
```
# mkdir –p ~/elements/mongodb
# cd ~/elements/mongodb
# mkdir pre-install.d
# touch 10-mongodb
# chmod a+x 10-mongodb
# echo "
!/bin/bash
set -eux
echo "[mongodb]" > /etc/yum.repos.d/mongodb.repo
echo "name=mongodb_repo" >> /etc/yum.repos.d/mongodb.repo
echo "baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/" >>
echo "enabled=1" >> /etc/yum.repos.d/mongodb.repo
echo "gpgcheck=0" >> /etc/yum.repos.d/mongodb.repo
" >10-mongodb

# mkdir install.d
# touch 10-mongodb
# chmod a+x 10-mongodb

# echo "
!/bin/bash
set -eux
install-packages mongo-10gen mongo-10gen-server">10-mongodb
# cd ..

# mkdir finalise.d
# touch 10-mongodb
# chmod a+x 10-mongodb
# echo "
!/bin/bash
set -eux
#rm -Rf /etc/yum.repos.d/mongodb.repo">10-mongodb
# cd ..

# disk-image-create –a amd64 –o rhel-mongodb base rhel baremetal mongodb
```

### 构建tripleo 镜像

```
# git clone https://git.openstack.org/openstack/tripleo-image-elements.git
# git clone https://git.openstack.org/openstack/heat-templates.git
# export ELEMENTS_PATH=tripleo-image-elements/elements:heat-templates/hot/software-config/elements

# export BASE_ELEMENTS="centos7 selinux-permissive"
# export AGENT_ELEMENTS="os-collect-config os-refresh-config os-apply-config"
# export DEPLOYMENT_BASE_ELEMENTS="heat-config heat-config-script"
# export DEPLOYMENT_TOOL="heat-config-cfn-init heat-config-puppet"
# export IMAGE_NAME=centos7-software-config

# disk-image-create -a amd64 -o fedora-heat-cfntools vm fedora heat-cfntools
```

### 构建trove 镜像

下载相关文件
```
# git clone https://github.com/openstack/tripleo-image-elements.git
# git clone https://github.com/denismakogon/trove-guest-image-elements.git
# export ELEMENTS_PATH=tripleo-image-elements/elements:trove-guest-image-elements/elements:diskimage-builder/elements
```

版本选择
```
Ubuntu:
# export DISTRO=ubuntu
# export DIB_RELEASE=trusty
CentOS:
# export DISTRO=centos
# export DIB_RELEASE=GenericCloud
# export DIB_EXTLINUX=1
```

仅开发模式:
```
export DIB_DEV_USER_USERNAME=ks
export DIB_DEV_USER_PASSWORD=w8EmsJXTMaG0e97NK8lM
export DIB_DEV_USER_SHELL=/bin/bash
export DIB_DEV_USER_PWDLESS_SUDO=true
export DIB_DEV_USER_AUTHORIZED_KEYS=/home/sa709c/.ssh/authorized_keys
```

Trove Guest agent 配置:
```
export DIB_TROVE_RABBITMQ_HOSTS=172.24.4.18:5672
export DIB_TROVE_RABBIT_USERID=guest
export DIB_TROVE_RABBIT_PASSWORD=guest
export DIB_TROVE_RABBIT_USE_SSL=false
export DIB_TROVE_AUTH_URL=http://172.24.8.18:35357/v2.0
export DIB_TROVE_NOVA_PROXY_ADMIN_USER=nova
export DIB_TROVE_NOVA_PROXY_ADMIN_PASS=devstack
export DIB_TROVE_NOVA_PROXY_ADMIN_TENANT_NAME=service
export DIB_TROVE_SWIFT_URL=http://172.24.8.18:8080/v1/AUTH_
export DIB_TROVE_SWIFT_SERVICE_TYPE=object-store
export DIB_TROVE_BACKUP_SWIFT_CONTAINER=database_backups

export DATASTORE="postgresql"
export DATASTORE="mysql"
export DATASTORE="cassandra"
export DATASTORE_VERSION="2.1.1"

disk-image-create -a amd64 \
    -o ${DISTRO}-${DATASTORE}-${DATASTORE_VERSION}-guest-image \
    -x --qemu-img-options compat=0.10 \
    ${DISTRO}-${DATASTORE}-guest-image

Note. Only anonymous HTTP(S) accessable Git repos are allowed.
```       

选择安装ssh：
```
openssh-server
```
如果为vCenter编译镜像，选择安装vmware-tools:
```
${DISTRO}-vmware-tools
```
>注意：
1、这个配置仅对 Debian/Ubuntu 14.04 Trusty Tahr和CentOS 7.x有效。
2、如果key导入失败，可手动修改key导入命令，elements\ubuntu-mysql\pre-install.d\10-percona-apt-key。
wget -O - http://www.percona.com/redir/downloads/RPM-GPG-KEY-percona | gpg --import
gpg --armor --export 1C4CBDCDCD2EFD2A | apt-key add -


```
# git clone https://github.com/openstack/trove-integration.git
# git clone https://github.com/openstack/tripleo-image-elements.git

export HOST_USERNAME
export HOST_SCP_USERNAME
export GUEST_USERNAME
export NETWORK_GATEWAY
export REDSTACK_SCRIPTS
export SERVICE_TYPE
export PATH_TROVE
export ESCAPED_PATH_TROVE
export SSH_DIR
export GUEST_LOGDIR
export ESCAPED_GUEST_LOGDIR
export ELEMENTS_PATH=$REDSTACK_SCRIPTS/files/elements:$PATH_TRIPLEO_ELEMENTS/elements
export DIB_CLOUD_INIT_DATASOURCES="ConfigDrive"
local QEMU_IMG_OPTIONS=$(! $(qemu-img | grep -q 'version 1') && echo "--qemu-img-options compat=0.10")

# disk-image-create -a amd64 -o "${IMAGE_NAME}" \
   -x ${QEMU_IMG_OPTIONS} ${DISTRO} ${EXTRA_ELEMENTS} \
   vm heat-cfntools cloud-init-datasources ${DISTRO}-guest \
   ${DISTRO}-${SERVICE_TYPE}

# disk-image-create -a amd64 -o ${output-file} Ubuntu vm heat-cfntools \
         cloud-init-datasources ubuntu-guest ubuntu-percona
```


# 参考资料
http://docs.openstack.org/developer/diskimage-builder/
http://docs.openstack.org/developer/trove/dev/building_guest_images.html              
https://www.rdoproject.org/blog/2015/03/creation-of-trove-compatible-images-for-rdo/
https://github.com/denismakogon/trove-guest-image-elements
https://github.com/openstack/diskimage-builder
http://www.lnmpy.com/disk-image-builder/
