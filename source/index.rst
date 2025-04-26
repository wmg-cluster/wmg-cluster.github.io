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
   |    Tutorial:  https://wmg.ustc.edu.cn/wiki                 |
   |  Job Status:  https://wmg.ustc.edu.cn/cluster/status.html  |
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
      <span style="color:red; font-weight:bold">务必</span>记得使用 srun 命令或者通过 srun --pty bash 启动交互式 Bash 环境！！！
      <div style="margin: 1em 0"></div>

.. toctree::
   :maxdepth: 3
   :caption: 目录

   wmc-docs.md
   job_scripts.md
   handy_script_utilities.md

..  Indices and tables
..  ==================

..  * :ref:`genindex`
..  * :ref:`modindex`
..  * :ref:`search`