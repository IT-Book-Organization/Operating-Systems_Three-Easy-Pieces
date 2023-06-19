# 운영체제 개요

### 운영체제란?
- 프로그램을 쉽게 실행한다. 
- 프로그램 간 메모리 공유를 가능케 한다. 
- 장치와 상호 작용을 가능케한다.
- 시스템을 사용하기 편리하면서 정확하고 올바르게 동작시킬 책임을 갖는다.
위 책임을 갖는 소프트웨어를 **운영체제(Operating System)**이라고 한다.

위 일들을 하기 위해 
- 가상 형태의 자원을 제공하는 가상화 기법을 이용한다.
  - 이 때문에 가상 머신이라고도 불린다.
- 응용 프로그램이 이용 가능한 수백 개의 시스템 콜(fork(), wait(), read() 등등)을 제공한다.
- 많은 프로그램이 CPU(자원)를 공유하여 동시에 실행될 수 있게 한다.
  - 이 때문에 자원 관리자 라고도 불린다.

이 책에서는 운영체제를 세가지 주된 아이디어로 나누어 설명한다.
- Virtualization
- Concurrency
- Persistence

## 가상화 (Virtualization)

### 2.1 CPU 가상화
소프트웨어가 모든 하드웨어 자원을 자기가 갖고 있다는 illusion을 제공하는 것이다.

```
ex : cpu가 4개 밖에 없더라도 수많은 프로그램을 동시에 실행할 수 있다. 
이는 프로그램에 자기가 하나의 CPU를 점유하고 있다는 illusion을 일으킨다.
```

### 2.2 메모리 가상화
프로그램이 system call이라는 interface를 이용해서 OS에게 자원을 요청하면 OS가 CPU, 메모리, 디스크와 각종 장치들을 제공한다.

```
ex. 메모리를 달라 -> malloc() -> system call -> OS가 실제 메모리를 찾음 -> OS가 메모리를 virtual 형태로 만들어서 제공
```

자신이 malloc()을 통해 부여받은 메모리는 다른 프로세스와 공유될 수 있으나 마치 자신의 메모리를 갖고 있는 것처럼 보인다.

이는 운영체제의 메모리 가상화 기능 덕에 가능하다.
각 프로세스는 **가상 주소 공간**을 갖고 운영체제는 이 가상 주소를 컴퓨터의 물리 메모리로 mapping하는 역할을 한다.(이를 address translation 이라고 한다.)

이를 통해, 하나의 프로그램이 수행하는 각종 메모리 연산은 다른 프로그램의 주소 공간에 영향을 주지 않아 실행 중인 프로그램의 입장에서는 자신만의 물리 공간을 할당 받은 것처럼 느낀다.

그러나 실제로는 물리 메모리는 공유 자원이다.

## 2.3 병행성(Concurrency)
OS는 자원을 정확하고 효율적으로 제공해야한다.

multi-thread 코드에서, 두 thread가 모두 `volatile int counter`에 접근해 counter++;를 수행한다면 loop의 크기가 작을 때는 첫 번째 thread가 실행되고 두 번째 thread가 실행되기 전에 첫 번째 thread의 수행이 끝나서 올바른 결과가 나올 수 있지만 loop의 크기가 커지면 두 thread가 실행되는 시간이 겹치면서 `counter에` 접근하려고 한다 (race condition).

두 thread중 하나는 `counter의` 값을 update하겠지만 둘 중 어느 thread가 값을 update했는지는 알 수 없다. 

그냥 그 순간 scheduling된 thread가 `counter의` 값을 update하는 것이다.

race condition을 방지하기 위해 locking과 같은 메커니즘을 사용하며 이것이 concurrency에 해당한다.

위의 경우에 race condition이 발생하는 이유는 ++가 atomic하지 않기 때문이다.

어떤 instruction이 atomic하다는 것은 그 instruction이 실행되는 여러 step 사이에 그 무엇도 끼어들 수 없는 것을 의미한다. atomic하게 만들기 위해 mutex같은 것을 사용해야 한다.

## 2.4 영속성(Persistence)
소프트웨어는 자신의 data를 non-volatile memory에 저장하고 싶어한다. OS가 물리적 공간을 어떻게 다루는지와 file system 등에 대해서 배운다.

file system은 디렉토리와 파일로 구성되어 있다.

user는 file system을 통해 raw disk에 접근한다. 

user는 원래 raw disk에 직접 접근할 수 없어야 한다(지금은 가능한 것도 많지만).

user가 file system을 이용할 때는 syscall을 사용해야한다.

## 2.5 설계 목표

- 성능 (오버헤드 최소화)
  - 사용하기 쉬운 시스템을 만드는 것은 의미 있지만 반드시 해야하지는 않다.
  - 기능을 과도한 오버헤드 없이 제공하는 것이 가장 우선시 될 목표다.
- 보호
  - 응용 프로그램 간의 고립을 통해 다른 프로그램의 메모리에 침범해 영향을 주는 것을 방지한다.
- 보안