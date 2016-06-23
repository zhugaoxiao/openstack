# diskimage-builder那点事

支持的image：
- fedora
- debian
- ubuntu
- opensuse
- rhel

## 安装


可以使用yum、pip或者手动下载，yum安装需要添加epel源，git安装需切换至相关目录运行。
```
# yum install python-pip
# pip install diskimage-builder

# yum install diskimage-builder

# yum -y install git
# git clone https://github.com/openstack/diskimage-builder.git
```


## 使用

### 构建普通镜像

```
# disk-image-create vm fedora -a amd64 -o fedora-image.qcow2

DIB_RELEASE=trusty
disk-image-create vm ubuntu -a amd64 -o ubuntu-trusty.qcow2

disk-image-create -o base -a amd64 vm base ubuntu cloud-init-nocloud

disk-image-create –o <image_prefix> –a amd64 –u base rhel baremetal
```

### 构建tripleo 镜像

```
# git clone https://git.openstack.org/openstack/diskimage-builder.git
# git clone https://git.openstack.org/openstack/tripleo-image-elements.git
# git clone https://git.openstack.org/openstack/heat-templates.git
export ELEMENTS_PATH=tripleo-image-elements/elements
# export ELEMENTS_PATH=tripleo-image-elements/elements:heat-templates/hot/software-config/elements

#export BASE_ELEMENTS="centos7 selinux-permissive"
#export AGENT_ELEMENTS="os-collect-config os-refresh-config os-apply-config"
#export DEPLOYMENT_BASE_ELEMENTS="heat-config heat-config-script"
#export DEPLOYMENT_TOOL="heat-config-cfn-init heat-config-puppet"
#export IMAGE_NAME=centos7-software-config
```


### 构建heat 镜像

```
git clone https://github.com/openstack/tripleo-image-elements.git

# export ELEMENTS_PATH=tripleo-image-elements/elements

# disk-image-create vm fedora heat-cfntools -a amd64 -o fedora-heat-cfntools
```

### 构建trove 镜像

下载相关文件
```
# pip install pyyaml
# git clone https://github.com/openstack/tripleo-image-elements.git
# git clone https://github.com/denismakogon/trove-guest-image-elements.git
# export ELEMENTS_PATH=tripleo-image-elements/elements:trove-guest-image-elements/elements:diskimage-builder/elements
```

版本选择
```
Ubuntu:
    export DISTRO=ubuntu
    export DIB_RELEASE=trusty

CentOS:
    export DISTRO=centos
    export DIB_RELEASE=GenericCloud
    export DIB_EXTLINUX=1
```

Development purposes only:
```
export DIB_DEV_USER_USERNAME=ks
export DIB_DEV_USER_PASSWORD=w8EmsJXTMaG0e97NK8lM
export DIB_DEV_USER_SHELL=/bin/bash
export DIB_DEV_USER_PWDLESS_SUDO=true
export DIB_DEV_USER_AUTHORIZED_KEYS=/home/sa709c/.ssh/authorized_keys
```

For Trove Guest agent config:
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


export DATASTORE="mysql"

disk-image-create -a amd64 \
    -o ${DISTRO}-${DATASTORE}-${DATASTORE_VERSION}-guest-image \
    -x --qemu-img-options compat=0.10 \
    ${DISTRO}-${DATASTORE}-guest-image

Note. Only anonymous HTTP(S) accessable Git repos are allowed.

For example:

    Cassandra datastore:

        export DATASTORE="cassandra"
        export DATASTORE_VERSION="2.1.1"

If DATASTORE VERSION wasn't mentioned each datastore image element would use its own default version.
For example:

    PostgreSQL datastore:

        export DATASTORE="postgresql"
        default datastore version 9.3 would be picked
```        
If you want to build image for development purposes please add next elements into disk-image-create command:
```
openssh-server
```
If you want to build image for vCenter please add next elements:
```
${DISTRO}-vmware-tools
```
Note: this elements are orientied to work with Debian/Ubuntu 14.04 Trusty Tahr, CentOS 7.x






```
正式
# git clone
https://github.com/openstack/trove-integration.git
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
local QEMU_IMG_OPTIONS=$(! $(qemu-img | grep -q 'version 1') && \
     echo "--qemu-img-options compat=0.10")
${PATH_DISKIMAGEBUILDER}/bin/disk-image-create -a amd64 -o "${IMAGE_NAME}" \
     -x ${QEMU_IMG_OPTIONS} ${DISTRO} ${EXTRA_ELEMENTS} \
     vm heat-cfntools cloud-init-datasources ${DISTRO}-guest \
     ${DISTRO}-${SERVICE_TYPE}


     ${DIB} -a amd64 -o ${output-file} Ubuntu vm heat-cfntools \
         cloud-init-datasources ubuntu-guest ubuntu-percona
disk-image-create -a amd64 -o ubuntu-amd64 -p mysql-server,tmux vm ubuntu
```





# 参考资料
http://docs.openstack.org/developer/trove/dev/building_guest_images.html              
https://www.rdoproject.org/blog/2015/03/creation-of-trove-compatible-images-for-rdo/

https://github.com/denismakogon/trove-guest-image-elements
