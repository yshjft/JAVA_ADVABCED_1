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