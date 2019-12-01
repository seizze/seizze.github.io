---
layout: post
title:  "iOS의 Responder와 Responder Chain 이해하기"
date:   2019-11-26 11:11:50 +0900
tags: iOS swift UIKit
---

UIKit과 관련된 애플 문서를 볼 때 빈번하게 등장하는 Reponder Chain에 관련한 내용을 정리합니다. 여러 애플 문서를 참고하였으며, 포스트의 맨 아래 References에서 확인할 수 있습니다.

<center><img src="{{ "/assets/img/2019-11-26-iOS의-Responder와-Responder-Chain-이해하기/ios-responder-chain.png" | absolute_url }}" alt="ios-responder-chain" style="zoom:48%;"/></center>

## Responder Object and UIResponder Class

Responder는 이벤트를 핸들링하고 이벤트에 반응할 수 있는 객체입니다. 모든 responder 객체는 `UIResponder`에서 상속된 클래스들의 인스턴스입니다. 이 클래스는 이벤트 핸들링을 위한 인터페이스와 responder들의 기본적인 행위를 정의합니다. `UIApplication`, `UIViewController` 객체들, 모든 `UIView` 객체들(`UIWindow` 객체 포함)을 포함한 많은 주요 객체들 또한 reponder입니다. 이벤트가 일어나면, UIKit은 이벤트 핸들링을 위해 해당 이벤트를 앱의 reponder 객체들에게 보냅니다.

<center><img src="{{ "/assets/img/2019-11-26-iOS의-Responder와-Responder-Chain-이해하기/uiresponder.png" | absolute_url }}" alt="uiresponder" style="zoom:90%;"/></center>

이벤트의 종류엔 터치 이벤트, 모션 이벤트, 원격 조종 이벤트, press 이벤트 등이 있습니다. 특정 이벤트를 핸들링하기 위해서는 responder가 해당 이벤트에 대응되는 메서드들을 오버라이드하여 구현해야 합니다. 예를 들어, 터치 이벤트를 핸들링하기 위해서는 responder가 `touchesBegan(_:with:)`, `touchesMoved(_:with:)`, `touchesEnded(_:with:)`, `touchesCancelled(_:with:)` 메서드를 구현해야 합니다. 터치의 경우에, responder는 터치의 변화를 트래킹하고 앱의 인터페이스를 적절히 업데이트하기 위해서 UIKit에서 제공하는 이벤트 정보를 이용합니다.

Responder들은 `UIEvent` 객체를 처리할 수도 있고, input view를 통해 커스텀 input을 받아들일 수도 있다. 시스템 키보드가 이 input view에 대한 가장 명료한 예시입니다. 사용자가 화면의 `UITextField` 또는 `UITextView` 객체를 탭하면, 해당 뷰는 first responder가 되고, input view를 디스플레이합니다. 이 경우의 input view는 시스템 키보드입니다. 이와 비슷하게, 커스텀 input view를 생성하여 다른 responder가 활성화됐을 때 이 input view를 디스플레이할 수도 있습니다. 커스텀 input view를 responder에 연결하려면, 해당 뷰를 responder의 `inputView` 프로퍼티에 할당하세요.

다음은 UIKit에 정의되어 있는 UIResponder 클래스입니다.

```swift
@available(iOS 2.0, *)
open class UIResponder : NSObject, UIResponderStandardEditActions {

    open var next: UIResponder? { get }
    
    open var canBecomeFirstResponder: Bool { get } // default is NO
    open func becomeFirstResponder() -> Bool

    open var canResignFirstResponder: Bool { get } // default is YES
    open func resignFirstResponder() -> Bool

    open var isFirstResponder: Bool { get }
    // ...
}
```

## The Responder Chain

이벤트 핸들링 말고도, UIKit responder들은 처리되지 않은 이벤트를 앱의 다른 파트로 forwarding하는 일도 담당합니다. **Responder chain**은 responder 객체들이 이벤트나 액션 메시지를 핸들링할 책임을 앱의 다른 객체들에게 전송할 수 있도록 해 줍니다. 특정 정해진 responder가 이벤트를 핸들링하지 않을 경우, 해당 reponder는 그 이벤트를 reponder chain의 다음 객체에게로 포워딩합니다. 메시지는 처리될 때까지 계속 chain의 상위 객체들로 이동합니다. 마지막까지 처리되지 않을 경우, 앱이 해당 메시지를 버립니다.

<center><img src="{{ "/assets/img/2019-11-26-iOS의-Responder와-Responder-Chain-이해하기/ios-responder-chain.png" | absolute_url }}" alt="ios-responder-chain" style="zoom:48%;"/></center>

### The path of an event

일반적인 이벤트는 responder chain에서 뷰(first responder 또는 터치된 뷰)에서 시작해서, 뷰 계층을 따라 window 객체를 거쳐 app 객체에 도달할 때까지 이동합니다. UIKit은 일정한 규칙을 사용하여 responder chain을 동적으로 관리하는데, 이 규칙에 의해 어떤 객체가 다음 순서로 이벤트를 받을지가 결정됩니다. 예를 들어, 뷰는 뷰의 슈퍼뷰에게로 이벤트를 포워딩하고, 뷰 계층의 루트 뷰는 루트 뷰의 뷰 컨트롤러에게로 이벤트를 포워딩합니다.

다음 그림은 이벤트가 responder chain을 따라 한 responder에서 다음 responder로 이동하는 내용을 예시와 함께 보여줍니다.

<center><img src="{{ "/assets/img/2019-11-26-iOS의-Responder와-Responder-Chain-이해하기/ios-responder-chain-example.png" | absolute_url }}" alt="ios-responder-chain-example"/></center>

* Text field가 이벤트를 핸들링하지 않으면, UIKit은 text field의 부모인 UIView 객체에게로 이벤트를 보내고, 이어서 윈도우의 루트 뷰에게로 보냅니다.
* 루트 뷰에서, Responder chain은 이벤트를 윈도우로 보내기 전에 방향을 바꾸어 루트 뷰를 소유하고 있는 뷰 컨트롤러에게로 보냅니다.
* 만약 윈도우가 이벤트를 처리하지 못하면, UIKit은 이벤트를 `UIApplication` 객체에게로 보냅니다. 그리고 app delegate가 `UIResponder`의 인스턴스이고 responder chain의 일부가 아니라면 이벤트를 app delegate에게로 보냅니다.

## The First Responder

앱에서 많은 종류의 이벤트들을 처음으로 받는 responder 객체를 **first responder**라고 합니다. First responder는 대체로 앱이 이벤트를 핸들링하기 가장 적합하다고 간주하는 responder 객체입니다. 앱이 이벤트를 받으면, UIKit이 해당 이벤트를 가장 적합한 responder 객체인 first responder에게로 보냅니다.

이벤트를 받기 위해서는, responder는 자신이 first responder가 될 수 있음을 나타내야 합니다. first responder가 될 수 있게 하려면, `UIResponder`의 서브클래스에서 `canBecomeFirstResponder` 프로퍼티를 오버라이드하여 true를 리턴하도록 만들어야 합니다.

이벤트 메시지들을 받는 것 이외에, responder는 target이 특정되지 않은 액션 메시지들을 받을 수도 있습니다. (액션 메시지들은 버튼이나 사용자가 조절 가능한 control들 등 컨트롤들로부터 보내집니다.)

### Determining an Event's First Responder

UIKit은 받은 이벤트의 종류에 따라서 특정 객체를 해당 이벤트의 first responder로 지정합니다.

| Event type   | First responder   |
|:-------------|:------------------|
| Touch events | 터치가 일어난 뷰 |
| Press events | 포커스를 가진 뷰 |
| Shake-motion events | UIKit이 지정한 객체 또는 직접 지정 |
| Remote-control events | UIKit이 지정한 객체 또는 직접 지정 |
| Editing menu messages | UIKit이 지정한 객체 또는 직접 지정 |

컨트롤은 연관된 타겟 객체와 직접 액션 메시지를 이용해 소통합니다. 액션 메시지는 이벤트는 아니지만 responder chain을 이용합니다. 컨트롤의 타겟 객체가 nil일 경우, UIKit은 first responder에서 시작하여 적절한 액션 메시지를 구현한 객체를 만날 때까지 responder chain을 따라 이동합니다.

뷰에 있는 Gesture recognizer의 경우 또한 터치 등을 인식하지 못하면 UIKit은 뷰로 터치를 보내고, 뷰도 터치를 처리하지 않을 경우 마찬가지로 responder chain을 따라 이벤트를 보냅니다.

## Determining Which Responder Contained a Touch Event

UIKit은 어디서 터치 이벤트가 발생했는지를 결정하기 위해 뷰 기반 hit-testing을 사용합니다. UIKit은 터치 위치를 뷰 계층에 있는 뷰 객체의 바운드와 비교합니다. `UIView`의 `hitTest(_:with:)` 메서드는 특정 터치를 포함하는 가장 깊은 서브뷰를 찾기 위해 뷰 계층을 따라서 이동하고, 이 가장 깊은 서브뷰가 터치 이벤트의 first responder가 됩니다.

> 터치 위치가 뷰의 경계 밖이라면, `hitTest(_:with:)` 메서드는 해당 뷰와 그 뷰의 모든 서브뷰들을 무시합니다. 결과적으로, 뷰의 `clipToBounds` 프로퍼티가 `false`라면, 그 뷰의 밖에 있는 서브뷰들은 터치를 포함하더라도 반환되지 않습니다.

터치가 일어나면, UIKit은 `UITouch` 객체를 만들고 뷰와 연결합니다. 터치 위치나 다른 파라미터들이 변경되면, UIKit은 같은 `UITouch` 객체를 새로운 정보로 업데이트합니다. 변경되지 않는 유일한 프로퍼티는 뷰입니다. 심지어 터치 위치가 원래 뷰의 바깥으로 이동했더라도, 터치의 `view` 프로퍼티의 값은 변하지 않습니다. 터치가 끝나면, UIKit은 `UITouch` 객체를 메모리에서 해제합니다.

## Altering the Responder Chain

Responder 객체의 `next` 프로퍼티를 오버라이드하여 responder chain을 변경할 수 있습니다. 이 작업을 할 때, next responder는 오버라이드한 프로퍼티에서 반환하는 객체입니다.

많은 UIKit 클래스들은 이미 이 프로퍼티를 오버라이드하여 특정 객체들을 반환하고 있습니다.
* `UIView` 객체 — 만약 해당 뷰가 뷰 컨트롤러의 루트 뷰라면, next responder는 뷰 컨트롤러입니다. 아닌 경우엔, next responder는 해당 뷰의 슈퍼뷰입니다.
* `UIViewController` 객체 — 만약 뷰 컨트롤러의 뷰가 윈도우의 루트 뷰라면, next responder는 윈도우 객체입니다. 만약 뷰 컨트롤러가 다른 뷰 컨트롤러에 의해 프레젠트된 경우, next responder는 presenting 뷰 컨트롤러 입니다.
* `UIWindow` 객체 — 윈도우의 next responder는 `UIApplication` 객체입니다.
* `UIApplication` 객체 — `UIApplication` 객체의 next responder는 app delegate입니다. 하지만 앱 딜리게이트가 `UIResponder`의 인스턴스이면서 뷰, 뷰 컨트롤러, 또는 앱 객체 자신이 아닐 때만 해당됩니다.

## References

* [Using Responders and the Responder Chain to Handle Events][using-responders]
* [UIResponder][uiresponder]
* [Responder object][responder-object]

[responder-object]: https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html
[uiresponder]: https://developer.apple.com/documentation/uikit/uiresponder
[using-responders]: https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events
