# awesome-interview
iOS面试题总结：总结今天去美团面试的情况和以后面试需要准备的东西，包括基本知识点、算法。

## 1、什么情况下使用weak？相比assign有什么不同？

  什么情况使用 weak 关键字？
  
在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。

不同点：

weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

assigin 可以用非 OC 对象,而 weak 必须用于 OC 对象


