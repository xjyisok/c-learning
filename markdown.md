# 第三次作业
## 作业1：CreateThread实验
### 编译器MSVC，MinGW64 IDE:VS2022,VScode(本实验部分函数MinGW64不支持所以灵活使用MSVC和Vscode)
原始代码如下：(使用MinGW64)
```
~~const unsigned int THREAD_NUM = 50; 
unsigned int g_Count = 0;
DWORD WINAPI ThreadFunc(LPVOID);

int main()
{
HANDLE hThread[THREAD_NUM]; 
for (int i = 0; i < THREAD_NUM; i++)
{
    hThread[i] = CreateThread(NULL, 0, ThreadFunc, &i, 0, NULL); 
}
WaitForMultipleObjects(THREAD_NUM,hThread,true, INFINITE); 
std::cout<<g_Count<<std::endl;
return 0;
}
DWORD WINAPI ThreadFunc(LPVOID p)
{
Sleep(50); 
g_Count++;
Sleep(50);
printf("我是， pid = %d 的子线程 当前gcount值为:%d\n", GetCurrentThreadId(),g_Count);
return 0;}
```
多次运行结果如下
```
C:\Users\XDUAI\Desktop\windowsmpi>g++ winapitest.cpp -o winapitest

C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
46

C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
45

C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
48

C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
49
```
不同NUM_THREADS下输出  

| NUM_THREADS   | 10   | 30   |   50   |
|--------|--------|--------|-------|
| 第一次 | 9   | 18   |49|
| 第二次 | 9   | 17   |47|
| 第三次 | 8   | 19   |48|

### 原理
从实验结果可以看出得到的g_count值总是小于应有的输出值即```THREAD_NULL=50```这是因为线程间存在竞争。每一个线程在创建时会先sleep(50ms)由于线程的创建速度是及其迅速的所以在sleep结束后几乎所有的线程同时读取g_count。线程中```g_count++```这一行代码做的操作本质上是将```g_count```的值写入寄存器然后加一后再写入内存中的g_count。那么假设在同一时间有n个线程同读取了g_count那么他们做的实际上是重复的操作即将g_count在寄存器中+1然后写入内存。这也就意味着多个线程对于g_count的操作相同，g_count始终只增加了1。线程的数量是固定的，竞争的线程数量越多则g_count最后输出的值越小。  
如果将前后的sleep(50)都去掉那么输出的值基本都是50正确。这是因为线程的创建速度相比于寄存器写入操作而言时间要长的多因此，在下一个线程创建完成之时上一个线程已经完成了g_count++操作。也就不存在竞争。
### 互斥量
代码：
```
const unsigned int THREAD_NUM = 50; 
unsigned int g_Count = 0;
HANDLE hmutex;
DWORD WINAPI ThreadFunc(LPVOID);
int main()
{
hmutex=CreateMutex(NULL,FALSE,NULL);
HANDLE hThread[THREAD_NUM]; 
for (int i = 0; i < THREAD_NUM; i++)
{
    hThread[i] = CreateThread(NULL, 0, ThreadFunc, &i, 0, NULL);
}
WaitForMultipleObjects(THREAD_NUM,hThread,true, INFINITE); 
printf("%d",g_Count);
CloseHandle(hmutex);
return 0;
}
DWORD WINAPI ThreadFunc(LPVOID p)
{
WaitForSingleObject(hmutex,INFINITE);
Sleep(50);
g_Count++; 
Sleep(50);
ReleaseMutex(hmutex);
printf("我是， pid = %d 的子线程 当前gcount值为:%d\n", GetCurrentThreadId(),g_Count);
return 0;
}
```
### 原理
这是使用互斥量，每个线程在进行g_count++写操作前都必须先获取互斥量，互斥量每次只能被一个线程获取，在对全局变量g_count进行写操作完成后释放互斥量让其他线程继续对g_count进行操作。防止在同一时间有多个线程对g_count做写操作造成竞争。
###信号量
代码
```
const unsigned int THREAD_NUM = 50; 
unsigned int g_Count = 0;
HANDLE hsemphore;
DWORD WINAPI ThreadFunc(LPVOID);
int main()
{
hsemphore=CreateSemaphore(NULL,1,1,NULL);
HANDLE hThread[THREAD_NUM]; 
for (int i = 0; i < THREAD_NUM; i++)
{
    hThread[i] = CreateThread(NULL, 0, ThreadFunc, &i, 0, NULL);
}
WaitForMultipleObjects(THREAD_NUM,hThread,true, INFINITE); 
printf("%d",g_Count);
CloseHandle(hsemphore);
return 0;
}
DWORD WINAPI ThreadFunc(LPVOID p)
{
WaitForSingleObject(hsemphore,INFINITE);
Sleep(50);
g_Count++; 
Sleep(50);
ReleaseSemaphore(hsemphore,1,NULL);
```
运行结果
```
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
```
从代码上来看信号量和互斥量看似功能几乎差不多。但是依旧存在以下区别  





