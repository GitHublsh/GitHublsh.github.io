---
title: Hexo+Github搭建个人博客（2）
date: 2016-09-14 14:01:42
tags: [hexo,github]
---
	
### Hexo+Github搭建个人博客（2）
#### 开始动手配置diy博客
	
---
	
##一、hexo站点配置文件
	
打开存放整个博客文件下_config.yml：
	
	# Hexo Configuration
	## Docs: https://hexo.io/docs/configuration.html
	## Source: https://github.com/hexojs/hexo/
	
	# Site
	title: Neil's blog	#网站标题
	subtitle: Let's start from here  #网站副标题
	description: 优秀不够，你是否无可替代
	author: Neil Liu	#博主的名字
	email: codeneil@163.com		#博主的邮箱
	language: zh-Hans	#网站使用的语言
	timezone:	#网站时区
	
	# URL
	## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
	url: http://yoursite.com
	root: /
	permalink: :year/:month/:day/:title/
	permalink_defaults:
	
	# Directory	目录配置
	source_dir: source	#源文件夹，这个文件夹用来存放内容。
	public_dir: public	#公共文件夹，这个文件夹用于存放生成的站点文件。
	tag_dir: tags	#标签文件夹
	archive_dir: archives	#归档文件夹
	category_dir: categories	#分类文件夹
	code_dir: downloads/code
	i18n_dir: :lang	#国际化（i18n）文件夹
	skip_render:	#跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。
	
	# Writing
	new_post_name: :title.md # File name of new posts
	default_layout: post	# 默认布局
	titlecase: false # Transform title into titlecase
	external_link: true # Open external links in new tab
	filename_case: 0
	render_drafts: false
	post_asset_folder: false	#启动 Asset 文件夹
	relative_link: false
	future: true
	highlight:	#代码块的设置 
	  enable: true
	  line_number: true
	  auto_detect: true
	  tab_replace:
	
	# Category & Tag	 分类和标签的设置
	default_category: uncategorized	 #默认分类
	category_map:	#分类别名
	tag_map:#标签别名
	
	# Archives 默认值为2，这里都修改为1，相应页面就只会列出标题，而非全文
	## 2: Enable pagination
	## 1: Disable pagination
	## 0: Fully Disable
	archive: 1
	category: 1
	tag: 1
	
	# Date / Time format
	## Hexo uses Moment.js to parse and display date
	## You can customize the date format as defined in
	## http://momentjs.com/docs/#/displaying/format/
	date_format: YYYY-MM-DD
	time_format: HH:mm:ss
	
	# Pagination
	## Set per_page to 0 to disable pagination
	per_page: 5
	pagination_dir: page	#分页目录
	
	# Extensions
	## Plugins: https://hexo.io/plugins/
	## Themes: https://hexo.io/themes/
	theme: next	# hexo主题
	
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	  type: git
	  repository: git@github.com:Githublsh/Githublsh.github.io.git
	  branch: master
	  
	  
	  
### 二、hexo主题配合文件

打开themes\next下的_config.yml文件

1. 开启打赏功能

		reward_comment: 让我感受一下知识的力量~
		wechatpay: /images/WechatIMG3.jpeg
		alipay: /images/WechatIMG5.jpeg

2. 开启个人微信二维码

		# Wechat Subscriber
		wechat_subscriber:
		  enabled: true
		  qcode: /images/WechatIMG7.jpeg
		  description: 个人微信，欢迎交流
		  
		  
3. 开启侧边栏社交链接

		social:
		  GitHub: https://github.com/your-user-name
		  Twitter: https://twitter.com/your-user-name
		  微博: http://weibo.com/your-user-name
		  豆瓣: http://douban.com/people/your-user-name
		  知乎: http://www.zhihu.com/people/your-user-name
		  # so on
		  
4. 选择 Scheme

		#scheme: Muse
		scheme: Mist
		#scheme: Pisces
		
		
#### 更多自定义配置请参考<a href="http://theme-next.iissnan.com/theme-settings.html">NexT主题配置</a>

 

