---
title: Using `npm` behind a proxy
category: ['Linux', 'Npm', 'Nodejs']
tags: ['linux', 'npm', 'nodejs', 'proxy']
---

`npm` stands for `Node Package Manager`, and is the default package manager for the JavaScript runtime environment `Node.js`.

To permanently set the configuration we can do this.
	
	npm config set proxy http://proxy.company.com:8080
	npm config set https-proxy http://proxy.company.com:8080


If you need to specify credentials, they can be passed in the url using the following syntax.

	http://user_name:password@proxy.company.com:8080

To Redirect traffic to proxy.

	npm --https-proxy=http://proxy.company.com:8080 -g install npmbox
	npm --https-proxy=http://proxy.company.com:8080 -g install kafka-node

More Details.

	http://wil.boayue.com/blog/2013/06/14/using-npm-behind-a-proxy/