---
layout: post
title: "解决github page warning"
description: ""
category: linux
tags: []
---
{% include JB/setup %}

使用github page挂网站，每次发布都有一封邮件warning邮件。


	The page build completed successfully, but returned the following warning:

	GitHub Pages recently underwent some improvements (https://github.com/blog/1715-faster-more-awesome-github-pages) to make your site faster and more awesome, but we've noticed that dp0304.com isn't properly configured to take advantage of these new features. While your site will continue to work just fine, updating your domain's configuration offers some additional speed and performance benefits. Instructions on updating your site's IP address can be found at https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages#step-2-configure-dns-records, and of course, you can always get in touch with a human at support@github.com. For the more technical minded folks who want to skip the help docs: your site's DNS records are pointed to a deprecated IP address. 

	For information on troubleshooting Jekyll see:

	 https://help.github.com/articles/using-jekyll-with-pages#troubleshooting

	If you have any questions please contact us at https://github.com/contact.


大概是指github page升级了cdn，更新配置可以更快。   
`dig dp0304.github.io +nostats +nocomments +nocmd`

	➜  dp0304.github.com git:(master) dig dp0304.github.io  +nostats +nocomments +nocmd

	; <<>> DiG 9.8.3-P1 <<>> dp0304.github.io +nostats +nocomments +nocmd
	;; global options: +cmd
	;dp0304.github.io.		IN	A
	dp0304.github.io.	3600	IN	CNAME	github.map.fastly.net.
	github.map.fastly.net.	18	IN	A	103.245.222.133


然后登陆你自己域名的服务商域名管理界面，添加这个ip。  


--EOF
