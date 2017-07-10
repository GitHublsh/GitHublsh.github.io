---
title: hexo+gitpage
date: 2017-07-10 18:02:08
tags: [hexo,github]
---

## Hexo+Github搭建个人博客

### 一、搭建环境

* Git环境配置
* node.js环境配置
* GitHub账号配置
	* 首先，你得有一个Github账号。没有的话，手动再见~
	* 创建一个Repository
		* Repository name填写yourname.github.io,其他的地方看心情填写。
		* 创建之后，开启pages功能，setting-->Github Pages-->Automatic page generator。
		* ok

* hexo环境搭建
	* npm install -g hexo
	* 安装完成后，自己在本地文件夹新建一个本地blog文件夹（例如：\GitBlog）
	* 命令行进入文件目录，
		* hexo init,然后等新建完成
		* npm install,将在文件中安装node_modeules
		* hexo s,[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop. 就可以看到本地的blog了

		
		
### 二、基本配置

本地的blog已经可以看到了，接下来就是配置后部署到Github,然后大家都可以看到写的东西了。

---
* 回到本地新建的Gitblog文件夹，配置_config.yml文件

		# Deployment
		## Docs: http://hexo.io/docs/deployment.html
		deploy:
		  type:
		  
	修改后：
			
		deploy:
		  type: git
		  repository: git@github.com:Githublsh/Githublsh.github.io.git
		  branch: master
		  
		  
	repository为Github新建的代码库的地址，如上。
	
* 部署到Github上
	* hexo deploy(如果报GitHub权限的问题，配置下SSH即可)
		* ssh-keygen -t rsa -C "your_email@example.com"创建ssh
		* 添加SSH key到GitHub，将公钥拷贝到Github-->Setting-->SSH and GPG keys-->new SSH keys.
	* 部署成功后，打开(我的blog)https://githublsh.github.io/

		
### 三、小结

* 部署步骤

	1. hexo clean
	2. hexo g
	3. hexo d

* 修改后，本地预览

	1. hexo g
	2. hexo s
	3. 查看页面效果http://localhost:4000