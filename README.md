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