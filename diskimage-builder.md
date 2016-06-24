# diskimage-builder那点事

diskimage-builder 是一个用于为云环境构建操作系统镜像的工具，是TripleO项目的子项目。它支持当前主流的Linux版本并且可以生成各种通用格式（qcow2,vhd,raw等），裸机文件系统镜像和ram-disk镜像。

支持下列系统类型，暂时不支持Windows系列，Windows系列可以使用另外的工具：
- fedora
- debian
- ubuntu
- centos
- rhel
- gentoo

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

* fedora

```
# disk-image-create -a amd64 -o fedora -t qcow2 vm fedora  
# disk-image-create -u -o fedora-image -a amd64 fedora baremetal local-conf dhcp-all-interfaces

```

* ubuntu

```
# export DIB_RELEASE=trusty
# export DIB_DISTRIBUTION_MIRROR=http://cn.archive.ubuntu.com/ubuntu

# disk-image-create -a amd64 -o ubuntu vm ubuntu
# disk-image-create -a amd64 -o ubuntu-baremetal vm base ubuntu baremetal
# disk-image-create -a amd64 -o ubuntu-nocloud vm base ubuntu cloud-init-nocloud
# disk-image-create -a amd64 -o ubuntu-mysql -p mysql-server,tmux vm ubuntu
```

* rhel

构建包括内核和ramdisk的镜像，会包括rhel-baremetal.qcow2, rhel-baremetal.vmlinux和rhel-baremetal.kernel三个文件。

```
disk-image-create –o rhel-baremetal –a amd64 –u base rhel baremetal
```


### 构建pxe镜像

构建pxe镜像，会包括initramfs和kernel两个文件。

```
# ramdisk-image-create -a amd64 – pxe deploy

# ramdisk-image-create -o deploy-baremetal.ramdisk deploy-baremetal

# ramdisk-image-create -o deploy-ironic.ramdisk deploy-ironic
```

### 构建自定义镜像

* nginx
To create an Ubuntu image for a specific workload, you first need to create the nginx element.
Creating a nginx element
1. From a command prompt, create a directory structure of elements/nginxusing following command:
mkdir –p ~/elements/nginx
Note:The tilde (~) represents your home directory.
2. Change to this newly created directory:
cd ~/elements/nginx
3. From here, create another directory named install.d, which will contain the installation hooks:
mkdir install.d
4. In install.d, create a file named 15-nginxand change its execute permissions:
touch 15-nginxchmod a+x 15-nginx
5. Using an editor, such as vi, add the following lines to the 15-nginx file:
!/bin/bash
set -euxinstall-packages nginx

```
# disk-image-create -o base -a amd64 vm base baremetal nginx
```

* mongoDB
Creating a mongodb element
1. From a command prompt, create the directory structure elements/mongodbusing following command:
mkdir –p ~/elements/mongodb
Note:The tilde (~) represents the path to your home directory.
2. Change to this newly created directory:
cd ~/elements/mongodb
3. From here, create another directory named pre-install.d, which will contain the pre-installation hooks to
setup yum repositories:
mkdir pre-install.d
4. In pre-install.d, create a file named 10-mongodband change its execute permissions:
touch 10-mongodbchmod a+x 10-mongodb
5. Using an editor, such as vi, add the following lines to the 10-mongodb file:
!/bin/bash
set -eux
echo "[mongodb]" > /etc/yum.repos.d/mongodb.repo
echo "name=mongodb_repo" >> /etc/yum.repos.d/mongodb.repo
echo "baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/" >>
echo "enabled=1" >> /etc/yum.repos.d/mongodb.repo
echo "gpgcheck=0" >> /etc/yum.repos.d/mongodb.repo
While still in ~/elements/mongodb, create another directory named install.d, which will contain the
installation hooks:
mkdir install.d
7. In install.d, create a file named 10-mongodband change its execute permissions:
touch 10-mongodbchmod a+x 10-mongodb
8. Using your editor, add the following lines to the 10-mongodb file:
!/bin/bash
set -euxinstall-packages mongo-10gen mongo-10gen-server
9. While still in ~/elements/mongodb, create another directory named finalise.d, which will contain the
finalise hooks:
mkdir finalise.d
10. In finalise.d, create a file named 10-mongodband change its execute permissions:
touch 10-mongodbchmod a+x 10-mongodb
11. Using an editor, add the following lines to the 10-mongodb file:
!/bin/bash
set -euxrm -Rf /etc/yum.repos.d/mongodb.repo

```
disk-image-create –o <image_prefix> –a amd64–u base rhel baremetal mongodb
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
正式
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
