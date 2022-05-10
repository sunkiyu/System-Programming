# VirtualAlloc

__LPVOID VirtualAlloc__(   
   [in, optional] LPVOID lpAddress,   
   [in]           SIZE_T dwSize,   
   [in]           DWORD  flAllocationType,   
   [in]           DWORD  flProtect                                                                                                      
);                                             
                                                                                                   
__Parameters__
[in, optional] lpAddress   

* 할당 메모리의 시작 주소입니다. 만약 메모리가 예약되어 있다면, 할당 단위의 가장 가까운 배수로 내림합니다.   
* 메모리가 이미 예약되어 있고 커밋 중인 경우 주소는 다음 페이지 경계로 내림됩니다.   

[in] dwSize   

* 영역의 크기(바이트)입니다. lpAddress 매개변수가 NULL 이면 이 값은 다음 페이지 경계로 반올림됩니다.    
* 그렇지 않으면 할당된 페이지에는 lpAddress 에서 lpAddress + dwSize 범위의 하나 이상의 바이트를 포함하는 모든 페이지가 포함됩니다 .   
* 즉, 페이지 경계를 가로지르는 2바이트 범위로 인해 두 페이지가 모두 할당된 영역에 포함됩니다.   
