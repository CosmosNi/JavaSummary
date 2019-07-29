# java8新特性

###### 1.1 lambda

常用函数接口：

- Predicate T->boolean 接收T类型的对象并返回boolean值

  - IntPredicate 接收Integer类型对象
  - LongPredicate 接收Long类型对象
  - DoublePredicate 接收Double类型对象

- Consumer T->void 接收T类型的对象并进行处理

  - IntConsumer 接收Integer类型对象
  - LongConsumer 接收Long类型对象
  - LongConsumer 接收Double类型对象

- Function<T,R> T->R 接收T类型对象，进行处理后变成R类型对象

  - IntFunction 接收Integer类型对象，进行处理后变成R类型对象
  - IntToDoubleFunction
  - IntToLongFunction
  - LongFunction
  - LongToDoubleFunction
  - LongToIntFunction
  - DoubleFunction
  - ToIntFunction 接收T类型对象，返回Integer
  - ToDoubleFunction
  - ToLongFunction

  - Supplier ()->T Supplier主要是用来创建对象的。可以将耗时操作放在get里，在程序中，传递是Supplier对象，只有真正调用get方法时才执行运算，这就是所谓的惰性求值。
    - BooleanSupplier
    - IntSupplier
    - LongSupplier
    - DoubleSupplier
  - UnaryOperator T->T 接收T类型对象，经过处理后返回T类型对象
    - IntUnaryOperator
    - LongUnaryOperator
    - DoubleUnaryOperator

- BinaryOperator (T,T)->T 对两个T类型对象进行处理返回T类型对象

  - IntBinaryOperator
  - LongBinaryOperator
  - DoubleBinaryOperator

- BiPredicate<L,R> (L,R)->boolean 对两个不同类型对象进行处理，返回boolean类型

- BiConsumer<T,U> (T,U)->void 对两个对象进行处理

  - ObjIntConsumer
  - ObjLongConsumer
  - objDoubleConsumer

- BiFunction<T,U,R> (T,U)->R 对T,U两个类型的对象进行处理并返回R类型数据

  - ToIntBiFunction<T,U>
  - ToLongBiFunction<T,U>
  - ToDoubleBiFunction<T,U>



###### 1.2 Stream

定义：从支持数据处理操作的源生成元素的序列。

详细定义：

1. 元素序列：就像集合一样，流也提供了一个接口，可以访问特定元素类型的一组有序值。集合讲的是数据，流讲的是计算。
2. 源：流会使用一个提供数据的源，如集合，数组或输入/输出资源。
3. 数据处理操作：流的数据处理功能类似于数据库的操作，以及函数式编程语言中的常用操作。流操作可以顺序执行，也可并行执行。

流操作特点：

1. 流水线：很多操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大的流水线。流水线的操作可以看做对数据源进行数据库式查询。
2. 内部迭代：与使用迭代显式迭代的集合不同，流的迭代操作是在背后进行的。
3. 流只能遍历一次。

#### [1.3.2.1 stream常用方法](https://cosmosni.gitee.io/cosmos-tutorial/#/?id=_1321-stream常用方法)

(数值类型的流：IntStream，LongStream)

1. map：转换流，将一种类型的流转换为另外一种流。
2. filter：过滤流，过滤流中的元素。
3. flapMap：拆解流，将流中每一个元素拆解成一个流。
4. sorted：对流进行排序。
5. limit:元素不能超过指定长度。短路操作。
6. distinct:去重。返回一个元素各异（根据流生成的hashcode和equal的方法实现）的的流。
7. skip(n):返回一个扔掉前n个元素的流。
8. flatMap:将一个流中的每个值都转换成另一个流，然后将所有的流链接起来成为一个流，见simpleStreamTest2。
9. anyMatch：流中是否有一个元素能匹配给定的谓词。
10. allMatch：类似anyMatch，查看流流中元素是否都能匹配给定的谓词。
11. noneMatch：allMatch相反。
12. findAny:返回当前元素的任意元素。
13. findFirst：查找第一个元素。
14. reduce：数值计算。