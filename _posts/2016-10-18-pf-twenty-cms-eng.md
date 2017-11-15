---
layout: post
title: '[Project] Twenty CMS - English'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>I worked as an intern at TWENTY(Start-up - Media platform for university students) and developed a content management system.

---

* Language/Framework : Python/Django
* DB : MySQL, Django ORM
* Server : nCloud, uCloud, Ubuntu 14.04.4 LTS
* Charts : D3.js
* ETC : TDD, Git, Vim, PyCharm

---

My part was the DASHBOARD section, which analyzes the service usage log and makes statistics, the content management page, and the individual content statistics analysis page.  

![main](/assets/images/twenty_main_page.png)

This is the main Dashboard, created to allow the user to intuitively check the status of the current service when logged in.  

![postlist](/assets/images/twenty_postlist.png)

Content list of the content management system.  
A page where you can manage news content that is published daily in Twenty applications.  
There was a problem that the performance of the application deteriorated as the content increased.  
So, I have improved performance through query and pageator optimization.  

![postview](/assets/images/twenty_postview.png)

A page where you can view content and statistics for individual content.