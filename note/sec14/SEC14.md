## Section13

### graceful shutdown
#### 소개
* 새로운 요청을 막고 기존에 진행하던 작업을 마무리한 후 종료하는 방식을 graceful shutdown 이라고 한다.

#### ExecutorService의 종료 메서드
* 서비스 종료
  * ```void shutdown()```
    * 새로운 작업을 받지 않고, 이미 제출된 작업을 모두 완료한 후에 종료한다. 
    * 논 블로킹 메서드(이 메서드를 호출한 스레드는 대기하지 않고 즉시 다음 코드를 호출한다.)
  * ```List<Runnable> shutdownNow()```
    * 실행 중인 작업을 중단하고, 대기 중인 작업을 반환하며 즉시 종료한다.
    * 실행 중인 작업을 중단하기 위해 인터럽트를 발생시킨다. 
    * 논 블로킹 메서드
* 서비스 상태 확인
  * ```boolean isShutdown()```
    * 서비스가 종료되었는지 확인한다.
  * ```boolean isTerminated()```
    * shutdown() , shutdownNow() 호출 후, 모든 작업이 완료되었는지 확인한다
* 작업 완료 대기
  * ```boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException```
    * 서비스 종료시 모든 작업이 완료될 때까지 대기한다. 이때 지정된 시간까지만 대기한다.
    * 블로킹 메서드
  * ```close()```
    * 자바 19부터 지원
    * shutdown() + 작업이 완료되거나 인터럽트가 발생할 때 까지 무한정 반복 대기
    * 호출한 스레드에 인터럽트가 발생해도 shutdownNow()를 호출한다
      * close()를 호출한 스레드(= 대기 중인 스레드)가 외부에서 인터럽트를 받으면, awaitTermination()이 InterruptedException을 던지는데, 이때 그냥 대기를 포기하고 나가는 게 아니라 추가로 shutdownNow()까지 호출해서 작업 중인 스레드들에도 인터럽트를 전파한다라는 뜻

#### 구현
* close()
  * shutdown() 을 호출하고, 하루를 기다려도 작업이 완료되지 않으면 shutdownNow() 를 호출
  * 하지만 하루는 너무 길다
* shutdown() 을 통해 우아한 종료를 시도하고, 기다리는 시간동안 종료되지 않으면 shutdownNow() 통해 강제 종료하는 방식을 구현해야 한다.,
  * 대기 시간은 보통 30 ~ 60초 정도 설정한다.
  * 구현 방식의 경우  ExecutorService 공식 API 문서에서 제안하는 방식이 있다.

```java
static void shutdownAndAwaitTermination(ExecutorService es) {
  es.shutdown(); // non-blocking, 새로운 작업을 받지 않는다. 처리 중이거나, 큐에 이미 대기중인 작업은 처리한다. 이후에 풀의 스레드를 종료한다.
  try {
    // 이미 대기중인 작업들을 모두 완료할 때 까지 10초 기다린다.
    log("서비스 정상 종료 시도");
    if (!es.awaitTermination(10, TimeUnit.SECONDS)) {
      // 정상 종료가 너무 오래 걸리면...
      log("서비스 정상 종료 실패 -> 강제 종료 시도");
      es.shutdownNow();
      // 작업이 취소될 때 까지 대기한다.
      if (!es.awaitTermination(10, TimeUnit.SECONDS)) {
        log("서비스가 종료되지 않았습니다.");
      }
    }
  } catch (InterruptedException ex) {
    // awaitTermination()으로 대기중인 현재 스레드가 인터럽트 될 수 있다.
    es.shutdownNow();
  }
}
```
* 서비스 종료 (shutdown) → 대기 (awaitTermination) → 서비스 정상종료 실패 → 강제 종료 시도 (shutdownNow) → 대기 (awaitTermination)
  * hutdownNow() 가 작업 중인 스레드에 인터럽트를 호출하는 것은 맞으나 인터럽트를 호출하더라도 여러가지 이유로 작업에 시간이 걸릴 수 있기에 잠시 대기하는 것이다.
  * 최악의 경우 인터럽트를 받을 수 없는 코드를 수행 중이라면 인터럽트 예외가 발생하지 않고, 스레드도 계속 수행될 수 있다.
* 서비스 종료시 기본적으로 우아한 종료를 선택하고, 우아한 종료가 되지 않으면 무한정 기다릴 수는 없으니, 그 다음으로 강제 종료를 하는 방식으로 접근하는 것이 좋다.

### Executor 스레드 풀 관리
#### 속성
* corePoolSize
  * 스레드 풀에서 관리되는 기본 스레드의 수
* maximumPoolSize
  * 스레드 풀에서 관리되는 최대 스레드 수
  * 블로킹 큐가 꽉 찼을 때(급할 때), corePoolSize를 초과해서 maximumPoolSize까지 스레드를 추가(초과 스레드)로 만들어서 작업을 처리할 수 있다.
* keepAliveTime , TimeUnit unit
  * 기본 스레드 수를 초과해서 만들어진 초과 스레드가 생존할 수 있는 대기 시간, 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.
* BlockingQueue workQueue
  * 작업을 보관할 블로킹 큐

#### 스레드 미리 생성하기
* 기본적으로 요청이 들어와야 스레드가 만들어진다.
* 하지만 응답시간이 중요한 경우 스레드를 미리 만들어 놓는 것이 좋다.
* ThreadPoolExecutor.prestartAllCoreThreads() 를 사용하면 기본 스레드를 미리 생성할 수 있다.

#### Executor 전략 - 고정 풀 전략
* newSingleThreadPool
  ```java
  new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
  ```
* 스레드 풀에 기본 스레드 1개만 사용한다. 
* 큐 사이즈에 제한이 없다. ( LinkedBlockingQueue )
* 주로 간단히 사용하거나, 테스트 용도로 사용한다.

#### Executor 전략 - 고정 풀 전략
* newFixedThreadPool(nThreads)
  ```java
  new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
  ```
* 스레드 풀에 nThreads 만큼의 기본 스레드를 생성한다. 초과 스레드는 생성하지 않는다.
* 큐 사이즈에 제한이 없다. ( LinkedBlockingQueue )
* 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식이다.
* 그러나 사용자가 점진적으로 증가되거나, 갑작스럽게 요청이 증가하는 환경에서는 알맞은 방식은 아니다.

#### Executor 전략 - 캐시 풀 전략
* newCachedThreadPool()
 ```java
 new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
 ```
* 기본 스레드를 사용하지 않고, 60초 생존 주기를 가진 초과 스레드만 사용한다.
* 초과 스레드의 수에 제한이 없으며 큐에 작업을 저장하지 않는다.
* 작업 수에 맞추어 스레드 수가 변하기 때문에, 작업의 처리 속도가 빠르고, CPU, 메모리를 매우 유연하게 사용할 수 있다.
* 그러나 스레드 수가 너무 늘어나 시스템이 느려지거나 다운 될 수 있는 위험이 존재한다.