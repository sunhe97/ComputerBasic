# STL

# 第二章空间适配器

## alloc的构建与销毁，构建使用operator new 申请空间，placement new在起始内存地址调用构造函数`new(p) ClassName(1)`;alloc销毁对于不包含指针数据这种析构无关紧要的类型一次删除，其他的类型遍历来从头到尾调用析构函数再删除，类型区分使用萃取机。

## 第一级配置器使用malloc， free，realloc申请释放空间，如果申请失败会调用set_malloc_handler，循环重新申请，realloc也是

## 第二级配置器申请一个内存池，有一个16大小的vector分别储存8，16，24，...，128字节的16个链表,第二级配置器用union作为节点，不分配时用于指向下一节点，分配出去用于储存数据。第二级配置器在freelist中没有相应空闲内存，会向内存池申请20个size的内存，如果不够申请相应个数，如果内存池连一个都不够，就malloc重新申请内存，如果堆满了，在大小大于等于size的一个节点申请空余空间看能分配几个size大小的块，最后不行调用一级配置器。



# c++

## `enum Level {leveleror = 0, levelgood};  Level m = levelerrpr; Level is a integar` 对于类中的enum，可以用类名加enum元素传值，class log， log.set(log::Error);所以enum中的名字不能与类中变量，函数重名。
