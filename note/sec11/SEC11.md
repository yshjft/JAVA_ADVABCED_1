## Section11

### 원자적 연산
#### 원자적 연산
* 원자적 연산은 멀티스레드 상황에서 아무런 문제가 발생하지 않는다. 
* 하지만 원자적 연산이 아닌 경우에는 synchronized 블럭이나 Lock 등을 사용해서 안전한 임계 영역을 만들어야 한다.

#### AtomicInteger
* 자바에서 제공하는 멀티스레드 상황에서 안전하게 증가 연산을 수행할 수 있는 클래스

#### 성능 비교
* BasicInteger
  * 가장 빠르다. 
  * CPU 캐시를 적극 사용한다. CPU 캐시의 위력을 알 수 있다. 
  * 안전한 임계 영역도 없고, volatile 도 사용하지 않기 때문에 멀티스레드 상황에는 사용할 수 없다. 
  * 단일 스레드가 사용하는 경우에 효율적이다.
* VolatileInteger 
  * volatile 을 사용해서 CPU 캐시를 사용하지 않고 메인 메모리를 사용한다. 
  * 안전한 임계 영역이 없기 때문에 멀티스레드 상황에는 사용할 수 없다. 
  * 단일 스레드가 사용하기에는 BasicInteger 보다 느리다. 그리고 멀티스레드 상황에도 안전하지 않다.
* SyncInteger
  * synchronized 를 사용한 안전한 임계 영역이 있기 때문에 멀티스레드 상황에도 안전하게 사용할 수 있다.
  * MyAtomicInteger 보다 성능이 느리다.
* MyAtomicInteger 
  * 자바가 제공하는 AtomicInteger 를 사용한다. 멀티스레드 상황에 안전하게 사용할 수 있다. 
  * 성능도 synchronized , Lock(ReentrantLock) 을 사용하는 경우보다 1.5 ~ 2배 정도 빠르다
  *  AtomicInteger 가 제공하는 incrementAndGet() 메서드는 락을 사용하지 않고, 원자적 연산을 만들어낸다. (그래서 빠른거다)