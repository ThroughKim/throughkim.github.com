---
layout: post
title: 사진블로그
---

>교내 웹프로그래밍 강의의 프로젝트로 제작한 사진중심의 블로그입니다.

---

* Language : Java, JSP, CSS
* Library : cos
* DB : MySQL
* Web Server : Apache Tomcat
* 기타 : Git, IntelliJ
* [소스코드](https://github.com/ThroughKim/photoblog)

---

인스타그램의 UI를 모방하여 제작하였고, Java와 JSP를 이용해 만들었습니다.

![메인](/images/photoblog_main.png)

메인화면은 팔로우한 사용자의 포스팅만을 보여줍니다. 

![사진업로드](/images/photoblog_photo_select.png)
![포스팅작성](/images/photoblog_postedit.png)

사진을 업로드하고 포스팅을 작성하는 페이지입니다. 사진 업로드에는 cos 라이브러리를 사용하였고, 포스팅 내용의 해시태그를 자동으로 인식하여 DB에 저장하면서 하이퍼링크를 생성하도록 작성했습니다.

![해시태그 포스팅](/images/photoblog_hashtag.png)

해시태그 기능도 구현하여 해시태그를 클릭하면 동일한 해시태그가 포함된 게시물을 보여줍니다.

![자기프로필](/images/photoblog_profile.png)

프로필창에서 자신이 올린 게시물을 확인할 수 있습니다. CSS의 Flex기능을 이용했습니다.

![팔로잉프로필](/images/photoblog_following.png)
![언팔로잉프로필](/images/photoblog_unfollowing.png)

타인의 프로필페이지에서 간단히 버튼을 이용해 팔로잉/언팔로잉을 할 수 있도록 구현했습니다.

![프로필 편집](/images/photoblog_profile_edit.png)

프로필을 편집하고 자신의 프로필사진을 변경할 수 있는 페이지입니다.






