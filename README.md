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
  
  > 2、使用了 atomic 属性会严重影响性能 ；
    
    比如下面的代码就会发生崩溃:
    
      // .h文件
      // 下面的代码就会发生崩溃
        @property (nonatomic, copy) NSMutableArray *mutableArray;
      
      // .m文件
      // 下面的代码就会发生崩溃
        NSMutableArray *array = [NSMutableArray arrayWithObjects:@1,@2,nil];
        self.mutableArray = array;
        [self.mutableArray removeObjectAtIndex:0];
      
    接下来就会奔溃：
      
          -[__NSArrayI removeObjectAtIndex:]: unrecognized selector sent to instance 0x7fcd1bc30460
      
  第2条原因，如下：
  
  > 该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明 nonatomic 可以节省这些虽然很小但是不必要额外开销。
  
  在默认情况下，由编译器所合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用同步锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的”(atomic))。

  在iOS开发中，你会发现，几乎所有属性都声明为 nonatomic。

  一般情况下并不要求属性必须是“原子的”，因为这并不能保证“线程安全” ( thread safety)，若要实现“线程安全”的操作，还需采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为 atomic，也还是会读到不同的属性值。

  因此，开发iOS程序时一般都会使用 nonatomic 属性。但是在开发 Mac OS X 程序时，使用 atomic 属性通常都不会有性能瓶颈。

