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