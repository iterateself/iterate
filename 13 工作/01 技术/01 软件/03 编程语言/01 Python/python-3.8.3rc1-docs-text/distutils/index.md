分发 Python 模块（遗留版本）
****************************

作者:
   Greg Ward ， Anthony Baxter

电子邮箱:
   distutils-sig@python.org

参见:

  分发 Python 模块
     最新的模块分发文档

注解:

  这篇文档只有在
  https://setuptools.readthedocs.io/en/latest/setuptools.html 上的
  "setuptools" 文档独立涵盖此处包含的所有相关信息之前，才会单独保留。

注解:

  本指南仅介绍构建和分发扩展的基本工具，这些扩展是作为此Python版本的一
  部分提供的。 第三方工具提供更易于使用和更安全的替代方案。有关详细信
  息，请参阅 Python 打包用户指南中的 快速推荐部分 。

本文档从模块开发人员的角度描述了 Python Distribution Utilities
("Distutils")。 描述了 "setuptools" 构建所依赖的下层功能，以允许
Python 开发者方便地为更广泛的受众编写 Python 模块和扩展。

* 1. An Introduction to Distutils

  * 1.1. Concepts & Terminology

  * 1.2. 一个简单的例子

  * 1.3. General Python terminology

  * 1.4. Distutils-specific terminology

* 2. 编写安装脚本

  * 2.1. Listing whole packages

  * 2.2. Listing individual modules

  * 2.3. Describing extension modules

  * 2.4. Relationships between Distributions and Packages

  * 2.5. Installing Scripts

  * 2.6. Installing Package Data

  * 2.7. Installing Additional Files

  * 2.8. Additional meta-data

  * 2.9. Debugging the setup script

* 3. Writing the Setup Configuration File

* 4. Creating a Source Distribution

  * 4.1. Specifying the files to distribute

  * 4.2. Manifest-related options

* 5. Creating Built Distributions

  * 5.1. 创建RPM软件包

  * 5.2. Creating Windows Installers

  * 5.3. Cross-compiling on Windows

  * 5.4. Vista User Access Control (UAC)

* 6. Distutils 示例

  * 6.1. 纯 Python 分发（通过 module）

  * 6.2. 纯 Python 分发（通过 包）

  * 6.3. 单个扩展模块

  * 6.4. 检查一个包

  * 6.5. 读取元数据

* 7. 扩展 Distutils

  * 7.1. 集成新的命令

  * 7.2. 添加新的发布类型

* 8. 命令参考

  * 8.1. 安装模块: **install** 命令族

  * 8.2. 创建源码发行包: **sdist** 命令

* 9. API参考引用

  * 9.1. "distutils.core" --- 分发包功能的核心

  * 9.2. "distutils.ccompiler" --- CCompiler基类

  * 9.3. "distutils.unixccompiler" --- Unix C Compiler

  * 9.4. "distutils.msvccompiler" --- Microsoft Compiler

  * 9.5. "distutils.bcppcompiler" --- Borland Compiler

  * 9.6. "distutils.cygwincompiler" --- Cygwin Compiler

  * 9.7. "distutils.archive_util" ---  Archiving utilities

  * 9.8. "distutils.dep_util" --- Dependency checking

  * 9.9. "distutils.dir_util" --- Directory tree operations

  * 9.10. "distutils.file_util" --- Single file operations

  * 9.11. "distutils.util" --- Miscellaneous other utility functions

  * 9.12. "distutils.dist" --- The Distribution class

  * 9.13. "distutils.extension" --- The Extension class

  * 9.14. "distutils.debug" --- Distutils debug mode

  * 9.15. "distutils.errors" --- Distutils exceptions

  * 9.16. "distutils.fancy_getopt" --- Wrapper around the standard
    getopt module

  * 9.17. "distutils.filelist" --- The FileList class

  * 9.18. "distutils.log" --- Simple **PEP 282**-style logging

  * 9.19. "distutils.spawn" --- Spawn a sub-process

  * 9.20. "distutils.sysconfig" --- System configuration information

  * 9.21. "distutils.text_file" --- The TextFile class

  * 9.22. "distutils.version" --- Version number classes

  * 9.23. "distutils.cmd" --- Abstract base class for Distutils
    commands

  * 9.24. Creating a new Distutils command

  * 9.25. "distutils.command" --- Individual Distutils commands

  * 9.26. "distutils.command.bdist" --- Build a binary installer

  * 9.27. "distutils.command.bdist_packager" --- Abstract base class
    for packagers

  * 9.28. "distutils.command.bdist_dumb" --- Build a "dumb" installer

  * 9.29. "distutils.command.bdist_msi" --- Build a Microsoft
    Installer binary package

  * 9.30. "distutils.command.bdist_rpm" --- Build a binary
    distribution as a Redhat RPM and SRPM

  * 9.31. "distutils.command.bdist_wininst" --- Build a Windows
    installer

  * 9.32. "distutils.command.sdist" --- Build a source distribution

  * 9.33. "distutils.command.build" --- Build all files of a package

  * 9.34. "distutils.command.build_clib" --- Build any C libraries in
    a package

  * 9.35. "distutils.command.build_ext" --- Build any extensions in a
    package

  * 9.36. "distutils.command.build_py" --- Build the .py/.pyc files of
    a package

  * 9.37. "distutils.command.build_scripts" --- Build the scripts of a
    package

  * 9.38. "distutils.command.clean" --- Clean a package build area

  * 9.39. "distutils.command.config" --- Perform package configuration

  * 9.40. "distutils.command.install" --- Install a package

  * 9.41. "distutils.command.install_data" --- Install data files from
    a package

  * 9.42. "distutils.command.install_headers" --- Install C/C++ header
    files from a package

  * 9.43. "distutils.command.install_lib" --- Install library files
    from a package

  * 9.44. "distutils.command.install_scripts" --- Install script files
    from a package

  * 9.45. "distutils.command.register" --- Register a module with the
    Python Package Index

  * 9.46. "distutils.command.check" --- Check the meta-data of a
    package
