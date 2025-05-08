.. wmc-docs documentation master file, created by
   sphinx-quickstart on Thu Apr 24 15:47:44 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. wmc-docs documentation

𝐖ei𝐌ing 𝐂luster User Guide
===========================================

.. code-block:: text
   :class: ascii-art

    ____________________________________________________________
   | __      __   _       _              ___                    |
   | \ \    / /__(_)_ __ (_)_ _  __ _   / __|_ _ ___ _  _ _ __  |
   |  \ \/\/ / -_) | '  \| | ' \/ _' | | (_ | '_/ _ \ || | '_ \ |
   |   \_/\_/\___|_|_|_|_|_|_||_\__, |  \___|_| \___/\_,_| .__/ |
   |                            |___/                    |_|    |
   |============================================================|
   |                                                            |
   |  Bonvenon! This is the computer cluster of Weiming Group.  |
   |                                                            |
   |  Tutorial:  https://wmg-cluster.github.io/                 |
   |                                                            |
   |  Remember to run commands with 'srun' while debugging!     |
   |____________________________________________________________|


.. Add your content using ``reStructuredText`` syntax. See the
   `reStructuredText <https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html>`_
   documentation for details.

.. admonition:: 重要提示
   :class: danger

   .. raw:: html

      <div style="margin: 1em 0"></div>
      <span style="color:red; font-weight:bold">严禁</span>在登录节点上直接运行计算任务！！！后果包括但不限于<span style="color:red; font-weight:bold">集群登录节点过载，导致用户集体掉线</span>。
      <div style="margin: 1em 0"></div>
      启动交互式 Bash 环境<span style="color:red; font-weight:bold">务必</span>记得使用 <code>srun</code> 命令或者通过 <code>srun --pty bash</code> ！！！
      <div style="margin: 1em 0"></div>

.. admonition:: 文档改进计划
   :class: note

   如果发现文档存在错漏需要更正、信息过时需要更新或者有好的内容（例如一些集群使用技巧或工具）愿意分享，请联系集群管理员或在 GitHub 提交 `Discussions <https://github.com/wmg-cluster/wmg-cluster.github.io/discussions>`_ 。

.. toctree::
   :maxdepth: 3
   :caption: 目录

   wmc-docs.md
   job_scripts.md
   useful_softwares.md
   FAQs.md

..  Indices and tables
..  ==================

..  * :ref:`genindex`
..  * :ref:`modindex`
..  * :ref:`search`