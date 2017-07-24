---
title: Hexo+Github搭建个人博客（4）
date: 2016-09-24 17:28:09
tags: [hexo,github]
---

### Hexo+Github搭建个人博客遇到的问题

几日不更博，自己的blog就跳出来捣乱了

很开心的写了一篇blog，好的，机械化操作：
hexo g --> hexo d


坐等部署，然后等着我的是

	Permission denied (publickey).
	fatal: Could not read from remote repository.
	
	Please make sure you have the correct access rights
	and the repository exists.
	FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
	Error: Permission denied (publickey).
	fatal: Could not read from remote repository.
	
	Please make sure you have the correct access rights
	and the repository exists.
	
	    at ChildProcess.<anonymous> (/Users/liushihan/GitHublsh/node_modules/hexo-util/lib/spawn.js:37:17)
	    at emitTwo (events.js:125:13)
	    at ChildProcess.emit (events.js:213:7)
	    at maybeClose (internal/child_process.js:897:16)
	    at Process.ChildProcess._handle.onexit (internal/child_process.js:208:5)


什么鬼，感觉受到了伤害，之前明明就是好的。现在hexo不能部署了，sourcetree也不能连接git服务器了，几天不更就坏了？SSH配置了没问题啊~ 

Calm down!!!

* 那我看看自己的ssh key

   usr/lsh/.ssh

   打开一看，都在啊（此处有坑）

* 测测有没有问题

  ssh -T git@github.com
  
  结果居然连不上了。之前一直这样更新博客的，突然连不上。
  
* 查看GitHub ssh配置状态

	也是配好的，和之前一样没动过。
	
* 难道是hexo git node 版本要更新？

 好吧，来吧，更新吧 少年~
 
* 然并卵

有点生气了，简单粗暴点，删掉之前的ssh key ，重新生成一次

ssh-keygen -t rsa -C "liu601545126@qq.com"

生成成功，会显示文件路径。

* 换个姿势，再来一次hexo g->hexo d

WTF!

	*** Please tell me who you are.
	
	Run
	
	  git config --global user.email "you@example.com"
	  git config --global user.name "Your Name"
	
	to set your account's default identity.
	Omit --global to set the identity only in this repository.
	
	
我读书少，你别骗我，git我之前可是配置过的~
算了，再配一次吧~

git config --global user.email "youremail@example.com"

git config --global user.name "yourname"

* 万事具备，只需deploy

然而，现实又给了我一个响亮的耳光

	FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
	Error: Permission denied (publickey).
	fatal: Could not read from remote repository.
	
	Please make sure you have the correct access rights
	and the repository exists.
	
	    at ChildProcess.<anonymous> (/Users/liushihan/GitHublsh/node_modules/hexo-util/lib/spawn.js:37:17)
	    at emitTwo (events.js:125:13)
	    at ChildProcess.emit (events.js:213:7)
	    at maybeClose (internal/child_process.js:897:16)
	    at Socket.stream.socket.on (internal/child_process.js:340:11)
	    at emitOne (events.js:115:13)
	    at Socket.emit (events.js:210:7)
	    at Pipe._handle.close [as _onclose] (net.js:549:12)
	    
	    
	Error: EACCES: permission denied, unlink '/Users/liushihan/GitHublsh/.deploy_git/about/index.html'
	
	
我不信！！！说我没权限？

show me the code

	ssh -T git@github.com
	Hi GitHublsh! You've successfully authenticated, but GitHub does not provide shell access.
	
明明就···明明就···有···

* 看了下错误，冷静分析一下

应该是这个博客本地库存在未知错误

还是简单粗暴，删掉，重新把git上面的库clone下来，重新生成部署。

	To github.com:Githublsh/Githublsh.github.io.git
	   0b24cdb..ccae3f1  HEAD -> master
	Branch master set up to track remote branch master from git@github.com:Githublsh/Githublsh.github.io.git.
	INFO  Deploy done: git
	
终于成功了···

### 小结
1. 回头看看，从一开始的ssh key ,和后来的重新生成的ssh key有点不一样（废话，key是唯一的肯定不一样啊），我说的不是这个，我发现之前的key的文件名自己改了，打开看了一下，好像是SourceTree改的。好吧，当然是原谅它啊~

2. 到最后发现，是自己的本地库也存在未知错误，果断删掉，重新clone.

3. 遇到没有头绪的问题，那就从头开始解决。




