# c-learning
my note of c++
# c++ 编译和链接
在 c++中代码的运行分为两个阶段，编译和链接以下为例  
在`log.py`中实现一个简单的函数如下  
void log(const char* message){
std::cout<<message<<std::endl}  
在`main.py`中对其进行声明  
`#include<iostream>  using namespace std  void log(const char *message)  int main(){const char* message="xjy"  log(message)}`
