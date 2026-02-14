## Section003
### 스레드 시작1
#### 자바 메모리 구조
* 메서드 영역
    * 클래스 정보, static 영역, 런타임 상수 풀
* 스택 영역
    * 지역변수, 중간 연산 결과, 메서드 호출 정보 포함
    * 각 스레드별로 하나의 실행 스택이 생성됨
* 힙 영역
    * 객체와 배열이 생성되는 영역
    * GC가 이루어지는 영역

#### 스레드 생성
* Runnable 인터페이스를 구현하는 방법
* Thread 클래스를 상속 받는 방법
    * run() 메서드 재정의
    * 다만 run()을 직접 호출할 일은 절대 없다. start()에 의해 호출된다.
* [예제] (src/thread/start/HelloThreadMain.java 확인)
  ![img.png](note/image/sec003.png)
    * main 스레드가 run() 메서드를 실행하는게 아니다.
        * main 스레드가 start()를 통해 Thread-0 스레드가 run()을 실행하도록 시킨다.
    * 스레드는 순서와 실행 기간을 모두 보장하지 않는다.

### 스레드 시작2
#### start() vs run()
* start() 대신에 재정의한 run() 메서드를 직접 호출하면?
  ![img_1.png](note/image/sec003_1.png)
    * main 스레드가 모든 것을 처리한다.
    *  별도의 스레드에서 재정의한 run() 메서드를 실행하려면, 반드시 start() 메서드를 호출해야 한다.


### 데몬 스레드
#### 데몬 스레드
* 사용자 스레드
    * 프로그램 주요 작업 수행
    * 작업이 완료될 때까지 실행
    * 모든 사용자 스레드가 종료되면 JVM도 종료
        * 데몬 스레드가 아닌 모든 스레드가 종료되면, 자바 프로그램도 종료된다.
* 데몬 스레드
    * 백그라운드에서 보조적인 역할 수행
    * 사용자 스레드가 종료되면 데몬 스레드는 자동으로 종료
* 참고) run() 메서드 안에서는 CheckedException을 반드시 잡아야 한다. 밖으로 던질 수 없다.

### 스레드 생성 - Runnable
* [예제] (src/thread/start/HelloRunnableMain.java 확인)

#### Thread 상속 VS Runnable 구현
* **Runnable 구현이 더 좋은 선택이다.**
* Thread 상속
    * 구현 간단
    * BUT 상속 제한
    * BUT 유연성 부족
* Runnable 인터페이스 구현 방식
    * 상속의 자유로움
    * 코드 분리(스레드와 실행할 작업 분리)
    * 여러 스레드가 동일한 Runnable 객체를 공유할 수 있어 자원 관리 효율적
    * BUT 코드가 복잡해진다.
        * Runnable 객체 추가하고 이를 Thread 객체로 전달해야하는 과정 추가됨

### 여러 스레드 만들기
* [예제1] (src/thread/start/ManyThreadMainV1.java 확인)
* [예제2] (src/thread/start/ManyThreadMainV2.java 확인)


### Runnable을 만드는 다양한 방법
#### 정적 중첩 클래스
* [예제] src/thread/start/InnerRunnableMainV1.java 확인)

#### 익명 클래스 사용
* [예제] src/thread/start/InnerRunnableMainV2.java 확인)

#### 익명 클래스 변수 없이 직접 전달
* [예제] src/thread/start/InnerRunnableMainV3.java 확인)

#### 람다
* [예제] src/thread/start/InnerRunnableMainV4.java 확인)
