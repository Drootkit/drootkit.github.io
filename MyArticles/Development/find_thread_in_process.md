# 查找进程中的线程

本程序实现了针对输入的proc id，查看该进程中的所有的线程，并且获取线程id的功能。

```c
#include<Windows.h>
#include<stdio.h>
#include<stdlib.h>
#include<tchar.h>
#include <TlHelp32.h>

int main(int argc , char* argv[])
{
    // 没有获取到参数的情况
    if (argc <= 1)
    {
        printf("Usage: %s <filename>\n", argv[0]);
        printf("Example: ftip [proc Id]\n", argv[0]);
        return 1;
    }

    int notePID = atoi(argv[1]);

    // 申请空间
    LPDWORD pThreadIdList = NULL;
    DWORD dwThreadIdListLength = 0;
    DWORD dwThreadIdListMaxCount = 2000;
    HANDLE hThreadSnap = INVALID_HANDLE_VALUE;

    pThreadIdList = (LPDWORD)VirtualAlloc(NULL, dwThreadIdListMaxCount * sizeof(DWORD), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (pThreadIdList == NULL)
    {
        printf("VirtualAlloc Failed\n我也没辙 :-)\n");
        return 1;
    }

    RtlZeroMemory(pThreadIdList, dwThreadIdListMaxCount * sizeof(DWORD));
    THREADENTRY32 th32 = { 0 };

    // 拍摄快照, 通过第一个参数快照系统中的所有线程，指定进程的快照
    hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, notePID);

    if (hThreadSnap == INVALID_HANDLE_VALUE)
    {
        printf("Get all threads in system Failed\n我也没辙 :-)\n");
        return 1;
    }

    // 结构的大小
    th32.dwSize = sizeof(THREADENTRY32);

    //遍历所有THREADENTRY32结构, 按顺序填入数组【有关系统快照中遇到的任何进程的第一个线程的信息。】
    //函数返回的快照的句柄 , 指向 THREADENTRY32 结构的指针
    BOOL bRet = Thread32First(hThreadSnap, &th32);
    int i = 0;
    while (bRet)
    {
        printf("[=> %d ] the thread id is  < %d >\t\t", ++i, th32.th32ThreadID);
        if(i%2==0 && i!=0)
        {
            printf("\n");
        }
        // 先检查当前线程是不是指定进程下的线程
        if (th32.th32OwnerProcessID == notePID)
        {
            if (dwThreadIdListLength >= dwThreadIdListMaxCount)
            {
                break;
            }
            pThreadIdList[dwThreadIdListLength++] = th32.th32ThreadID;
        }
        // 遇到的任何进程的下一个线程的信息
        bRet = Thread32Next(hThreadSnap, &th32);
    }

    printf(TEXT("\nThere are %d threads in process %d.\n"), dwThreadIdListLength, notePID);
    return 0;
}
```

实现了查找目标进程中所有的线程号和线程数量。