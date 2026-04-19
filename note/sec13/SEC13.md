## Section13

## Executor 프레임워크 (1)


### 스레드 직접 사용할 때의 문제점
#### 스레드 생성 비용으로 인한 문제
* 필요할 때마 스레드를 만들어 사용하는 경우 스레드 생성하기 위한 비용을 매번 담당행야 한다.

#### 스레드 관리 문제
* 스레드를 문한하게 사용할 경우 시스템 전체를 다운 시킬 수 있다.

#### Runnable 인터페이스 불편함
* 반환 값이 없으며 예외 처리(ex. checked exception)의 불편함이 크다.

#### 해결
* Executor 를 사용하라
  * 스레드 풀을 이용하여 1,2 번 문제 해결
  * Runnable 불편함 문제 해결
  * 생산자 소비자 문제 해결

### ThreadPoolExecutor
#### ThreadPoolExecutor
* 생산자/소비자 패턴
  * 생산자: es.execute(작업) 를 호출하는 스레드
  * 소비자: 스레드 풀에 있는 스레드
* 생성자
  ```java
  ExecutorService es = new ThreadPoolExecutor(2,2,0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
  ```
  * corePoolSize : 스레드 풀에서 관리되는 기본 스레드의 수
  * maximumPoolSize : 스레드 풀에서 관리되는 최대 스레드 수
  * keepAliveTime, TimeUnit unit : 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간(이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거됨)
  * BlockingQueue workQueue : 작업을 보관할 블로킹 큐
  * 참고) 요청이 들어와야 스레드를 만든다.

#### Runnable의 불편함
* 반환 값이 없다.
* 예외 처리가 힘들다.(checked exception을 던질 수 없다.)


### Future
#### Callable
```java
package java.util.concurrent;

public interface Callable<V> {
 V call() throws Exception;
}
```
* 반환 값이 존재하며 checked exception을 throw 할 수 있다.

#### Callable 사용
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
   ExecutorService es = Executors.newFixedThreadPool(1);
   Future<Integer> future = es.submit(new MyCallable());
   Integer result = future.get();
   es.close();
 }
 
 static class MyCallable implements Callable<Integer> {
   @Override
   public Integer call() {
     // ...
   }
 }
```
* es.submit()을 통해 스레드에 callable 작업을 전달
* futrure.get()을 통해 callable이 반환한 결과 확인
* 멀티스레드를 매우 편하게 사용 가능

#### Future 분석
* Future: 전달한 작업의 미래 결과를 담고 있다.
* 실행 분석
  ```java
  Future<Integer> future = es.submit(new MyCallable())
  ``` 
    * ExecutorService에 Task(MyCallable)을 전달
    * ExecutorService는 Future(FutureTask) 객체 생성
    * 생성된 Future 객체 안에 Task 보관
      * Future 내부에는 **Task의 작업 완료 여부** 와 **작업 결과 값** 을 가지고 있다.
    * Future 객체는 블로킹 큐에 보관된다. 
    * 요청 스레드에 Future 객체는 바로 반환된다.
  
  ```
  ExecutorService(ThreadPoolExecutor)
  
  블로킹 큐에 있는 Task를 꺼냄
   → 스레드 풀에 있는 스레드가 Task의 run() 호출
   → run() 메서드가 Task의 call() 호출 
  ```

  ```java
  Integer result = future.get();
  ```
    * 요청 스레드는 Fuutre.get()을 통하여 Task의 결과를 받아 볼 수 있다.
    * 하지만 Task가 아직 완료되지 않을 수도 있다.
      * 이런 경우 요청 스레드는 Task가 완료될 때까지 기다려야 한다.
      * 요청 스레드의 상태: RUNNABLE → WAITING
      * Future 완료
        * 완료 여부: O
        * 결과 값 존재
      * Future 비완료
        * 완료 여부: X
        * 결과 값 NONE
        * 요청 스레드 대기(Blocking)
          * 참고) 블로킹 메서드: Thread.join, Future.get()
    * Task가 완료되면
      * 스레드는 Future의 Task의 반환 결과를 담고 완료 처리
      * if 요청 스레드 대기) 
        * 스레드(Task를 수행한 스레드)는 요청 스레드를 깨움 (WAITING → RUNNABLE)

#### Future 왜 필요한가?
* Future 반환
  ```java
  Future<Integer> future1 = es.submit(task1); // 여기는 블로킹 아님
  Future<Integer> future2 = es.submit(task2); // 여기는 블로킹 아님
  
  Integer sum1 = future1.get(); // 여기서 블로킹
  Integer sum2 = future2.get(); // 여기서 블로킹
  ```

* Future를 반환하지 않는다면
  ```java
  Integer sum1 = es.submit(task1); // 여기서 블로킹
  Integer sum2 = es.submit(task2); // 여기서 블로킹
  ```
  * 이러한 경우 멀티스레드를 제대로 활용하지 못하게 된다.

#### Future의 잘못된 사용
* 잘못된 사용 1
  ```java
  Future<Integer> future1 = es.submit(task1); // non-blocking
  Integer sum1 = future1.get(); // blocking, 2초 대기
  
  Future<Integer> future2 = es.submit(task2); // non-blocking
  Integer sum2 = future2.get(); // blocking, 2초 대기
  ```
* 잘못된 사용 2
  ```java
  Integer sum1 = es.submit(task1).get(); // get()에서 블로킹
  Integer sum2 = es.submit(task2).get(); // get()에서 블로킹
  ```