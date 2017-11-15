---
layout: post
title: '[Project] PhotoBlog - English'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>It is a photograph-oriented micro blog developed as a project of web-programming class on campus.

---

* Language : Java, JSP, CSS
* Library : cos
* DB : MySQL
* Web Server : Apache Tomcat
* ETC : Git, IntelliJ
* [Source Code](https://github.com/ThroughKim/photoblog)

---

It was created by imitating the UI of the Instagram, and developed using Java and JSP.

![Main](/assets/images/photoblog_main.png)

The main page only shows the posts of the users you follow.

![Uploading page](/assets/images/photoblog_photo_select.png)
![Post writing page](/assets/images/photoblog_postedit.png)

This is the page where you upload photos and create posts. The cos library was used for uploading photos, and the hash tags of the posting contents were automatically recognized and stored in the DB to create a hyperlink.

![Posting with hashtag](/assets/images/photoblog_hashtag.png)

Hashtag featue implemented, clicking on the hashtag will show the post with the same hashtag.

![Profile page](/assets/images/photoblog_profile.png)

You can see your posts in the profile page. Used Flex function of CSS.

![Following profile](/assets/images/photoblog_following.png)
![Unfollowing profile](/assets/images/photoblog_unfollowing.png)

Following/ Unfollowing user by clicking button

![프로필 편집](/assets/images/photoblog_profile_edit.png)

A page where you can edit your profile and change your profile picture.