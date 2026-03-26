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


### BlockingQueue
#### BlockingQueue
* BoundedQueueV5의 로직이 이미 구현되어 있다고 생각하면 편하다.
* 인터페이스
  * ArrayBlockingQueue: 배열 기반, 버퍼 크기 고정
  * LinkedBlockingQueue: 링크 기반, 버퍼 크기 고정 or 비고정
  * BlockingQueue의 구현체들은 BoundedQueueV5의 로직이 구현하여 제공한다.
* 데이터 추가 메서드
  * add(), offer(), put(), offer(타임아웃)
* 데이터 획득 메서드
  * take(), poll(타임아웃), remove(..)
* Queue 를 상속 받아 큐의 기능을 사용할 수 있다.

#### 기능 설명
* Throws Exception - 대기시 예외
  * add(e): 지정된 요소를 큐에 추가하며, 큐가 가득 차면 IllegalStateException 예외를 던진다.
  * remove(): 큐에서 요소를 제거하며 반환한다. 큐가 비어 있으면 NoSuchElementException 예외를 던진다.
  * element(): 큐의 머리 요소를 반환하지만, 요소를 큐에서 제거하지 않는다. 큐가 비어 있으면 NoSuchElementException 예외를 던진다.
* Special Value - 대기시 즉시 반환 
  * offer(e): 지정된 요소를 큐에 추가하려고 시도하며, 큐가 가득 차면 false 를 반환한다. 
  * poll(): 큐에서 요소를 제거하고 반환한다. 큐가 비어 있으면 null 을 반환한다. 
  * peek(): 큐의 머리 요소를 반환하지만, 요소를 큐에서 제거하지 않는다. 큐가 비어 있으면 null 을 반환한다.
* Blocks - 대기 
  * put(e): 지정된 요소를 큐에 추가할 때까지 대기한다. 큐가 가득 차면 공간이 생길 때까지 대기한다. 
  * take(): 큐에서 요소를 제거하고 반환한다. 큐가 비어 있으면 요소가 준비될 때까지 대기한다. 
  * Examine (관찰): 해당 사항 없음.
* Times Out - 시간 대기 
  * offer(e, time, unit): 지정된 요소를 큐에 추가하려고 시도하며, 지정된 시간 동안 큐가 비워지기를 기다리다가 
  * 시간이 초과되면 false 를 반환한다. 
  * poll(time, unit): 큐에서 요소를 제거하고 반환한다. 큐에 요소가 없다면 지정된 시간 동안 요소가 준비되기를 기다리다가 시간이 초과되면 null 을 반환한다. 
  * Examine (관찰): 해당 사항 없음.