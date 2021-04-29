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


#### 10/98 Replay Subjects
- Behavior Subject는 가장 최근 Next event 하나를 저장했다가 새로운 Observer로 전달한다
    - 최신 이벤트를 제외한 나머지 모든 이벤트는 사라진다

- 두 개 이상의 Event를 저장해두고 새로운 Observer로 전달하고 싶다면 Replay Subject를 사용한다
- Replay Subject는 create 메소드로 생성한다
- Subject를 생성할 때 buffer의 크기를 지정하는데 bufferSize를 3으로 하면 세 개의 Event를 저장하는 buffer가 생성된다
- buffer에는 가장 마지막에 전달된 세 개의 Event가 전달된다
- Replay Subject는 지정된 buffer 크기만큼 최신 Event를 저장하고 새로운 Observer에게 전달한다
- buffer는 메모리에 저장되기 때문에 항상 메모리 사용량에 신경 써야 한다
- 필요 이상으로 큰 buffer를 쓰는 것은 피해야 한다
- Replay Subject는 종료 여부에 관계 없이 항상 buffer에 저장돼 있는 Event를 새로운 Observer에게 전달한다


#### 11/98 Async Subjects
- Async Subject는 이전의 Subject들과 Event를 전달하는 시점에 차이가 있다
- Publish Subject, Behavior Subject, Replay Subject는 Subject로 Event가 전달되면 즉시 Observer에게 전달한다
- 반면 Async Subject는 Subject로 Completed event가 전달되기 전까지 어떤 Event도 Observer로 전달하지 않는다
- Completed event가 전달되면 그 시점에 가장 최근에 전달된 Next event 하나를 Observer에게 전달한다
- Async Subject는 Completed event가 전달된 시점을 기준으로 가장 최근에 전달된 하나의 Next event를 Observer에게 전달한다
- 만약 Async Subject로 전달된 Next event가 없다면 그냥 Completed event만 전달하고 종료한다
- Error event가 전달된 경우에는 Next event가 Observer에게 전달되지 않고, Error event만 전달되고 종료한다


#### 12/98 Relays
- RxSwift는 두가지 Relay를 제공한다.
    1. PublishRelay
    2. BehaviorRelay
- Relay는 Subject와 유사한 특징을 가지고 있고, 내부에 Subject를 래핑하고 있다
- Publish Relay는 Publish Subject를 래핑하고 있고, Behavior Relay는 Behavior Subject를 래핑하고 있다
- Relay는 Subject와 마찬가지로 다른 Source로부터 이벤트를 받아서 구독자에게 전달한다
- 가장 큰 차이는 Next event만 전달한다는 것이다
- Completed evnet와 Error event는 전달 받지도 않고, 전달 하지도 않는다
- 그래서 Subject와 달리 종료되지 않는다
- Observer가 Dispose 되기 전까지 계속 Event를 처리한다
- 주로 UI 이벤트 처리에 활용된다
- Relay는 RxSwift 프레임워크가 아닌 RxCocoa 프레임워크를 통해 제공된다
- Subject에서는 onNext를 사용하지만, Relay에서 Next event를 전달할 때는 accept 메소드를 사용한다
- accept 메소드를 호출하고 값을 전달하면 Observer에게 Next 이벤트가 전달된다
- Behavior Relay는 Behavior Subject와 마찬가지로 하나의 값을 생성자로 전달한다
- Behavior Relay는 value라는 속성을 제공한다
    - Behavior Relay가 저장하고 있는 Next event에 접근해서 여기에 저장되어 있는 값을 리턴한다 
    - 이 속성은 읽기 전용이고, 이 안에 있는 값을 바꿀 수는 없다
    - 값을 바꾸고 싶다면, accept 메소드를 통해 새로운 Next event를 전달해야 한다



***

### [4] Create Operators
#### 13/98 just, of, from
- 하나의 요소를 방출하는 Observable을 생성할 때는 just 연산자를 사용한다
- 두 개 이상의 요소를 방출하는 Observable을 생성할 때는 of 연산자를 사용한다
- just와 of 연산자는 항목을 그대로 방출하기 때문에, 배열을 전달하면 배열이 방출된다
- 배열에 저장된 요소를 하나씩 방출하는 Observable이 필요하다면 from 연산자를 사용한다

1. just
    - just는 하나의 항목을 방출하는 Observable을 생성한다
    - just는 ObservableType 프로토콜에 Type 메소드로 선언되어 있다
    - 파라미터로 하나의 요소를 받아서 Observable을 리턴한다
    - from 연산자와 자주 혼동하게 되는데, just로 생성한 Observable은 파라미터로 전달한 요소를 그대로 방출한다는 사실을 기억하라
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let element = "😀"

    Observable.just(element)
       .subscribe { print($0) }
       .disposed(by: disposeBag)
       
    --> 출력결과
    next(😀)
    completed
    <--

    Observable.just([1, 2, 3])
       .subscribe { print($0) }
       .disposed(by: disposeBag)
       
    --> 출력결과
    next([1, 2, 3])
    completed
    <--
       
    </code>
    </pre>

2. of
    - 만약 두 개 이상의 요소를 방출할 Observable을 만들어야 한다면 just로는 불가능하고, of 연산자를 사용해야 한다
    - of의 경우 가변 파라미터로 설정되어 있어서 여러 개의 값을 동시에 전달할 수 있다
    - of 역시 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
    - 방출할 요소를 원하는 수만큼 전달할 수 있다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let apple = "🍏"
    let orange = "🍊"
    let kiwi = "🥝"

    Observable.of(apple, orange, kiwi)
       .subscribe { element in print(element) }
       .disposed(by: disposeBag)
    
    --> 출력결과
    next(🍏)
    next(🍊)
    next(🥝)
    completed
    <--

    Observable.of([1, 2], [3, 4], [5, 6])
       .subscribe { element in print(element) }
       .disposed(by: disposeBag)
       
   --> 출력결과
   next([1, 2])
   next([3, 4])
   next([5, 6])
   completed
   <--
       
       
    </code>
    </pre>


3. from
    - 배열에 저장된 요소를 하나씩 방출하고 싶다면 from 연산자를 사용하면 된다
    - from 역시 ObserableType 프로토콜의 Type 메소드로 선언되어 있다
    - 첫 번째 파라미터로 배열을 받고, 리턴형은 배열이 아니라 배열에 포함된 요소이다
    - 배열에 포함된 요소를 하나씩 순서대로 방출한다
    - sequence 형식을 전달할 수도 있다
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let fruits = ["🍏", "🍎", "🍋", "🍓", "🍇"]

    Observable.from(fruits)
       .subscribe { element in print(element) }
       .disposed(by: disposeBag)
       
    --> 출력결과
    next(🍏)
    next(🍎)
    next(🍋)
    next(🍓)
    next(🍇)
    completed
    <--
    </code>
    </pre>


#### 14/98 range, generate
- 정수를 지정된 수만큼 방출하는 Observable을 생성하기 위해서 range 연산자와 generate 연산자를 사용한다

1. range
    - range 연산자는 시작 값에서 1씩 증가하는 squence를 생성한다
    - 증가되는 크기를 바꾸거나 감소하는 sequence를 생성하는 것은 불가능하다
    
    <pre>
    <code>
    let disposeBag = DisposeBag()

    Observable.range(start: 1, count: 5)  // start -> 시작할 정수를 입력( 실수x ), count -> 방출할 정수의 수를 전달
      .subsribe { print($0) }
      .disposed(by: disposeBag)
    --> 출력결과
    next(1)
    next(2)
    next(3)
    next(4)
    next(5)
    completed
    <-- 
    </code>
    </pre>
    
    
2. generate
    - generate 연산자의 경우 range 연산자와 다르게 파라미터 형식이 정수로 제한되지 않는다
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    
    Observable.generate(initialState: 0,      // initialState -> 시작값을 전달한다. 즉, 가장 먼저 방출되는 값이 들어간다
                        condition: { $0 <= 10},      // condition    -> true를 리턴하는 경우에만 요소가 방출된다, false를 리턴하면 Completed event를 전달하고 바로 종료한다
                        iterate: { $0 + 2 })              // iterate          -> 값을 바꾸는 코드를 전달한다. 보통 값을 증가시키거나 감소시키는 코드를 전달한다
        .subscribe { print($0)}
        .disposed(by: disposeBag)
        
    --> 출력결과
    next(0)
    next(2)
    next(4)
    next(6)
    next(8)
    next(10)
    completed
    <--
    </code>
    </pre>
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let red = "🔴"
    let blue = "🔵"

    Observable.generate(initialState: red,
                        condition: { $0.count < 6},
                        iterate: { $0.count.isMultiple(of: 2) ? $0 + red : $0 + blue })
        .subscribe { print($0) }
        .disposed(by: disposeBag)
        
    --> 출력결과
    next(🔴)
    next(🔴🔵)
    next(🔴🔵🔴)
    next(🔴🔵🔴🔵)
    next(🔴🔵🔴🔵🔴)
    completed
    <--
    </code>
    </pre>


#### 15/98 repeatElement 
- 동일한 요소를 반복적으로 방출하는 Observable을 생성
- repeatElement는 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
- 첫 번째 파라미터로 요소를 전달하면 이 요소를 반복적으로 방출하는 Observable을 리턴한다
- 반복적의 의미는 설명에 나와 있는 대로(infinitely) 무한적으로 방출하는 것을 의미한다
- repeatElement 연산자를 사용할 때는 방출되는 횟수를 제한해주는 것이 아주 중요하다
- take 연산자를 사용해서 방출 횟수를 지정할 수 있다

<pre>
<code>
let disposeBag = DisposeBag()
let element = "❤️"
Observable.repeatElement(element)
  .subscribe { print($0) }
--> 출력결과
next(❤️)  ... 무한반복
<--

Observable.repeatElement(element)
  .take(3)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
--> 출력결과
next("❤️")
next("❤️")
next("❤️")
completed
<--
</code>
</pre>


#### 16/98 deferred
- 특정 조건에 따라서 Observable을 생성할 수 있다
- Observable을 리턴하는 클로저를 파라미터로 받는다

<pre>
<code>
let disposeBag = DisposeBag()
let animals = ["🐶", "🐱", "🐹"]
let fruits = ["🍎", "🍐", "🍋"]
var flag = true


let factory: Observable<String> = Observable.deferred {
    flag.toggle()
    
    if flag {
        return Observable.from(animals)
    } else {
        return Observable.from(fruits)
    }
}

factory
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

factory
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

factory
    .subscribe{ print($0) }
    .disposed(by: disposeBag)
    
    
--> 출력결과
next(🍎)
next(🍐)
next(🍋)
completed
next(🐶)
next(🐱)
next(🐹)
completed
next(🍎)
next(🍐)
next(🍋)
completed
<--
    
</code>
</pre>


#### 17/98 create
- Observable이 동작하는 방식을 직접 구현
- Observable을 종료하기 위해서는 onError 또는 onCompleted 메소드를 반드시 호출해야 한다
- 둘 중 하나라도 호출하면 Observable이 종료되기 때문에, 그 이후에 onNext를 호출하면 요소가 방출되지 않는다
- onNext를 호출하려면 onCompleted() 메소드 또는 onError() 전에 호출해야 한다
<pre>
<code>
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

Observable<String>.create { (observer) -> Disposable in
    
    guard let url = URL(string: "https://www.apple.com") else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    
    guard let html = try? String(contentsOf: url, encoding: .utf8) else {
        observer.onError(MyError.error)
        return Disposables.create()
    }
    
    observer.onNext(html)
    observer.onCompleted()
    
    return Disposables.create()
}
.subscribe { print($0) }
.disposed(by: disposeBag)


--> 출력결과
https://www.apple.com - HTML ...
...
completed
<--
</code>
</pre>


#### 18/98 empty, error
- 두 연산자가 생성한 Observable은 Next event를 전달하지 않는다는 공통점이 있다
- 둘 다 어떠한 요소도 방출하지 않는다

1. empty
    - empty 연산자는 completed event를 전달하는 Observable을 생성한다 
    - 요소를 방출하지 않기 때문에 요소의 형식은 중요하지 않다
    - empty 연산자는 파라미터가 없다
    - observer가 아무런 동작 없이 종료되어야 할 때 사용한다
    
    <pre>
    <code>
    let disposeBag = DisposeBag()

    Observable<Void>.empty()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    completed
    <--
    </code>
    </pre>

2. error
    - error 연산자는 error 이벤트를 전달하고 종료한다
    
    <pre>
    <code>
    let disposeBag = DisposeBag()

    enum MyError: Error {
       case error
    }

    Observable<Void>.error(MyError.error)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    error(error)
    <--
    </code>
    </pre>


***

### [5] Filtering Operators
#### 19/98 ignoreElementsOperator 
- ignoreElements는 Observable이 방출하는 Next event를 필터링하고 Completed event와 Error event만 Observer에 전달한다
- ignoreElements는 파라미터를 받지 않는다
- 리턴형은 Completable인데, Completable은 트레이츠라고 부르는 특별한 Observable이다
- Completable은 completed 또는 Error event만 전달하고 Next event는 무시한다
- 주로 작업의 성공과 실패에만 관심이 있을 때 사용한다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["🍎", "🍓", "🍇"]

Observable.from(fruits)
  .ignoreElements()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
--> 출력결과
completed
<--
</code>
</pre>


#### 20/98 elementAt Operator
- elementAt은 특정 인덱스에 위치한 요소를 제한적으로 방출한다
- elementAt은 정수 인덱스를 파라미터로 받아서 Observable을 리턴한다
- 해당 인덱스의 요소를 방출하고 Completed event를 전달받는다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["🍎", "🍓", "🍇"]

Observable.from(fruits)
  .elementAt(1)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
--> 출력결과
next(🍓)
completed
<--
</code>
</pre>


#### 21/98 filter Operator
- filter 연산자는 클로저를 파라미터로 받는다
- true를 리턴하는 요소가 연산자가 리턴하는 Observable에 포함된다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .filter { $0.isMultiple(of: 2) } // 짝수
  .subscribe { print($0) }
  .dispose(by: disposeBag)
--> 출력결과
next(2)
next(4)
next(6)
next(8)
next(10)
completed
<--
</code>
</pre>


#### 22/98 skip, skipWhile, skipUntil Operator
- 특정 요소를 무시

1. skip
    - skip 연산자는 정수를 파라미터로 받는다
    - Observable이 방출하는 요소 중에서 지정된 수만큼 무시한 다음에 이후에 방출되는 요소만 Observer로 전달한다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5]

    Observable.from(numbers)
        .skip(3)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    next(4)
    next(5)
    completed
    <--
    </code>
    </pre>
    
2. skipWhile
    - skipWhile은 클로저를 파라미터로 받는다 
    - 이 클로저는 filter 연산자와 마찬가지로 predicate로 사용되고, 클로저에서 true를 리턴하는 동안 방출되는 요소를 무시한다
    - 클로저에서 false를 리턴하면 그때부터 요소를 방출하고, 이후에는 조건에 관계 없이 모든 요소를 방출한다
    - 연산자는 방출되는 요소를 포함한 Observable을 리턴한다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 3, 4, 8, 9, 10]


    Observable.from(numbers)
        .skipWhile { !$0.isMultiple(of: 2) } // 홀수
        .subscribe { print($0) }
        .disposed(by: disposeBag)
        
    --> 출력결과
    next(4)
    next(8)
    next(9)
    next(10)
    completed
    <--
    </code>
    </pre>
    
    
3. skipUntil
    - skipUntil 연산자는 Observable 타입을 파라미터로 받는다
    - 다른 Observable을 파라미터로 받고, 이 Observable이 Next event를 전달하기 전까지, 원본 Observable이 전달하는 Event를 무시한다
    - 이런 특징 때문에 파라미터로 전달되는 Observable을 trigger라고 부르기도 한다
    <pre>
    <code>
    let disposeBag = DisposeBag()

    let subject = PublishSubject<Int>()
    let trigger = PublishSubject<Int>()

    subject.skipUntil(trigger)
        .subscribe { print($0) }
        .dispose(by: disposeBag)
      
    subject.onNext(1)
    // 아직 trigger가 요소를 방출한적이 없기 때문에 subject가 방출한 요소는 Observer로 전달되지 않는다

    trigger.onNext(0)
    // 이번에는 trigger에서 요소를 방출하고 있다. 그런데 subject가 이전에 방출했던 요소는 여전히 Observer로 전달되지 않는다
    // skipUntil은 trigger가 요소를 방출한 이후부터 원본 Observable에서 방출되는 요소들을 Observer로 전달한다

    subject.onNext(2)

    --> 출력결과
    next(2)
    completed
    <--
    </code>
    </pre>


#### 23/98 take, takeWhile, takeUntil, takeLast Operator
1. take
    - 정수를 파라미터로 받아서 해당 숫자만큼만 요소를 방출한다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5]

    Observable.from(numbers)
        .take(3)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    next(1)
    next(2)
    next(3)
    completed
    <--
    </code>
    </pre>
    
2. takeWhile
    - 클로저를 파라미터로 받아서 predicate로 사용한다
    - true를 리턴하면 Observer에게 전달된다
    - 연산자가 리턴하는 Observable에는 최종적으로 조건을 만족하는 요소들만 포함된다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    Observable.from(numbers)
        .takeWhile { !$0.isMultiple(of: 2) }
        .subscribe { print($0) }
        .disposed(by: disposeBag)
        
    --> 출력결과
    next(1) // 이후에도 홀수가 방출되지만 구독자로는 전달되지 않는다 (takeWhile 연산자는 클로저가 false를 리턴하면 더 이상 요소를 방출하지 않는다)
    completed
    <--
    </code>
    </pre>

3. takeUntil
    - Observable을 파라미터로 받는다
    - 파라미터로 전달한 Observable에서 Next event를 전달하기 전까지 원본 Observable에서 Next event를 전달한다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    let subject = PublishSubject<Int>()
    let trigger = PublishSubject<Int>()


    subject.takeUntil(trigger)
        .subscribe{ print($0) }
        .disposed(by: disposeBag)

    subject.onNext(1) // 아직 trigger가 Next event를 전달하지 않았기 때문에 요소를 방출하다
    subject.onNext(2) // 아직 trigger가 Next event를 전달하지 않았기 때문에 요소를 방출하다

    trigger.onNext(0) // completed evnet 전달
    subject.onNext(3) // 정상적으로 실행되지만 completed event가 전달 되었기 때문에 더이상 요소를 방출하지 않는다.
    --> 출력결과
    next(1)
    next(2)
    completed
    <--
    </code>
    </pre>

4. takeLast
    - 정수를 파라미터로 받아서 Observable을 리턴한다
    - 리턴되는 Observable에는 원본 Observable이 방출한 요소 중에서 마지막으로 방출한 n개의 요소가 포함된다
    - 가장 중요한 점은 Observer로 전달되는 시점이 딜레이 된다는 점이다
    - Error event가 전달되면 buffer에 있는 요소는 절달되지않고 Error event만 전달된다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    let subject = PublishSubject<Int>()

    subject.takeLast(2)
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    numbers.forEach { subject.onNext($0) } // 아무것도 출력이 안되지만 코드는 정상실행. takeLast는 마지막에 방출한 9와 10을 buffer에 저장하고 있다

    subject.onNext(11) // 새로운 요소를 방출하면 buffer에 저장되어있는 값이 10과 11로 업데이트된다

    subject.onCompleted() // 이때 buffer에 저장된 요소가 Observer로 방출되고 Completed event가 전달된다
    
    --> 출력결과
    next(10)
    next(11)
    completed
    <--
    
    </code>
    </pre>
    
    
    #### 24/98 single Operator
    - single 연산자는 원본 Observable에서 첫 번째 요소만 방출하거나, 조건과 일치하는 첫 번째 요소만 방출한다
    - 두 개 이상의 요소가 방출되는 경우 Error가 발생한다
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    Observable.just(1)
        .single()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    next(1)
    completed
    <--
    </code>
    </pre>
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    Observable.from(numbers)
        .single()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    next(1)
    error(Sequence contains more than one element.)
    <--
    </code>
    </pre>
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    Observable.from(numbers)
        .single{ $0 == 3}
        .subscribe{ print($0) }
        .disposed(by: disposeBag)
    
    --> 출력결과
    next(3)
    completed
    <--
    </code>
    </pre>
    
    <pre>
    <code>
    let disposeBag = DisposeBag()
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    let subject = PublishSubject<Int>()

    subject.single()
        .subscribe{ print($0) }
        .disposed(by: disposeBag)

    subject.onNext(100)
    
    --> 출력결과
    next(100)  
    // 새로운 요소를 방출하면 구독자에게 바로 전달된다.
    // 다른 요소가 방출될 수도 있으므로 single 연산자가 리턴하는 Observable은 원본 Observable에서 completed event가 전달할 때까지 대기한다
    // completed event가 전달되는 시점에 하나의 요소만 방출된 시점이라면 Observer에게 completed event가 전달되고, 그 사이에 다른 요소가 방출되었다면 Observer에게는 error event가 전달된다. 이와 같은 방식을 통해 하나의 요소만 방출되는 것을 보장받을 수 있다
    <--
    </code>
    </pre>


#### 25/98 distinctUntilChange Operator
- distinctUntilChange 연산자는 동일한 항목이 연속적으로 방출되지 않도록 필터링 해준다
- 원본 Observable에서 전달되는 두 개의 요소를 순서대로 비교한 다음에 이전 요소와 동일하면 방출하지 않는다
- 두 개의 요소를 비교할 때는 비교 연산자로 비교한다
- 단순히 연속적으로 방출되는 동일한 요소만 확인한다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 1, 3, 2, 2, 3, 1, 5, 5, 7, 7, 7]


Observable.from(numbers)
    .distinctUntilChanged()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)
    
--> 출력결과
next(1)
next(3)
next(2)
next(3)
next(1)
next(5)
next(7)
completed
<--
</code>
</pre>


#### 26/98 debounce, throttle Operator
- 두 연산자는 짧은 시간동안 반복적으로 방출되는 이벤트를 제어한다는 공통점이 있다
- 연산자로 전달하는 파라미터도 동일하다
- 하지만 연산의 결과는 완전히 다르다
- throttle 연산자는 next event를 지정된 주기마다 하나씩 Observer에게 전달한다
- 반면 debounce 연산자는 next event가 전달된 다음 지정된 시간이 경과하기까지 다른 evnet가 전달되지 않는다면 마지막으로 방출된 event를 Observer에게 전달한다
- 짧은 시간 동안 반복되는 tap 이벤트나 delegate 이벤트를 처리할 때는 throttle을 사용하고, debounce는 주로 검색 기능을 구현할 때 사용한다
- debounce를 활용해 사용자가 짧은 시간동안 연속적으로 타이핑을 할 때는 검색작업을 실행하지 않다가, 타이핑을 멈추면 검색을 실행한다
- toArray 연산자는 별도의 파라미터를 받지는 않는다
- 하나의 요소를 방출하거나 error event를 방출하는 ObservableType의 메소드이다
- 하나의 요소를 방출하고 바로 종료한다

1. debounce
    - 첫 번째 파라미터(dueTime: RxTimeInterval)에는 시간을 전달한다
    - 이 시간은 연산자가 next event를 방출할지 결정하는 조건으로 사용된다
    - Observer가 next event를 방출한 다음 지정된 시간 동안 다른 next event를 방출하지 않는다면 해당 시점에 가장 마지막으로 방출된 next event를 Observer에게 전달한다
    - 반대로 지정된 시간 이내에 또 다른 next event를 방출했다면 타이머를 초기화한다
    - 타이머를 초기화한 다음에 다시 지정된 시간 동안 대기한다
    - 이 시간 이내에 다른 event가 방출되지 않는다면 마지막 event를 방출하고 event가 방출된다면 타이머를 다시 초기화한다
    - 두 번째 파라미터(scheduler: SchedulerType)에는 타이머를 실행할 scheduler를 전달한다
    
2. throttle
    - 세 개의 파라미터(dueTime: RxTimeInterval, latest: Bool, scheduler: SchedulerType)를 받는다
    - 기본값을 가진 두 번째 파라미터는 생략하는 경우가 많기 때문에 debounce와 파라미터가 동일하다고 생각해도 무방하다
    - 첫 번째 파라미터에는 반복 주기를 전달하고, 세 번째 파라미터에는 scheduler를 전달한다
    - 지정된 주기 동안 하나의 event만 Observer에게 전달한다
    - 보통 두 번째 파라미터는 기본값을 사용하는데, 주기를 엄격하게 지킨다
    - 항상 지정된 주기마다 하나씩 event를 전달한다
    - 두 번째 파라미터에 false를 부여하면, 반복 주기가 경과한 다음 가장 먼저 방출되는 event를 Observer에게 전달한다    


***

### [6] Tranforming Operators
#### 27/98 toArray Operator
- Observable이 방출하는 모든 요소를 배열에 담은 다음, 이 배열을 방출하는 Observable을 생성한다
<pre>
<code>
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()

subject
    .toArray()
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(1)
subject.onNext(2)

subject.onCompleted()
--> 출력결과
success([1, 2])
<--
</code>
</pre>


#### 28/98 map Operator
- Observable 배출하는 항목을 대상으로 함수를 실행하고 결과를 방출하는 Observable를 리턴한다
- 사용하다보면 파라미터와 동일한 형식을 리턴해야 한다고 생각하는 경우가 많지만 그런 제약은 없다
- Observable이 방출하는 요소들을 대상으로 클로저를 실행하고 그 결과를 Observer에게 전달한다
- 클로저로 전달되는 파라미터의 형식은 소스 Observable이 방출하는 요소와 동일하다
- 하지만 클로저가 리턴하는 값의 형식은 고정되어 있지 않으며, 원하는 형식으로 리턴할 수 있다

<pre>
<code>
let disposeBag = DisposeBag()
let skills = ["Siwft", "SwiftUI", "RxSwift"]

Observable.from(skills)
//  .map { "Hello, \($0)"}
  .map { $0.count }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
// next(Hello, Swift)
// next(Hello, SwiftUI)
// next(Hello, RxSwift)
next(5)
next(7)
next(7)
completed
</code>
</pre>


#### 29/98 flatMap Operator
- 모든 Observable이 방출하는 항목을 모아서 최종적으로 하나의 Observable을 리턴한다
- 클로저를 파라미터로 받는데, BehaviorSubject를 원하는대로 변환한 다음 새로운 Observable을 리턴해야 한다
- flatMap이 내부적으로 여러 개의 Observable을 생성하지만, 최종적으로 모든 Observable이 하나의 Observable로 합쳐지고, 방출되는 항목들이 순서대로 Observer에게 전달된다
- 원본 Observable이 방출하는 항목을 새로운 Observable로 변환한다. 새로운 Observable은 항목이 업데이트 될 때마다 새로운 항목을 방출한다
- 이렇게 생성된 모든 Observable은 최종적으로 하나의 Observable로 합쳐지고, 모든 항목들이 이 Observable을 통해서 Observer로 전달 된다
- 단순히 처음에 방출된 항목만 Observer로 전달되는 것이 아니라 업데이트된 최신 항목도 Observer로 전달된다
- 이 연산자는 네트워크 요청을 구현할 때 자주 활용한다

<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject<Int>(value: 1)
let b = BehaviorSubject<Int>(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
    .flatMap { $0.asObservable() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)

--> 출력결과
next(1)
next(2)
next(11)
next(22)
<--
</code>
</pre>


#### 30/98 flatMapFirst, flatMapLatest Operator
1. flatMapFirst
    - flatMapFirst의 연산자, 리턴형은 flatMap과 동일하다
    - 하지만 연산자가 리턴하는 Observable에는 처음에 변환된 Observable이 방출하는 항목만 포함된다
    <pre>
    <code>
    let disposeBag = DisposeBag()

    let a = BehaviorSubject(value: 1)
    let b = BehaviorSubject(value: 2)

    let subject = PublishSubject<BehaviorSubject<Int>>()

    subject
       .flatMapFirst { $0.asObservable() }
       .subscribe { print($0) }
       .disposed(by: disposeBag)

    subject.onNext(a)
    subject.onNext(b)

    a.onNext(11)
    b.onNext(22)
    b.onNext(222)
    a.onNext(111)
    
    --> 출력결과
    next(1)
    next(11)
    next(111)
    <--
    </code>
    </pre>
    
2. flatMapLatest
    - flatMapLatest는 원본 Observable이 방출하는 항목을 Observable로 변환하는 것은 동일하다
    - 반면 모든 Observable이 방출하는 항목을 하나로 병합하지 않는다
    - 대신 가장 최근의 항목을 방출한 Observable을 제외한 나머지는 모두 무시한다
    <pre>
    <code>
    let disposeBag = DisposeBag()

    let a = BehaviorSubject(value: 1)
    let b = BehaviorSubject(value: 2)

    let subject = PublishSubject<BehaviorSubject<Int>>()

    subject
       .flatMapLatest { $0.asObservable() }
       .subscribe { print($0) }
       .disposed(by: disposeBag)

    subject.onNext(a)

    a.onNext(11)

    subject.onNext(b)

    b.onNext(22)

    a.onNext(11) // Observer로 전달되지 않아 출력되지 않는다

    subject.onNext(a)

    b.onNext(222) // Observer로 전달되지 않아 출력되지 않는다
    a.onNext(111)
    
    --> 출력결과
    next(1)
    next(11)
    next(2)
    next(22)
    next(11)
    next(111)
    <--
    </code>
    </pre>


#### 31/98 scan Operator
- 기본값으로 연산을 시작하고, 원본 Observable이 방출하는 항목을 대상으로 변환을 실행한 다음 결과를 방출하는 하나의 Observable을 리턴한다
- 원본이 방출하는 항목의 수와 Observer로 전달되는 항목의 수가 동일하다
- 첫 번째 파라미터로 기본값을 전달하고, 두 번째 파라미터에는 클로저를 전달한다
- 이 연산자는 작업 결과를 누적 시키면서 중간 결과와 최종 결과가 모두 필요한 경우에 사용한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 5)
    .scan(0, accumulator: +)
    .subscribe{ print($0) }
    .disposed(by: disposeBag)

--> 출력결과
next(1) // 1+0
next(3) // 1+2
next(6) // 3+3
next(10) // 6+4
next(15) // 10+5
completed
<--
</code>
</pre>


#### 32/98 buffer Operator
- 특정 주기 동안 옵저버블이 방출하는 항목을 수집하고 하나의 배열로 리턴한다
- RxSwift에서는 이런 동작을 Controlled Buffering 이라고 한다
- 세 개의 파라미터(timeSpan: 항목을 수집할 시간(DispatchTimeInterval), count: 수집할 항목의 최대 숫자(Int), scheduler: SchedulerType)
- 연산자의 리턴형은 Type 파라미터가 배열로 선언되어 있다
- 지정된 시간동안 수집한 항목들을 배열에 담아서 리턴한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .buffer(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5) // 무한루프 방지
  .subscribe { print($0) }
  .disposed(by: disposerBag)
==> 출력 결과
next([0])
next([1, 2, 3])  
next([4, 5])
next([6, 7])
next([8, 9])
completed  
// Observable은 1초마다 항목을 방출하고 있고, buffer 연산자는 2초마다 세 개씩 수집하고 있다
// buffer 연산자는 첫 번째 파라미터로 전달한 timeSpan이 경과하면 수집된 항목들을 즉시 방출한다
// 두 번째 파라미터로 지정한 수만큼 수집되지 않았더라도 즉시 방출한다 (0만 출력되거나 1, 2, 3 세 개가 출력되는 경우)
</code>
</pre>


#### 33/98 window Operator
- window 연산자는 버퍼 연산자처럼 timeSpan과 maxCount를 지정해서 원본 Observable이 방출하는 항목들을 작은 단위의 Observable로 분해한다
- buffer와 달리 window 연산자는 수집된 항목을 방출하는 Observable을 리턴한다
- 리턴된 Observable이 무엇을 방출하고, 언제 완료되는지 이해하는 것이 중요하다
- 파라미터는 buffer와 똑같이 첫 번째로는 timeSpan, 두 번째로는 count, 세 번째로는 scheduler를 전달한다
- buffer 연산자와의 차이는 리턴형에 있다. buffer는 수집된 배열을 방출하는 Observable을 리턴, window 연산자는 Observable을 방출하는 Observable을 리턴
- Observable이 방출하는 Observable을 inner Observable이라고 한다
- inner Observable은 지정된 최대항목수만큼 방출하거나 지정된 시간이 경과하면 completed event를 전달하고 종료한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .window(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
    .take(5)
    .subscribe{
        print($0)
        
        if let observable = $0.element {
            observable.subscribe { print(" inner: \($0)")}
        }
    }
    .disposed(by: disposeBag)
--> 출력결과
next(RxSwift.AddRef<Swift.Int>)
 inner: next(0)
 inner: next(1)
 inner: next(2)
 inner: completed // 5초가 지나지 않았지만 max count (3) 만큼 항목을 배출했기때문에
next(RxSwift.AddRef<Swift.Int>)
 inner: next(3)
 inner: next(4)
 inner: next(5)
 inner: completed
next(RxSwift.AddRef<Swift.Int>)
 inner: next(6)
 inner: next(7)
 inner: next(8)
 inner: completed
next(RxSwift.AddRef<Swift.Int>)
 inner: next(9)
 inner: next(10)
 inner: next(11)
 inner: completed
next(RxSwift.AddRef<Swift.Int>)
completed
 inner: next(12)
 inner: next(13)
 inner: next(14)
 inner: completed
<-- 
</code>
</pre>


#### 34/98 groupBy Operator
- 연산자를 실행하면 클로저에서 동일한 값을 리턴하는 요소끼리 그룹으로 묶이고 그룹에 속한 요소들은 개별 Observable을 통해 방출된다
- 연산자가 리턴하는 Observable을 보면 TypeParameter가 grouped Observable로 선언되어 있다
- 방출하는 요소와 함께 key가 저장되어 있다

1. key와 GroupedObservable 출력
<pre>
<code>
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
    .groupBy { $0.count }
    .subscribe(onNext: { (groupedObservable) in
        print(" key = \(groupedObservable.key)")
        groupedObservable.subscribe{ print("   \($0)")}
    })
    .disposed(by: disposeBag)
--> 출력결과
key = 5
  next(Apple)
key = 6
  next(Banana)
  next(Orange)
key = 4
  next(Book)
  next(City)
key = 3
  next(Axe)
  completed
  completed
  completed
  completed
<--
</code>
</pre>

2. 문자열 길이로 그룹핑
<pre>
<code>
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
    .groupBy{ $0.count }
    .flatMap{ $0.toArray() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)
--> 출력결과
next(["Banana", "Orange"])
next(["Axe"])
next(["Apple"])
next(["Book", "City"])
completed
<--
</code>
</pre>

3. 첫번째 문자로 그룹핑
<pre>
<code>
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
    .groupBy{ $0.first ?? Character(" ")}
    .flatMap{ $0.toArray() }
    .subscribe{ print($0) }
    .disposed(by: disposeBag)
--> 출력결과
next(["Banana", "Book"])
next(["Orange"])
next(["City"])
next(["Apple", "Axe"])
completed
<--
</code>
</pre>

4. 홀수, 짝수로 그룹핑
<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
    .groupBy { $0.isMultiple(of: 2) }
    .flatMap{ $0.toArray() }
    .subscribe{ print($0)}
    .disposed(by: disposeBag)
--> 출력결과
next([1, 3, 5, 7, 9])
next([2, 4, 6, 8, 10])
completed
<--
</code>
</pre>


***

### [7] Combining Operators
#### 35/98 startWith Operator
- Observable이 요소를 방출하기 전에 다른 항목들을 앞 부분에 추가한다
- 주로 기본값이나 시작값을 지정할 때 활용한다
- 파라미터로 전달하는 하나 이상의 값을 Observable sequence 앞 부분에 추가한다. 그 다음 새로운 Observable을 리턴한다
- 가장 마지막에 호출한 연산자로 전달한 값이 가장먼저 방출된다

<pre>
<code>
let bag = DisposeBag()
let numbers = [1, 2, 3]

Observable.from(numbers)
    .startWith(0)
    .startWith(-3)
    .startWith(-1, -2)
    .subscribe{ print($0) }
    .disposed(by: bag)
    
--> 출력결과
next(-1)
next(-2)
next(0)
next(-3)
next(1)
next(2)
next(3)
completed
<--
</code>
</pre>


#### 36/98 concat Operator
- 두 개의 Observable을 연결
- 연결된 모든 Observable이 방출하는 요소들이 방출 순서대로 정렬되지는 않는다
- 이전 Observable이 모든 요소들을 방출하고 completed event를 전달해야 이어진 Observable이 방출을 시작한다

1. Type 메소드
    - 파라미터로 전달된 Collection에 있는 모든 Observable을 순서대로 연결한 하나의 Observable을 리턴한다

2. Instance 메소드
    - 대상 Observable이 completed event를 전달한 경우에 파라미터로 전달한 Observable을 연결한다
    - 만약 error event가 전달된다면 Observable은 연결되지 않는다 ( type 메소드로 구현된 concat연산자도 마찬가지)
    
<pre>
<code>
let bag = DisposeBag()
let fruits = Observable.from(["🍏", "🍎", "🥝"])
let animals = Observable.from(["🐶", "🐱", "🐹"])

// Type
Observable.concat([fruits, animals])
    .subscribe{ print($0) }
    .disposed(by: bag)

// Instance
animals.concat(fruits)
    .subscribe{ print($0) }
    .disposed(by: bag)
    
--> 출력결과
next(🍏)
next(🍎)
next(🥝)
next(🐶)
next(🐱)
next(🐹)
completed
next(🐶)
next(🐱)
next(🐹)
next(🍏)
next(🍎)
next(🥝)
completed
<--
</code>
</pre>
    

#### 37/98 merge Operator
- 여러 Observable이 방출하는 event를 하나의 Observable에서 방출하도록 병합하는 merge 연산자
- concat 연산자와 혼동하기 쉽지만 동작 방식이 다르다
- concat은 하나의 Observable이 모든 요소를 방출하고 completed event를 전달하면 이어지는 Observable을 연결
- merge는 두 개 이상의 Observable을 병합하고 모든 Observable에서 방출하는 요소들을 순서대로 방출하는 Observable을 리턴한다
- merge는 두 개 이상의 Observable이 방출하는 요소들을 병합한 하나의 Observable을 리턴한다
- 단순히 뒤에 연결하는 것이 아니라 하나의 Observable로 합쳐준다



#### 38/98 combineLatest Operator
- 소스 Observable이 방출하는 최신 요소를 병합하는 combineLatest 연산자
- combine은 결합한다는 의미이다
- 소스 Observable을 결합한 다음 파라미터로 전달한 함수를 실행하고 결과를 방출하는 새로운 Observable을 리턴한다
- 핵심은 연산자가 리턴한 Observable이 언제 event를 방출하는지 이해하는 것이다
- 두 개의 Observable과 클로저를 파라미터를 받는다
- Observable이 next event를 통해 전달하는 요소들은 클로저 파라미터를 통해 클로저에 전달된다
- 이후 클로저는 실행 결과를 리턴하고 연산자는 최종적으로 이 결과를 방출하는 Observable을 리턴한다
- 다양한 오버로딩이 있는데, 클로저를 전달하지 않는 경우에는 리턴 타입이 달라진다
- 파라미터로 전달한 Observable이 방출하는 요소들을 하나의 tuple로 합친 다음, 이 tuple을 방출하는 Observable을 리턴한다
- Observable을 최대 여덟개까지 전달할 수 있는 연산자들이 선언되어 있다
- 파라미터의 수만 다르고 동작 방식은 동일하다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()


Observable.combineLatest(greetings, languages) { lhs, rhs -> String in
    return "\(lhs) \(rhs)"
}
    .subscribe { print($0) }
    .disposed(by: bag)

greetings.onNext("Hi")
languages.onNext("World!")

greetings.onNext("Hello")
languages.onNext("RxSwift")

//greetings.onCompleted()
greetings.onError(MyError.error)
languages.onNext("SwiftUI")

languages.onCompleted() // 모든 Observable이 completed event를 전달하면 이 시점에 Observer에게 completed event가 전달된다

--> 출력결과
next(Hi World!)
next(Hello World!)
next(Hello RxSwift)
error(error) // error가 하나라도 전달되면 그 즉시 구독자에게 error event를 전달하고 종료한다.
// next(Hello SwiftUI)
// completed
<--
</code>
</pre>


#### 39/98 zip Operator
- Indexed Sequencing을 구현하는 zip 연산자
- 소스 Observable이 방출하는 요소를 결합한다
- Observable을 결합하고 클로저를 실행한 다음 이 결과를 방출하는 result Observable을 리턴한다
- zip 연산자는 클로저에게 중복되는 요소를 전달하지 않고, index가 일치하는 짝을 전달한다
- 소스 Observable이 방출하는 요소들을 순서를 일치시켜 결합하는 것을 Indexed Sequencing이라고 한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let numbers = PublishSubject<Int>()
let strings = PublishSubject<String>()

Observable.zip(numbers, strings) { "\($0) - \($1)" }
    .subscribe { print($0) }
    .disposed(by: bag)

numbers.onNext(1)
strings.onNext("one")

numbers.onNext(2)
strings.onNext("two")

//numbers.onCompleted()
numbers.onError(MyError.error) // 하나라도 error event를 전달하면 즉시 구독자에게 error event가 전달되고 종료된다

strings.onNext("three")
strings.onCompleted()

--> 출력결과
next(1 - one)
next(2 - two)
error(error)
//completed
<--
</code>
</pre>


#### 40/98 withLatestFrom Operator
- 연산자를 호출하는 Observable을 trigger Observable이라고 부르고 파라미터로 전달하는 Observable을 data Observable이라고 부른다
- trigger Observable이 next event를 방출하면 data Observable이 가장 최근에 방출한 Next event를 Observer에게 전달

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

trigger.withLatestFrom(data)
    .subscribe { print($0) }
    .disposed(by: bag)

data.onNext("Hello")
trigger.onNext(())
trigger.onNext(())

//data.onCompleted()
//data.onError(MyError.error)
//trigger.onNext(())

trigger.onCompleted() 

--> 출력결과
next(Hello)
next(Hello)
// error(error) // completed와 달리 error는 바로 전달된다
// next(Hello) // completed가 아닌 next가 전달 된다
completed // 바로 전달된다 ( error도 마찬가지 )
<--
</code>
</pre>


#### 41/98 sample Operator
- trigger Observable이 next event를 전달할 때마다 data Observable이 next event를 방출하지만, 동일한 next event를 반복해서 방출하지 않는 sample 연산자
- dataObservable.withLatestFrom(triggerObservable) 과 같은 형태로 사용한다 (withLatestFrom 연산자와 반대)
- data Observable에서 연산자를 호출하고 trigger Observable을 파라미터로 전달한다
- trigger Observable에서 next event를 전달할 때마다 data Observable이 최신 Event를 방출한다
- 동일한 next event를 반복해서 방출하지 않는 차이가 있다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

data.sample(trigger)
    .subscribe{ print($0) }
    .disposed(by: bag)

trigger.onNext(())
data.onNext("Hello")

trigger.onNext(())
trigger.onNext(()) // 동일한 연산자는 방출되지 않는다

//data.onCompleted()
//trigger.onNext(())

data.onError(MyError.error)

--> 출력결과
next(Hello)
//completed
error(error) //  trigger Observable이 next event를 방출하지 않더라도 즉시 Observer에게 전달된다

<--
</code>
</pre>



#### 42/98 switchLatest Operator
- 가장 최근에 방출된 Observable을 구독하고, 이 Observable이 전달하는 event를 Observer에게 전달하는 switchLatest 연산자
- 가장 최근 Observable이 방출하는 event를 Observer에게 전달한다
- 어떤 Observable이 가장 최근 Observable인지 이해하는 것이 핵심이다
- 주로 Observable을 방출하는 Observable에서 사용된다
- source Observable이 가장 최근에 방출한 Observable을 구독하고 여기에서 전달하는 next event를 방출하는 새로운 Observable을 리턴한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()
  
let source = PublishSubject<Observable<String>>()
  
source
  .switchLatest()
  .subscribe { print($0) }
  .disposed(by: bag)
  
a.onNext("1")
b.onNext("b")

source.onNext(a) //  a를 최신 Observable로 설정

a.onNext("2")
b.onNext("b")

source.onNext(b) // b를 최신 Observable로 설정

a.onNext("3")
b.onNext("c")

// a.onCompleted() // completed 이벤트가 전달되지 않는다
// b.onCompleted() // completed 이벤트가 전달되지 않는다

// source.onCompleted() // completed 이벤트가 전달된다

a.onError(MyError.error) // 최신 Observable이 아니기 때문에 error event가 전달되지 않는다
b.onError(MyError.error) // 최신 Observable인 b는 error event를 받으면 즉시 Observer에게 전달 가능하다

--> 출력결과
next(2)
next(c)
// completed
error(error)
<--
</code>
</pre>



#### 43/98 reduce Operator
- seed 값과 Observable이 방출하는 요소를 대상으로 클로저를 실행하고 최종 결과를 Observable로 방출하는 reduce 연산자
- scan 연산자와 비교하면 쉽게 이해할 수 있다
- reduce 연산자는 seed value와 accumulator 클로저를 파라미터로 받는다
- seed value와 소스 Observable이 방출하는 요소를 대상으로 클로저를 실행하고, result Observable을 통해 결과를 방출한다 (scan 연산자와 동일)
- accumulator 클로저의 실행 결과가 클로저로 다시 전달되는 것도 scan과 동일
- 하지만 reduce 연산자는 result Observable을 통해 최종 결과 하나만 방출한다. scan은 중간 과정까지 모두 방출한다
- 세 번째 파라미터 mapResult는 최종 결과를 다른 형식으로 바꾸고 싶을 때 사용한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let o = Observable.range(start: 1, count: 5)

print("== scan")

o.scan(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: bag)
  
print("== reduce")

o.reduce(0, accumulator: +)
  .subsribe { print($0) }
  .disposed(by: bag)
  
--> 출력결과
== scan 
next(1) // 1
next(3) // 1 + 2
next(6) // 3 + 3
next(10) // 6 + 4
next(15) // 10 + 5
completed
== reduce
next(15) // 최종결과 하나만 출력된다(scan 연산자와의 가장 큰 차이)
completed
<--
</code>
</pre>


---

### [8] Conditional Operators
#### 44/98 amb Operator
- 두 개 이상의 소스 Observable 중에서 가장 먼저 Next event를 전달하는 Observable을 구독하고 나머지는 무시한다
- 여러 Observable 중에서 가장 먼저 event를 방출하는 Observable을 선택하는 amb 연산자
- 여러 서버로 요청을 전달하고 가장 빠른 응답을 처리하는 패턴을 구현할 수 있다
- amb 연산자는 하나의 Observable을 파라미터로 받는다
- 두 Observable 중에서 먼저 event를 전달하는 Observable을 구독하고 이 Observable의 event를 Observer에게 전달하는 새로운 Observable을 리턴한다
- 세 개 이상의 Observable을 대상으로 연산자를 사용해야 한다면 Type 메소드로 구현된 연산자를 사용하면 된다
- 이때는 모든 소스 Observable을 배열 형태로 전달한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()
let c = PublishSubject<String>()

//a.amb(b)
Observable.amb([a,b,c]) // 여러 소스 Observable 받기
    .subscribe { print($0) }
    .disposed(by: bag)

a.onNext("A")
b.onNext("B")

b.onCompleted() // 전달 X
a.onCompleted()

--> 출력결과
next(A)
completed
<--
</code>
</pre>


---

### [9] Time-based Operators
#### 45/98 interval Operator
- 특정 주기마다 정수를 방출
- 첫 번째 파라미터로 반복 주기(RxTimeInterval -> Dispatch Time Interval과 같다)를 받고, 두 번째 파라미터로 정수를 방출할 scheduler를 지정
- 연산자가 리턴하는 Observable은 지정된 주기마다 정수를 반복적으로 방출한다
- 종료 시점을 지정하지 않기 때문에 직접 dispose 하기 전까지 계속해서 방출한다
- 방출하는 정수의 형식은 Int로 한정되지 않는다. 다양한 정수 형식 지원이 가능하다 ( FixedWidthInteger)
- double이나 문자열은 사용할 수 없다
- interval 연산자가 생성하는 Observable은 내부에 timer를 가지고 있다
- timer가 시작되는 시점은 생성 시점이 아니다. 구독자가 구독을 시작하는 시점이다
- Observable에서 새로운 구독자가 추가될 때마다 새로운 timer가 생성된다
- 새로운 구독이 추가되는 시점에 내부에 있는 timer가 시작된다는 점이 interval 연산자의 핵심이다

<pre>
<code>
let i = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

let subscription1 = i.subscribe { print("1 >> \($0)") }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    subscription1.dispose()
}


var subscription2: Disposable?

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    subscription2 = i.subscribe { print("2 >> \($0)") }
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
    subscription2?.dispose()
}

--> 출력결과
1 >> next(0)
1 >> next(1)
1 >> next(2)
2 >> next(0)
1 >> next(3)
2 >> next(1)
1 >> next(4)
2 >> next(2)
2 >> next(3)
<--
</code>
</pre>


#### 46/98 timer Operator
- interval과 마찬가지로 정수를 반복적으로 방출하는 Observable을 생성한다
- 하지만 지연 시간과 반복 주기를 설정할 수 있다
- interval과 마찬가지로 Type 메소드로 구현되어 있다
- 리턴되는 Observable이 방출하는 요소는 fixedWidthInteger로 제한되어 있다
- 첫 번째 파라미터는 첫 번째 요소가 방출되기까지의 상대적인 시간이다
- 두 번째 파라미터는 반복주기이다. 기본 값은 nil로 돼 있다. 값에 따라 타이머 연산자의 동작 방식이 달라진다
- 세 번째 파라미터는 타이머가 동작할 scheduler를 전달한다

<pre>
<code>
let bag = DisposeBag()


Observable<Int>.timer(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: bag)


// 5초 뒤에 정지
let i = Observable<Int>.timer(.seconds(1), period: .milliseconds(1000), scheduler: MainScheduler.instance)
let subscription = i.subscribe{ print($0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    subscription.dispose()
}
</code>
</pre>



#### 47/98 timeout Operator
- timeout 연산자는 소스 Observable이 방출하는 모든 요소에 timeout 정책을 적용한다
- 첫 번째 파라미터로 timeout 시간을 전달하는데 이 시간 안에 Next event를 전달하지 않으면 Error event를 전달하고 종료시킨다
- 에러 형식은 'RxError.timeout'이다
- 반대로 시간 내에 Next event를 전달하면 그대로 Observer에게 전달한다

<pre>
<code>
let bag = DisposeBag()

let subject = PublishSubject<Int>()

// 3초 이내에 새로운 event가 전달되지 않으면 Error event가 전달
subject.timeout(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe{ print($0) }
    .disposed(by: bag)

// Observable<Int>.timer(.seconds(1), period: .seconds(1), scheduler: MainScheduler.intance) // next(1 ... )
// Observable<Int>.timer(.seconds(5), period: .seconds(1), scheduler: MainScheduler.intance) // error
Observable<Int>.timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.instance) // error
    .subscribe(onNext: { subject.onNext($0)})
    .disposed(by: bag)
    
// Error 대신 0을 전달
// subject.timeout(.seconds(3), other: Observable.just(0), scheduler: MainScheduler.instance)
//    .subscribe{ print($0) }
//    .disposed(by: bag)
    
</code>
</pre>


#### 48/98 delay Operator
- Next event가 구독자로 전달되는 시점을 지정한 시간만큼 지연시킨다
- 첫 번째 파라미터에는 지연시킬 시간을 전달
- 두 번째 파라미터에는 delay timer를 실행할 scheduler를 전달
- Error event는 지연 없이 즉시 전달된다

<pre>
<code>
let bag = DisposeBag()

func currentTimeString() -> String {
   let f = DateFormatter()
   f.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
   return f.string(from: Date())
}

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(10)
    .debug()
    .delay(.seconds(5), scheduler: MainScheduler.instance)
    .subscribe{ print(currentTimeString(), $0) }
    .disposed(by: bag)
    
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(10)
    .debug()
    .delaySubscription(.seconds(7), scheduler: MainScheduler.instance) // 7초 동안 아무 Log도 출력되지 않는다
    .subscribe{ print(currentTimeString(), $0)}
    .disposed(by: bag)
        
</code>
</pre>


---

### [10] Sharing Subscription
#### 49/98 Sharing Subscription
- 구독 공유를 통해서 불필요한 중복 작업을 피하는 방법
<pre>
<code>
let bag = DisposeBag()

let source = Observable<String>.create { observer in
   let url = URL(string: "https://kxcoding-study.azurewebsites.net/api/string")!
   let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
      if let data = data, let html = String(data: data, encoding: .utf8) {
         observer.onNext(html)
      }
      
      observer.onCompleted()
   }
   task.resume()
   
   return Disposables.create {
      task.cancel()
   }
}
.debug()
.share() // share 연산자를 추가하면 모든 구독자가 구독을 공유하기 때문에 중복을 제거해준다

source.subscribe().disposed(by: bag)
source.subscribe().disposed(by: bag) // 실행 X
source.subscribe().disposed(by: bag) // 실행 X
</code>
</pre>


#### 50/98 multicast Operator
- multicast Operator와 Connectable Observable
- multicast 연산자는 subject를 파라미터로 받는다
- 원본 Observable이 방출하는 Event는 Observer에게 전달되는 것이 아니라 이 subject로 전달된다
- subject는 전달받은 Event를 등록된 다수의 Observer에게 전달한다
- 기본적으로 unicast 방식으로 동작하는 Observable을 multicast 방식으로 바꿔준다
- 이를 위해 ConnectableObservable을 리턴한다
- 일반 Observable은 Observer가 추가되면 새로운 sequence가 시작된다( event 방출 시작 )
- ConnectableObservable은 sequence가 시작되는 시점이 다르다
- Observer가 추가되어도 sequence는 시작되지 않고, connect 메소드를 호출하는 시점에 sequence가 시작된다
- 원본 Observable이 전달하는 Event는 Observer에게 바로 전달되는 것이 아니라 첫 번째 파라미터로 전달한 subject로 전달한다
- 전달받은 subject가 등록된 모든 Observer에게 Event를 전달한다
- 모든 Observer가 등록된 이후에 하나의 sequence가 시작되는 패턴을 구현할 수 있다
- ConnectableObservableAdapter는 원본 Observable과 subject를 연결해주는 특별한 클래스이다

<pre>
<code>
let bag = DisposeBag()
let subject = PublishSubject<Int>()

let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()

--> 출력결과
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
<--
</code>
</pre>


#### 51/98 publish Operator
- multicast 연산자를 호출하고 새로운 Publish Subject를 만들어서 파라미터로 전달한다
- 그 다음 multicast가 리턴하는 ConnectableObservable을 그대로 리턴한다
- multicast 연산자는 Observable을 공유하기 위해서 내부적으로 subject를 사용한다
- 파라미터로 Publish Subject를 전달한다면 직접 생성해서 전달하는 것보다 publish 연산자를 사용해서 활용하는 방법이 단순하고 좋다
- Publish Subject를 자동으로 생성해준다는 점을 제외하면 나머지는 multicast와 동일하다

<pre>
<code>
// multicast
let subject = PublishSubject<Int>()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject)

// publish
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).publish()

</code>
</pre>


#### 52/98 replay Operator
- multicast 연산자로 Publish Subject를 전달한다면 Publish 연산자를 사용하고, Replay Subject를 전달하면 replay 연산자를 사용한다
- 두 연산자 모두 multicast를 조금 더 쉽게 사용하도록 도와주는 유틸리티 연산자이다
- 보통은 파라미터를 통해 buffer의 크기를 지정하지만, buffer 크기에 제한이 없는 replayAll 연산자도 있다
- 하지만 경우에 따라 메모리 사용량이 급격하게 증가하는 경우가 있어 가급적 사용하지 않는다
- replay 연산자를 사용할 때 buffer 크기를 지정하는 데 유의해야 한다. 필요 이상으로 크게 잡을 경우 메모리 문제가 발생할 가능성이 높기 때문이다

<pre>
<code>
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).replay(5)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()

--> 출력결과
🔵 next(0)
🔵 next(1)
🔴 next(0)
🔴 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
<--
</code>
</pre>



#### 53/98 refCount Operator
- refCount 연산자는 다른 연산자와 달리 ConnectableObservableType
- 다시 말해서 일반 Observable에서는 사용할 수 없고, ConnectableObservable에서만 사용할 수 있다
- 파라미터는 없고, Observable을 리턴한다
- refCount는 ConnectableObservable을 통해 생성하는 특별한 Observable이다
- 앞으로 이 Observable을 refCountObservable이라 부르겠다
- refCountObservable은 내부에 ConnectableObservable을 유지하면서 새로운 Observer가 추가되는 시점에 자동으로 커넥트 메소드를 호출한다
- 그리고 Observer가 구독을 중지하고 더 이상 다른 Observer가 없다면 ConnectableObservable에서 sequence를 중지한다
- 그러다가 새로운 구독자가 추가되면 다시 커넥트 메소드를 호출한다
- 이 때 ConnectableObservable에서는 새로운 sequence가 시작된다

<pre>
<code>
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().publish().refCount()

let observer1 = source
   .subscribe { print("🔵", $0) }

//source.connect() // refCount는 내부적으로 connect를 호출하기 때문에 별도로 connect 연산자를 호출할 필요가 없다

DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 뒤에 구독 중지
   observer1.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // 7초 뒤에 구독 시작
   let observer2 = source.subscribe { print("🔴", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 뒤에 구독 중지
      observer2.dispose()
   }
}
</code>
</pre>



#### 54/98 share Operator
- share 연산자가 리턴하는 Observable은 refCount Observable이다
- share 연산자는 두 개의 파라미터를 받는다
    1. 첫 번째 파라미터(replay: Int = 0)는 replay buffer의 크기이다
        - 파라미터로 0을 전달하면 multicast를 호출할 때 Publish Subject를 전달한다
        - 0보다 큰 값을 전달한다면 replay Subject를 전달한다
        - 기본값이 0으로 선언되어 있기 때문에 다른 값을 전달하지 않는다면 새로운 Observer는 구독 이후에 방출되는 event만 전달 받는다
        - multicast 연산자를 호출하니까 하나의 subject를 통해 sequence를 공유한다
    
    2. 두 번째 파라미터(scope: SubjectLifetimeScope = .whileConnected)는 이 subject의 수명을 결정한다
        - 기본값은 whileConnected로 선언되어 있다
        - 새로운 구독자가 추가되면(새로운 connection이 시작되면) 새로운 subject가 생성된다
        - connection이 종료되면 subject는 사라진다
        - connection마다 새로운 subject가 생성되기 때문에 connection은 다른 connection과 격리된다
        - 반대로 두 번째 파라미터에 forever를 전달하면 모든 connection이 하나의 subject를 공유한다

<pre>
<code>
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().share(replay: 5, scope: .forever) // forever를 전달하면 모든 connection이 하나의 subject를 공유한다

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
   observer1.dispose()
   observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // 7초 뒤에 새로운 sequence가 시작된다
   let observer3 = source.subscribe { print("⚫️", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 그리고 3초 뒤에 구독 중지
      observer3.dispose()
   }
}

</code>
</pre>




---

### [11] Scheduler
#### 55/98 Scheduler
- RxSwift에서는 GCD 대신 Scheduler 사용
- Scheduler는 특정 코드가 실행되는 context를 추상화한 것이다
- context는 로우레벨 스레드가 될 수도 있고, DispatchQueue나 OperationQueue가 될 수도 있다
- Scheduler는 추상화된 context이기 때문에 Thread와 1:1로 매칭되지 않는다
- 하나의 Thread에 두 개 이상의 개별 scheduler가 존재하거나, 하나의 scheduler가 두 개의 Thread에 걸쳐 있는 경우도 있다
- 사용 예
    1. UI 업데이트
        - GCD       : Main Queue
        - RxSwift  : Main Scheduler
        
    2. Network 요청이나 파일 처리 작업
        - GDC       : Global Queue
        - RxSwift  : Background Scheduler
        

- RxSwift는 GCD와 마찬가지로 다양한 기본 scheduler를 제공한다
- 내부적으로 GCD와 유사한 방식으로 동작하고, 실행할 작업을 스케줄링 한다
- 스케줄링 방식에 따라 Serial Scheduler와 Concurrent Scheduler로 구분한다
- CurrentThreadScheduler가 가장 기본적인 scheduler이다
- Main Thread와 연관된 scheduler는 MainScheduler이다. Main Queue처럼 UI를 업데이트할 때 사용한다
- 작업을 실행할 DispatchQueue를 직접 지정하고 싶다면 SerialDispatchQueueScheduler나 ConcurrentDispatchQueueScheduler를 활용한다
- 앞에서 사용한 Main Scheduler는 Serial DispatchQueue의 일종이다
- Background 작업을 실행할 때는 DispatchQueue Scheduler를 사용한다
- 실행 순서를 제외하거나 동시에 실행 가능한 작업 수를 제한하고 싶다면 OperationQueueScheduler를 사용한다. 이 scheduler는 DispatchQueue가 아닌 OperationQueue를 사용해서 생성한다
- Unit Test에서 사용하는 TestScheduler가 있다
- scheduler를 직접 구현할 수도 있다( = Custom Scheduler)


<pre>
<code>
let bag = DisposeBag()

let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .subscribeOn(MainScheduler.instance) // Observable이 시작되는 Scheduler를 지정
    .filter { num -> Bool in
      print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
      return num.isMultiple(of: 2)
    }
    .observeOn(backgroundScheduler) // 이어지는 연산자가 실행되는 Scheduler를 지정, 다른 Scheduler로 변경하기 전까지 계속 사용
    .map { num -> Int in
      print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
      return num * 2
    }
    .observeOn(MainScheduler.instance)
    .subscribe {
        print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
        print($0)
    }
    .disposed(by: bag)
</code>
</pre>



---

### [12] Error Handling
#### 56/98 Error Handling
- Observable에서 전달한 Error event가 Observer에게 전달되면 구독이 종료되고 더 이상 새로운 Event가 전달되지 않는다
- 더 이상 새로운 Event를 처리할 수 없게 된다
- 두 가지 방법으로 문제를 해결
    1. catchError -  Error event가 전달되면 새로운 Observable을 리턴
        - Observable이 전달하는 Next 이벤트와 completed 이벤트는 그대로 구독자에게 전달
        - 반면 Error event가 전달되면 새로운 Observable을 Observer에게 전달
        
    2. retry - Error가 발생한 경우 Observable을 다시 구독
        - Error가 발생하지 않을 때까지 무한정 재시도 하거나, 재시도 횟수를 제한할 수 있다
