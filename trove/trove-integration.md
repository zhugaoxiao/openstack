# trove-integration使用

官方提供了 trove-integration 组件，其实质是通过 DevStack 实现安装 Trove 服务，本次我们就使用trove-integration来安装trove。


# 环境准备

主机环境： vitualbox，vagrant，ubuntu 14.04，4GB内存

## 配置环境

```
$ sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/' /etc/apt/sources.list
$ sudo sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/' /etc/apt/sources.list
$ sudo -s
# mkdir ~/.pip
# cat > ~/.pip/pip.conf <<EOF
[global]
index-url = https://pypi.mirrors.ustc.edu.cn/simple
EOF
# exit
$ sudo apt-get update
$ sudo apt-get install git-core -y
$ sudo ntpdate ntp.sjtu.edu.cn
```

## 配置用户
必须使用普通用户进行安装，如果已有，可以直接使用，不然手动创建一个新用户。这里我直接使用vagrant用户。
```
# adduser ubuntu
# echo "ubuntu ALL = (ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ubuntu
# chmod 440 /etc/sudoers.d/ubuntu
```

# trove安装

克隆trove-integration
```
$ cd ~
$ git clone https://github.com/openstack/trove-integration.git
```

进入脚本目录，并开始进行 Trove 安装：
```
$ cd trove-integration/scripts/
exportGIT_BASE=http://git.trystack.cn
$ ./redstack install
.........................................................................
2016-06-09 08:24:04.617 | stack.sh completed in 3431 seconds.
~/trove-integration/scripts
*******************************************************************************
FINISHED INSTALL
*******************************************************************************
```
过程中会自动下载并安装相关依赖、软件等，耐心等待过程结束，最后会提示完成安装。
```
$ source devstack/openrc admin admin
$ nova image-list
```

或者可以手动下载好，加速安装过程。
```
$ cd /opt/stack
$ for i in nova cinder glance heat horizon keystone neutron swift neutron-lbaas neutron-fwaas requirements \
heat-cfntools heat-templates tempest \
os-apply-config os-collect-config os-refresh-config \
python-heatclient python-neutronclient python-novaclient python-openstackclient python-troveclient \
dib-utils diskimage-builder tripleo-image-elements trove trove-dashboard
do
git clone http://git.trystack.cn/openstack/$i
done
```

# trove镜像创建

使用mysql作为参数构建和添加mysql镜像，过程中因为集成测试会修改/etc/trove/test.conf配置。
```
$ ./redstack kick-start mysql
```
运行./redstack kick-start mysql制作mysql的vm镜像，并把镜像导入Glance，在Trove中建立datastore（比如mysql）和version（比如5.6版本）信息，。（其中version信息是包含存放在Glance中的vm镜像信息）这个过程也有点长，因为是使用diskimage-builder根据定义好的element去动态下载打包并配置镜像的。
利用trove-instegration项目构建的vm镜像，并没有把guestagent打包进去，而是通过vm文件系统中/etc/init/troveguestagent脚本，在vm启动的时候从宿主机上把guestagent拷贝过去。

初始化测试配置并配置测试用户（覆盖/etc/trove/test.conf）。
```
$ ./redstack test-init
```
编译镜像并添加至 glance
```
$  ./redstack build-image mysql
```
查看镜像
```
$ source devstack/openrc admin admin
$ openstack image list
+--------------------------------------+---------------------------------+--------+
| ID                                   | Name                            | Status |
+--------------------------------------+---------------------------------+--------+
| 766ec9cb-a3f3-40bf-8c0d-20b21e0b65d4 | ubuntu_mysql                    | active |
| 17342914-34a6-427b-b3a8-c6b3e00ebd0d | cirros-0.3.4-x86_64-uec         | active |
| 0d0e02ab-14f4-4063-b71f-a6668fe7234c | cirros-0.3.4-x86_64-uec-ramdisk | active |
| 242a0c32-007b-4afa-890d-9085d92a0eb3 | cirros-0.3.4-x86_64-uec-kernel  | active |
+--------------------------------------+---------------------------------+--------+
```

添加TripleO-Image-Elements到Trove-Integration/redstack镜像。

如果你想添加诸如 [os-apply-config](https://github.com/openstack/tripleo-image-elements/tree/master/elements/os-apply-config)的元素到ubuntu-mysql，你需要添加变量EXTRA_ELEMENTS到trove-integration/scripts/redstack.rc文件。默认情况下，redstack kick-start能自动找到os-apply-config，因为它克隆了所有的tripleo-image-elements。 redstack kick-start使用的元素路径是ELEMENTS_PATH=$REDSTACK_SCRIPTS/files/elements:$PATH_TRIPLEO_ELEMENTS/elements。
添加一个元素：
EXTRA_ELEMENTS="os-apply-config"
添加多个元素：
EXTRA_ELEMENTS="os-apply-config os-refresh-config"

现在运行./redstack kick-start mysql会自动添加os-apply-config元素到ubuntu-mysql镜像。

现在你可以添加文件夹os-apply-config到Trove-Integration/scripts/elements/ubuntu-mysql镜像来添加自定义配置文件了。

添加文件(foo.conf)和文件夹(os-apply-config/etc/init)到元素ubuntu-mysql (ubuntu-mysql/os-apply-config/etc/init/foo.conf)，当./redstack kick-start mysql 构建镜像时，会添加文件foo.conf到/etc/init。

重新构建镜像

一旦镜像构建好，你必须从nova中删除镜像并且删除~/images/ubuntu-mysql文件，再使用./redstack kick-start mysql重新构建镜像。如果你不这样做，会继续使用原来存在的镜像。


# trove试用
1）创建主实例

trove create my_inst_master 8 --size 10 --database my_inst_db --users admin:admin123 --datastore mysql --datastore_version 5.6
实例名字为my_inst_master，实例规格ID是8（512MB内存），硬盘卷大小是10GB，并且创建数据库my_inst_db和用户admin（密码是admin123），数据库引擎类型是mysql，版本是5.6版本。

2）创建主实例的备份（先随便在创建的数据库实例里创建表和插入一些数据）

trove backup-create my_inst_master my_bak.0001
创建数据库实例my_inst_master的当前的备份，备份名字为my_bak.0001。

3）从主实例的备份创建两个从实例，并且建立主从关系

trove create my_inst_slave 8 --size 10 --backup my_bak.0001 --replica_of my_inst_master --replica_count 2
创建两个数据库实例，名字以mysql_inst_slave开头，实例规格ID为8，硬盘卷大小是10GB，并且用备份名为my_bak.0001的备份导入数据，且建立到实例my_inst_master的主从复制关系。

4）动态Resize主实例规格

trove resize-instance my_inst_master 2
动态调整实例my_inst_master的instance规格为2（内存2GB大小）

trove resize-volume my_inst_master 20
动态调整实例my_inst_master的硬盘卷大小为20GB


# 重置环境
## trove重启
Stop all the services running in the screens and refresh the environment:
```
$ killall -9 screen
$ screen -wipe
$ RECLONE=yes ./redstack install
$ ./redstack kick-start mysql
```
or
```
$ RECLONE=yes ./redstack install
$ ./redstack test-init
$ ./redstack build-image mysql
```

## 主机重启

当虚拟机重启后，cinder使用的卷组并不会自动重建，所以在运行rejoin之前，需要将恢复卷组的工作，放入开机启动的脚本中。
```
# cat /etc/init.d/cinder-setup-backing-file
losetup /dev/loop1 /opt/stack/data/stack-volumes-default-backing-file
losetup /dev/loop2 /opt/stack/data/stack-volumes-lvmdriver-1-backing-file
exit 0
# chmod 755 /etc/init.d/cinder-setup-backing-file
# ln -s /etc/init.d/cinder-setup-backing-filen /etc/rc2.d/S10cinder-setup-backing-file

$ sudo losetup -a

```
If the VM was restarted, then the process for bringing up Openstack and Trove is quite simple
```
$./redstack start-deps
$./redstack start
```
Use screen to ensure all modules have started without error
```
$screen -r stack
```

## screen语法
执行完rejoin_stack.sh后，需要使用screen语法来控制openstack的进程
帮助 ctrl+a+?
查看screen导航 ctrl+a+"  注需要使用shift键
退出screen，有两种方法:
方法1：attach screen   ctrl+a+d
方法2：exit screen       ctrl+a+K
查看下一个screen ctrl+a+n
查看上一个screen ctrl+a+p
保存screen的日志到文件 ctrl+a+H，再按一次停止保存。

查看screen
# screen -ls
There is a screen on:
        2678.stack      (Attached)
1 Socket in /var/run/screen/S-root.

重连接Re-attach screen   
screen -r 2678

# Running Integration Tests

Check the values in /etc/trove/test.conf in case it has been re-initialized prior to running the tests. For example, from the previous mysql steps:

"dbaas_datastore": "%datastore_type%",
"dbaas_datastore_version": "%datastore_version%",
should be:

"dbaas_datastore": "mysql",
"dbaas_datastore_version": "5.5",
Once Trove is running on DevStack, you can use the dev scripts to run the integration tests locally.

$./redstack int-tests
This will runs all of the blackbox tests by default. Use the --group option to run a different group:

$./redstack int-tests --group=simple_blackbox
You can also specify the TESTS_USE_INSTANCE_ID environment variable to have the integration tests use an existing instance for the tests rather than creating a new one.

$./TESTS_DO_NOT_DELETE_INSTANCE=True TESTS_USE_INSTANCE_ID=INSTANCE_UUID ./redstack int-tests --group=simple_blackbox

参考资料：
https://wiki.openstack.org/wiki/Trove/trove-integration
https://github.com/openstack/trove-integration
