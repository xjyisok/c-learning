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
### 互斥量(MinGW64)
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
###信号量(MinGW64)
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
* 1互斥量只允许一个线程访问资源但是信号量允许多个线程同时访问资源，可以通过设置最大计数量来改变
* 2互斥量相比于信号量开销较小适用于保护单个资源，信号量适用于需要控制对资源池的访问场景。例如，限制对数据库连接池、线程池或其他可共享资源的访问
### 条件变量(MSVC)
代码
```
const unsigned int THREAD_NUM = 50;  // 线程数量
unsigned int g_Count = 0;  // 共享计数器
CRITICAL_SECTION cs;  // 临界区对象
CONDITION_VARIABLE cv;  // 条件变量

DWORD WINAPI ThreadFunc(LPVOID) {
    EnterCriticalSection(&cs);  // 进入临界区
    g_Count++;
    Sleep(50);  // 模拟工作
    LeaveCriticalSection(&cs);  // 离开临界区

    // 通知主线程，g_Count 已更新
    WakeConditionVariable(&cv);  
    return 0;
}

int main() {
    // 初始化临界区和条件变量
    InitializeCriticalSection(&cs);
    InitializeConditionVariable(&cv);

    HANDLE hThread[THREAD_NUM];  // 线程句柄数组
    for (int i = 0; i < THREAD_NUM; i++) {
        hThread[i] = CreateThread(NULL, 0, ThreadFunc, NULL, 0, NULL);
    }

    // 等待所有线程完成
    EnterCriticalSection(&cs);  // 主线程进入临界区

    while (g_Count < THREAD_NUM) {
        // 主线程等待，直到所有线程完成 g_Count++
        SleepConditionVariableCS(&cv, &cs, INFINITE);
    }

    LeaveCriticalSection(&cs);  // 主线程离开临界区

    printf("最终 g_Count 值: %d\n", g_Count);

    // 清理资源
    for (int i = 0; i < THREAD_NUM; i++) {
        CloseHandle(hThread[i]);
    }
    DeleteCriticalSection(&cs);  // 删除临界区

    return 0;
}
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
### 读写锁
代码
```
const unsigned int THREAD_NUM = 50;
unsigned int g_Count = 0;
SRWLOCK g_CountLock;

DWORD WINAPI ThreadFunc(LPVOID p) {
    // 睡眠一段时间以模拟工作
    Sleep(50);

    // 获取写锁
    AcquireSRWLockExclusive(&g_CountLock);
    g_Count++;
    // 释放写锁
    ReleaseSRWLockExclusive(&g_CountLock);

    // 睡眠一段时间以模拟工作
    Sleep(50);
    return 0;
}

int main() {
    HANDLE hThread[THREAD_NUM];

    // 初始化 SRWLOCK
    InitializeSRWLock(&g_CountLock);

    for (int i = 0; i < THREAD_NUM; i++) {
        hThread[i] = CreateThread(NULL, 0, ThreadFunc, NULL, 0, NULL);
    }

    WaitForMultipleObjects(THREAD_NUM, hThread, TRUE, INFINITE);

    // 输出 g_Count 的值
    std::cout << "Final g_Count: " << g_Count << std::endl;

    return 0;
}
```
实验结果
```
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
C:\Users\XDUAI\Desktop\windowsmpi>winapitest.exe
50
```
### 原理
读写锁保证了在同一时间内只有一个线程允许对变量进行读和写操作，保证数据一致性。

## 作业2Pthread实现多线程矩阵乘法(Mingw64)。
代码如下
```
#include<iostream>
#include<pthread.h>
#include<time.h>
#define thread_count 4
#define MAX 512
int a[MAX][MAX];
int b[MAX][MAX];
int result[MAX][MAX];
int resultsingle[MAX][MAX];
int bt[MAX][MAX];
typedef struct{
    int thread_id;
    int num_threads;
}thread_data_t;
void singlemultiply(int c[MAX][MAX],int d[MAX][MAX]){
    int output[MAX][MAX];
    for(int i=0;i<MAX;i++){
        for(int j=0;j<MAX;j++){
            for(int k=0;k<MAX;k++){
                resultsingle[i][j]+=c[i][k]*d[j][k];
            }
        }
    }
}
void* multiply(void* arg){
    thread_data_t* data=(thread_data_t*)arg;
    int thread_id=data->thread_id;
    int num_threads=data->num_threads;
    //int start_rows=(1000-1000%num_threads+num_threads)/num_threads*thread_id;
    //int rows=std::min((1000-1000%num_threads+num_threads)/num_threads,1000-start_rows);
    //int rows=(thread_id!=num_threads-1)?(100-100%num_threads+num_threads)/num_threads:100-start_rows;
    int start_rows = (MAX / num_threads) * thread_id;
    int end_rows = (thread_id == num_threads - 1) ? MAX : start_rows + (MAX / num_threads);
    for(int i=start_rows;i<end_rows;i++){
       for(int j=0;j<MAX;j++){
        for(int k=0;k<MAX;k++){
           result[i][j]+=a[i][k]*bt[j][k];
        }
       }
    }
    pthread_exit(NULL);
}
int main(){
    for(int i=0;i<MAX;i++){
    for(int j=0;j<MAX;j++){
        a[i][j]=rand();
        b[i][j]=rand();
    }
}
    for(int i=0;i<MAX;i++){
        for(int j=0;j<MAX;j++){
            bt[j][i]=b[i][j];
        }
    }
    clock_t start_time=clock();
    singlemultiply(a,bt);
    clock_t end_time=clock();
    double single_time_taken=((double)(end_time - start_time)) / CLOCKS_PER_SEC;
    pthread_t threads[thread_count];
    thread_data_t thread_data[thread_count];
    start_time = clock();
    for(int i=0;i<thread_count;i++){
        thread_data[i].thread_id=i;
        thread_data[i].num_threads=thread_count;
        pthread_create(&threads[i],NULL,multiply,(void*)&thread_data[i]);
    }
    for(int i=0;i<thread_count;i++){
        pthread_join(threads[i],NULL);
    }
    end_time = clock();
    double time_taken=((double)(end_time - start_time)) / CLOCKS_PER_SEC;
    double speedup = single_time_taken / time_taken;
    printf("use %d threads computetime:%.6fsec speedup:%.2f",thread_count,time_taken,speedup);
    //printf("use %d threads,computetime: %.6f second,speedeup: %.2f\n", thread_count, time_taken, speedup);
}
```
代码解释：
这里将输入的矩阵和最后的结果矩阵作为全局变量，线程函数的输入参数为一个元素为线程id和线程数量的结构体。因为只需要知道线程id和线程数量就能得知每个线程负责的行。同时为了减少cache miss（矩阵相乘右边的矩阵读取方式是按照列读取和二维数组在内存中的存储顺序相悖，缓存在从内存中预取数据时是按照行取的，所以在取b中的数据时由于取数据的顺去和缓存读取的顺序不同，大概率b中使用的数据在缓存中不存在，只能重新从内存中读取开销十分巨大），因此这里对b进行转置。在线程函数中对转置矩阵进行行读取，减少cache miss提高代码性能。
### 输出结果：（运行十次取平均值）
| NUM_THREADS|64*64|128*128|256*256|512*512|
|--------|--------|--------|-------|-------|
|2|1.5|1.76|1.76|1.97|
|4|<1|2.63|3.41|3.65|
|8|<1|3.61|3.80|4.61|
|16|<1|2.75|4.14|4.43|
|32|<1|3.28|5.06|5.38|
|64|<1|2.08|4.96|5.67|
* 结果分析：从不同大小的矩阵在不同线程下的加速比来看，加速比由于进程间调度以及缓存中cachemiss的影响，在固定任务中存在加速比上限，不能做到随着线程的增加做到无线增加。同时如果任务本身规模就十分的小，采取过多的线程对于计算速度的影响可能是反向的。此外在运行过程中在问题规模较小时我甚至发现在少数的几次输出结果中加速比大于县城数量。通过对代码的分析我认为可能是因为我首先运行了单线程矩阵相乘的函数，在运行```singlemultiply```后缓存中已经存在了矩阵```a,b```的元素，大大减少了后续多线程矩阵相乘的cache miss率。由于在问题规模较小时本身计算时间就很小，在这种情况下，缓存cachemiss降低带来的计算时间降低很可能接近多线程带来的收益。
## 作业3：OPENMPI helloworld
代码
```
#include <stdio.h>
#include <omp.h>

int main(int argc, char **argv) { 
    int nthreads, thread_id; 
    printf("I am the main thread.\n"); 

    #pragma omp parallel private(nthreads, thread_id)
    { 
        nthreads = omp_get_num_threads(); 
        thread_id = omp_get_thread_num(); 
        printf("Hello. I am thread %d out of a team of %d\n", thread_id, nthreads);
    } 

    printf("Here I am, back to the main thread.\n"); 
    return 0;
}
```
C:\Users\XDUAI\Desktop\openmptest>g++ -fopenmp helloworld.cpp -o hello

C:\Users\XDUAI\Desktop\openmptest>hello.exe
I am the main thread.
Hello. I am thread 0 out of a team of 12
Hello. I am thread 6 out of a team of 12
Hello. I am thread 4 out of a team of 12
Hello. I am thread 8 out of a team of 12
Hello. I am thread 11 out of a team of 12
Hello. I am thread 7 out of a team of 12
Hello. I am thread 1 out of a team of 12
Hello. I am thread 9 out of a team of 12
Hello. I am thread 5 out of a team of 12
Hello. I am thread 3 out of a team of 12
Hello. I am thread 2 out of a team of 12
Hello. I am thread 10 out of a team of 12
Here I am, back to the main thread.
```
在默认情况下创建了12个线程，这是因为我的cpu是12核的
设置线程数量为4
```
C:\Users\XDUAI\Desktop\openmptest>g++ -fopenmp helloworld.cpp -o hello 

C:\Users\XDUAI\Desktop\openmptest>hello.exe
I am the main thread.
Hello. I am thread 0 out of a team of 0
Hello. I am thread 1 out of a team of 4199854
Hello. I am thread 2 out of a team of 16016720
Hello. I am thread 3 out of a team of 4199854
Here I am, back to the main thread.
```


