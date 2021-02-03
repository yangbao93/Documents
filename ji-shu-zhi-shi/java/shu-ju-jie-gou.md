# 数据结构

数据结构一般和算法相关。算法就是用合适操作，合适的数据结构去处理数据，从而获得一个正确的结果。

1. 数组Array 1. 具有固定大小，在初始化的时候就会设置，且不可变； 2. 强类型，数据的数据类型在初始化的时候就指定，不需要额外时间来定义，可以提升效率； 3. 因为大小固定，类型固定，因此运行时占用很少的内存，效率高。foreach支持 4. 定义方法：
   * String\[\] str = new String\[5\]
   * String\[\] str = new String\[\]{"a","b"}
   * String\[\] str = {"a","b"}
2. ArrayList 1. 长度可变，动态增加； 2. 非强类型，运行时候消耗内存较多；
3. HashTable哈希表 1. key-value形式，并非强类型，也不固定大小限制
4. Stack栈 1. 入栈和出栈，遵循LIFO（先进后出）
5. Queue队列 1. 和栈一样，具有优先级定义的结构体，遵循FIFO（先进先出 ），非强类型，无固定大小
