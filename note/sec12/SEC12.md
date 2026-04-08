## Section11

### 동시성 컬랙션
#### 동시성 컬랙션이 필요한 이유
* 우리가 일반적으로 아는 컬랙션을 원자적인 연산이 아니기에 멀티 스레드 상황에서 문제가 발생한다.
* 따라서 동시성 컬랙션을 사용해야 한다.

#### synchronized + 프록시 패턴 (동기화 프록시)
* synchronized를 사용하면 임계영역을 동기화 할 수 있다. 그리고 이를 통해 동기화를 지원하는 컬랙션을 구현할 수 있다.
* 하지만 모든 종류의 컬랙션에 맞는 synchronized 컬랙션을 만드는 것은 힘들다.
* 따라서 프록시 패턴을 사용한다. synchronized를 프록시 클래스에 적용하고 클라이언트에서 프록시 클래스를 사용하게 만든다.
  ```
  클라이언트 → (프록시) → Collection
  ```
#### Collections.synchronizedList()
* 자바에서 제공하는 동기화 프록시다.
* 아래와 같이 사용할 수 있다.
  ```java
  Collections.synchronizedList(new ArrayList());
  ```
* 해당 코드는 결과적으로 다음과 같은 코드다.
  ```java
  new SynchronizedRandomAccessList(new ArrayList());
  ``` 

#### 동기화 프록시 방식의 단점
결론은 최적화를 적용하지 못해 성능이 떨어진다는 것이다.

* 동기화 오버헤드 발생한다.
* 잠금 법위가 넓다.
* 정교한 동기화가 불가능하다.


#### 동시성 컬랙션
정교한 동기화를 구현하면서 동시에 성능도 최적화 되어 있다.

* List
  * CopyOnWriteArrayList ArrayList 의 대안
* Set 
  * CopyOnWriteArraySet HashSet 의 대안 
  * ConcurrentSkipListSet TreeSet 의 대안(정렬된 순서 유지, Comparator 사용 가능)
* Map 
  * ConcurrentHashMap : HashMap 의 대안 
  * ConcurrentSkipListMap : TreeMap 의 대안(정렬된 순서 유지, Comparator 사용 가능)
* Queue 
  * ConcurrentLinkedQueue : 동시성 큐, 비 차단(non-blocking) 큐이다. 
* Deque 
  * ConcurrentLinkedDeque : 동시성 데크, 비 차단(non-blocking) 큐이다

※ LinkedHashSet , LinkedHashMap 처럼 입력 순서를 유지하는 동시에 멀티스레드 환경에서 사용할 수 있는 Set , Map 구현체는 제공하지 않는다. 필요하다면 Collections.synchronizedXxx() 를 사용해야 한다.

스레드를 아예 차단하는 블로킹 큐도 있다.

* ArrayBlockingQueue 
  * 크기가 고정된 블로킹 큐 
  * 공정(fair) 모드를 사용할 수 있다. 공정(fair) 모드를 사용하면 성능이 저하될 수 있다.
* LinkedBlockingQueue 
  * 크기가 무한하거나 고정된 블로킹 큐 
* PriorityBlockingQueue 
  * 우선순위가 높은 요소를 먼저 처리하는 블로킹 큐 
* SynchronousQueue
  * 직거래 큐 
  * 쉽게 이야기해서 중간에 큐 없이 생산자, 소비자가 직접 거래한다. 
* DelayQueue 
  * 지연된 요소를 처리하는 블로킹 큐로, 각 요소는 지정된 지연 시간이 지난 후에야 소비될 수 있다. 일정 시간이 지난 후 작업을 처리해야 하는 스케줄링 작업에 사용된다.