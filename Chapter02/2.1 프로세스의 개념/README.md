# Process

한 프로그램만 CPU를 사용할 수 있기 때문에 OS는 많은 CPU가 존재하는 것 같은 illusion을 주어야 한다.

즉 여러 프로그램이 각각 자신의 CPU를 갖고 있는 것 같은 착각을 갖도록 한다.

여기에 필요한 것이 `Time sharing`이다.

time sharing은 CPU Virtualization의 key technique이다. 우리는 CPU의 physical area를 sharing할 수는 없지만 time을 sharing할 수는 있다.

긴 시간을 작은 조각으로 쪼개서 각 조각을 하나의 프로그램에게 준다. 그 시간의 조각 동안 한 프로그램을 실행시키고 그 시간이 끝나면 프로그램의 실행을 멈추고 다른 프로그램으로 switch한다.

(이때, 어떤 대기 중인 프로세스가 여럿이라면 선택해서 실행하여야 하는데, 이를 **스케줄링 정책**이라한다,)

그리고 그 프로그램의 시간을 다 소모하면 다음 프로그램으로 switch한다. 이를 반복하면 많은 프로그램들이 자신만의 CPU를 소유하고 있다고 생각하도록 만들 수 있다.

time sharing에 드는 potential cost는 performance이다. performance는 switch를 할 때에 드는 overhead의 영향을 받는다.

time sharing을 하기 위해서는 OS가 언제든 running program의 상태를 저장(혹은 캡쳐, resume)할 수 있어야 한다.
(멈춘 process를 다시 실행하려면 멈췄던 상태를 저장해야한다.)

즉, 이는 OS가 프로그램의 execution을 통제할 수 있어야 함을 의미한다.

## 프로세스의 개념

```
실행 중인 프로그램을 프로세스라고 한다.
```

프로그램이 실행되면 CPU나 메모리의 상태를 바꾸고 때로는 storage(ex. disk)를 바꾸기도 한다.
이 scope를 CPU와 메모리에 한정시키면 프로세스의 상태는 CPU상태와 메모리로 이루어져 있다.

A process consists of:
- CPU state (registers) 
  - 실질적으로 register들이 CPU의 상태를 hold하고 있다. program counter, stack pointer, general register들이 여기에 해당한다.
- Memory (address space) 
  - 프로그램이 차지하고 있는 공간에 대한 것이다. instruction과 data section이 이에 해당된다.
  CPU state와 memory를 capture할 수 있다면 running program을 표현할 수 있다.

`c = a + b;`와 같이 high-level programming에서는 한 줄로 보이는 코드여도 내부적으로는 load, store등의 여러 instruction이 실행된다. 

한 instruction을 실행하면 CPU의 state가 바뀌고(CPU state라는 것은 결국 multiple register이므로 register가 바뀌는 것이다) CPU에서 메모리로 어떤 value를 저장하면 메모리도 바뀐다.

따라서 OS가 CPU와 메모리를 capture할 수 있다면 프로세스를 표현할 수 있게 된다.

나중에 suspend되었던 프로그램을 다시 실행하고 싶으면 메모리에 저장해둔 값을 restore해서 resume하면 된다.

### CPU state의 특별한 레지스터
- 프로그램 카운터 (PC)
  - 어느 명령어가 실행 중인지 나타낸다.
- 스택 포인터, 프레임 포인터
  - 함수의 변수와 리턴 주소를 저장하는 스택을 관리할 때 사용하는 레지스터이다.

## Process API

프로세스에 대한 다음의 기능들은 운영체제가 기본적으로 제공해야한다.
- Create
- Destroy
- Wait
- Miscellaneous Control
- Status

## 프로세스 생성 : deep

프로그램이 어떻게 프로세스로 변형되는가?

1. 디스크(또는 SSD) 상의 프로그램 코드와 정적 데이터를 메모리, 프로세스의 주소 공간에 탑재한다.
   - 초기에는 코드와 데이터를 모두 메모리에 탑재하였다면, 현대의 운영체제들은 프로그램을 실행하면서 코드나 데이터가 필요할 때 필요한 부분만 메모리에 탑재한다. (페이징, 스와핑)
2. 특정 크기의 메모리 공간을 프로그램에 스택 용도로 할당한다.
   - 스택은 지역 변수, 함수 인자, 리턴 주소 등을 저장하기 위해 사용된다.
3. 프로그램의 힙을 위한 메모리 영역을 할당한다.
   - 힙에는 주로 동적으로 할당된 데이터를 저장하기 위해 사용된다. e.g. 크기가 가변적인 자료구조
   - c 프로그래밍에서는 malloc(), free() 등으로 이러한 힙 메모리를 할당받을 수 있다.
4. 입출력과 관계된 초기화 작업을 수행한다.
   - 표준 입출력, 표준 에러에 해당하는 세 개의 파일 디스크립터를 갖는다.
5. 프로그램의 시작 지점(entry point) 즉, main() 에서 부터 프로그램을 시작한다.

## 프로세스 상태
- Running : CPU에서 현재 실행되고 있는 프로세스
- Ready : suspend되었거나 이제 막 user에 의해 create돼서 initialize되고 running되기를 기다리는 상태. 실행시킬 수 있지만 OS가 그 시점에 실행하기로 선택하지 않은 프로세스를 의미함. ready queue나 list를 사용함.
- Block : I/O를 원하면 memory와 disk의 속도 gap으로 인한 시간이 필요한데 그 때 spinning하면서 CPU cycle을 소모하는 것은 비효율적이기 때문에 프로세스를 block함. request를 fully satisfy하기 전까지 block state로 만듦. I/O나 peripheral들로 인해 발생함.


프로세스는 운영체제의 스케줄링 정책에 따라 스케줄이 되면 준비 상태에서 실행 상태로 전이한다.

## 프로세스를 위한 자료구조
운영체제도 결국엔 프로그램이다. ROM에 위치하여 컴퓨터 구동 시 가장 먼저 실행된다.

다른 프로그램들과 마찬가지로 다양한 정보를 유지하기 위한 자료구조를 가지고 있다.

- 프로세스 상태를 파악하기 위해 준비 상태의 프로세스들을 위한 프로세스 리스트
- 프로세스가 중단되었을 때 해당 프로세스의 레지스터 값들을 저장하는 레지스터 문맥

### xv6 proc 구조
```
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
int eip;
int esp;
int ebx;
int ecx;
int edx;
int esi;
int edi;
int ebp;
};

// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING,
RUNNABLE, RUNNING, ZOMBIE };

// the information xv6 tracks about each process
// including its register context and state
struct proc {
char *mem; // Start of process memory
uint sz; // Size of process memory
char *kstack; // Bottom of kernel stack

// for this process
enum proc_state state; // Process state
int pid; // Process ID
struct proc *parent; // Parent process
void *chan; // If !zero, sleeping on chan
int killed; // If !zero, has been killed
struct file *ofile[NOFILE]; // Open files
struct inode *cwd; // Current directory
struct context context; // Switch here to run process
struct trapframe *tf; // Trap frame for the
};


```

proc은 PCB이며 PCB는 key structure이다. 가장 먼저 initialize하고 조심히 manage해야 한다. running program을 manage하기 때문이다. 그리고 이것이 CPU virtualization의 key이다.

mem, sz, kstack은 memory state를 capture하는데에 사용된다.
context는 CPU state를 capture하는데에 사용된다.