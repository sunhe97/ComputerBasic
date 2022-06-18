# GamePlay

## Actor与Component

### UObject

+ 元数据
+ 反射生成
+ GC
+ 序列化
+ 编辑器可见
+ Class Default Object

### Actor

+ Replication（网络复制）
+ Spawn
+ Tick

#### actor加component实现不同的actor，component继承于UObject，actor有一个rootcomponent作为actor的总transform，childactorcomponent用来实现，子actor与父actor之间属性的相互设置，actorcomponent与actorcomponent不能嵌套，scenecomponent之间可以嵌套。AttachToActor和AttachToComponent，这两个方法确定了Actor的SceneComponent之间的父子关系，是通过SceneComponent中的AttachChildren数组存储的，Actor中的Children数组是通过SetOwner方法修改的



# 反射系统

## 工具生成代码实现反射

## 过程

+ UHT解析代码
+ 