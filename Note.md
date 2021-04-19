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
