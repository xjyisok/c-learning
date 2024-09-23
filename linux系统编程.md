# 软连接硬链接
## 软连接
```
ln -s /path/to/targetfile/ softlinkname
```
## 硬链接  
```
ln file.h file
```
* linux下软连接指向的是文件的路径，而不是文件的具体内容，依赖目标文件的存在
* 由于是软连接指向的是文件的路径，所以如果在软连接创建时如果使用的是相对路径那么无法在其他目录下访问这个软连接例如
  在filedir文件夹下有file文件
```
cd filedir
ln -s file file.s
mv file.s home/filedir1
cd ~
cd filedir1
cat file.s
```
* 这时会出问题因为file.s是相对目录下创建的，要想在其他目录下也访问，得在创建阶段就将```path/to/targetfile/```设置为绝对路径  

* 硬链接与原始文件指向相同的数据块，和原始文件共享相同的Inode。硬链接并非为原始文件创建了一个副本，只是创建了另外一个指向原始文件数据块的路径  
* 删除一个硬链接时并不会对其余硬链接有影响，即使删除源文件也可以通过硬链接访问原文件数据块  
* 只有当所有指向该 inode 的链接都被删除后，文件系统才会真正释放该文件的数据。
## xargs 和 exec区别
### exec
```
find ./ -type -f -exec ls{} -ld\;
```
### xargs  
```
find ./ -name '*amns*'| xargs ls -ld
```
相比于exec xargs依赖管道可以分片处理，当搜索出来的文件过多时不会一次性全部执行，而是分片执行。
* 但是xargs按照空格划分如果文件名中带空格格会划分失败，这时候就需要添加-print0更改划分方式
```
find ./ -name '*amns*' -print0| xargs -print0 ls -ld 
```
