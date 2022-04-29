출처 : '뇌를 자극하는 윈도우즈 시스템 프로그래밍' 책

# 절차적 함수 호출 지원 CPU 모델

## 스택 프레임 구조 
* 스택 프레임? 
* 함수 호출 과정에서 할당되는 메모리 블록(지역 변수 선언으로 인해 할당되는 메모리 블록)   
* 함수 호출이 완료(return)되면 주소를 알고 있어도 기존에 선언된 지역변수 접근 불가, 즉 할당된 메모리가 반환됨.   
* 함수가 반환되면 함수의 스택 프레임은 모두 반환   

## sp 레지스터
* 지역변수를 위한 메모리 공간을 스택이라 부르는 이유는 FILO(First In Last Out) 구조 이기 때문이다.   
* 즉, 스택 프레임은 가장 먼저 할당되면 가장 나중에 반환, 가장 나중에 할당된것이 가장 먼저 반환된다.   
* 스택에 데이터를 쌓거나 반환하기 위해 쌓아 올린 스택 위치를 기억해야하며, CPU내에 sp(Stack Pointer)라는 이름의 레지스터가 그 역할을 담당.   
* sp가 가르키는 위치를 아래로 이동시키며 스택 프레임을 반환한다.   
* 호출 완료된 함수를 빠져나오는 시점에서 sp를 얼마만큼 이동시킬 것인가? 프레임 포인터 레지스터가 이 역할을 한다.   

 ## 프레임 포인터(Frame Pointer) 레지스터
 * 새로운 함수가 호출될 때마다 이 값을 0으로 초기화 하며, 새로운 변수가 선언될 때마가 그 크기만큼 값을 증가시킨다.   
 * 되돌아갈 (함수 호출 이전의) sp의 위치만 저장하는데 이역할을 fp(프레임 포인터)레지스터가 한다.  
 * 함수 호출이 중첩되어 일어나면 fp가 덮어 써지는 문제가 발생한다.   
 *  => 함수 호출이 일어날 때마다 fp 레지스터에 저장되어 있는 값을 스택에 저장하고 새로운 값을 fp 레지스터에 채운다.     

## 함수 호출 인자의 전달과 PUSH & POP 명령어 디자인
* 함수호출 인자의 전달방식    
* 전달인자는 어디에 두는 것이 좋겠는가? 어떻게 함수 내부로 전달되는가?    
* 함수 호출 시 전달되는 인자들은 모두 스택에 저장, 지역 변수가 스택에 할당되는 방식과 동일.    
* 현재 sp가 가르키는 위치에 인자를 저장하고 sp값을 증가시킨 다음 다음 인자를 저장하며 sp값을 증가 시킨다.    

## 함수 호출(Procedure Call)에 의한 실행의 이동
* 프로그램이 실행되는 원리와 프로그램 작성 시 정의하고 호출되는 함수의 원리를 보자
* 과정에서 pc 레지스터 (프로그램 카운터라 불린다)의 역할을 알게 된다   

## 프로그램 카운터(Program Counter)
* 프로그램 실행시 스택(파라미터,지역변수),코드(실행 코드),데이터(전역, 정적 변수),힙(동적 할당)    
* 코드 영역은 프로그램이 동작하기 위한 프로그램 코드(컴파일된 명령어 집합)이 올라가는 위치.   
* 코드 영역에 실행될 명령어들이 올라가 순차적 실행   
* Fetch, Decode, Excecution 이 세 단계 중 Fetch가 명령어를 cpu내부로 가져오고 가져오는 위치가 바로 프로그램 코드 영역.   
* 그 후 Fetch, Decode, Execution 된다.   
* 명령어 길이가 4바이트, 그리고 실행중인 프로그램이 현재 1036번지 명령어이면 다음번에는 1040번지 명령어가 fetch되어야함.   
* 이 때, 어느 위치 명령어까지 가져와 실행했는지 기억해 둬야 다음번 실행 명령어를 가져 올 수 있다.   
* 명령어를 순차적 fetch하기 위해 pc레지스터를 둔다.    

## 함수 호출 규약(Calling Convention)
* 스택은 왼쪽 -> 오른쪽 순서로 스택에 쌓이는 구조가 있고, 반대로 오른쪽 -> 왼쪽으로 스택에 쌓이는 구조가 있다.   
* 스택 프레임 반환?   
* 함수 호출이 완룓된 이후 sp 레지스터를 복원하는 등 작업을 하는데 주체가 호출자일수도 호출된 함수일 수도 있다.   
* 이처럼 함수 호출시 인자를 전달하는 방식과 스택 프레임을 반환하는 방식을 가르켜 함수 호출 규약이라한다.

## __cdecl, __stdcall
* 함수 선언부에서 __stdcall을 본 적 있을 것이다. 이것이 바로 함수 호출 규약을 지정하는 것이다.    
* WINAPI, APIENTRY, CALLBACK 이런것들은 다음과 같이 정의된 매크로이다.   
* #define CALLBACK __stdacll   
* #define WINAPI __stdcall   
* 따로 함수 호출 규약 선언을 하지 않은 것들은 프로젝트 속성창의 디폴트 선언을 따르게 된다.   
 
## 함수 호출 규약의 종류와 의미
* 32비트 기반 함수 호출 규약부터 설명하겠다   
* __cdecl은 c/c++의 디폴트 호출규약   
* 인자 전달 방식이 c언어 스타일, 오른쪽에 전달되는 인자가 먼저 스택에 쌓임.   
* 반환시 호출자(caller)가 스택프레임 반환   

## __cdecl과 __stdcall의 차이점?
* 차이는 스택 프레임 반환 주체    
* __stdcall : 호출된 함수 callee가 스택 프레임 반환   
* __cdecl : 호출 함수 caller가 스택 프레임 반환   
* __fastcall : 함수 호출을 빠르게 처리하기 위함.   
=> 첫번째 전달인자와 두번째 전달인자는 레지스터의 사용 유무를 설명   
=> 이 둘은 ecx, edx를 통해 저장되는데 이것은 레지스터의 이름이다.   
=> 레지스터를 사용하고 있는데, 이것이 함수 호출이 빨라지는 근거가 된다.   
=> 2개를 넘어서는 인자는 스택을 활용하게 된다.   

## 64비트 기반 함수 호출 규약을 보자
* 64비트 시스템에서는 함수 호출 규약이 운영체제에 따라서 나뉘게 된다.   
* Windows 기반에서는 총 8개의 레지스터를 활용해서 전달되는 인자를 저장한다.   
* 실제로 레지스터에 저장되는 전달인자의 개수는 4개에 지나지 않는다.   
* 표에 표시된 rcx/xmm0은 첫번째 전달인자가 rcx 혹은 xmm0에 저장된다는 것을 뜻한다.   
* 따라서 총 4개의 전달인자까지만 레지스터를 통해 처리한다.   
* Linux나 BSD 계열의 시스템에서는 훨씬 더 많은 수의 레지스터를 전달되는 인자에 할당하고 있음을 볼 수 있다.   
* 최대 14개의 인자까지 레지스터를 통해 처리한다.   
 