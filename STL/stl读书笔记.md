# 第二章空间适配器

## alloc的构建与销毁，构建使用operator new 申请空间，placement new在起始内存地址调用构造函数`new(p) ClassName(1)`;alloc销毁对于不包含指针数据这种析构无关紧要的类型一次删除，其他的类型遍历来从头到尾调用析构函数再删除，类型区分使用萃取机。

## 第一级配置器使用malloc， free，realloc申请释放空间，如果申请失败会调用set_malloc_handler，循环重新申请，realloc也是

## 第二级配置器申请一个内存池，有一个16大小的vector分别储存8，16，24，...，128字节的16个链表

