# Magnum

> * 原文地址：[OpenStack Magnum Wiki](https://wiki.openstack.org/wiki/Magnum)

Magnum 由 OpenStack Container Team开发的OpenStack API服务，用于把像 Docker 和 Kubernetes 之类的container orchestration engines 作为 OpenStack 上第一类可用资源。Magnum 使用 Heat 來编排(orchestrate) OS 镜像(image)，其中包含 Docker 和 Kubernetes，并按照集群配置运行在任何的虚拟机或者裸机(bare metal)中。点击下面5分钟的 Magnum Demo 短片，了解Magnum如何工作。

[![ScreenShot](https://wiki.openstack.org/w/images/8/88/Demo-Preview-Frame.png)](https://vimeo.com/128538940)


## 目录

- [最新消息](#最新消息)
- [开始与下载](#开始与下载)
- [贡献](#贡献)
- [架构](#架构)
- [资源](#资源)
- [IRC](#IRC)
- [会议](#会议)
- [常见问题](#常见问题)


## 最新消息
* 2015-10-20 We have published a list of [sessions to attend](https://wiki.openstack.org/wiki/Magnum/Summit) at the Mitaka Design Summit in Tokyo
* 2015-08-26 Magnum will be presenting [a session on Magnum](http://sched.co/49xE) at the 2015 OpenStack Summit in Tokyo on October 28 4:40pm - 5:20pm [Video](https://www.openstack.org/summit/tokyo-2015/videos/presentation/openstack-magnum-containers-as-a-service)
* 2015-05-21 We presented [a session on Magnum](https://openstacksummitmay2015vancouver.sched.org/event/ec3936678ef22681408088ec52a4e80b) at the 2015 OpenStack Summit in Vancouver on Thursday, May 21 9:00am - 9:40am US/Pacific. [Video](https://www.openstack.org/summit/vancouver-2015/summit-videos/presentation/magnum-containers-as-a-service-for-openstack)
* 2015-03-24 Magnum has [officially joined](https://review.openstack.org/161080) the OpenStack project list upon approval by a unanimous vote by the Technical Committee.
* 2015-03-09 Our Kilo-2 release is now available for download.
* 2015-01-20 We have announced Magnum's first release, now available for download.

## 开始与下载
开始使用 Magnum 请参考：[快速开始指南](http://docs.openstack.org/developer/magnum/dev/dev-quickstart.html)

版本 1.0.0.0b1 (Liberty Beta 1)下载：
* [magnum](http://tarballs.openstack.org/magnum/magnum-1.0.0.0b1.tar.gz)
* [python-magnumclient](http://tarballs.openstack.org/python-magnumclient/python-magnumclient-1.0.0.0b1.tar.gz)

## 贡献
该项目由我们的 OpenStack Containers Team 积极开发。我们每周使用 [IRC](https://wiki.openstack.org/wiki/Meetings/Containers) 进行在线交流，Magnum的会议通常由我们的 PTL [Adrian Otto](https://launchpad.net/~aotto) 主持。
* 我们需要您向[ Magnum贡献](https://wiki.openstack.org/wiki/Magnum/Contributing) !

## 架构
![magnum_architecture](https://wiki.openstack.org/w/images/thumb/6/61/Magnum_architecture.png/800px-Magnum_architecture.png)
Bay 创建/更新/删除

## 资源
* Launchpad 项目地址
	* [Magnum Launchpad Project](http://launchpad.net/magnum) for Blueprints
	* [python-magnumclient Launchpad Project](http://launchpad.net/python-magnumclient) for Blueprints
* 邮件列表
	* 项目相关讨论组[OpenStack Mailing List](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-dev) 
		* Add [Magnum] to the subject line of new posts about this project.
	* 当前讨论归档组[OpenStack Mailing List Archives](http://lists.openstack.org/pipermail/openstack-dev/) 
* Code Reviews
	* [Gerrit Reviews](https://review.openstack.org/#/q/status:open+magnum,n,z)
* Code Repository
	* git clone git://git.openstack.org/openstack/magnum
* Specification
	* [Containers Service Spec](https://review.openstack.org/136103)
	* [Container Networking Model Spec](https://review.openstack.org/204686/)
* 参考资料
	* [Magnum Networking Details](https://wiki.openstack.org/wiki/Magnum/Networking)
	* [Network Driver Support Matrix](https://wiki.openstack.org/wiki/Magnum/NetworkDriverMatrix)
	* [Labels Support Matrix](https://wiki.openstack.org/wiki/Magnum/LabelMatrix)
	* [Acronyms](https://wiki.openstack.org/wiki/Magnum/Acronyms)
	* [IRC Logs - OpenStack Containers](http://eavesdrop.openstack.org/irclogs/%23openstack-containers/)
	* [Meeting Minutes - OpenStack Containers](http://eavesdrop.openstack.org/meetings/containers/2015/)

## IRC
我们的开发人员使用 freenode 的 ```#openstack-containers``` 进行开发讨论。

## 会议
* 在每四下午4点（时区UTC）进行 Containers [IRC 会议](https://wiki.openstack.org/wiki/Meetings/Containers)[\[schedule\]](https://wiki.openstack.org/wiki/Meetings/Containers)。
* [2014 Containers 会议归档](http://eavesdrop.openstack.org/meetings/containers/2014/)

## 常见问题
#### 1) Magnum 与 Nova 的差异？
Magnum 提供一个专用的 API 來管理应用容器，其与 Nova(machine) 实例最大的差异是生命周期和操作。实际上 Magnum 使用 Nova instances 运行应用容器。

#### 2) Magnum 与 Docker 或 Kubernetes 的差异？
Magnum 提供的异步 API，与 Keystone 和多租戶(multi-tenancy)部署兼容。它并不会对内部执行编排，而是依赖 OpenStack Orchestration。Magnum 同时使用 Kubernetes 和 Docker 作为其组件。

#### 3) Magnum 与 Nova-Docker 相同吗？
不相同，对于 Nova 來说 Nova-Docker 是一個 virt 驱动，允许容器像 Nova 实例一样创建。如果你想把容器作为轻量级的虚拟机， Nova-Docker 非常适合这种场景。Magnum 提供了一些超出 Nova API 能处理范围的容器特定功能，并实现了自己的 API，這些特征在某种程度上与其他 OpenStack 服务一致。使用 Magnum 启动的容器是通过 Heat 创建并运行在 Nova instance 上。

#### 4) Magnum为谁而生？
Magnum 是为想提供自助式容器主机代管服务给用户的OpenStack 云运营商(公有或私有)应运而生的。Magnum 简化了与 OpenStack 的集成，并允许可以启动像 Nova instances、Cinder Volumes、Trove Databases 等云端资源的云端使用者，也可以在环境中创建应用容器运行应用，并提供超出现有云资源的高级功能特性。可以创建 IaaS 资源的身份凭证同样可以使用 Magnum 运行容器化应用。Magnum 的一些高级特性示例还包括：扩展应用到制定数量实例的能力，当存在错误事件时，自动重新运行实例恢复应用的能力，且与使用虚拟机相比能更好的将应用封装在一起。

#### 5) 若我使用 Heat 中的 Docker resource 能否实现相同目的？
不可以，Docker Heat resource 不提供资源调度或者容器技术使用的选择，它是专门针对 Docker 且使用 Glance 來存储容器镜像。目前还不支持分层镜像功能，如果分层的镜像与本地缓存基础镜像一起使用，这可能导致容器花费较长的时间启动。而Magnum 充分利用了 Docker 提供的所有效率上的好处。


#### 6) 在 Magnum 中，multi-tenancy意味着什么(Magnum 安全吗)?
通过 Magnum 建立的诸如容器(containers)、服务（Services）、Pods、Buys 等，只有创建他们的租戶才可以查看和读取。Bays 是非共享的，意味着即使相邻的租户，他们的容器也是运行在不同核心上的。這是一個关键的安全特性，使得同一租户的容器在相同的 Pods 或 Buys中，而不同租户各自运行在不同的核心（在不同的 Nova instances）上。这与不使用Magnum而使用 Kubernetes 的系统不同，Kubernetes 设计本意是只供单租户使用的，并将安全隔离设计丢给了系统实现者。使用 Magnum 提供了与 Nova 一样的安全等级隔离，即Nova提供的属于不同租户的虚拟机运行在相同计算节点的隔离能力。

