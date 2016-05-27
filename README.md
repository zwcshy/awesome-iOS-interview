# awesome-interview
iOS面试题总结：总结今天去美团面试的情况和以后面试需要准备的东西，包括基本知识点、算法。

### 1、什么情况下使用weak？相比assign有什么不同？

  什么情况使用 weak 关键字？
  
  > 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。

  不同点：
  > weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

  assigin 可以用非 OC 对象,而 weak 必须用于 OC 对象

### 2、怎么用 copy 关键字？

  用途：
  
  > 1) NSString、NSArray、NSDictionary 等等经常使用copy关键字:
    是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。
    
  > 2) block 也经常使用 copy 关键字:
    block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作。如果不写 copy ，该类的调用者有可能会忘记或者根本不知道“编译器会自动对 block 进行了 copy 操作”，他们有可能会在调用之前自行拷贝属性值。这种操作多余而低效。
    
### 3、这个写法会出什么问题: @property (copy) NSMutableArray *array;

  两个问题：
  
  > 1、添加,删除,修改数组内的元素的时候,程序会因为找不到对应的方法而崩溃.因为 copy 就是复制一个不可变 NSArray 的对象；
    2、使用了 atomic 属性会严重影响性能 ；
    
    比如下面的代码就会发生崩溃:
    
        // .h文件
        // http://weibo.com/luohanchenyilong/
        // https://github.com/ChenYilong
        // 下面的代码就会发生崩溃
        @property (nonatomic, copy) NSMutableArray *mutableArray;
