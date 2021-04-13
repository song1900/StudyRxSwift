#  StudyRxSwift - KxCoding


### [1] Hello, RxSwift
#### 1/98 Hello, RxSwift
- RxSwift를 공부하는 순서
<br>1. Swift Language
<br>2. Functional Programming, Protocol Oriented Programming
<br>3. RxSwift

- RxSwift의 장점
<br> 단순하고 직관적인 코드를 작성할 수 있다

- RxSwift 공부하는 법
<br> 처음부터 RxSwift가 무엇인지 명확히 이해하는 것은 불가능하다
<br> 처음부터 이해하려고 하면 지친다. 코드를 통해 작은 것부터 하나씩 배워 나가라
<br> 처음부터 이해되는 것은 말이 안 되니 이해 안 되는 부분은 그냥 건너 뛰어라


#### 2/98 Hello Reactive Programming
- RxSwift로 코드를 작성하면 값이나 상태의 변화에 따라서 새로운 결과를 도출하는 코드를 비교적 쉽게 작성할 수 있다
  - 반응형 프로그래밍(Reactive Programming)


### [2] Key Concepts
#### 3/98 Observable and Obervers #1
- 가장 중요한 Observable
<br> Obserbable은 Observable Sequence 또는 Sequence 라고 부른다
<br> Observable은 Event를 전달한다
<br> Observer는 Observable을 감시하고 있다가 전달되는 Event를 처리한다
<br> Observable을 감시하는것을 구독한다고 표현한다. 그래서 Observer를 구독자라고 부르기도한다
- Observable은  세가지 Event를 전달한다(Next, Error, Completed)
    - Observable에서 발생한 새 Event는 Next Event를 통해서 구독자로 전달된다
    - Event에 값이 포함되어 있다면 Next 이벤트와 함께 전달된다.
    - RxSwift에서는 이를 Emission(방출, 배출)이라고 한다.
    - Observable에서 error가 발생하면 Error event가 작동되고, 정상적으로 작동되면 Completed가 작동된다. 
    - Error event나 Completed event는 Emission이라고 하지 않고 Notification이라고 부른다.

- Observable을 생성하는 두 가지 방법
    - (1) Create 연산자를 통해서 Observable의 동작을 직접 구현하는 방법
        - Create 연산자는 Observable Protocol type에 선언되어 있는 Type 메소드이다
        - RxSwift에서는 이런 메소드들을 연산자라고 부른다
        - Create 연산자는 하나의 클로저를 파라미터로 받는다 (Observer를 받아서 Disposable을 리턴한다)
        - Disposable은 메모리 정리에 필요한 객체이다.
            <br>
            <pre>
            <code>
            Observable<Int>.create { (observer) -> Disposable in
              observer.on(.next(0))
              observer.onNext(1)
              
              observer.onCompleted()
              
              return Disposables.create()
            }
            </code>
            </pre>

    - (2) Create가 아닌 다른 연산자를 사용하는 방법
        - 미리 정의된 규칙에 따라 이벤트를 전달하는 from 연산자
        - from 연산자를 활용해서 create 연산자로 만든 Observable과 동일한 이벤트를 전달하는 Observable을 생성한다
        - from 연산자는 파라미터로 전달한 배열에 있는 요소를 순서대로 방출하고 Completed 이벤트를 전달하는 Observable을 생성한다
        - 단순히 순서대로 방출되는 Observable을 생성할 때는 create 연산자로 직접 구현하는 것보다 from을 활용하는 것이 좋다
        - Observable이 생성된 상태일 뿐 방출되거나 event가 전달되지 않는다
        - Observable은 event가 어떤 순서로 전달되어야 하는지 정의할 뿐이다
        - 구현했던 클로저가 실행되거나 from 연산자로 만든 Observable에서 event가 전달되는 시점은 Observer가 Observable을 구독하는 시점이다
        - 이 시점에 Next event를 통해서 두 개의 정수가 순서대로 방출되고 이어서 completed event가 전달된다
        <br>
        <pre>
        <code>
        Observable.from([0, 1])
        </code>
        </pre>
    
#### 4/98 Observable and Obervers #2
- 실제로 event가 전달되는 시점은 observer가 구독을 시작하는 시점
    - Observer는 Observable에서 전달되는 event를 처리한다.
    - 이것을 구독한다고 표현한다
    - 그래서 observer를 구독자라고 표현하기도 한다
    
- observer가 구독을 시작하는 방법
    - Observable에서 subscribe 메소드를 호출하는 것
    - subscribe 메소드는 Observable과 observer를 연결한다
    - 두 요소를 연결해야 이벤트가 전달되므로 Rx에서 가장 기초적이고 필수적이다
        <br>
        <pre>
        <code>

        let o1 = Observable<Int>.create { (observer) -> Disposable in
            observer.on(.next(0))
            observer.onNext(1)
            
            observer.onCompleted()
            
            return Disposables.create()
        }

        //  첫번째 방법
        o1.subscribe {
            print($0)
            
            if let elem = $0.element {
                print(elem)
            }
        }
        --> 출력결과
        next(0)
        0
        next(1)
        1
        completed
        <--  

        //  두번째 방법
        o1.subscribe { (elem) in
            print(elem)
        }
        --> 출력결과
        0
        1
        <--  

        </code>
        </pre>

- Observer는 동시에 두가지 Event를 처리하지 않는다
    - Observable은 Observer가 하나의 Event를 처리한 후에 이어지는 Event를 전달한다
    - 여러 Event를 동시에 전달하지 않는다
        <br>
        <pre>
        <code>
        o1.subscribe {
            print("== Start == ")
            print($0)
            
            if let elem = $0.element {
                print(elem)
            }
            
            print("== End ==")
        }
        --> 출력결과
        == Start == 
        next(0)
        0
        == End ==
        == Start == 
        next(1)
        1
        == End ==
        == Start == 
        completed
        == End ==
        <--  
        
        </code>
        </pre>
