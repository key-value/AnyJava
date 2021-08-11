
# Array 和 ArrayList 有何区别？
1.  Array 可以容纳基本类型和对象，而 ArrayList 只能容纳对象；
2.  Array 是指定大小的，而 ArrayList 大小是不固定的。

# 扩容机制
1.  当使用 add 方法的时候首先调用 ensureCapacityInternal 方法，传入 size+1 进去，检查是否需要扩充 elementData 数组的大小；
2.  newCapacity = 扩充数组为原来的 1.5 倍(不能自定义)，如果还不够，就使用它指定要扩充的大小 minCapacity ，然后判断 minCapacity 是否大于 MAX_ARRAY_SIZE(Integer.MAX_VALUE – 8) ，如果大于，就取 Integer.MAX_VALUE；
3.  扩容的主要方法：grow；
4.  ArrayList 中 copy 数组的核心就是 System.arraycopy 方法，将 original 数组的所有数据复制到 copy 数组中，这是一个本地方法。