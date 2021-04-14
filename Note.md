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

***

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


#### 5/98 Disposables
- Disposed는 Observable이 전달하는 Event는 아니다
- Parameter로 클로저를 던달하면 Observable과 관련된 모든 리소스가 제거된 후에 호출된다
- Observable이 Completed나 Error Event로 해제되었다면 리소스도 해제된다
- 그럼에도 되도록 해제할 때는 Disposable을 사용하는 것이 좋다
- dispose()를 직접 호출하기보다는 DisposeBag을 사용하는 것이 좋다
- dispose() 메소드를 직접 호출하면 completed event가 전달되지 않으므로, 직접 사용하지 않는 것이 좋다
- 만약 특정 시점에 해지하고 싶다면 take until 메소드를 사용하는 것이 좋다

#### 6/98 Operators

- Observable과 관련된 여러 메소드들을 연산자라고 부른다
- 연산자의 특징
    - 연산자들은 Observable 상에서 동작하고, 새로운 Observable을 리턴한다
    - Observable을 리턴하기 때문에 두 개 이상의 연산자를 연달아 호출할 수 있다
    - 연산자는 보통 subscribe 메소드 앞에 추가한다
    - 그래야 구독자로 전달된 최종 데이터가 우리가 원하는 데이터가 된다
    - 연산자를 호출할 때는 호출 순서에 주의해야 한다 -> 호출 순서에 따라 다른 결과가 나올 수 있다
  
<pre>
<code>
let bag = DisposeBag()
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9])
    .take(5) // take 연산자는 소스 옵저버블이 방출하는 요소 중에서 파라미터로 지정한 수만큼 방출하는 새로운 옵저버블을 생성한다 -> 처음 5개의 요소만 전달한다
    .filter { $0.isMultiple(of: 2) } // 짝수만 구독자로 전달 -> 1~5 사이의 짝수인 2와 4만 전달됨
    .subscribe { print($0) }
    .disposed(by: bag)
--> 출력결과
next(2)
next(4)
Completed
<--
</code>
</pre>


***

### [3] Subjects
#### 7/98 Subjects Overview
- Subject를 이해하기 위해서는 Observer와 Observable에 대해 알아야한다
- Observable은 Event를 전달한다
- Observer는 Observable을 구독하고 전달되는 Event를 처리한다
- Observable은 Observer와 달리 다른 Observable을 구독하지 못한다
- 마찬가지로 Observer는 다른 Observer로 Event를 전달하지 못한다
- 반면 Subject는 다른 Observable로부터 Event를 받아서 Observer로 전달할 수 있다
- 다시 말해 Subject는 Observable인 동시에 Observer이다

- RxSwift는 네 가지 Subject를 제공한다
    1. PublishSubject
        - Subject로 전달되는 새로운 Event를 Observer로 전달한다
    2. BehaviorSubject
        - 생성시점의 시작 이벤트를 지정한다 그리고 Subject로 전달되는 Event 중에서 가장 마지막에 전달된 최신 Event를 저장해 두었다가 새로운 Observer에게 최신 Event를 전달한다
    3. ReplaySubject 
        - 하나 이상의 최신 Event를 Buffer에 저장한다. Observer가 구독을 시작하면 Buffer에 있는 모든 Event를 전달한다
    4. AsyncSubject
        - Subject로 Completed event가 전달되는 시점에 마지막으로 전달된 Next event를 Observer로 전달한다
    
- RxSwift는 Subject를 래핑하고 있는 두 가지 Relay를 제공한다
    1. PublishRelay
        -  PublishSubject를 래핑한 것
    2. BehaviorRelay
        - BehaviorSubject를 래핑한 것
        
    - Relay는 일반적인 Subject와 달리 Next event만 받고 나머지 Completed, Error event는 받지 않는다
    - 주로 종료 없이 계속 전달되는 Event Sequence를 처리할 때 활용한다 


#### 8/98 Publish Subject
- PublishSubject는 Subject로 전달되는 Event를 Observer에게 전달하는 가장 기본적인 형태의 Subject이다
- Subject는 Observable인 동시에 Observer이다
- 다른 Source로부터 Event를 전달받을 수 있고, 다른 Observer로 이벤트를 전달할 수 있다
- Observable에서 Observer로 Next event를 전달할 때 Observer로 onNext 메소드를 호출하고, 파라미터로 요소를 전달한다
- Subject 역시 Observer이기 때문에 onNext를 호출할 수 있다
- PublishSubject는 구독 이후에 전달되는 새로운 Event만 Observer로 전달한다
- PublishSubject는 Event가 전달되면 즉시 Observer에게 전달한다. 그래서 Subject가 최초로 생성되는 시점과 첫 번째 구독이 시작되는 시점 사이에 전달되는 Event는 그냥 사라진다.
- Event가 사라지는 것이 문제가 된다면 Replay Subject를 사용하거나 Hold Observable을 사용한다


#### 9/98 Behavior Subject
- Behavior Subject는 Publish Subject와 유사한 방식으로 동작한다
- Subject로 전달된 Event를 Observer로 전달하는 것은 동일하다
- 하지만 Subject를 생성하는 방식에 차이가 있다
- Behavior Subject를 생성할 때는 Publish Subject와 다르게 하나의 값을 전달한다
- 또 다른 차이는 Subject를 구독할 때 나타난다
- Publish Subject는 내부에 Event가 저장되지 않은 상태로 생성된다
- 그래서 Subject로 Event가 전달되기 전까지 Observer로 Event가 전달되지 않는다
- Behavior Subject를 생성하면 내부에 Next event가 생성되고, Observer로 전달한 값이 저장된다
- 새로운 구독자가 추가되면 저장되어 있던 Next event가 바로 전달된다
- 다시 Behavior Subject로 Next event를 전달하면 Observer로 Next event가 전달된다
- 이 시점에 새로운 Observer가 추가되면 가장 최신 Next event를 Observer로 전달한다

<pre>
<code>
let b = BehaviorSubject<Int>(value: 0)

b.subscribe { print("BehaviorSubject1 >>", $0) }
    .disposed(by: disposeBag)

b.onNext(1)

b.subscribe { print("BehaviorSubject2 >>", $0) }
    .disposed(by: disposeBag)

//b.onCompleted()
b.onError(MyError.error)

b.subscribe { print("BehaviorSubject3 >>", $0) }
    .disposed(by: disposeBag)
    
--> 출력결과
BehaviorSubject1 >> next(0)
BehaviorSubject1 >> next(1)
BehaviorSubject2 >> next(1)
BehaviorSubject1 >> error(error) // or completed
BehaviorSubject2 >> error(error) // or completed
BehaviorSubject3 >> error(error) // or completed
<--
    
</code>
</pre>
