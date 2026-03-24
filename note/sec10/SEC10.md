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
