Kolla 一览
==============

Kolla项目是OpenStack [Big Tent
Governance](http://governance.openstack.org/reference/projects/index.html)的一部分。Kolla的愿景是：

    Kolla提供生产级的容器和操作OpenStack云的部署工具。

Kolla 提供 [Docker](http://docker.com/) 容器和[Ansible](http://ansible.com) playbooks来满足Kolla的愿景。Kolla已经非常成熟了可以直接使用，不过也允许你进行充分的自定义。
这就使得运营商只需要一点点经验就可以快速部署OpenStack并且当经验丰富时可以修改OpenStack的配置来满足自己更具体的要求。

入门
===============

通过阅读[docs.openstack.org](http://docs.openstack.org/developer/kolla)学习Kolla。

通过阅读[Developer
Quickstart](http://docs.openstack.org/developer/kolla/quickstart.html)来入门。

Kolla提供镜像来部署下列OpenStack项目：

- [Aodh](http://docs.openstack.org/developer/aodh/)
- [Ceilometer](http://docs.openstack.org/developer/ceilometer/)
- [Cinder](http://docs.openstack.org/developer/cinder/)
- [Designate](http://docs.openstack.org/developer/designate/)
- [Glance](http://docs.openstack.org/developer/glance/)
- [Gnocchi](http://docs.openstack.org/developer/gnocchi/)
- [Heat](http://docs.openstack.org/developer/heat/)
- [Horizon](http://docs.openstack.org/developer/horizon/)
- [Ironic](http://docs.openstack.org/developer/ironic/)
- [Keystone](http://docs.openstack.org/developer/keystone/)
- [Magnum](http://docs.openstack.org/developer/magnum/)
- [Manila](http://docs.openstack.org/developer/manila/)
- [Mistral](http://docs.openstack.org/developer/mistral/)
- [Murano](http://docs.openstack.org/developer/murano/)
- [Nova](http://docs.openstack.org/developer/nova/)
- [Neutron](http://docs.openstack.org/developer/neutron/)
- [Swift](http://docs.openstack.org/developer/swift/)
- [Tempest](http://docs.openstack.org/developer/tempest/)
- [Trove](http://docs.openstack.org/developer/trove/)
- [Zaqar](http://docs.openstack.org/developer/zaqar/)

同样提供这些基础设施组件：

- [Ceph](http://ceph.com/) implementation for Cinder, Glance and Nova
- [Openvswitch](http://openvswitch.org/) and Linuxbridge backends for Neutron
- [MongoDB](https://www.mongodb.org/) as a database backend for Ceilometer
  and Gnocchi
- [RabbitMQ](https://www.rabbitmq.com/) as a messaging backend for
  communication between services.
- [HAProxy](http://www.haproxy.org/) and
  [Keepalived](http://www.keepalived.org/) for high availability of services
  and their endpoints.
- [MariaDB and Galera](https://mariadb.com/kb/en/mariadb/galera-cluster/)for
  highly available MySQL databases
- [Heka](http://hekad.readthedocs.org/en/) A distributed and
  scalable logging system for openstack services.

Docker 镜像
=============

Kolla 项目的维护者负责编译[Docker images](https://docs.docker.com/userguide/dockerimages/)。关于如何向镜像贡献的详细步骤可以参考 [image building
guide](http://docs.openstack.org/developer/kolla/image-building.html)。

The Kolla developers build images in the kollaglue namespace for every tagged
release and implement an Ansible deployment for many but not all of them.

Kolla开发人员在kollaglue命名空间下为每个tagged的版本编译镜像并且Ansible部署实现不过也不是全部。

你可以在[Docker Hub](https://hub.docker.com/u/kollaglue/)或者使用 Docker CLI查看可用镜像：

    $ sudo docker search kollaglue

目录
===========

-  ansible - Contains Ansible playbooks to deploy Kolla in Docker
   containers.
-  demos - Contains a few demos to use with Kolla.
-  dev/heat - Contains an OpenStack-Heat based development environment.
-  dev/vagrant - Contains a vagrant VirtualBox/Libvirt based development
   environment.
-  doc - Contains documentation.
-  etc - Contains a reference etc directory structure which requires
   configuration of a small number of configuration variables to achieve
   a working All-in-One (AIO) deployment.
-  docker - Contains jinja2 templates for the docker build system.
-  tools - Contains tools for interacting with Kolla.
-  specs - Contains the Kolla communities key arguments about
   architectural shifts in the code base.
-  tests - Contains functional testing tools.

参与其中
================

需要新特性？发现bug？请让我们知道！感谢所有的贡献并且应该遵循标准[Gerrit
workflow](http://docs.openstack.org/infra/manual/developers.html)。

-  我们使用irc的 #kolla 频道进行沟通。
-  在[Launchpad](https://launchpad.net/kolla)提交bug, blueprints, 跟踪发布版本等等。
-  参加每周 [meetings](https://wiki.openstack.org/wiki/Meetings/Kolla)。
-  贡献 [code](https://github.com/openstack/kolla)。

贡献者
============

查看谁在 [contributing
code](http://stackalytics.com/?module=kolla-group&metric=commits) 和
[contributing
reviews](http://stackalytics.com/?module=kolla-group&metric=marks)。

https://github.com/openstack/kolla
https://governance.openstack.org/reference/projects/kolla.html
https://wiki.openstack.org/wiki/Kolla
