## Section08

### LockSupport
* 무한 대기(synchronized의 단점)를 해결하기 위해 사용함

#### 대표적인 기능
* park() 
  * 스레드를 WAITING 상태로 변경한다 
  * 스레드를 대기 상태로 둔다. 참고로 park 의 뜻이 "주차하다", "두다"라는 뜻이다.
* parkNanos(nanos)
  * 스레드를 나노초 동안만 TIMED_WAITING 상태로 변경한다. 
  * 지정한 나노초가 지나면 TIMED_WAITING 상태에서 빠져나오고 RUNNABLE 상태로 변경된다.
* unpark(thread)
  * WAITING 상태의 대상 스레드를 RUNNABLE 상태로 변경한다.
  * 대기 상태의 스레드는 자신의 코드를 실행할 수 없기에 외부 스레드를 통해 빠져나와야 한다.
  * 이는 해당 메서드에 스레드 지정 매개변수가 있는 이유이기도 하다.

#### 인터럽트 사용
* 인터럽트를 통해 waiting 상태의 thread를 runnable 상태로 변경할 수 있다.
  * 당연히 인터럽트 상태도 true이다.

#### Blocked vs WAITING
* 인터럽트
  * BLOCKED 상태는 인터럽트가 걸려도 대기 상태를 빠져나오지 못한다. (물론 인터럽트 상태는 true)
  * WAITING 상태는 인터럽트가 걸리면 RUNNABLE로 변경
* 용도
  * BLOCKED는 synchronized 에서 락을 획득하기 위해 대기할 때 사용
  * WAITING은 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태
    *  WAITING
       * Thread.join(), LockSupport.park(), Object.wait()
    *  TIMED_WAITING
       * Thread.join(long millis), LockSupport.parkNanos(ns), Object.wait(long timeout)

#### 문제
* LockSupport는 너무 저수준이다.
* synchronized을 대체하려면 개발자가 구현해야 하는게 너무 많다.
* 그래서 ReentrantLock이 나왔다.

### ReentrantLock
#### 무한 대기 해결
* Lock 인터페이스
    ```java
    package java.util.concurrent.locks;
    public interface Lock {
         void lock();
         void lockInterruptibly() throws InterruptedException;
         boolean tryLock();
         boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
         void unlock();
         Condition newCondition();
    }
    ```
  * ```void lock()```
    * 락을 획득한다. 만약 다른 스레드가 이미 락을 획득했다면, 락이 풀릴 때까지 현재 스레드는 대기한다. 
    * 인터럽트에 응답하지 않는다. (정확히는 아주 잠시 WAITING 에서 RUNNABLE이 된 후 다시 WAITING이 되는 것이다.)
    * 여기서 사용하는 락은 객체 내부에 있는 모니터 락이 아니다!
  * ```void lockInterruptibly()```
    * 락 획득을 시도하되, 다른 스레드가 인터럽트할 수 있도록 한다. 만약 다른 스레드가 이미 락을 획득했다면, 현재 스레드는 락을 획득할 때까지 대기한다. 
    * 대기 중에 인터럽트가 발생하면 InterruptedException 이 발생하 며 락 획득을 포기한다.
  * ```boolean tryLock()```
    * 락 획득을 시도하고, 즉시 성공 여부를 반환한다. 
    * 다른 스레드가 이미 락을 획득했다면 false 를 반환하고, 그렇지 않으면 락을 획득하고 true 를 반환한다.
  * ```boolean tryLock(long time, TimeUnit unit)```
    * 주어진 시간 동안 락 획득을 시도한다. 
    * 주어진 시간 안에 락을 획득하면 true 를 반환한다. 주어진 시간이 지나도 락을 획득하지 못한 경우 false 를 반환한다. 
    * 대기 중 인터럽트가 발생하면 InterruptedException 이 발생하며 락 획득을 포기한다.
  * ```void unlock()```
    * 락을 해제한다.
    * 락을 획득한 스레드가 호출해야 하며, 그렇지 않으면 IllegalMonitorStateException 이 발생한다.
  * ```Condition newCondition()```
    * Condition 객체를 생성하여 반환한다. Condition 객체는 락과 결합되어 사용되며, 스레드가 특정 조건을 기다리거나 신호를 받을 수 있도록 한다.

#### 공정성 해결 
* 비공정 모드
  * ```new ReentrantLock()```
  * 성능 우선
  * 선점 가능
  * 기아 현상 가능성
* 공정 모드
  * ```new ReentrantLock(true)```
  * 공정성 보장
  * 기아 현상 방지
  * 성능 저하

#### 활용 - lock()
* 대기큐
  * 만약 락 획득을 실패하는 경우 스레드는 WAITING 상태로 대기큐에서 기다리게 된다.
  * 이 때 내부적으로 LockSupport.park()가 호출된다.
* 락 반납이 이루어진 후
  * 대기 큐의 스레드를 하나 깨운다. (RUNNABLE 상태로 만든다)
  * 대기 큐에서 깨워진 스레드가 만약 락을 획득하지 못하면 다시 waiting 상태가되어 다시 대기 큐에 보관된다.
    * 락을 획득하지 못하는 경우 → 락 획득을 시도하는 잠깐 사이에 새로운 스레드가 먼저 락을 가져가는 경우 발생 (비공정 모드)
    * 공정 모드인 경우 대기 큐에서 먼저 대기한 스레드가 먼저 락을 가져간다.
* Lock을 활용하면 메모리 가시성 문제도 해결된다.

#### 활용 - tryLock()
* ```boolean tryLock()```
  * 락 획득을 시도하고, 즉시 성공 여부를 반환한다.
* ```boolean tryLock(long time, TimeUnit unit)```
  * 주어진 시간 동안 락 획득을 시도한다. 
  * 락 획득을 실패하는 경우 LockSupport.park()가 호출되어 WAITING 상태로 만든다.