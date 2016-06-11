# Mission Statement

The OpenStack Open Source Database as a Service Mission: To provide scalable and reliable Cloud Database as a Service provisioning functionality for both relational and non-relational database engines, and to continue to improve its fully-featured and extensible open source framework.

OpenStack 开源数据库即服务愿景：提供扩展性和可靠云数据作为服务为关系型和非关系型引擎，并且持续提供开源框架的全特性和扩展性。
# Description

Trove is Database as a Service for OpenStack.  It's designed to run entirely on [OpenStack](https://wiki.openstack.org/wiki/OpenStack), with the goal of allowing users to quickly and easily utilize the features of a relational or non-relational database without the burden of handling complex administrative tasks.  Cloud users and database administrators can provision and manage multiple database instances as needed. Initially, the service will focus on providing resource isolation at high performance while automating complex administrative tasks including deployment, configuration, patching, backups, restores, and monitoring.

Trove是OpenStack的数据库即服务。它被设计为完全的运行在[OpenStack](https://wiki.openstack.org/wiki/OpenStack)上，目标是允许用户迅速并且方便的使用关系型和非关系型数据库特性，没有处理复杂配置任务的负担。云用户和数据库管理员可以分配和管理多个数据库实例。初期，服务焦点在提供资源隔离在高性能和自动复杂任务包括部署，配置，补丁，备份，恢复和监控。

# 设计
Trove is designed to support a single-tenant database within a Nova instance.  There will be no restrictions on how Nova is configured, since Trove interacts with other [OpenStack](https://wiki.openstack.org/wiki/OpenStack) components purely through the API. More detailed Architecture info can be found  [here](https://wiki.openstack.org/wiki/TroveArchitecture)。

Trove被设计为支持单租户数据库在Nova实例。由于Trove与其他[OpenStack](https://wiki.openstack.org/wiki/OpenStack) 组件仅仅通过API交互，在如何配置Nova上没有任何限制。更多架构信息详情看[这里](https://wiki.openstack.org/wiki/TroveArchitecture)。

# trove-api
The **trove-api** service provides a RESTful API that supports JSON and XML to provision and manage Trove instances.
**trove-api** 服务提供支持JSON和XML的RESTful API并且管理Trove实例。
* A REST-ful component
* Entry point - Trove/bin/trove-api
* Uses a WSGI launcher configured by Trove/etc/trove/api-paste.ini
 * Defines the pipeline of filters; tokenauth, ratelimit, etc.
 * Defines the app_factory for the troveapp as trove.common.api:app_factory
* The API class (a wsgi Router) wires the REST paths to the appropriate Controllers
 * Implementation of the Controllers are under the relevant module (versions/instance/flavor/limits), in the service.py module
* Controllers usually redirect implementation to a class in the models.py module
* At this point, an api module of another component (TaskManager, GuestAgent, etc.) is used to send the request onwards through RabbitMQ
它是一个WSGI组件负责监听服务外部的请求。

# trove-taskmanager
The **trove-taskmanager** service does the heavy lifting as far as provisioning instances, managing the lifecycle of instances, and performing operations on the Database instance.
**trove-taskmanager** 服务完成具体的管理命令，如创建实例，管理实例的生命周期，并执行对数据库的操作。

* A service that listens on a RabbitMQ topic
* 监听RabbitMQ topic来得到请求。
* Entry point - Trove/bin/trove-taskmanager
* Runs as a RpcService configured by Trove/etc/trove/trove-taskmanager.conf.sample which defines trove.taskmanager.manager.Manager as the manager - basically this is the entry point for requests arriving through the queue
* As described above, requests for this component are pushed to MQ from another component using the TaskManager's api module using _cast() or _call() (sync/a-sync) and putting the method's name as a parameter
* Trove/openstack/common/rpc/dispatcher.py- RpcDispatcher.dispatch() invokes the proper method in the Manager by some equivalent to reflection
* The Manager then redirect the handling to an object from the models.py module. It loads an object from the relevant class with the context  and instance_id
* Actual handling is usually done in the models.py module

# trove-guestagent
The **guestagent** is a service that runs within the guest instance, responsible for managing and performing operations on the Database itself.  The Guest Agent listens for RPC messages through the message bus and performs the requested operation.
**guestagent**主要提供具体数据库的运行和管理，并且对数据库本身进行操作。guestagent同样监听RabbitMQ topic来得到请求，并且运行在每一个数据库实例上。每一种数据库都需要一个自己的guestagent实现，目前只有MySQL agent。
* Similar to TaskManager in the sense of running as a service that listens on a RabbitMQ topic
* GuestAgent runs on every DB instance, and a dedicated MQ topic is used (identified as the instance's id)
* Entry point - Trove/bin/trove-guestagent
* Runs as a RpcService configured by Trove/etc/trove/trove-guestagent.conf.sample which defines trove.guestagent.manager.Manager as the manager - basically this is the entry point for requests arriving through the queue
* As described above, requests for this component are pushed to MQ from another component using the GuestAgent's api module using _cast() or _call() (sync/a-sync) and putting the method's name as a parameter
* Trove/openstack/common/rpc/dispatcher.py- RpcDispatcher.dispatch() invokes the proper method in the Manager by some equivalent to reflection
* The Manager then redirect the handling to an object (usually) from the dbaas.py module.
* Actual handling is usually done in the dbaas.py module

# Source Code Repositories
* Trove Server (https://github.com/openstack/trove)
* Trove Integration (https://github.com/openstack/trove-integration)
* Trove Client (https://github.com/openstack/python-troveclient)


# 安装和部署
* How to install trove as part of devstack: [trove/installation](https://wiki.openstack.org/wiki/Trove/installation)
* How to use trove-integration: [trove/trove-integration](https://wiki.openstack.org/wiki/Trove/trove-integration)
* How to set up unit tests to run with tox: [trove/unit-testing](https://wiki.openstack.org/wiki/Trove/unit-testing)
* How to set up a testing environment and run redstack tests after installation: [trove/integration-testing](https://wiki.openstack.org/wiki/Trove/integration-testing)
* How to set up your Mac dev environment to debug: [trove/dev-env](https://wiki.openstack.org/wiki/Trove/dev-env)
* Releasing python-troveclient [trove/release-python-troveclient](https://wiki.openstack.org/wiki/Trove/release-python-troveclient)
* Creating release notes with Reno [trove/create-release-notes-with-reno](https://wiki.openstack.org/wiki/Trove/create-release-notes-with-reno)


# 开发
* Quota Management is currently in development: [trove/trove-quotas](https://wiki.openstack.org/wiki/Trove/trove-quotas)
* Security Groups is currently in design/development: [trove/trove-security-groups](https://wiki.openstack.org/wiki/Trove/trove-security-groups)
* Snapshot Design: [trove/snapshot-design](https://wiki.openstack.org/wiki/Trove/snapshot-design)
* Versions and Types Design: [trove/trove-versions-types](https://wiki.openstack.org/wiki/Trove/trove-versions-types)
* Configuration Edits: [Trove/Configurations](https://wiki.openstack.org/wiki/Trove/Configurations)
* Notification Events: [trove/trove-notifications](https://wiki.openstack.org/wiki/Trove/trove-notifications)
* Diagrams: [trove/trove-diagrams](https://wiki.openstack.org/wiki/Trove/trove-diagrams)
* Capabilities: [trove/trove-capabilities](https://wiki.openstack.org/wiki/Trove/trove-capabilities)
* Data Volume Snapshoting: [trove/volume-data-snapshot-design](https://wiki.openstack.org/wiki/Trove/volume-data-snapshot-design)
* MySQL Replication (v1) [trove/Blueprints/Trove-v1-MySQL-Replication](https://wiki.openstack.org/wiki/Trove/Blueprints/Trove-v1-MySQL-Replication)
* A proposal for Guest Agent controlling the data store [trove/Blueprints/guest-agent-datastore-control-abstraction](https://wiki.openstack.org/wiki/Trove/Blueprints/guest-agent-datastore-control-abstraction)


来源：

https://wiki.openstack.org/wiki/Trove
