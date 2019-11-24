---
layout: post
title:  "iOS의 Window에 대해"
date:   2019-11-21 18:11:05 +0900
tags: iOS swift UIKit
---

최근 본 기술면접에서 iOS의 Window에 대한 질문이 나왔고, 기본이 되는 개념이지만 잘 모르고 있다는 생각이 들어 정리합니다. [View Programming Guide for iOS][view-programming-guide]의 Window 파트에서 필요하다고 생각하는 부분을 요약하고 부가적인 내용을 추가하였습니다.

<center><img src="{{ "/assets/img/2019-11-21-iOS의-Window에-대해/uiwindow-instance.png" | absolute_url }}" alt="uiwindow-instance" style="zoom:48%;"/></center>

## Windows

모든 iOS App은 하나 이상의 `UIWindow` 인스턴스가 필요하며, 어떤 앱들은 하나 이상의 윈도우를 포함할 수도 있습니다. 윈도우 객체는 이런 일들을 합니다:
* 어플리케이션의 visible content를 포함
* 어플리케이션의 뷰나 다른 객체에 터치 이벤트를 전달하는 중요한 역할을 수행합니다.
* 화면 회전 변화(orientation change)에 대한 대응을 쉽게 할 수 있도록 앱의 View Controller들과 상호작용합니다.

iOS에서, 윈도우는 다른 뷰들을 담는 빈 컨테이너로 동작하며, 대부분 앱의 전체 라이프타임 동안 하나의 윈도우만 생성하고 이 윈도우는 앱 실행 초기에 로드됩니다. (새로운 컨텐츠나 화면 등을 보여주기 위해 새로운 윈도우를 생성하는 것이 아닙니다.) 앱이 외부 디스플레이나 비디오를 지원하는 앱은 추가 윈도우를 생성하기도 합니다. 이외에 전화가 오거나 하는 특정 이벤트로 인해 생성되는 다른 추가적인 화면들은 대체로 시스템이 생성합니다.

### Tasks That Involve Windows

어플리케이션이 윈도우와 상호작용하는 경우는 대부분 앱을 시작할 때 윈도우를 생성하는 경우이지만, 몇몇 어플리케이션과 관련된 일을 수행하기 위해 윈도우 객체를 이용하기도 합니다.
* 윈도우와 특정 뷰 사이의 좌표 변환 시
* 앱이 윈도우를 보여주거나 숨길 때, 윈도우가 key window가 됐을 때나 key window에서 해제 됐을 때 등 윈도우 관련 변화에 대한 Notification을 받을 때 등
  * `UIWindowDidBecomeVisibleNotification`
  * `UIWindowDidBecomeHiddenNotification`
  * `UIWindowDidBecomeKeyNotification`
  * `UIWindowDidResignKeyNotification`

> key window: 터치 관련된 이벤트가 아닌 이벤트나, 키보드 이벤트 등을 받고 있는 윈도우. 한 시점에 하나의 윈도우만 키 윈도우이다.

### Creating and Configuring a Window

앱의 launch time에 윈도우를 생성 후에, 앱의 application delegate object에서 그 레퍼런스를 저장하고 유지해야 합니다.

Xcode 프로젝트를 생성하면, 다음 코드와 같이 `AppDelegate` 클래스의 프로퍼티로 **윈도우 객체에 대한 레퍼런스**를 가지고 있습니다. 코드에서 명시된 바와 같이 이 `AppDelegate` 클래스는 `UIApplicationDelegate` 프로토콜을 따르므로 이 클래스가 앱의 application delegate object가 됩니다.

```swift
//
//  AppDelegate.swift
//
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

}
```

> UIKit이 앱의 런치 사이클에서 app delegate object를 생성하며 앱이 실행되는 동안 이 객체는 계속 존재하기 때문에 위처럼 `AppDelegate` 클래스의 프로퍼티로 윈도우 객체에 대한 레퍼런스를 갖고 있으면 앱이 실행되는 동안 그 레퍼런스가 유지됩니다. 만약 AppDelegate 클래스의 프로퍼티로 선언하지 않고 `application(_:didFinishLaunchingWithOptions)`등 AppDelegate 안의 메서드 안에서 로컬 변수로 선언한다면, 메서드가 끝날 때 UIWindow의 인스턴스에 대한 레퍼런스 카운트가 0이 되어 메모리에서 해제되므로 앱을 실행시켰을 때 화면이 나오지 않게 됩니다. 

앱이 추가 윈도우를 생성한다면, 필요할 때 **lazily** 생성하도록 해야 합니다. 예를 들어 앱이 외부 디스플레이를 지원한다면 디스플레이가 실제로 연결되기를 기다렸다가 연결 된 후에 새 윈도우를 생성해야 합니다.

앱이 foreground로 launch되는지, background로 launch되는지와 상관 없이, 반드시 앱의 launch time에 윈도우를 셍성해야 합니다. 윈도우를 생성하고 설정하는 작업은 큰 작업량이 소모되는 작업이 아닙니다. 앱이 background로 런치 될 경우엔, 윈도우는 생성하되 foreground로 들어가기 전까지 윈도우를 visible하게 만들지 않으면 됩니다.

#### Creating Windows in Interface Builder

Xcode 프로젝트 템플릿에 이미 윈도우를 생성하는 작업이 되어 있습니다. 템플릿의 application delegate object에 이 윈도우에 대한 outlet이 선언되어 있으며, 이 outlet을 통해 코드에서 윈도우 객체에 접근할 수 있습니다.

#### Creating a Window Programmatically

어플리케이션의 메인 윈도우를 코드로 생성하려면 application delegate 객체에 있는 `application(_:didFinishLaunchingWithOptions:)` 메서드에서 다음과 같이 하면 됩니다.

```swift
//
//  AppDelegate.swift
//
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        
        if let window = window {
            window.rootViewController = RootTabBarViewController()
            window.makeKeyAndVisible()
        }
        
        return true
    }
}
```

이전에 언급한 바와 같이 window는 application delegate에 선언된 프로퍼티이며, 윈도우 객체에 대한 레퍼런스를 유지합니다. 외부 디스플레이를 위한 윈도우를 생성해야 한다면, 이 레퍼런스를 다른 변수에 할당 해야 하며 해당 디스플레이를 나타내는 main이 아닌 `UIScreen` 객체의 bound를 명시해야 합니다.

윈도우를 생성할 때, 사이즈를 항상 스크린에 꽉 차도록 설정해야 하며 화면을 상태바에 맞게 조절하기 위해서 윈도우의 사이즈를 줄이면 안됩니다. 상태 바는 윈도우의 가장 위에 항상 떠있기 때문에, 화면을 상태바에 맞게 조절하기 위해서는 윈도우에 올리는 뷰의 크기를 줄여야 합니다. 그리고 뷰 컨트롤러를 이용한다면, 뷰 컨트롤러가 자동으로 뷰의 사이즈를 조절해야 합니다.

#### Changing the Window Level

각 `UIWindow` 객체는 설정할 수 있는 `windowLevel` 프로퍼티를 갖고 있습니다. 이 프로퍼티는 해당 윈도우가 다른 윈도우에 비해 어느 곳에 위치할지를 결정합니다. 대부분의 경우에 앱 윈도우의 레벨을 변경할 일은 없습니다. 새 윈도우는 생성 시에 자동으로 normal window level로 할당됩니다. normal window level은 이 윈도우가 앱과 관련된 컨텐츠를 표시함을 의미합니다. 높은 윈도우 레벨은 앱 컨텐츠 위에 정보를 띄울 필요가 있을 때를 위해 예약되어 있습니다. 예를 들어 시스템 상태바나 경고 메시지 입니다. 이런 레벨에 윈도우를 직접 할당할 수 있긴 하지만, 특정 인터페이스를 사용하면 보통 시스템이 자동으로 이 일을 해줍니다. 예를 들어 상태바를 보여줄 때나 숨길 때, 또는 경고 창을 보여줄 때, 시스템이 자동으로 필요한 윈도우를 생성하고 이 요소들을 디스플레이해 줍니다.

## 마무리

이상 iOS의 Window에 대한 정리였습니다.

[view-programming-guide]: https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingWindows/CreatingWindows.html#//apple_ref/doc/uid/TP40009503-CH4-SW1
