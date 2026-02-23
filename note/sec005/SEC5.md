## Section05

### 인터럽트
#### 시작
* 인터럽트를 사용하면 WAITING, TIMED_WAITING 같은 대기 상태의 스레드를 직접 깨워서, 작동하는 RUNNABLE 상태로 만들 수 있다.
* 특정 스레드의 인스턴스에 interrupt() 메서드를 호출하면, 해당 스레드에 인터럽트가 발생한다.
  * 인터럽트를 받은 스레드는 대기 상태에서 깨어나 RUNNABLE 상태가 된다.
  * interrupt() 를 호출했다고 해서 즉각 InterruptedException이 발생하는게 아니라 sleep()처럼 InterruptedException을 던지는 메서드를 호출하는 경우 발생한다.
  * InterruptedException이 발생하면서 인터럽트 상태는 해제된다.
* 인터럽트를 사용하면 대기중인 스레드를 바로 깨워서 실행 가능한 상태로 바꿀 수 있다.