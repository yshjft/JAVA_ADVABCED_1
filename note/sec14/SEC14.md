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