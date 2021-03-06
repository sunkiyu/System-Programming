# Memory Order & atomic

[참조 : 씹어먹는 C\+\+ - \<15 - 3. C\+\+ memory order 와 atomic 객체\>](https://modoocode.com/271)

## Memory는 느리다.
* CPU와 RAM은 물리적으로 떨어짐.   
* CPU가 RAM에서 데이터를 읽어오기 위해서는 시간이 오래걸림.   

## 캐시
* RAM이 느리기 떄문에 Cache를 도입   
* 캐시란 CPU 칩 안의 조그마한 메모리   
* 캐시는 CPU에서 연산 수행하는 부분과 거의 붙어있다 싶이해서 읽기/쓰기 속도 매우 빠름.   
* CPU에서 가장 많이 접근하는 메모리는 L1 캐시, 그 다음 자주 접근 L2, L3 캐시 순으로 놓임   
* CPU가 특정 수소 데이터에 접근한다면 일단 캐시에 있는지 확인 후 있으면 읽고 없으면 멤모리까지 갔다온다.   
* 캐시에 있는 데이터를 다시 요청해서 시간 절약하는 것을 Cache Hit라 한다.    
* 반대로 캐시에 요청한 데이터가 없어 메모리까지 갔다오는 것을 Cache miss라 한다   
* 메모리를 읽으면 일단 캐시에 저장 -> 캐시가 다 찼다면 "보통" LRU - Least Recently Used 가장 오랫동안 참조되지 않은 캐시를 날리고 새롭게 기록한다.   
* 따라서 LRU - Least Recently Used에서는 최근데 접근한 데이터를 자주 반복 접근헀다면 유리   


## 코드 재배치 
* 현대의 CPU 한 번에 한 명령어씩 실행하는 것이 아니다.   

### CPU 파이프라이닝 (pipelining)
* 여러 개의 작업을 동시에 실행하는 것을 파이프라이닝(pipelining) 이라고 함.   
* CPU도 마찬가지로 동작, fetch->decode->excecute->write를 동시에 수행.   
* but, 각 명령어 마다 실행 속도는 천차만별   
* 따라서 오랜 시간이 걸리는 명령어가 있으면 다른 명령어들이 밀림 -> 따라서 컴파일러가 CPU 파이프라인을 효율적으로 활용하도록 명령어 재배치함.   
* 물론 전제 조건은 명령어를 재배치해도 최종 결과물은 같아야한다. ( 싱글 스레드일 경우만 보장됨)   
* 따라서 멀티스레드 환경에서는 예상치 못한 결과가 나올 수 있다.   

### 컴파일러뿐만 아니라 CPU도 명령어를 재배치한다.

```cpp
int a =0,b=0;
a=1;//캐시에 없음
b=1;//캐시에 있음 
```
a = 1 의 경우 현재 a가 캐시에 없으므로 매우 오래 걸림.   
b = 1 의 경우 현재 b가 캐시에 있으므로 빠르게 처리 가능.   
따라서, CPU에서 b = 1; 이 a = 1; 보다 먼저 처리될 수 있다.   
따라서, b == 1인데 a == 0 인 순간을 관찰할 수 있다.

## 수정 순서(modification order)
* C++ 의 모든 객체는 수정 순서를 정의할 수 있다.   
* 수정 순서는 어떤 객체의 값을 실시간으로 값의 변화를 기록한 것이라고 보면된다.(실제로 존재하는 것이 아닌 가상의 순서다 있다고 가정)   
* 5 -> 8 -> 6 ->3 (a의 수정 순서가 이와 같다고 가정)  
* C++에서 보장하는 사실은, 원자적 연산을 할 경우 모든 스레드에서 같은 객체에 대해 동일한 수정 순서를 관찰 할 수 있다   
* 순서가 동일하다는 것은 어떤 스레드가 a의 값을 읽었을 때 8이라면, 그 다음에 읽어지는 a값은 반드시 8,6,3 중 하나여야 한다는 것   
* 수정 순서가 거꾸로 거슬러 5를 읽는 일은 없음   
* 모든 스레드에서 변수의 수정 순서에 동의하면 문제가 안됨. 즉, 같은 시간에 변수 a를 관찰했다고 굳이 모든 스레드가 동일 값을 관찰할 필요는 없다.   
* 정확히 같은 시간에 스레드 1,2에서 a 값을 관찰했을 떄 스레드 1에서는 5, 스레드 2에서는 8을 관찰해도 문제될 것이 없다는것   
* 심지어 동일 코드를 각기 다른 스레드에서 실행했을 때 실행하는 순서가 달라도 결과만 같다면 문제 안됨   
* 하지만 이 조건을 강제할 수 없는 이유는 CPU 캐시가 각 코어별로 존재하기 때문...    
* 각 코어들은 자신만의 L1, L2캐시들을 갖고 있다   
* 따라서, 스레드 1에서 a = 5 로 수정 후 자신의 캐시에만 기록해 놓고 다른 코어에 알리지 않으면 스레드 3에서 a 값을 확인할 때, 5를 얻는다는 보장이 없음   
* 매번 값을 기록할 때 마다, 모든 캐시에 동기화 시킨다면 시간을 꽤나 잡아먹는다.   
* C++에서는 로우 레벨 언어 답게 이를 세밀하게 조정할 수 있다.   

## 원자성(atomicity)
* C++ 에서 모든 스레드들이 수정 순서에 동의해야만 하는 경우는 바로 모든 연산들이 원자적 일 때.   
* 원자적 연산이 아닌 경우 모든 스레드에서 같은 수정 순서를 관찰할 수 있음이 보장되지 않아 동기화 방법을 통해 처리 필요   
* 그렇지 않으면 프로그램이 정의되지 않은 행동을 할 수 있다.   
* 원자전 연산이란, CPU가 명령어 1개로 처리하는 명령으로 중간에 다른 스레드가 끼어들 여기가 전혀 없는 연산을 말함    
* 즉, "연산을 반 정도 했다"는 있을 수 없고 연산을 "했다" "안했다" 두 가지 상황만 존재. 즉, 원차처럼 쪼갤 수 없음    
* C++ 에서는 원자적 연산을 쉽게 할 수 있도록 여러가지 도구를 지원   
* 이러한 원자적 연산은 올바른 연산을 위해 굳이 뮤텍스가 필요없다. 즉 속도가 더 빠르다.   

## atomic 템플릿
* atomic 의 템플릿 인자로 원자적으로 만들고 싶은 타입을 전달하면 됨.   
* 증감 연산에서 아무런 뮤텍스로 보호하지 않아도 정확한 값을 얻을 수 있다.   
* 원래 CPU는 한 명령어에서 메모리에 읽기 혹은 쓰기 둘 중 하나 밖에 하지 못함   
* 메모리에 읽기 쓰기 동시에 하는 명령은 없다.   
* 하지만 atomic 템플릿을 사용하면 읽기 쓰기를 모두 해버림    
* 어느 CPU에서 명령어를 실행할지 x86 컴파일러가 알고 있기 때문에 가능.   
* atomic 객체들이 위와 같은 연산이 정말 원자적으로 가능한지 확인하려면 is_lock_free() 함수르 호출해보면 됨   
* lock_free가 true ? lock 없이 실행 가능하다 즉, 원자적 실행 가능   

## memory_order
* atomic 객체들의 경우 원자적 연산 시 메모리에 접근할 때 어떤 방식으로 접근할지 지정 가능.   
* memory_order_relexed 가장 느슨한 조건    
* memory_order_relexed 방식으로 메모리에 읽고 쓸 경우 주위의 다른 메모리 접근들과 순서가 바뀌어도 무방   
* store/load는 atomic 객체들에 대핸 원자적으로 읽기/쓰기를 지원   
* memory_order_relaxed 는 앞서 말했듯이, 메모리 연산들 사이에서 어떠한 제약조건도 없음   
* 서로 다른 변수의 relaxed 메모리 연산은 CPU 마음대로 재배치 할 수 있다.   
* 순으로 CPU 가 순서를 재배치 하여 실행해도 무방하다는 뜻   
* memory_order_relaxed 는 CPU 에서 메모리 연산 순서에 관련해서 무한한 자유를 줌    
* 덕분에 CPU 에서 매우 빠른 속도로 실행할 수 있음   

## memory_order_acquire 과 memory_order_release
```cpp
*data = 10;//(1)
is_ready->store(true, memory_order_relaxed);//(2)
//memory_order_relaxed 이므로 (1)과 (2)가 순서가 뒤바뀔 수도 있다.   

while (!is_ready->load(memory_order_relaxed)) {//(3)
}

std::cout << "Data : " << *data << std::endl; //(4)
//memory_order_relaxed 이므로 (3)과 (4)가 순서가 뒤바뀔 수도 있다
//따라서, 이와 같은 조직에서는 memory_order_relaxed를 사용할 수 없다.

*data = 10; //(5)
is_ready->store(true, std::memory_order_release); //(6)
//memory_order_release는 해당 명령 이전의 모든 메모리 명령어들이 해당 명령 이후로 재배치 되는 것을 금지
//만약 같은 변수를 memory_order_acquire로 읽는 스레드가 있다면 memory_order_release이전에 오는 
//모든 메모리 명령어들이 해당 스레드에 의해 관찰 될 수 있어야함
//memory_order_acquire 의 경우, release 와는 반대로 해당 명령 뒤에 오는 모든 메모리 명령들이 해당 명령 위로 재배치 되는 것을 금지
```
```cpp
data[0].store(1, memory_order_relaxed);
data[1].store(2, memory_order_relaxed);
data[2].store(3, memory_order_relaxed);
is_ready.store(true, std::memory_order_release);
```
* 여기서 data 의 원소들을 store 하는 명령들은 모두 relaxed 때문에 자기들 끼리는 CPU 에서 마음대로 재배치될 수 있지만   
* 아래 release 명령을 넘어가서 재배치될 수는 없음   
* data 들의 값을 확인했을 때 언제나 정확히 1, 2, 3 이 들어있음   

## memory_order_acq_rel
* memory_order_acq_rel는 이름에서도 알 수 있듯, acquire과 release를 모두 수행   
* 읽기/쓰기 모두를 수행하는 명령들, 예를 들어 fetch_add와 같은 함수에서 사용될 수 있다.   

## memory_order_seq_cst
* memory_order_seq_cst 는 메모리 명령의 순차적 일관성(sequential consistency) 을 보장   
* 즉, 메모리 명령 재배치가 없고, 모든 스레드에서 모든 시점에 동일 값을 관찰 가능한, 프로그래머가 생각하는 그대로 CPU가 동작하는 방식   
* memory_order_seq_cst 를 사용하게 된다면, 해당 명령을 사용하는 메모리 연산들 끼리는 모든 쓰레드에서 동일한 연산 순서를 관찰할 수 있도록 보장   
* 우리가 atomic 객체를 사용할 때, memory_order 를 지정해주지 않는다면 디폴트로 memory_order_seq_cst 가 지정.   
* counter ++ 은 사실 counter.fetch_add(1, memory_order_seq_cst) 와 동일한 연산.

```cpp
std::atomic<bool> x(false);
std::atomic<bool> y(false);
std::atomic<int> z(0);

void write_x() { x.store(true, std::memory_order_release); }

void write_y() { y.store(true, std::memory_order_release); }

void read_x_then_y() {
  while (!x.load(std::memory_order_acquire)) {
  }
  if (y.load(std::memory_order_acquire)) {
    ++z;
  }
}

void read_y_then_x() {
  while (!y.load(std::memory_order_acquire)) {
  }
  if (x.load(std::memory_order_acquire)) {
    ++z;
  }
}

int main() {
  thread a(write_x);
  thread b(write_y);
  thread c(read_x_then_y);
  thread d(read_y_then_x);
  a.join();
  b.join();
  c.join();
  d.join();
  std::cout << "z : " << z << std::endl;
}
```
* 위 예제에서 x,y가 모두 true가 되어야 모든 스레드가 종료되어 z가 찍힌다.      
* x true가 되어 while 문을 빠져나왔을 때 y가 꼭 1이 아닐 수 있다.      
* y true가 되어 while 문을 빠져나왔을 때 x가 꼭 1이 아닐 수 있다.   
* 따라서 z값은 0,1,2 모두가 나올 수 있다.    

```cpp
#include <atomic>
#include <iostream>
#include <thread>
using std::memory_order_seq_cst;
using std::thread;

std::atomic<bool> x(false);
std::atomic<bool> y(false);
std::atomic<int> z(0);

void write_x() { x.store(true, memory_order_seq_cst); }

void write_y() { y.store(true, memory_order_seq_cst); }

void read_x_then_y() {
  while (!x.load(memory_order_seq_cst)) {
  }
  if (y.load(memory_order_seq_cst)) {
    ++z;
  }
}

void read_y_then_x() {
  while (!y.load(memory_order_seq_cst)) {
  }
  if (x.load(memory_order_seq_cst)) {
    ++z;
  }
}

int main() {
  thread a(write_x);
  thread b(write_y);
  thread c(read_x_then_y);
  thread d(read_y_then_x);
  a.join();
  b.join();
  c.join();
  d.join();
  std::cout << "z : " << z << std::endl;
}
```
* 위 예제에서 x,y가 모두 true가 되어야 모든 스레드가 종료되어 z가 찍힌다.      
* x true 가 되면 이 후 모든 스레드가 읽을 때 x == true 임을 보장.      
* y true 가 되면 이 후 모든 스레드가 읽을 때 y == true 임을 보장.   
* x가 true가 된 시점에 y도 true라면 두 함수의 if문이 모두 실행 되므로 2가 나올 수 있다.  
* 따라서 z값은 1,2 가 나올 수 있다. 0은 나올 수 없다 왜냐? 모든 스레드가 종료되었다는 것은 x,y 가 모두 true 가 되었다는 뜻인데    
* 설령 x가 true 가 된 시점에 y가 false 였어도 어쨌든 추후에 y가 true 가 되어(스레드가 모두 종료하기 위해) while문을 빠져나왔을 때 x가 1을 보장하므로 if문에서 무조건 z++가 된다.   
* 설령 y가 true 가 된 시점에 x가 false 였어도 어쨌든 추후에 x가 true 가 되어(스레드가 모두 종료하기 위해) while문을 빠져나왔을 때 y가 1을 보장하므로 if문에서 무조건 z++가 된다.     

## memory_order_seq_cst 는 비싼 연산!
* 인텔 혹은 AMD 의 x86(-64) CPU 의 경우에는 사실 거의 순차적 일관성이 보장되서 memory_order_seq_cst 를 강제하더라도 그 차이가 크지 않다.   
* ARM 계열의 CPU 와 같은 경우 순차적 일관성을 보장하기 위해서는 CPU 의 동기화 비용이 매우 크므로 꼭 필요할 경우만 사용!   
