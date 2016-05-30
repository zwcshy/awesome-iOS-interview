# awesome-iOS-interview
iOS面试题总结：总结今天（2016.05.27）去美团面试的情况和以后面试需要准备的东西，包括基本知识点、算法。

### 1、什么情况下使用weak？相比assign有什么不同？assign、retain、copy的区别？

  什么情况使用 weak 关键字？
  
  > 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。

  不同点：
  > weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

  assigin 可以用非 OC 对象,而 weak 必须用于 OC 对象
  
  assign、retain、copy的区别：
  > assign：简单赋值，不更改索引计数。使用对象L基本数据类型(NSInteger)、C数据类型(int、float、char等);
  > retain：该属性在赋值的时候，先release之前的值，然后再赋新值给属性，引用+1;
  > copy：前一个值发送release消息，基本上像retain，但是它没有增加引用计数，是分配一块新的内存来放置它
    
  > 总结：copy是创建一个新对象，retain是创建一个指针！

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

### 4、以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？
  RunLoop只能运行在一种mode下，如果要换mode，当前的loop也需要停下重启成新的。利用这个机制，ScrollView滚动过程中NSDefaultRunLoopMode（kCFRunLoopDefaultMode）的mode会切换到UITrackingRunLoopMode来保证ScrollView的流畅滑动：只能在NSDefaultRunLoopMode模式下处理的事件会影响ScrollView的滑动。

  > 如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。

  > 同时因为mode还是可定制的，所以：

  > Timer计时会被scrollView的滑动影响的问题可以通过将timer添加到NSRunLoopCommonModes（kCFRunLoopCommonModes）来解决。代码如下：
  
    //将timer添加到NSDefaultRunLoopMode中
    [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timerTick:) userInfo:nil repeats:YES];
    
    //然后再添加到NSRunLoopCommonModes里
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timerTick:) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

### 5、BAD_ACCESS在什么情况下出现？
  > 访问了野指针，比如对一个已经释放的对象执行了release、访问已经释放对象的成员变量或者发消息。 死循环
  
### 6、如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？
  > 若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。
  
  具体步骤：
    1、需声明该类遵从 NSCopying 协议
    
    2、实现 NSCopying 协议。该协议只有一个方法:
    
     - (id)copyWithZone:(NSZone *)zone;

### 7、UITableView的优化主要从三个方面入手：

> ① 提前计算并缓存好高度（布局），因为heightForRowAtIndexPath:是调用最频繁的方法；

> ② 异步绘制，遇到复杂界面，遇到性能瓶颈时，可能就是突破口；

> ③ 滑动时按需加载，这个在大量图片展示，网络加载的时候很管用！（SDWebImage已经实现异步加载，配合这条性能杠杠的）。

除了上面最主要的三个方面外，还有很多几乎大伙都很熟知的优化点：

> ④ 正确使用reuseIdentifier来重用Cells

> ⑤ 尽量使所有的view opaque，包括Cell自身

> ⑥ 尽量少用或不用透明图层

> ⑦ 如果Cell内现实的内容来自web，使用异步加载，缓存请求结果

> ⑧ 减少subviews的数量

> ⑨ 在heightForRowAtIndexPath:中尽量不使用cellForRowAtIndexPath:，如果你需要用到它，只用一次然后缓存结果

> ⑩ 尽量少用addView给Cell动态添加View，可以初始化时就添加，然后通过hide来控制是否显示

## 内存管理
### 1.什么是ARC？
  > ARC是automatic reference counting自动引用计数，在程序编译时自动加入retain/release。在对象被创建时retain count+1，在对象被release时count-1，当count=0时，销毁对象。程序中加入autoreleasepool对象会由系统自动加上autorelease方法，如果该对象引用计数为0，则销毁。那么ARC是为了解决MRC手动管理内存存在的一些而诞生的。
  
  MRC下内存管理的缺点：
  
    1)释放一个堆内存时，首先要确定指向这个堆空间的指针都被release了。(避免提前释放)
    2)释放指针指向的堆空间，首先要确定哪些指向同一个堆，这些指针只能释放一次。(避免释放多次，造成内存泄露)
    3)模块化操作时，对象可能被多个模块创建和使用，不能确定最后由谁释放
    4)多线程操作时，不确定哪个线程最后使用完毕。
    
  虽然ARC给我们编程带来的很多好多，但也可能出现内存泄露。如下面两种情况：
  
    1)循环参照：A有个属性参照B，B有个属性参照A，如果都是strong参照的话，两个对象都无法释放。
    2)死循环：如果有个ViewController中有无限循环，也会导致即使ViewController对应的view消失了，ViewController也不能释放。
    
### 2.block一般用那个关键字修饰，为什么？
  > block一般使用copy关键之进行修饰，block使用copy是从MRC遗留下来的“传统”，在MRC中，方法内容的block是在栈区的，使用copy可以把它放到堆区。但在ARC中写不写都行：编译器自动对block进行了copy操作。
  
### 3.用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？
  > 用@property声明 NSString、NSArray、NSDictionary 经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。
  
  > 如果我们使用是strong,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性。
  
  > copy此特质所表达的所属关系与strong类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。 当属性类型为NSString时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个NSMutableString类的实例。这个类是NSString的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。

### 4.runloop、autorelease pool以及线程之间的关系。
  > 每个线程(包含主线程)都有一个Runloop。对于每一个Runloop，系统会隐式创建一个Autorelease pool，这样所有的release pool会构成一个像callstack一样的一个栈式结构，在每一个Runloop结束时，当前栈顶的Autorelease pool会被销毁，这样这个pool里的每个Object会被release。
  
### 5.@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的。
  > “属性”(property)有两大概念：ivar(实例变量)、存取方法(access method=getter)，即@property = ivar + getter + setter。
  
  > 例如下面的这个类：
  > @interface WBTextView :UITextView  
  >   @property (nonatomic,copy)NSString *placehold;  
  >   @property (nonatomic,copy)UIColor *placeholdColor;  
  > @end
  
  > 类完成属性的定以后，编译器会自动编写访问这些属性的方法(自动合成autosynthesis)，上述代码写出来的类等效与下面的代码：
  > @interface WBTextView :UITextView  
  >  - (NSString *)placehold;  
  >  -(void)setPlacehold:(NSString *)placehold;  
  >  -(UIColor *)placeholdColor;  
  >  -(void)setPlaceholdColor:(UIColor *)placeholdColor;  
  > @end
