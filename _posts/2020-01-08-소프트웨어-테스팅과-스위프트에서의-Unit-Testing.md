---
layout: post
title: "소프트웨어 테스팅과 스위프트에서의 Unit Testing"
date: 2020-01-08 19:21:48 +0900
tags: Swift iOS macOS UnitTesting UnitTest Xcode
---

소프트웨어 테스팅에 대한 기초 개념부터, 스위프트에서의 유닛 테스트 방법에 대해 정리합니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/test-succeeded.png" | absolute_url }}" alt="test-succeeded" style="zoom:48%;"/></center>
## 소프트웨어 테스팅의 4 레벨

애플리케이션을 릴리즈하기 전에, 의도한 대로 동작하는지 테스트하는 과정이 필요합니다. 테스팅에는 4가지 주요한 단계가 있는데, 바로 unit testing, integration testing, sytem testing, acceptance testing 입니다.

#### Unit Testing

유닛 테스트에서는, 소프트웨어의 유닛이나 컴포넌트들이 잘 동작하는지 확인하기 위한 평가를 수행합니다. 유닛 테스트의 주요 목적은 애플리케이션이 설계된 대로 동작하는지를 결정하는 것이며, 화이트박스 테스팅 기법이 보통 사용됩니다. 유닛 테스트에서의 유닛은 함수일수도 있고, 개별 프로그램, 또는 절차일 수도 있습니다. 유닛 테스트의 가장 큰 이점 중 하나는, 프로그램의 작은 부분이 바뀔 때마다 전체 테스팅 과정을 다시 수행하여 문제가 발생했을 경우 빠르게 이슈를 해결할 수 있다는 점 입니다. 흔히 정식 테스팅 전 개발자들이 유닛 테스트를 수행합니다.

#### Integration Testing

프로그램의 유닛들을 모두 모아 그룹으로 테스트하는 것을 인터그레이션 테스팅이라 합니다. 유닛들을 함께 동작시켜서 모듈과 함수들 간의 인터페이스에 결함이 있는지를 찾는 것이 목적입니다. 개별적인 유닛들이 잘 동작하더라도, 잘못 통합시킬 경우 전체적인 소프트웨어의 기능에 영향을 주게 됩니다. 인터그레이션 테스팅의 경우 다양한 테스트 방법들을 이용할 수 있으며, 유닛들이 어떻게 정의되어 있느냐에 따라 사용되는 테스트 기법이 달라집니다.

#### System Testing

시스템 테스팅에서는 완벽하게 통합된 전체 앱을 테스트합니다. 시스템이 요구사항을 모두 충족시키는지, Quality Standards를 만족시키는지를 평가하는 것이 목적입니다. 프로덕션과 아주 유사한 환경에서, 개발에 참여하지 않은 테스터들에 의해 수행됩니다. 클라이언트가 요구하는 기술적, 기능적, 비기능적, 비지니스적 요구사항들을 만족하는지를 검증하는 과정이기 때문에 아주 중요합니다.

#### Acceptance Testing

인수 테스트에서는 시스템을 릴리즈할 수 있는지를 테스트합니다. 소프트웨어 개발 도중에 일어나는 요구사항 변경이 사용자의 의도를 다소 잘못 해석했을 수도 있기 때문에, 사용자가 직접 프로그램이 자신의 비즈니스 니즈에 맞는지를 테스트합니다. 이 테스트를 통과하면, 프로그램이 프로덕션 단계에 들어갑니다.

## 스위프트의 Unit Test

#### Xcode에서 Unit Test 시작하기

다음으로 Xcode를 이용한 스위프트의 유닛 테스트에 대해 알아보겠습니다.

유닛 테스트를 하기 위해서는 프로젝트에 별도의 타겟이 필요합니다.  iOS 프로젝트를 생성할 경우, 다음 그림과 같이 프로젝트 생성 화면에서 **Include Unit Tests**를 선택하면 됩니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/include-unit-tests.png" | absolute_url }}" alt="include-unit-tests" style="zoom:48%;"/></center>
이미 프로젝트를 생성했을 경우 다음 그림에 표시된 **Add a target** 버튼을 클릭 후 **Unit Testing Bundle**을 선택합니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/add-a-target.png" | absolute_url }}" alt="add-a-target" style="zoom:48%;"/></center>
만약 macOS Command Line Tool로 프로젝트를 생성할 경우, **Add a target** 버튼으로 타겟을 추가하는 방법으로 진행 후에, Scheme Editor에서 다음 사진처럼 **Test**를 선택 후 **Add test target** 버튼을 눌러 테스트 타겟을 추가해야 합니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/scheme-editing.png" | absolute_url }}" alt="scheme-editing" style="zoom:48%;"/></center>
그 다음, 아래 사진처럼 **Target Membership**을 설정합니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/target-membership.png" | absolute_url }}" alt="target-membership" style="zoom:48%;"/></center>
처음엔 테스트 타겟에 체크가 되어있지 않으며, 이 부분을 체크해야 테스트 타겟에서 테스트할 타겟의 파일들을 인식할 수 있습니다.

{% include callout.html content="**주의할 점**은, 테스트하고 싶은 파일들에 대해 모두 개별적으로 **Target Membership**을 설정해 줘야 합니다." type="success" %}

#### 살펴보기

먼저 테스트 타겟을 추가했을 때 기본적으로 생성되는 파일을 자세히 살펴보겠습니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/basic-file.png" | absolute_url }}" alt="basic-file" style="zoom:48%;"/></center>
가장 위에 XCTest 프레임워크가 임포트되어있는 것을 볼 수 있습니다. XCTest는 Xcode에서 유닛 테스트, performance 테스트, UI 테스트를 하기 위한 프레임워크입니다. 이 프레임워크 안에 테스트를 위한 여러 클래스들이 존재합니다.

다음으로, `UnitTestingAppTests`가 `XCTestCase`를 서브클래싱한 것을 볼 수 있습니다. 하나의 테스트 케이스는 `XCTestCase` 클래스를 상속받은 하나의 클래스 단위로 작성합니다. 테스트 케이스는 연관된 테스트 메서드들의 모음입니다. `XCTestCase` 클래스는 `XCTest` 클래스를 상속합니다.

* **`XCTest`** — 테스트를 생성, 관리, 실행하기 위한 추상 베이스 클래스입니다. 각 테스트 메서드들이 불리기 전/후에 호출되는 `setUp()`과 `tearDown()` 인스턴스 메서드가 정의되어 있습니다.
* **`XCTestCase`** — `XCTest`를 상속하며, 클래스(하나의 테스트 케이스) 안의 모든 테스트 메서드들이 불리기 전/후에 호출되는 `setUp()`과 `tearDown()` 클래스 메서드가 정의되어 있습니다.

`setUp()`은 테스트 메서드 실행 전 초기 상태를 준비하는 데 사용하고, `tearDown()`은 테스트 완료 후 cleanup 작업을 수행할 때 사용합니다.

4가지 메서드들 중 상황에 맞는 메서드를 선택적으로 골라 override하여 작성하면 됩니다. 기본적으로 생성되는 파일에는 인스턴스 메서드 두 가지에 대한 오버라이드가 되어 있습니다.

추가적으로, 테스트 메서드 실행 중 `addTeardownBlock(() -> Void)` 메서드를 이용하여  teardown 코드 블록을 등록할 수 있습니다. 이 블록들은 해당 테스트 메서드가 끝난 후, `teakDown()` 인스턴스 메서드가 호출되기 전 last-in, first-out 순서로 호출됩니다. 등록된 teardown 블록들이나 오버라이드된 `teakDown()` 메서드들은 테스트 메서드의 성공, 실패 여부와 관계 없이 호출됩니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/order-of-execution.png" | absolute_url }}" alt="order-of-execution" style="zoom:40%;"/></center>
참고로, 테스트 케이스의 `setUp()`과 `tearDown()` 인스턴스 메서드는 assert 문을 포함할 수 있습니다. 해당 assert 문들은 모든 테스트 메서드들의 실행 전후로 검사됩니다. 하지만, 테스트 assertion들은 테스트 클래스 인스턴스가 필요하기 때문에, 테스트 케이스의 `setUp()`과 `tearDown()` 클래스 메서드에는 assert 문을 포함할 수 없습니다.

#### 테스트하기

먼저 테스트할 간단한 유닛을 추가해보겠습니다.

```swift
class Calculator {
    private var number: Int
    
    init(with number: Int) {
        self.number = number
    }
    
    var numberDescription: String {
        return "The number is: \(number)"
    }
    
    func multiply(by otherNumber: Int) {
        number *= otherNumber
    }
    
    func divide(by otherNumber: Int) {
        number /= otherNumber
    }
}
```

매개변수로 전달된 값으로 인스턴스 안의 프로퍼티를 곱하고 나누는 함수들과, 이 프로퍼티의 값을 설명하는 문자열을 반환하는 연산 프로퍼티가 구현되어 있습니다.

다음으로, 테스트 케이스에서 테스트할 모듈을 임포트해야 합니다.

```swift
import XCTest
@testable import UnitTestingApp
```

임포트할 때, `@testable` 어노테이션을 명시해야 합니다. 애플리케이션 코드를 작성할 때, 클래스, 함수 등에 access control 키워드를 지정하지 않으면, 디폴트로 `internal`로 지정되어 모듈 안에서만 이용할 수 있으며 다른 모듈에서는 접근할 수 없습니다. 유닛 테스트를 위해서는 테스트할 모듈의 코드를 유닛 테스트 모듈에서 접근할 수 있어야 합니다. 프로덕트 모듈을 임포트할 때 `@testable` 어노테이션을 붙이면, 유닛 테스트 타겟이 `internal`인 클래스나 함수 등에 접근할 수 있게 됩니다.

{% include callout.html content="`@testable` 어노테이션을 붙이고 프로덕트 모듈을 한 번 컴파일한 후부터 테스트 모듈에서 프로덕트 모듈을 인식합니다." type="success" %}

이제 테스트 코드를 작성합니다.

```swift
class UnitTestingAppTests: XCTestCase {
    
    var calculator: Calculator!

    override func setUp() {
        calculator = Calculator(with: 10)
    }

    func testMultiply() {
        calculator.multiply(by: 6)
        XCTAssertEqual(calculator.numberDescription, "The number is: 60")
    }

    func testDivide() {
        calculator.divide(by: 2)
        XCTAssertEqual(calculator.numberDescription, "The number is: 5")
    }
}
```

우선, 이전에 설명했던 대로 `setUp()` 인스턴스 메서드는 각 메서드가 호출되기 전에 호출되므로, 테스트할 클래스를 초기화하는 코드를 넣었습니다. `Calculator` 클래스의 곱하기와 나누기 기능을 테스트하는 테스트 메서드인 `testMultiply()`와 `testDivide()`도 작성했습니다. 

테스트 메서드는 `XCTestCase` 서브클래스의 인스턴스 메서드로 작성하며, 파라미터와 리턴값이 없습니다. 또한 소문자 단어 `test`로 시작하는 이름이어야 합니다. 이런 규칙들을 지키면 Xcode의 XCTest 프레임워크에 의해 자동으로 인식되어 Test navigator에 나타나며, 테스트 시 실행됩니다. 지키지 않으면 인식할 수 없기 때문에 테스트 시 실행되지 않고, Test navigator에도 나타나지 않습니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/test-navigator.png" | absolute_url }}" alt="test-navigator" style="zoom:40%;"/></center>
각 테스트 메서드들은 작성된 코드를 실행하고, 특정 조건이 맞는지 검사합니다. 이 때 `XCTest` 프레임워크에 포함된 `XCTAssert` 함수들을 이용합니다. 이 함수들을 이용한 문장을 test assertion이라고 합니다. 여기선 여러가지 `XCTAssert` 함수들 중 입력되는 표현식 두 가지가 같은지를 검사하는 `XCTAssertEqual`을 이용하였습니다.

단축키 ⌘ + U를 누르면 테스트를 실행합니다. 모든 테스트 메서드가 실행되며 test assertion들에 정의된 조건들이 모두 맞을 경우 테스트가 성공합니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/test-methods-succeeded.png" | absolute_url }}" alt="test-methods-succeeded" style="zoom:40%;"/></center>
하나라도 틀릴 경우 테스트가 실패합니다. 테스트 도중 실패한 테스트 메서드는 실패로 기록되어 테스트가 끝난 후에 Xcode에서 표시됩니다.

<center><img src="{{ "/assets/img/2020-01-08-소프트웨어-테스팅과-스위프트에서의-Unit-Testing/test-method-failed.png" | absolute_url }}" alt="test-method-failed" style="zoom:40%;"/></center>
`XCTAssertEqual()` 외에도 사용 가능한 `XCTAssert` 함수들을 몇 가지 정리해 보면 다음과 같습니다.

| 함수 | 설명 |
|:---|:---|
| <small>`XCTAssert()`,<br>`XCTAssertTrue()` | 표현식이 `True`인지 체크합니다. |
| <small>`XCTAssertFalse()` | 표현식이 `False`인지 체크합니다. |
| <small>`XCTAssertNil()` | 표현식이 `nil`인지 체크합니다. |
| <small>`XCTAssertNotNil()` | 표현식이 `nil`이 아닌 것을 체크합니다. `nil`일 경우 실패를 발생시킵니다. |
| <small>`XCTUnwrap()` | 표현식이 `nil`이 아닌 것을 체크하고 unwrap된 값을 반환합니다. 반환된 값을 테스트 안에서 뒤이어 사용할 수 있습니다. |
| <small>`XCTAssertEqual()` | 두 값이 같은지 체크합니다. 실수를 비교할 경우 `accuracy:` 매개변수를 통해 정확도를 전달합니다. |
| <small>`XCTAssertNotEqual()` | 두 값이 다른지 체크합니다. |
| <small>`XCTAssertGreaterThan()`, `XCTAssertGreaterThanOrEqual()`, `XCTAssertLessThanOrEqual()`, `XCTAssertLessThan()` | 각 조건을 체크합니다. |
| <small>`XCTAssertThrowsError()` | 함수 호출이 에러를 던지는지 체크합니다. |
| <small>`XCTAssertNoThrow()` | 함수 호출이 에러를 던지지 않는 것을 체크합니다. |
| <small>`XCTFail()` | 조건 체크 없이 즉시 실패를 발생시킵니다.  |

이 외에도 많은 assertion 함수들이 존재합니다. 더 자세한 내용은 [XCTest 프레임워크 문서][xctest]를 참고하세요.

## 마무리

이상으로 소프트웨어 테스팅에 대한 기초 지식부터, 스위프트에서의 XCTest 프레임워크, 그리고 Unit Test를 하는 방법까지 정리해보았습니다.

## References

* [The Four Levels of Software Testing][software-testing]
* [Add XCTest to a OS X Command Line Tool project][cmd-line-tool-unit-test]
* [XCTest Framework][xctest]
* [Understanding Setup and Teardown for Test Methods][test-methods]
* [Defining Test Cases and Test Methods][defining]
* [The swift programming language — Access Control][access-control]


[software-testing]: https://www.seguetech.com/the-four-levels-of-software-testing/
[cmd-line-tool-unit-test]: https://deangerber.com/blog/2015/10/21/add-xctest-to-a-os-x-command-line-tool-project/
[xctest]: https://developer.apple.com/documentation/xctest
[test-methods]: https://developer.apple.com/documentation/xctest/xctestcase/understanding_setup_and_teardown_for_test_methods
[access-control]: https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html
[defining]: https://developer.apple.com/documentation/xctest/defining_test_cases_and_test_methods
