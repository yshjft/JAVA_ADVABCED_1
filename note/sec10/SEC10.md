## Section10

### Lock Condition

#### 스레드 대기 공간 분리
* BoundedQueueV5.java
* synchronized는 일단 스레드 대기 공간을 분리할 수 없다.
* 그래서 ReentrantLock을 사용해야 한다.
  ```java
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
  ```
  * lock 이라는 객체 안에 대기 공간이 만들어진다.
  * condition.await()
    * 대기
  * condition.signal()
    * 깨우기
    * 보통 FIFO로 깨워짐 (자료구조가 queue라서)
* 생산자와 소비자 스레드 대기 공간을 분리한다.
  ```java
    Lock lock = new ReentrantLock();
    Condition producerCond = lock.newCondition();
    Condition consumerCond = lock.newCondition();
  ```
  * put() - 생산자 스레드가 호출
    * 큐가 가득찬 경우: producerCond.await() 호출
    * 데이터를 큐에 저장한 경우: consumerCond.signal()를 호출하여 소비자 깨움
  * take() - 소비자 스레드가 호출
    * 큐에 데이터가 없는 경우: consumerCond.await() 호출
    * 데이터를 큐에서 가져온 경우: producerCond.signal()를 호출하여 생산자 깨움
  * 이를 통해 비효율 문제를 해결 가능

#### 스레드의 대기
* 스레드가 락을 얻기 위해 Blocked 상태가 되어 있을 때에도 관리되는 곳이 있다.
* synchronized 대기
  * 락 대기 집합
* ReentrantLock 대기
  * 대기 큐 (ReentrantLock에서 제공하는 락 대기집합)

#### BlockingQueue
* 큐가 특정 조건이 만족될 때까지 스레드의 작업을 차단(blocking)한다.
* 데이터 추가 차단
  * 큐가 가득 차면 데이터 추가 작업( put() )을 시도하는 스레드는 공간이 생길 때까지 차단된다. 
* 데이터 획득 차단
  * 큐가 비어 있으면 획득 작업( take() )을 시도하는 스레드는 큐에 데이터가 들어올 때까지 차단된다.