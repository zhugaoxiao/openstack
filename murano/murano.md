# Murano

Murano 项目引入了一个应用目录，它可以让应用开发者和云管理员在浏览分类目录里发布各种云应用。云用户，包括那些没有经验的人，可以用应用目录一键构建可靠的应用环境。

The key goal is to provide UI and API which allows to compose and deploy composite environments on the Application abstraction level and then manage their lifecycle. The Service should be able to orchestrate complex circular dependent cases in order to setup complete environments with many dependent applications and services. However, the actual deployment itself will be done by the existing software orchestration tools (such as Heat), while the Murano project will become an integration point for various applications and services.

# 相关资源

* Project Wiki: https://wiki.openstack.org/wiki/Murano
* Project Roadmap: https://wiki.openstack.org/wiki/Murano/Roadmap
* Project Launchpad: https://launchpad.net/murano
* Murano Official Documentation: http://murano.readthedocs.org
* API draft: http://murano.mirantis.com/content/ch04.html
* Quickstart: [Installation Guide](http://murano.mirantis.com/content/ch05.html)
* Horizon screenshots: [Horizon screenshots](http://murano.mirantis.com/content/ch06.html)
