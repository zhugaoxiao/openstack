
First timers
============

开始OpenStack文档贡献最好的办法就是详细阅读安装文档并亲自动手实践。
把问题记录下来并且通过Lanchpad提交文档bug并给出相应修改意见。

另外bug分类和bug修复也不失为好的入门文档任务:

1.  访问 https://bugs.launchpad.net/openstack-manuals/+bugs
   和 https://bugs.launchpad.net/openstack-api-site/+bugs。

2. 当你确认bug时，根据[documentation bug triaging guidelines](http://docs.openstack.org/contributor-guide/doc-bugs.html#doc-bugs-triaging)对bug状态进行分类。你也可以跳过这步直接进行bug修复。

3. 如果你准备修复它，当它被别人确认后把它指给你自己。通过提交OpenStack文档要求的变更来修复它。

下面的图表表明了基本的配置工作流：

![figures/workflow-diagram.png](http://docs.openstack.org/contributor-guide/_images/workflow-diagram.png)



# Setting up for contribution

开始前，需要完成下面步骤：

1. 设置你的账号并且同意Individual Contributor License Agreement (ICLA)。详情[Account Setup](http://docs.openstack.org/infra/manual/developers.html#account-setup)。

2. 在Launchpad上加入`OpenStack Documentation Bug Team`。

为了设置你的环境，完成下一节的内容。

Set up a text editor
--------------------

选择你要使用的文本编辑器，比如：

* https://wiki.gnome.org/Apps/Gedit
* https://wiki.typo3.org/Editors_%28reST%29#Open_source_.28.3D_free_of_cost.29

为了保持文档整洁并易于比较，所有的OpenStack项目要求在[79个字符](https://www.python.org/dev/peps/pep-0008/#maximum-line-length)时换行，并且每行最后都不含空格。

你可以配置文本编辑器自动完成这些。

比如配置vim编辑器，修改配置文件`.vimrc`:

```
set list
set listchars=tab:>-,trail:-,extends:#,nbsp:-
set modeline
set tw=78
set tabstop=8 expandtab shiftwidth=4 softtabstop=4
```


Set up git and git-review
-------------------------

1. 安装[git](https://help.github.com/articles/set-up-git)。

如果你使用Windows，安装[Git for Windows](https://git-for-windows.github.io)。后续操作，通过Git Bash console执行命令。

2. 安装[git-review](http://docs.openstack.org/infra/manual/developers.html#installing-git-review)，用于提交补丁。

> note

>如果使用Windows，需要安装[Python](https://docs.python.org/3/using/windows.html)，同时确保安装setuptools和pip。


Set up SSH
----------

1. 在你用于commit的机器，生成SSH key：


      $ ssh-keygen –t rsa

2. 过程中你可以选择输入密码，如果输入了，需要记住它，因为在你每次提交时都需要输入。

3. 查看并拷贝SSH key:

   **Linux/Mac**
```
      $ less ~/.ssh/id_rsa.pub
```
   **Windows**
```
      $ notepad ~/.ssh/id_rsa.pub
```
4. 登陆review.openstack.org上的gerrit。

5. 在右上角，点击你的用户名。点击`Settings > SSH Public Keys`， 点击``Add Key``。将上一步的key拷贝到 ``Add SSH Public Key``的表单中并点击``Add``。

Set up a repository
-------------------

For the instructions on how to set up a repository so that you can work
on it locally, refer to the `Starting Work on a New Project`_
of the Infrastructure manual.

.. note::

   Substitute ``<projectname>`` in the examples included in this section
   with ``openstack-manuals`` as the documentation is mostly stored in
   the *openstack-manuals* repository. However, if you need specific
   guide sources, refer to *openstack/api-site*,
   *openstack/security-guide*, or *openstack/training-guides*
   repository.

See :ref:`troubleshoot_setup` if you have difficulty with a repository
setup.


Committing a change
~~~~~~~~~~~~~~~~~~~

1. Update the repository and create a new topic branch as described in
   the `Starting a Change`_ section of the Infrastructure manual.

2. Fix the bug in the docs.

   Read the :ref:`Writing style <stg_writing_style>` section, also pay
   attention to the :ref:`RST formatting conventions <rst_conv>` section.

3. Create your commit message. See `Committing a change`_ for details.

4. Create a patch for review.openstack.org following the `Submitting a Change
   for Review`_ instructions.

5. Follow the URL returned from git-review to check your commit::

     http://review.openstack.org/<COMMIT-NUMBER>

Celebrate and wait for reviews!


Responding to requests
~~~~~~~~~~~~~~~~~~~~~~

After you submit a patch, reviewers may ask you to make changes before
they approve the patch.

To submit changes to your patch, proceed with the following steps:

#. Copy the commit number from the review.openstack.org URL.

#. At the command line, change into your local copy of the repository.

#. Check out the patch:

   .. code-block:: console

      $ git review -d <COMMIT-NUMBER>

#. Make your edits.

#. Commit the changes using the `amend` flag:

   .. code-block:: console

      $ git commit -a --amend

   Ensure that the Change-ID line remains intact in your commit message. This
   prevents Gerrit from creating a new patch.

#. Push the changes to review as described in the `Updating a Change`_ section
   of the Infrastructure manual.

Wait for more reviews.


.. _troubleshoot_setup:

Troubleshooting your setup
~~~~~~~~~~~~~~~~~~~~~~~~~~

git and git review
------------------

* Authenticity error.

  The first time that you run git review, you might see this error::

    The authenticity of host '[review.openstack.org]:29418 ([198.101.231.251]:29418) can't be established.

  Type *yes* (all three letters) at the prompt.

* Gerrit connection error.

  When you connect to gerrit for the first time, you might see this error:

  .. code-block:: console

     Could not connect to gerrit.
     Enter your gerrit username:

  Enter the user name that matches the user name in the :guilabel:`Settings`
  page at review.openstack.org.

* Not a git repository error.

  If you see this error::

    fatal: Not a git repository (or any of the parent directories): .git
    You are not in a directory that is a git repository: A .git file was not found.

  Change into your local copy of the repository and re-run the command.

* Gerrit location unknown error.

  If you see this error::

    We don't know where your gerrit is. Please manually create a remote named "gerrit" and try again.

  You need to make a git remote that maps to the review.openstack.org ssh port
  for your repo. For example, for a user with the ``username_example`` username
  and the openstack-manuals repo, you should run this command::

    git remote add gerrit ssh://username_example@review.openstack.org:29418/openstack/openstack-manuals.git

* Remote rejected error.

  If you see this error::

    ! [remote rejected] HEAD -> refs/publish/master/addopenstackdocstheme (missing Change-Id in commit message footer)

  The first time you set up a gerrit remote and try to create a patch for
  review.openstack.org, you may see this message because the tool needs one
  more edit of your commit message in order to automatically insert
  the *Change-Id*. When this happens, run :code:`git commit -a --amend`,
  save the commit message and run :code:`git review -v` again.

* Permission denied error.

  If you see this error:

  .. code-block:: console

     Permission denied (publickey).

  Double check the :guilabel:`Settings` page at
  http://review.openstack.org to make sure your public key on the computer
  or virtual server has been copied to SSH public keys on
  https://review.openstack.org/#/settings/ssh-keys. If you have not adjusted
  your ``.ssh`` configuration, your system may not be connecting using
  the correct key for gerrit.

  List your local public key on Mac or Linux with:

  .. code-block:: console

     less ~/.ssh/id_rsa.pub

  On Windows, look for it in the same location.


Network
-------

If your network connection is weak, you might see this error:

.. code-block:: console

   Read from socket failed: Connection reset by peer

Try again when your network connection improves.

**Accessing gerrit over HTTP/HTTPS**

If you suspect that SSH over non-standards ports might be blocked or need to
access the web using http/https, you can configure git-review to `use an http
endpoint instead of ssh <http://docs.openstack.org/infra/manual/developers.html#accessing-gerrit-over-https>`_
as explained in the Infrastructure Manual.

Python
------

If you see this this error:

.. code-block:: console

   /usr/bin/env: python: No such file or directory

Your Python environment is not set up correctly. See the Python documentation
for your operating system.

i18n
----

If you see this error:

.. code-block:: console

   $ git review -s
   Problems encountered installing commit-msg hook
   The following command failed with exit code 1
      "scp  :hooks/commit-msg .git/hooks/commit-msg"
   -----------------------
   .git/hooks/commit-msg: No such file or directory
   -----------------------

You may have a LANGUAGE variable setup to something else than C. Try using
instead:

.. code-block:: console

   $ LANG=C LANGUAGE=C git review -s



.. Links

.. _`Account Setup`: http://docs.openstack.org/infra/manual/developers.html#account-setup
.. _`Sign the appropriate Individual Contributor License Agreement`: http://docs.openstack.org/infra/manual/developers.html#sign-the-appropriate-individual-contributor-license-agreement
.. _`Installing git-review`: http://docs.openstack.org/infra/manual/developers.html#installing-git-review
.. _`OpenStack Documentation Bug Team`: https://launchpad.net/~openstack-doc-bugs
.. _`OpenStack Foundation`: http://www.openstack.org/join
.. _`Development Workflow`: http://docs.openstack.org/infra/manual/developers.html#development-workflow
.. _`git`: http://msysgit.github.io
.. _`curl`: http://curl.haxx.se/
.. _`tar`: http://gnuwin32.sourceforge.net/packages/gtar.htm
.. _`7-zip`: http://sourceforge.net/projects/sevenzip/?source=recommended
.. _`Python 2.7 environment`: http://docs.python-guide.org/en/latest/starting/install/win/
.. _`79 characters maximum`: https://www.python.org/dev/peps/pep-0008/#maximum-line-length
.. _`GitHub help`: https://help.github.com/articles/set-up-git
.. _`Settings page on gerrit`: https://review.openstack.org/#/settings/
.. _`Settings > SSH Public Keys`: https://review.openstack.org/#/settings/ssh-keys
.. _`Starting Work on a New Project`: http://docs.openstack.org/infra/manual/developers.html#starting-work-on-a-new-project
.. _`Starting a Change`: http://docs.openstack.org/infra/manual/developers.html#starting-a-change
.. _`Committing a change`: http://docs.openstack.org/infra/manual/developers.html#committing-a-change
.. _`Submitting a Change for Review`: http://docs.openstack.org/infra/manual/developers.html#submitting-a-change-for-review
.. _`Updating a Change`: http://docs.openstack.org/infra/manual/developers.html#updating-a-change
