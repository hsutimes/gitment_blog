---
title: git教程
tags:
  - git
  - 文章
categories:
  - 文章
toc: false
date: 2019-08-16 15:47:37
---

# 名称
git - 愚蠢的内容跟踪器

# 概要

	git [--version] [ -  help] [-C <path>] [-c <name> = <value>]
	    [--exec-path [= <path>]] [--html-path] [--man-path] [--info-path]
	    [-p | --paginate | -P | --no-pager] [ -  no-replace-objects] [--bare]
	    [--git-dir = <path>] [ -  work-tree = <path>] [--namespace = <name>]
	    [--super前缀= <路径>]
	    <command> [<args>]

# 描述
Git是一个快速，可扩展的分布式版本控制系统，具有异常丰富的命令集，可提供高级操作和对内部的完全访问。

请参阅gittutorial [7]以开始使用，然后查看 giteveryday [7]以获取有用的最小命令集。在Git的用户手册有一个更深入的介绍。

掌握了基本概念后，您可以回到此页面了解Git提供的命令。您可以使用“git help command”了解有关各个Git命令的更多信息。 gitcli [7] 手册页概述了命令行命令语法。

可以在以下位置查看最新Git文档的格式化和超链接副本https://git.github.io/htmldocs/git.html。

# OPTIONS

- 版
打印git程序来自的Git套件版本。

- 救命
打印概要和最常用命令的列表。如果选择--all或-a已给出，则打印所有可用命令。如果命名了Git命令，则此选项将显示该命令的手册页。

其他选项可用于控制手册页的显示方式。有关更多信息，请参阅git-help [1]，因为git --help ...内部转换为git help ...。

-C <路径>
运行就好像git是在<path>而不是当前工作目录中启动的。当-C给出多个选项时，-C <path>相对于前面的每个后续非绝对值被解释-C <path>。如果<path>存在但是为空，例如-C ""，则保持当前工作目录不变。

此选项会影响预期的路径名的选项一样--git-dir，并 --work-tree在他们的路径名的解释，将相对于所造成的工作目录进行-C选择。例如，以下调用是等效的：

	git --git-dir = a.git --work-tree = b -C c status
	git --git-dir = c / a.git --work-tree = c / b status
	
-c <name> = <value>
将配置参数传递给命令。给定的值将覆盖配置文件中的值。<name>的格式与git config列出的格式相同 （以点分隔的子键）。

请注意，允许省略=in git -c foo.bar ...并设置 foo.bar为布尔值true（就像[foo]bar在配置文件中一样）。包括equals但空值（如git -c foo.bar= ...）设置foo.bar为git config --type=bool将转换为的空字符串false。

--exec路径[= <路径>]
安装核心Git程序的路径。这也可以通过设置GIT_EXEC_PATH环境变量来控制。如果没有给出路径，git将打印当前设置然后退出。

--html路径
打印路径，不带斜杠，安装Git的HTML文档并退出。

--man路径
打印manpath（请参阅参考资料man(1)）获取此版本Git的手册页并退出。

--info路径
打印安装记录此版本Git的Info文件的路径并退出。

-p
--paginate
如果标准输出是终端，则将所有输出传输到较少（或如果设置为$ PAGER）。这将覆盖pager.<cmd> 配置选项（请参阅下面的“配置机制”部分）。

-P
--no寻呼机
不要将Git输出传输到寻呼机。

--git-DIR = <路径>
设置存储库的路径。这也可以通过设置GIT_DIR环境变量来控制。它可以是当前工作目录的绝对路径或相对路径。

- 共同努力树= <路径>
设置工作树的路径。它可以是绝对路径或相对于当前工作目录的路径。这也可以通过设置GIT_WORK_TREE环境变量和core.worktree配置变量来控制（有关更详细的讨论，请参阅git-config [1]中的core.worktree ）。

--namespace = <路径>
设置Git名称空间。有关更多详细信息，请参阅gitnamespaces [7]。相当于设置GIT_NAMESPACE环境变量。

--super前缀= <路径>
目前仅供内部使用。设置一个前缀，该前缀提供从存储库上方到其根目录的路径。一个用途是给出调用它的超级项目的子模块上下文。

- 裸
将存储库视为裸存储库。如果未设置GIT_DIR环境，则将其设置为当前工作目录。

--no替换对象
不要使用替换引用来替换Git对象。有关更多信息，请参阅 git-replace [1]。

--literal按本义，pathspecs
按字面意思处理pathspecs（即没有globbing，没有pathspec魔法）。这相当于将GIT_LITERAL_PATHSPECS环境变量设置为1。

--glob-pathspecs
为所有pathspec添加“glob”魔法。这相当于将GIT_GLOB_PATHSPECS环境变量设置为1。可以使用pathspec magic“:( literal）”在各个pathspec上禁用通配符

--noglob-pathspecs
为所有pathspec添加“literal”魔法。这相当于将GIT_NOGLOB_PATHSPECS环境变量设置为1。可以使用pathspec magic“:( glob）”在各个pathspec上启用globbing

--icase-pathspecs
为所有pathspec添加“icase”魔法。这相当于将GIT_ICASE_PATHSPECS环境变量设置为1。

--no-可选锁
不要执行需要锁定的可选操作。这相当于设置GIT_OPTIONAL_LOCKS为0。

--list-CMDS =基团[，组...]
按组列出命令。这是一个内部/实验选项，可能会在将来更改或删除。支持的组包括：builtins，parseopt（使用parse-options的内置命令），main（libexec目录中的所有命令），其他（所有其他命令$PATH都有git-前缀），list- <category>（请参阅命令中的类别 - list.txt），nohelpers（排除帮助程序命令），别名和配置（从配置变量completion.commands检索命令列表）