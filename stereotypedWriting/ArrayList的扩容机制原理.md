### 初始参数

 从下面的代码中可以看到，初始容量为10

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

### 扩容代码解读

扩容我们可以从add方法入手查看

```java
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

可以看到，添加元素的时候，会调用 `ensureCapacityInternal`方法。这个方法名的意思为确认内部容量，顾名思义，就是判断当前容量能否满足使用需求。我们进入该方法内部

```java
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
   }
  private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
 private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
  }
```

我们可以看到，嵌套进行了调用，`calculateCapacity` 计算容量的方法会在初始化为空的时候与默认容量比较进行比较，返回较大数值。其他情况都会返回当前size+1

在`minCapacity - elementData.length > 0`条件下，调用扩容方法。即当目前size大于当前容量时扩容。

```java
   private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

我们可以从 `grow` 方法中可以看到，新的容量（newCapacity）使用了旧容量+ 旧容量大小左移1位。 

> \>>"（移位运算符）：>>1 右移一位相当于除 2，右移 n 位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了 1 位所以相当于 oldCapacity /2。对于大数据的 2 进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源。

旧容量为偶数的情况下，扩容后为1.5倍。为奇数的情况下，由于会舍弃掉小数位。向下取整。

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```

我们可以看出来当新容量大于 MAX_ARRAY_SIZE 时，会取用Integer.MAX_VALUE 作为容量。

### 总结

- ArrayList扩容发生在add()方法调用的时候， 调用ensureCapacityInternal()来扩容的，通过方法calculateCapacity(elementData, minCapacity)获取需要扩容的长度
- ensureExplicitCapacity方法可以判断是否需要扩容
- ArrayList扩容的关键方法grow():  获取到ArrayList中elementData数组的内存空间长度 扩容至原来的1.5倍
- 调用Arrays.copyOf方法将elementData数组指向新的内存空间时newCapacity的连续空间 从此方法中我们可以清晰的看出其实ArrayList扩容的本质就是计算出新的扩容数组的size后实例化，并将原有数组内容复制到新数组中去。