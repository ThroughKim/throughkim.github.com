---
layout: post
title: 'iOS Notification 만들기'
author: through.kim
date: 2016-12-27 11:39
tags: [ios, app, study]
image: '/files/covers/ios_app.jpg'
comments: true
---

# iOS Notification 만들기
> 본 포스팅은 OS X, Xcode 8, ios 10, Swift3.0 기반으로 작성되었습니다.

_APNS: Apple Push Notification Service_  

APNS를 이용해 ios 어플리케이션에 notification을 전송하는 방법을 기록해봅니다. 애플 개발자 아이디가 있으며, 계정에 테스트를 진행할 기기가 등록되어 있다는 조건하에 진행합니다.  
작업은 아래와 같은 순서로 진행됩니다.  

* TOC  
{:toc}

---

## 개발자 계정 세팅  

### 인증서 요청 생성  

애플 개발자 계정에서 Notificaiton 인증서를 만들기 위해서는 우선 아래와 같이 키체인을 이용해 CertificateSigningRequest라는 인증서를 만들어야 합니다.  
  
![키체인 메뉴](/files/ios_noti_images/1.png)

인증서 정보 입력 창에 자신이 사용하는 Apple ID(개발자 ID)와 이름을 입력 후 '디스크에 저장됨',  '본인이 키 쌍 정보 지정'을 선택 후 계속 합니다.  

![인증서 설정](/files/ios_noti_images/2.png)

이어서 키 크기 2048비트 RSA알고리즘으로 설정하고 인증서 요청을 생성하면 됩니다.

### APP ID 생성
푸시 알림을 수신할 앱의 ID를 먼저 생성해줍니다. 우선 [Apple Developer - Member Center](https://developer.apple.com/account/)에 접속합니다. 이어서 Certificates, Identifiers & Profiles를 클릭하여 이동합니다.  

![Apple Developer - Member Center](/files/ios_noti_images/3.png)

이어서 좌측 패널의 App IDs를 클릭후 +버튼을 눌러 새로운 앱을 추가해줍니다.  

![App IDs 추가](/files/ios_noti_images/4.png)  

추가할 앱의 이름을 입력해주고, 사용할 번들ID를 적어줍니다. APNS를 사용하려면 Explicit App ID를 체크하고 작성해야 합니다.  
Bundle ID는 보통 도메인 역순 + 앱 이름으로 작성합니다.

![앱 정보 입력](/files/ios_noti_images/5.png)

하단의 App Service에서 Push Notification을 선택해준 뒤 Continue를 클릭합니다.  
![푸시 알림 기능 선택](/files/ios_noti_images/6.png)

다음 창에서 Push Notification에 Configurable이 되어있는 것을 확인한 뒤 Register버튼을 클릭합니다.  
![Push Notification에 노란불을 확인](/files/ios_noti_images/7.png)

### APNS 인증서 발급

다시 App IDs탭으로 돌아오면 방금 만든 앱이 나타나있는 것을 확인할 수 있습니다. 앱을 클릭하고 Edit버튼을 클릭해줍니다.  
![App ID 생성](/files/ios_noti_images/8.png)

Push Notification을 확인해보면 개발용 인증서와 배포용 인증서를 만들 수 있는 것을 확인할 수 있습니다. 우선 개발용으로 생성합니다.  
![개발용 인증서 선택](/files/ios_noti_images/9.png)

다음 창에서 Continue를 클릭하면 아까 생성했던 CertificateSigningRequest파일을 업로드 하는 창이 나옵니다. Choose File버튼을 클릭하여 추가해줍니다.
![인증서 선택 후 계속](/files/ios_noti_images/10.png)

이제 인증서가 완성되었습니다. 다운로드 하여줍니다.
![인증서 완성](/files/ios_noti_images/11.png)

다운로드 된 인증서 파일을 더블클릭하여 실행하면 키체인에 등록이 됩니다.

---

### 서버용 APNS 인증서 발급  
키체인을 열어 방금 설치한 Apple Development(or Production) iOS Push Service 인증서를 찾아서 Export를 클릭합니다.
![Export하기](/files/ios_noti_images/12.png)

.p12확장자로 생성을 하면 됩니다. 저장을 클릭하면 암호 설정창이 팝업되며, 적당한 암호를 입력한 뒤 생성을 완료해줍니다.  
![p12파일 생성](/files/ios_noti_images/13.png) 

키체인에서 오른쪽에 화살표를 클릭하면 키가 노출되는데, 이 키 또한 방금전과 동일한 방법으로 .p12파일로 내보내기 해줍니다.
![키파일 생성](/files/ios_noti_images/14.png)

작업이 완료되면 지정한 경로에 cert.p12 key.p12와 같이 지정한 이름으로 인증서 파일 두 개가 생성되어있는 것을 확인할 수 있습니다. 이제 터미널에서 인증서 형식 변환을 해줍니다. .p12확장자 파일을 .pem파일로 변환해주는 작업입니다.

```bash
$ openssl pkcs12 -clcerts -nokeys -out cert.pem -in cert.p12
```
다 입력 후 엔터를 치면 비밀번호 입력라인이 나타납니다. 방금 전에 설정했던 비밀번호를 입력해줍니다. 작업이 완료되면 같은 폴더상에 cert.pem이라는 파일이 생성되어있는 것을 확인할 수 있습니다.  
이어서 key.p12에도 같은 작업을 적용해줍니다.  

```bash
$ openssl pkcs12 -nocerts -out key.pem -in key.p12
```
cert.pem과는 다르게 비밀번호 체크 후 다른 비밀번호를 설정해주는 단계가 있습니다.  

이제 위에서 만들어준 key.pem을 가지고 key.unencrypted.pem 을 만들어줍니다. 이 단계에서 방금 새로 생성해 준 패스워드가 쓰입니다.  

```bash
$ openssl rsa -in key.pem -out key.unencrypted.pem
```

이제 마지막 단계입니다. key와 cert를 합쳐 최종 APNS에 쓰이는 인증서를 만들어주는 작업입니다.  

```bash
$ cat cert.pem key.unencrypted.pem > apns.pem
```

이렇게 작업을 하면 총 6개의 파일이 폴더 내에 생성되게 됩니다.  

```bash
$ ls
cert.p12  key.p12  cert.pem  key.pem  key.unencrypted.pem  apns.pem
```

### 프로비저닝 생성
Xcode에서 앱을 개발할 때 사용할 프로비저닝을 생성해줍니다. Certificates, Identifiers & Profiles페이지 좌측패널 하단에 Provisioning Profiles를 선택하고 +버튼을 눌러 새로운 프로파일을 추가해줍니다.  
![Provisioning Profiles](/files/ios_noti_images/15.png)

iOS App Development를 선택 후 계속 버튼을 눌러줍니다.
![iOS App Development 선택](/files/ios_noti_images/16.png)

계속해서 미리 생성해두었던 App Id를 선택 후 Continue를 클릭합니다.
![App Id 선택](/files/ios_noti_images/17.png)

자기 이름으로 된 인증서를 선택해줍니다.
![인증서 선택](/files/ios_noti_images/18.png)

테스트를 진행할 기기를 선택합니다. 푸시 알림의 경우 에뮬레이터로 확인할 수 없어 반드시 기기를 등록해 확인해야 합니다.
![기기 선택](/files/ios_noti_images/19.png)

다음 페이지에서 적당한 프로필 이름을 작성해주면 프로비저닝이 완성됩니다.
![프로비저닝 완성](/files/ios_noti_images/20.png)
다운로드 버튼을 클릭하여 프로비저닝을 다운로드 하고, 다운로드 된 파일을 더블클릭하면 Xcode에 자동으로 적용됩니다.

---

## iOS Application 작성  

### 프로젝트 설정

iOS 어플리케이션은 Xcode를 이용해 작성합니다. 최신 iOS의 경우 Xcode구버전은 제대로 지원하지 않는 경우가 있으므로 반드시 Xcode를 최신 버전으로 업데이트 후 진행합니다.  
우선 Xcode에서 신규 프로젝트를 생성합니다.  
![신규 프로젝트 생성](/files/ios_noti_images/21.png)

지난번 App ID를 생성할 때 입력했던 정보대로 입력해줍니다. Product Name과 Organization Identifier를 App ID 생성할 때 입력했던 Bundle Identifier의 내용에 맞추어 작성합니다.
![프로젝트 identifier생성 주의](/files/ios_noti_images/22.png)

프로젝트 생성이 완료되면 프로젝트 설정 창이 뜹니다. 상단에서 Build Setting을 클릭 후 우측 검색창에서 code signing을 검색해 준 뒤, Debug 우측 창을 클릭하여, 설정해두었던 Push service인증서를 선택해줍니다. (뒤에 따라오는 identifier로 구분합니다)
![Push Service 인증서 선택](/files/ios_noti_images/23.png)

이어서 좌측 패널에서 Targets 밑에 있는 앱이름을 클릭하여 세팅을 해줍니다. 우선 Capability옵션에서 Push Notification기능을 켜줍니다.  
![Push Notification On](/files/ios_noti_images/24.png)

마지막으로 Build Phases 옵션에서 Link Binary With Libaraies를 클릭하고, +버튼을 눌러 'UserNotifications.framework'와 'PushKit.framework'를 추가해줍니다.
![라이브러리 추가](/files/ios_noti_images/25.png)

### AppDelegate.swift 수정

이제 `AppDelegate.swift`파일을 수정하여 푸시 알림을 받을 수 있도록 만들어줍니다.  

```swift
import UIKit
import UserNotifications

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?


    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        /**************************** Push service start *****************************/
        // iOS 10 support
        if #available(iOS 10, *) {
            UNUserNotificationCenter.current().requestAuthorization(options:[.badge, .alert, .sound]){ (granted, error) in }
            application.registerForRemoteNotifications()
        }
            // iOS 9 support
        else if #available(iOS 9, *) {
            UIApplication.shared.registerUserNotificationSettings(UIUserNotificationSettings(types: [.badge, .sound, .alert], categories: nil))
            UIApplication.shared.registerForRemoteNotifications()
        }
            // iOS 8 support
        else if #available(iOS 8, *) {
            UIApplication.shared.registerUserNotificationSettings(UIUserNotificationSettings(types: [.badge, .sound, .alert], categories: nil))
            UIApplication.shared.registerForRemoteNotifications()
        }
            // iOS 7 support
        else {  
            application.registerForRemoteNotifications(matching: [.badge, .sound, .alert])
        }
        /***************************** Push service end ******************************/
        return true
    }
    
    // Called when APNs has assigned the device a unique token
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // Convert token to string
        let deviceTokenString = deviceToken.reduce("", {$0 + String(format: "%02X", $1)})
        
        // Print it to console
        print("APNs device token: \(deviceTokenString)")
        
        // Persist it in your backend in case it's new
    }
    
    // Called when APNs failed to register the device for push notifications
    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        // Print the error to console (you should alert the user that registration failed)
        print("APNs registration failed: \(error)")
    }

    func applicationWillResignActive(_ application: UIApplication) {
        // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
        // Use this method to pause ongoing tasks, disable timers, and invalidate graphics rendering callbacks. Games should use this method to pause the game.
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
        // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
        // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        // Called as part of the transition from the background to the active state; here you can undo many of the changes made on entering the background.
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    }

    func applicationWillTerminate(_ application: UIApplication) {
        // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    }
    
    // Push notification received
    func application(_ application: UIApplication, didReceiveRemoteNotification data: [AnyHashable : Any]) {
        // Print notification payload data
        print("Push notification received: \(data)")
    }


}

```

### 구동 테스트

저장해준 뒤 아이폰을 USB로 연결하고 Run 버튼을 클릭해주면 아이폰에서 어플이 실행됩니다. 어플이 실행되고, 푸시 알림설정이 제대로 되었다면 다음과 같은 확인창이 팝업됩니다.  
![알림 수신 확인창](/files/ios_noti_images/26.png)

확인을 클릭하면 Xcode 하단 콘솔창에 Device 토큰이 표시되는 것을 확인할 수 있습니다.  
![콘솔창에 ID 출력](/files/ios_noti_images/27.png)



---

## APNS 테스터로 테스트
웹 상에있는 APNS테스트로 서버없이도 간단하게 푸시 알림을 테스트 해볼 수 있습니다.  

[APNS/GCM Tester](http://apns-gcm.bryantan.info/)  
  
위 사이트로 이동하여 방금 수신한 Device 토큰과 메세지, 그라고 APNS용 인증서를 업로드 해주면 간단히 테스트가 가능합니다.  
![APNS테스터](/files/ios_noti_images/28.png)

제출을 하면 방금 입력한 메세지가 푸시 알림으로 표시되는 것을 확인할 수 있습니다.
![푸시 알림 확인](/files/ios_noti_images/29.png)

---

## 서버에 업로드
apns인증서를 Node.js 혹은 Django 또는 다른 어떤 웹서버든 업로드 하여 푸시 알림 서버를 구성할 수 있습니다.


---

## References

- [APNS 따라하기 시리즈](http://qnibus.com/blog/how-to-make-certification-for-apns/)
- [Node.js를 이용하여 iOS 푸시 서비스 구현하기](http://blog.saltfactory.net/node/implementing-push-notification-service-for-ios.html)