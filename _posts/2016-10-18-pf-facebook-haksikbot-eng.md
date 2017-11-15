---
layout: post
title: '[Project] Facebook Cafeteria Menu Bot - English'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>I developed a bot that automatically posts menu of cafeteria on Facebook.

---

* Language/Framework : Python/Django
* Library : FacebookSDK, BeautifulSoup, PyVirtualDisplay
* Server : nCloud, Ubuntu 14.04.4 LTS
* ETC : Git, Vim, PyCharm, Crontab
* [Source code](https://github.com/ThroughKim/haksikbot)

---

![facebook bot main](/assets/images/fbhaksik_main.png)

Bot will post cafeteria's menu like image above, same time, everyday

![facebook bot image](/assets/images/fbhaksik_image.png)

Parsing cafeteria menu with Python's BeautifulSoup library, than Insert parsed data to HTML table. Using PyVirtualDisplay library take image of HTML table. lastly post image to Facebook using Cron function of Linux