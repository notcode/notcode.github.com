---
layout: post
title: "modeline: vim configuration per file"
date: 2013-07-02 22:10
comments: true
categories:
- vim
---

简介
---

vim有一个叫做modeline的功能，它允许对单独的文件设置vim变量。当用vim打开文件时，这些变量便可生效。

当编辑python文件时，必须保证编辑环境的缩进和文件中原有的缩进一致。有多个开发人员协作时，缩进不一致的问题经常发生。
使用modeline可以解决这样的问题。

具体做法如下：
    #!/usr/bin/env python2.7
    # vim: ts=4 sw=4 et

<!-- more -->

语法
---
modeline的语法如下：
    [text]{white}{vi:|vim:|ex:}[white]{options}


- [text]            any text or empty
- {white}           at least one blank character (<Space> or <Tab>)
- {vi:|vim:|ex:}    the string "vi:", "vim:" or "ex:"
- [white]           optional white space
- {options}         a list of option settings, separated with white space or ':', where each part between ':' is the argument for a ":set" command (can be empty)


使用范围
-------
modeline只在文件开头，或者末尾的N行有效，N默认为5，可以通过查看modelines变量得到
    :set modelines?


