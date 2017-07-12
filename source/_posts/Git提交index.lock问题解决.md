---
title: Git提交index.lock问题解决
date: 2016-09-12 19:07:26
tags:
---


## Git提交或者添加时，提示index.lock文件存在，无法提交或者添加 
### 解决办法一：
1. On linux/unix/gitbash/cygwin, try：

	rm -f .git/index.lock

2. On Windows Command Prompt, try:

	de >del .git\index.lock	de>

	

### 解决办法二：

de >de>

Go to: Tools > Options > Source Control
Select Current source control plug-in as: None

### 解决办法三：

check if the git still running (ps -ef | grep git)
if not, remove the locked file
if yes, kill the git process at first.
