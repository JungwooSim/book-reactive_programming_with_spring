# Part01

## 01. Reactive Programming

### Reactive System 이란?

클라이언트 요청에 즉각적으로 응답함으로써 지연 시간을 최소화하는 시스템

### The Reactive Manifesto (리액티브 선언문) 에서 지향하는 바는 무엇인가?

<img src="/img/1-1.png" width="600px;">

빠른 응답성을 바탕으로 유지보수와 확장이 용이한 시스템

Reactive Manifesto : [https://www.reactivemanifesto.org](https://www.reactivemanifesto.org/)

Reactive Manifesto 용어 : https://www.reactivemanifesto.org/ko/glossary

### Reactive Programming 이란?

Reactive System 을 구축하는데 필요한 program model

**설계 원칙**

- 비동기 메시지 기반 통신으로 동작해야 함
- 탄력적이고 회복성을 지녀야 함
- 높은 응답성을 지녀야 함
- 유지보수와 확장이 용이해야 함

### Reactive Programming 의 특징

- declarative programming
    - c, java 는 명령형 프로그래밍이다.
      즉, 실행할 동작을 구체적으로 명시하는 프로그래밍 코드 형태
    - 선언형 프로그래밍 방식은 실행할 동작을 구체적으로 명시하지 않고 이러이러한 동작을 하겠다고 목표만 선언한다
- data streams 와 the propagation of change
    - data streams : 데이터가 지속적으로 발생한다는 의미
    - the propagation of change : 지속적으로 데이터가 발생할 때마다 이것을 변화하는 이벤트로 보고, 이 이벤트를 발생시키면서 데이터를 계속적으로 전달하는 것을 의미

### Reactive Programming 구성

- Publisher : 입력으로 들어오는 데이터를 Subscriber 에 제공하는 역할
- Subscriber : Publisher 로부터 전달받은 데이터를 사용하는 역할
- Data Source : Publisher 의 입력으로 전달되는 데이터를 의미
- Operator : Publisher 와 Subscriber 중간에서 데이터를 가공하는 역할
## 02. Reactive Streams

### Reactive Streams 란?

데이터 스트림을 Non-Blocking 이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양

### Reactive Streams 구성요소

- Publisher : 데이터를 생성하고 통지(발행, 게시, 방출)하는 역할
- Subscriber : 구독한 Publisher 로부터 통지(발행, 게시, 방출)된 데이터를 전달받아서 처리하는 역할
- Subscription : Publisher 에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할
- Processor : Publisher 와 Subscriber 의 기능을 모두 가지고 있다. 즉, Subscriber 로서 다른 Publisher 를 구독할 수 있고, 반대로 Publisher 로서 다른 Subscriber 가 구독할 수 있다

### Reactive Streams 관련 용어 정의

- Signal : Publisher 와 Subscriber 간에 주고받는 상호작용
    - Ex. onSubscribe, onNext, onComplete, onError, request, cancel
- Demand : Subscriber 가 Publisher 에게 요청하는 데이터. 구체적으로는 Publisher 가 아직 Subscriber 에게 전달하지 않은 Subscriber 가 요청한 데이터
- Emit : Publisher 가 Subscriber 에게 데이터를 전달하는 것을 말한다.
    - Ex. 데이터를 emit 한다
- Upstream/Downstream
- Sequence : Flux 를 통해 데이터를 생성, emit 하고 filter 메서드를 통해 필터링 후, map 메서드를 통해 변환하는 이러한 과정 자체를 Sequence 라 한다
- Operator : just, filter, map 같은 메서들을 연산자라 하는데, 이러한 것을 말함
- Source : 최초의 데이터
    - Ex. Data Source, Source Publisher, Source Flux

### Reactive Streams 의 규칙

**Publisher 구현을 위한 주요 기본 규칙**

- Publisher 가 Subscriber 에게 보내는 onNext signal 의 총 개수는 항상 해당 Subscriber 의 구독을 통해 요청된 데이터의 총 개수보다 더 작거나 같아야 함
- Publisher 는 요청된 것보다 적은 수의 onNext signal 을 보내고 onComplete 또는 onError 를 호출하여 구독을 종료할 수 있다
- Publisher 의 데이터 처리가 실패하면 onError signal 을 보내야 한다
- Publisher 의 데이터 처리가 성공적으로 종료되면 onComplate signal 을 보내야 한다
- Publisher 가 Subscribe 에게 onError 또는 onComplate signal 을 보내는 경우 해당 Subscriber 의 구독은 취소된 것으로 간주되어야 한다
- 구독이 취소되면 Subscriber 는 결국 signal 을 받는 것을 중지해야 한다

**Subscriber 구현을 위한 주요 기본 규칙**

- Subscriber 는 Publisher 로부터 onNext signal 을 수신하기 위해 Subscription.request(n) 를 통해 Demand signal 을 Publisher 에게 보내야 한다
- Subscriber.onComplate() 및 Subscriber.onError(Throwable t) 는 Subscription 또는 Publisher 의 메서드를 호출해서는 안된다
    - Race Condition 을 방지하기 위해
- Subscriber.onComplate() 및 Subscriber.onError(Throwable t) 는 signal 을 수신한 후 구독이 취소된 것으로 간주해야 한다
- 구독이 더 이상 필요하지 않은 경우 Subscriber 는 Subscription.cancel() 을 호출해야 한다
- Subscriber.onSubscribe() 는 지정된 Subscriber 에 대해 최대 한번만 호출되어야 한다

**Subscription 구현을 위한 주요 기본 규칙**

- 구독은 Subscriber 가 onNext 또는 onSubscribe 내에 동기적으로 Subscription.request 를 호출하도록 허용해야 한다
- 구독이 취소된 후 추가적으로 호출되는 Subscription.request(long n) 는 효력이 없어야 한다
- 구독이 취소된 후 추가적으로 호출되는 Subscription.cancel() 은 효력이 없어야 한다
- 구독이 취소되지 않은 동안 Subscription.request(long n)의 매게변수가 0 보다 작거나 같으면 java.lang.lllegalArgumentException 과 함께 onError signal 을 보내야 한다
- 구독이 취소되지 않은 동안 Subscription.cancel() 은 Publisher 가 Subscriber 에게 보내는 signal 을 결국 중지하도록 요청해야 한다
- Subscription.cancel(), Subscription.request() 호출에 대한 응답으로 예외를 던지는 것을 허용하지 않는다
- 구독은 무제한 수의 request 호출을 지원해야 하고 최대 2^63 - 1 개의 Demand 를 지원해야 한다

### Reactive Streams 구현체

RxJava, Project Reactor, Akka Streams, Java Flow API, Reactive Extension

## 03. Blocking I/O 와 Non-Blocking I/O

I/O 는 일반적으로 컴퓨터 시스템이 외부의 입출력 장치들과 데이터를 주고 받는 것을 의미한다.

- Ex.
  - DB I/O : 데이터베이스에서 데이터를 조회하거나 추가하는 작업
  - Network I/O : web application 에서 web application 으로 네트워크 통신을 하는 경우

### Blocking I/O 의 특징

- 작업 스레드의 작업이 종료될 때까지 요청 스레드가 차단된다
- 스레드가 차단되는 문제를 보완하기 위해 멀티스레딩 기법을 사용할 수 있다
- 멀티스레딩 기법 사용시 컨텍스트 스위치 전환 비용, 메모리 사용 오버헤드, 스레드 풀의 응답 지연 등의 문제가 발생할 수 있다

### Non-Blocking I/O 의 특징

- 작업 스레드의 종료 여부와 상관없이 요청 스레드가 차단되지 않는다
- 적은 수의 스레드만 사용해 스레드 전환 비용이 적으므로, CPU 를 효율적으로 사용할 수 있다
- CPU 를 많이 사용하는 작업의 경우 성능에 악영향을 미칠 수 있다
- 사용자 요청 처리에서 응답까지 전 과정이 Non-Blocking 이어야 제대로 된 효과를 얻을 수 있다

### Non-Blocking I/O 방식의 통신이 적합한 시스템

- Blocking I/O 방식으로 처리하는데 한계가 있는 대량의 요청 트래픽이 발생하는 시스템
- micro service 기반 시스템
- streaming 또는 실시간 시스템

# Part02 Project Reactor

## 05. Reactor 개요 (Project Reactor)

Spring Framework 팀의 주도하에 개발된 리액티브 스트림즈의 구현체

Spring WebFlux 기반의 리액티브 애플리크에션을 제작하기 위한 핵심 역할을 담당

공식사이트 : [https://projectreactor.io](https://projectreactor.io/)

**특징**

- Reactive Streams : Reactive Streams 사양을 구현한 리액티브 라이브러리
- Non-Blocking : JVM 위에서 실행되는 Non-Blocking 어플리케이션을 제작하기 위해 필요한 핵심 기술
- Java’s Functional API : Publisher 와 Subscriber 간의 상호작용은 Java 의 함수형 프로그래밍 API 를 통해 이루어진다
- Flux[N] : N 개의 데이터를 emit 한다는 의미, 즉 Flux 는 0 ~ N 개, 즉 무한대의 데이터를 emit 할 수 있는 Reactor 의 Publisher
- Mono[0|1] : 데이터를 한 건도 emit 하지 않거나 단 한건만 emit 하는 단발성 데이터 emit 에 특화된 Publisher
- Well-suited for microservices : 마이크로 서비스 기반 시스템에서 수많은 서비스들 간에 지속적으로 발생하는 I/O 를 처리하기에 적합한 기술
- Backpressure-ready network : Publisher 로부터 전달받은 데이터를 처리하는 데 있어 과부하가 걸리지 않도록 제어하는 Backpressure 를 지원하는데, 다양한 전략을 제공해준다

## 07. Cold Sequence 와 Hot Sequence

- Subscriber 의 구독 시점이 달라도 구독을 할 때마다 Publisher 가 데이터를 처음부터 emit 하는 과정을 Cold Sequence 라고 한다
- Cold Sequence 흐름으로 동작하는 Publisher 를 Cold Publisher 라고 한다
- Publisher 가 데이터를 emit 하는 과정이 한 번만 일어나고, Subscriber 가 각각의 구독 시점 이후에 emit 된 데이터만 전달받는 것을 Hot Sequence 라고 한다
- Hot Sequence 흐름으로 동작하는 Publisher 를 Hot Publisher 라고 한다
- share(), cache() 등의 Operator 를 사용하여 Cold Sequence 를 Hot Sequence 로 변환할 수 있다
- Hot Sequence 는 Subscriber 의 최초 구독이 발생해야 Publisher 가 데이터를 emit 하는 Warm up 과 Subscriber 의 구독 여부와 상관없이 데이터를 emit 하는 Hot 으로 구분할 수 있다

### 08. Backpressure

- Publisher 가 끊임없이 emit 하는 무수히 많은 데이터를 적절하게 제어하여 데이터 처리에 있어 과부하가 걸리지 않도록 제어하는 데이터 처리 방식
- Reactor 에서 몇가지 전략을 지원한다
  - IGNORE : Backpressure 를 적용하지 않는 전략
  - ERROR : Downstream 의 데이터 처리 속도가 느려서 Upstream 의 emit 속도를 따라가지 못할 경우 에러를 발생 시키는 전략
  - DROP : Publisher 가 Downstream 으로 전달할 데이터가 버퍼에 가득 찰 경우, 버퍼 밖에서 대기 중인 데이터 중에서 먼저 emit 된 데이터부터 DROP 하는 전략
  - LATEST : Publisher 가 Downstream 으로 전달할 데이터가 버퍼에 가득 찰 경우, 버퍼 밖에서 대기 중인 데이터 중에서 가장 최근에 emit 된 데이터부터 버퍼에 채우는 전략
  - BUFFER : 버퍼의 데이터를 폐기하지 않고 버퍼링을 하는 전략이다. 만약 버퍼에 가득 찰 경우 아래 두가지 전략으로 삭제할 수 있다.
    - DROP_LATEST : Publisher 가 Downstream 으로 전달할 데이터가 버퍼에 가득 찰 경우, 가장 최근에 버퍼 안에 채워진 데이터를 Drop 하는 전략
    - DROP_OLDEST : Publisher 가  Downstream 으로 전달할 데이터가 버퍼에 가득 찰 경우, 버퍼 안에 채워진 데이터 중에서 가장 오래된 데이터를 Drop 하는 전략

### 09. Sinks

- Publisher 와 Subscriber 의 기능을 모두 지닌 Processor 의 향상된 기능을 제공
- 데이터를 emit 하는 Sinks 에는 크게 Sinks.One 와 Sinks.Many 가 있다
- [Sinks.One](http://Sinks.One) 는 한 건의 데이터를 프로그래밍 방식으로 emit 한다
- Sinks.Many 는 여러 건의 데이터를 프로그래밍 방식으로 emit 한다
- Sinks.Many 의 UnicastSpec 는 단 하나의 Subscriber 에게만 데이터를 emit 한다
- Sinks.Many 의 MulticastSpec 는 하나 이상의 Subscriber 에게 데이터를 emit 한다
- Sinks.Many 의 MulticastReplaySpec 은 emit 된 데이터 중에서 특정 시점으로 되돌린(replay) 데이터부터 emit 한다

-

### 10. Scheduler

- Reactor 에서의 Scheduler 는 비동기 프로그래밍을 위해 사용되는 스레드를 관리해 주는 역할
- subscribeOn() Operator 는 구독이 발생한 직후에, 실행될 스레드를 지정하는 Operator
- publishOn() Operator 는 Downstream 으로 Signal 을 전송할 때 실행되는 스레드를 제어하는 역할을 하는 Operator 이다
- parallel() Operator 는 Round Robin 방식으로 CPU 코어 개수만큼의 스레드를 병렬로 실행한다

**10.4 publishOn() 과 subscribeOn() 의 동작 이해**

- publishOn() Operator 는 한 개 이상의 사용할 수 있으며, 실행 스레드를 목적에 맞게 적절하게 분리할 수 있다
- subscribe() Operator 와 publishOn() Operator 를 함께 사용해서 원본 Publisher 에서 데이터를 emit 하는 스레드와 emit 된 데이터를 가공 처리하는 스레드를 적절하게 분리할 수 있다
- subscribeOn() 은 Operator 체인상에서 어떤 위치에 있든 구독 시점 직후, 즉 Publisher 가 데이터를 emit 하기 전에 실행 스레드를 변경한다

**10.5 Scheduler 의 종류**

- Schedulers.immediate() 는 별도의 스레드를 추가적으로 생성하지 않고, 현재 스레드에서 작업을 처리
- Schedulers.single() 는 스레드 하나만 생성해서 Scheduler 가 제거되기 전까지 재사용
- Schedulers.boundedElastic() 은 ExecutorService 기반의 Thread Pool 을 생성한 후, 그 안에서 정해진 수 만큼의 스레드를 사용하여 작업을 처리하고 작업이 종료된 스레드는 반납하여 재사용
- Schedulers.boundedElastic() 은 Blocking I/O 작업에 최적화되어 있다
- Schedulers.parallel() 은 Non-Blocking I/O 에 최적화되어 있는 Scheduler 로서 CPU 코어 수 만큼 스레드를 생성
- Schedulers.newSingle(), Schedulers.newBoundedElastic(), Schedulers.newParallel() 메서드를 사용해서 새로운 Scheduler 인스턴스를 생성할 수 있다.

### 11. Context

- 일반적으로 어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보를 의미
- 특징
  - Operator 체인에 전파되는 key / value 형태의 저장소
  - Subscriber 와 매핑되어 구독이 발생할 때마다 해당 구독과 연결된 하나의 Context 가 생긴다
  - contextWriter() Operator 를 사용해서 Context 에 데이터 쓰기 작업을 할 수 있따
  - deferContextual() Operator 를 사용해서 원본 데이터 소스 레벨에서 Context 의 데이터를 읽을 수 있다
  - transformDeferredContextal() Operator 를 사용해서 Operator 체인의 중간에 데이터를 읽을 수 있다
- 주의사항
  - Context 는 Operator 체인의 아래에서 위로 전파
  - 동일한 key 에 중복해서 저장하면 Operator 체인상에서 가장 위쪽에 위치한 contextWrite() 이 저장한 값으로 덮어쓴다
  - Inner Sequence 내부에서는 외부 Context 에 저장된 데이터를 읽을 수 있다
  - Inner Sequence 외부에서는 Inner Sequence 내부 Context 에 저장된 데이터를 읽을 수 없다
  - Context 는 인증 정보 같은 직교성(독립성)을 가지는 정보를 전송하는데 적합

### 12. Debugging

- Hooks.onOperatorDebug() 메서드를 호출해서 디버그 모드를 활성화 할 수 있다
- 디버그 모드 활성화 시 application 내에 있는 모든 Operator 의 Stacktrace 를 캡처하므로 프로덕트 환경에서는 사용하면 안된다
- Reactor Tools 에서 지원하는 ReactorDebugAgent 를 사용하여 프로덕트 환경에서 디버그 모드를 대체 할 수 있다
- checkpoint() Operator 를 사용하면 특정 Operator 체인 내의  Stacktrace 만 캡처 한다
- log() Operator 를 추가하면 추가한 지점의 Reactor Signal 을 출력한다
- log() Operator 는 사용 개수에 제한이 없으므로 1개 이상의 log() Operator 로 Reactor Sequence 의 내부 동작을 좀 더 상세하게 분석하며 디버깅 할 수 있다

### 14. Operators

**14.2 Sequence 생성을 위한 Operator**

- just() : Hot Publisher 이므로 Subscriber 의 구독 여부와 상관없이 데이터를 emit 하며, 구독이 발생하면 emit 된 데이터를 다시 replay 하여 Subscriber 에게 전달
- defer() : 구독이 발생하기 전까지 데이터의 emit 을 지연시키기 때문에 just() Operators 를 defer() 로 감싸게 되면 실제 구독이 발생해야 데이터를 emit 한다
- using() : 파라미터로 전달받은 resource 를 emit 하는 Flux 를 생성
- generator() : 프로그래밍 방식으로 Signal 이벤트를 발생시키며, 동기적으로 데이터를 하나씩 순차적으로 emit 한다
- create() : generator() Operator 와 마찬가지로 프로그래밍 방식으로 Signal 이벤트를 발생시키지만 generate() Operator 와 달리 한 번에 여러 건의 데이터를 비동기적으로 emit 할 수 있다

**14.3 Sequence 필터링 Operator**

- filter
- skip
- take
- next

**14.4 Sequence 변환 Operator**

- map
- flatMap
- concat
- merge
- zip
- and
- collectList
- collectMap

**14.5 Sequence 의 내부 동작 확인을 위한 Operator**

- doOnSubscribe() : Publisher 가 구독 중일 때, 트리거되는 동작을 추가
- doOnRequest() : Publisher 가 요청을 수신할 때, 트리거되는 동작을 추가
- doOnNext() : Publisher 가 데이터를 emit 할 때, 트리거되는 동작을 추가
- doOnComplete() : Publisher 가 성공적으로 완료되었을 때, 트리거되는 동작을 추가
- doOnError() : Publisher 가 에러가 발생한 상태로 종료되었을 때, 트리거되는 동작을 추가
- doOnCancel() : Publisher 가 취소되었을 때, 트리거되는 동작을 추가
- doOnTerminate() : Publisher 가 성공적으로 완료되었을 때, 또는 에러가 발생한 상태로 종료되었을 때, 트리거 되는 동작을 추가
- doOnEach() : Publisher 가 데이터를 emit 할 때, 성공적으로 완료되었을 때, 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가
- doOnDiscard() : Upstream  에 있는 전체 Operator 체인의 동작 중에서 Operator 에 의해 폐기되는 요소를 조건부로 정리할 수 있다
- doAfterTerminate() : Downstream 을 성공적으로 완료한 직후 또는 에러가 발생한 Publisher 가 종료된 직후에 트리거되는 동작을 추가
- doFirst() : Publisher 가 구독되기 전에 트리거되는 동작을 추가
- doFinally() : 에러를 포함하여 어떤 이유이든 간에 Publisher 가 종료된 후 트리거되는 동작을 추가

**14.6 에러 처리를 위한 Operator**

- error
- onErrorReturn
- onErrorResume
- onErrorContinue
- retry

**14.7 Sequence 의 동작 시간 측정을 위한 Operator**

- elapsed

**14.8 Flux Sequence 분할을 위한 Operator**

- window
- buffer
- bufferTimeout
- groupBy

**14.9 다수의 Subscriber 에게 Flux 를 Multicasting 하기 위한 Operator**

- publish
- autoConnect
- refCount