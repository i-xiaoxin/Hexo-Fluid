---
title: java集合体系
date: 2022-09-21 18:12:02
tags: java
index_img: /img/article1.jpg
banner_img: /img/post_banner.jpg
---

### 数组存在问题:

- 数组长度固定

- 存入的类型只能是一种(数据单一)

- 数组是线性结构，增删效率低

假设现在有一个情景需求，统计一个学校的学生,如果使用数组则需要固定长度,而一旦固定了之后,加入学校开除学生了则不能减少数组长度,同样如果学校中途又来了学生也没办法添加;所以数组是无法解决这个问题的,因为学生的统计是不固定的,有可能会增加也有可能会减少；所以出现了集合

集合的出现就是为了解决数组的问题,所以能够确定一点就是集合更加的强大;**集合存储数据必须是引用数据类型**

### 集合体系

> Collection接口体系

![Collection接口体系](https://raw.githubusercontent.com/i-xiaoxin/image/master/09.png)

> Map接口体系

![Map接口体系](https://raw.githubusercontent.com/i-xiaoxin/image/master/10.png)

### Collection接口

- `boolean add(E e)`:向集合中添加元素

- `boolean addAll(Collection<? extends E> c)`:向集合中添加一个集合元素

- `boolean contains(Object o)`:检测集合中是否包含某个元素

- `boolean containsAll(Collection<?> c)`:检测集合中是否包含另一个集合中的元素

- `boolean remove(Object o)`:从集合中移除指定数据(对象)

- `boolean removeAll(Collection<?> c)`:移除此集合的所有也包含在指定集合中的元素

- `boolean retainAll(Collection<?> c)`:移除刺激和的所有不包含在指定集合中的元素(移除非相同的元素,非交集元素)

- `int size()`:返回集合中的元素数量

- `boolean isEmpty()`:判断集合是否为空

- `Object[] toArray()`:将集合转为数组

- `void clear()`:清空集合内元素

- `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素

    代码案例：

    ```java
    public class CollectionDemo {
        public static void main(String[] args) {
            // 由于Collection是接口所以无法直接创建对象,所以使用其子类创建(ArrayList)
            // 以多态方式创建Collection接口集合,并且只能使用Collection接口中的方法
            Collection collection = new ArrayList();
            // 1. `boolean add(E e)`:向集合中添加元素
            collection.add("彭于晏");
            collection.add(18);
            collection.add(185.2);
            collection.add("男");
            System.out.println(collection);
            // 2. `boolean addAll(Collection<? extends E> c)`:向集合中添加一个集合元素
            Collection collection1 = new ArrayList();
            collection.add("吴彦祖");
            collection.add(19);
            collection.add(182.2);
            collection.add("男");
            collection.addAll(collection1);
            System.out.println(collection);
            // 3. `boolean contains(Object o)`:检测集合中是否包含某个元素
            boolean isContains = collection.contains("吴彦祖");
            System.out.println(isContains);
            isContains = collection.contains("吴彦祖1");
            System.out.println(isContains);
            // 4. `boolean containsAll(Collection<?> c)`:检测集合中是否包含另一个集合中的元素
            boolean isContainsAll = collection.containsAll(collection1);
            System.out.println(isContainsAll);
            // collection1中再添加一条数据
            collection1.add("帅");
            isContainsAll = collection.containsAll(collection1);
            System.out.println(isContainsAll);
            // 5.  `boolean remove(Object o)`:从集合中移除指定数据(对象)
            boolean isRemove = collection.remove("帅");
            System.out.println(isRemove);
            System.out.println(collection);
            isRemove = collection.remove("男");
            System.out.println(isRemove);
            System.out.println(collection);
            // 6. `boolean removeAll(Collection<?> c)`:移除此集合的所有也包含在指定集合中的元素
            // 7. `boolean retainAll(Collection<?> c)`:移除刺激和的所有不包含在指定集合中的元素(移除非相同的元素,非交集元素)
            // 8. `int size()`:返回集合中的元素数量
            System.out.println(collection.size());
            // 9. `boolean isEmpty()`:判断集合是否为空
            System.out.println(collection.isEmpty());
            // 10. `Object[] toArray()`:将集合转为数组
            // 11. `void clear()`:清空集合内元素
            collection.clear();
            System.out.println(collection);
            System.out.println(collection.isEmpty());
            // 12. `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素
        }
    }
    ```

###  List接口

 - `boolean add(E e)`:向List集合中添加元素`常用`
- `void add(int index, E element)`:向List集合中指定位置添加元素
- `boolean addAll(Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的末尾
- `boolean addAll(int index, Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的指定位置
- `boolean contains(Object o)`:检测此集合中是否包含指定元素`常用`
- `boolean containsAll(Collection<?> c)`:检测集合中是否包含另一个集合中的元素
- `E get(int index)`:获取集合中指定下标索引的元素并返回`常用`
- `E set(int index, E element)`:修改集合中指定下标索引的元素
- `int indexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回第一次发现的下标索引)
- `int lastIndexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回最后一次发现的下标索引)
- `List<E> subList(int fromIndex, int toIndex)`:切割集合,指定起始位置和结束位置
- `ListIterator<E> listIterator()`:返回List接口迭代器
- `int size()`:返回集合中的元素数量`常用`
- `boolean isEmpty()`:判断集合是否为空`常用`
- `Object[] toArray()`:将集合转为数组
- `void clear()`:清空集合内元素
- `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素

代码案例：

```java
public class ListDemo {
    public static void main(String[] args) {
        // 使用多态方式创建List接口对象-->使用其子类ArrayList
        List list = new ArrayList();
        // 1. `boolean add(E e)`:向List集合中添加元素`常用`
        list.add("彭于晏");
        list.add(18);
        list.add("男神");
        System.out.println(list);
        // 2. `void add(int index, E element)`:向List集合中指定位置添加元素
        list.add(2, 185.3);
        System.out.println(list);
        // 3. `boolean addAll(Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的末尾
        // 4. `boolean addAll(int index, Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的指定位置
        // 5. `boolean contains(Object o)`:检测此集合中是否包含指定元素`常用`
        boolean isContains = list.contains("彭于晏");
        System.out.println(isContains);
        isContains = list.contains("彭于晏1");
        System.out.println(isContains);
        // 6. `boolean containsAll(Collection<?> c)`:检测集合中是否包含另一个集合中的元素
        // 7. `E get(int index)`:获取集合中指定下标索引的元素并返回`常用`
        Object o = list.get(0);
        System.out.println(o);
        // 如果调用超出范围则报异常-->IndexOutOfBoundsException: Index: 100, Size: 4
        // Object o1 = list.get(100);
        // System.out.println(o1);
        // 8. `E set(int index, E element)`:修改集合中指定下标索引的元素
        // 返回的结果是修改前的内容
        Object set = list.set(2, 185.0);
        System.out.println(set);
        System.out.println(list);
        // 9. `int indexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回第一次发现的下标索引)
        // 在集合中多添加一些数据
        list.add("男");
        list.add("男神");
        System.out.println(list);
        int index = list.indexOf("男神");
        System.out.println("返回下标:" + index);
        // 如果不存在则返回-1
        index = list.indexOf("男神123");
        System.out.println("返回下标:" + index);
        // 10. `int lastIndexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回最后一次发现的下标索引)
        index = list.lastIndexOf("男神");
        System.out.println("返回下标:" + index);
        // 如果不存在则返回-1
        index = list.lastIndexOf("男神123");
        System.out.println("返回下标:" + index);
        // 11. `List<E> subList(int fromIndex, int toIndex)`:切割集合,指定起始位置和结束位置
        List subList = list.subList(0, 2);
        System.out.println(subList);
        // 12. `ListIterator<E> listIterator()`:返回List接口迭代器
        // 13. `int size()`:返回集合中的元素数量`常用`
        // 14. `boolean isEmpty()`:判断集合是否为空`常用`
        // 15. `Object[] toArray()`:将集合转为数组
        // 16. `void clear()`:清空集合内元素
        // 17. `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素


        System.out.println("========================================");
        // 使用迭代器
        // `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素
        Iterator iterator = list.iterator();
        // 迭代器方法boolean hasNext();-->判断iterator中是否还有可迭代的元素
        // 迭代器方法E next();-->输出迭代器中的元素-->类似于指针每次调用都会向后移
        // next如果超出范围则报错-->NoSuchElementException
        // 将next指针指向下一个元素
        /*Object next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);
        // 将next指针指向下一个元素
        next = iterator.next();
        System.out.println(next);*/
        // iterator.hasNext()验证是否有下一条数据
        /*boolean b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);
        iterator.next();
        b = iterator.hasNext();
        System.out.println(b);*/
        // 使用while循环
        // iterator.hasNext()判断是否存在元素
        // list.clear();
        // iterator = list.iterator();
        while (iterator.hasNext()) {
            // 将next指针指向下一条数据
            Object next = iterator.next();
            // 输出结果
            System.out.println(next);
        }
        System.out.println("========================================");
        // for循环遍历与上面的iterator迭代器相同都是遍历
        for (Object o1 : list) {
            System.out.println(o1);
        }
        System.out.println("========================================");
        // `ListIterator<E> listIterator()`:返回List接口迭代器
        ListIterator listIterator = list.listIterator();
        while (listIterator.hasNext()) {
            Object next = listIterator.next();
            System.out.println(next);
        }
    }
}
```

### List接口之实现类ArrayList

#### ArrayList概述

> ArrayList-->仅从名字看是数组集合-->ArrayList其底层是数组实现的,但是通过某些手段实现了可变长度的数组(动态数组),其容量可以自动增长动态增加容量
> 官方解释:
>
> List接口的可调整大小的数组实现(ArrayList是List接口的实现类,并且是动态数组大小)。实现了List接口的所有操作，并允许添加所有元素，包括null(但不建议添加null) 。除了实现List接口之外，该类还提供了一些自己的方法用来实现动态数组大小的方法;
>
> `(注意:ArrayList类与Vector类的方法相同,不同之处在于ArrayList是线程不安全,而Vector是线程安全)`

#### ArrayList中的底层实现源码

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    // ArrayList底层实现是使用数组实现的
    // transient-->(瞬时/瞬态/短暂的)-->如果通过网络传输此对象或向文件中写入此对象则不保留的字段(此字段不传输的内容)
    // 通过transient关键字修饰的只能在Java中使用,不能传输(例如写入文件和网络传输)
    transient Object[] elementData;
}
```

#### ArrayList添加方法

```java
// ArrayList集合的元素数量,与数组无关,因为是成员变量所以默认是0
private int size;
// 向集合末尾添加元素
public boolean add(E e) {
    // 判断当前集合的大小,size是当前添加的元素数量,如果没有添加则默认是0
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
// 此方法是add添加时调用的,用于判断集合空间的大小
private void ensureCapacityInternal(int minCapacity) {
    // 确保显式容量,并且参数是计算容量calculateCapacity()方法,用于计算当前容量
    // calculateCapacity()中的参数elementData是数组,minCapacity传入的最小容量
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
// 此方法是ensureExplicitCapacity中内部调用的,作用是计算容器的容量大小
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 使用传入的数组容器与默认数组容器使用==判断,判断是否相同的内存地址
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 返回Math数学类中的max方法计算出的大小
        // 如果DEFAULT_CAPACITY大则返回DEFAULT_CAPACITY
        // 如果minCapacity大则返回minCapacity
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 直接返回最小空间
    return minCapacity;
}
// 确保内部容量大小,minCapacity空间是从-->calculateCapacity方法计算而来
private void ensureExplicitCapacity(int minCapacity) {
    // 集合修改的次数(不用管)
    modCount++;
    // 使用传入的最小空间minCapacity减去数组长度elementData.length,如果大于0则空间溢出,
    if (minCapacity - elementData.length > 0)
        // 如果没有空间则进入继续计算,设置elementData数组的空间
        // 调用grow方法增加数组长度
        grow(minCapacity);
}
// 动态增加数组长度-->数组长度不足时会调用的方法
private void grow(int minCapacity) {
    // 第一步将数组长度赋值给old空间(oldCapacity)
    int oldCapacity = elementData.length;
    // 设置新空间(newCapacity) = oldCapacity(老空间)+(oldCapacity>>1)老空间右移一位,代表空间除2
    // 为什么计算空间时要加这么大的空间,原因是你都调用了并且能加到这个数值代表你是常用,所以不可能一直给你默认一个数值
    // 新数组空间的计算结果其实就是原空间的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 新空间减去传入的最小空间如果小于0则代表计算出来空间太小
    if (newCapacity - minCapacity < 0)
        // 如果小了则将最小空间重新赋值给新空间
        newCapacity = minCapacity;
    // 如果新空间减去最大空间(MAX_ARRAY_SIZE=2147483639)
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 如果大于则将新空间重新计算一个数值并赋值
        newCapacity = hugeCapacity(minCapacity);
    // 最终的扩容,将原数组与新空间通过Arrays.copyOf()方法拷贝至当前数组中,完成了数组的扩容
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/11.png)

#### ArrayList获取方法

``` java
// 获取下标索引元素方法
public E get(int index) {
    // 验证输入的index是否超出范围,超出则抛异常
    rangeCheck(index);
    // 返回index小表索引的元素
    return elementData(index);
}
// 获取index下标索引元素的方法
E elementData(int index) {
    // 返回数组当前index下标元素;例如:String[]strs = {"a","b","c"}-->strs[1]
    return (E) elementData[index];
}
```

#### ArrayList修改方法

```java
// 修改方法
public E set(int index, E element) {
    // 验证输入的index是否超出范围,超出则抛异常
    rangeCheck(index);
    // 获取当前数组index下标索引的元素
    E oldValue = elementData(index);
    // 将当前index下标索引数据修改为传入的数据
    elementData[index] = element;
    // 返回修改前的内容
    return oldValue;
}
```

#### ArrayList使用代码案例

```java
public class ArrayListDemo {
    public static void main(String[] args) {
        // 使用ArrayList集合,注意通常使用ArrayList都是使用多态方式创建既:List list=new ArrayList()这种方式
        ArrayList arrayList = new ArrayList();// 默认
        // 追加数据-->会扩容到10大小的空间
        arrayList.add("小明");
        System.out.println(arrayList);
        // 插入数据
        arrayList.add(0, "彭于晏");
        System.out.println(arrayList);
        // 修改数据
        arrayList.set(1, "吴彦祖");
        System.out.println(arrayList);
        // 移除数据
        arrayList.remove("吴彦祖");
        System.out.println(arrayList);
        // 获取数据
        Object o = arrayList.get(0);
        System.out.println(o);
    }
}
```

#### 重写实现ArrayList部分功能

```java
/**
 * 自定义实现ArrayList部分方法
 */
public class ArrayList {
    // 使用数组当做集合的底层存放
    private Object[] elementData;

    // 用于记录集合中元素数量的变量
    private int size;

    /**
     * 默认初始化数组空间即可
     */
    public ArrayList() {
        // 创建数组并分配默认空间-->默认10即可
        this.elementData = new Object[10];
    }

    /**
     * 创建自定义容量大小的构造方法
     *
     * @param initCapacity 初始化容量
     */
    public ArrayList(int initCapacity) {
        // 创建数组并分配默认空间-->默认10即可
        this.elementData = new Object[initCapacity];
    }

    /**
     * 添加元素方法
     *
     * @param element 传入需要添加的元素
     * @return 返回是否添加成功
     */
    public boolean add(Object element) {
        // 判断容量是否需要扩容
        ensureCapacityInternal();
        // 将数据添加至数组中-->其中size代表当前集合中的数据有多少
        this.elementData[size++] = element;
        return true;
    }

    private void ensureCapacityInternal() {
        // 使用size与数组长度对比,如果size大了则代表需要扩容
        /*if (size >= this.elementData.length) {
            // 将数组长度获取并赋值
            int oldLength = this.elementData.length;
            // 创建新数组并指定空间大小
            Object[] newElementData = new Object[oldLength + (oldLength >> 1) + 1];
            // 使用循环遍历方式将数据转义至新数组中-->带着大家使用原始方式来一遍
            // 根据老数组的长度计算循环
            for (int i = 0; i < this.elementData.length; i++) {
                // 循环将原始数组数据复制给新数组
                newElementData[i] = this.elementData[i];
            }
            // 重新赋值后再将this.elementData重新赋值指向新数组内存地址
            this.elementData = newElementData;
        }*/
        // 模仿官方写法
        if (size >= this.elementData.length) {
            // 将数组长度获取并赋值
            int oldCapacity = this.elementData.length;
            // 计算新空间大小
            int newCapacity = oldCapacity + (oldCapacity >> 1) + 1;
            // 使用数组工具类拷贝数组
            this.elementData = Arrays.copyOf(this.elementData, newCapacity);
        }
    }

    /**
     * 获取集合中的元素
     *
     * @param index 传入下标索引
     * @return 返回当前下标元素
     */
    public Object get(int index) {
        // 验证传入的index是否超出索引范围
        rangeCheck(index);
        // 返回数据
        return this.elementData[index];
    }

    /**
     * 检查传入的index是否超出范围
     *
     * @param index 传入的下标索引
     */
    private void rangeCheck(int index) {
        // 判断index是否小于0,以及是否大于size
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("数组下标索引超出范围:" + index);
        }
    }

    /**
     * 修改元素
     *
     * @param index   传入需要修改的下标索引
     * @param element 传入修改的数据
     * @return 返回修改前的数据
     */
    public Object set(int index, Object element) {
        // 验证下标是否合法
        rangeCheck(index);
        // 将老数据提取
        Object oldElementData = this.elementData[index];
        // 修改index下标索引的数据
        this.elementData[index] = element;
        return oldElementData;
    }

    /**
     * toString方法用于打印数据
     *
     * @return 返回的是数组中的元素数据
     */
    @Override
    public String toString() {
        return "ArrayList{" +
                "elementData=" + Arrays.toString(elementData) +
                '}';
    }
}
// 测试方法
public class MyArrayListDemo {
    public static void main(String[] args) {
        // 创建自己写的ArrayList
        ArrayList list = new ArrayList();
        // 添加数据
        for (int i = 0; i < 50; i++) {
            list.add("添加数据测试" + i);
        }
        // 打印结果
        System.out.println(list);
        // 获取数据
        System.out.println(list.get(1));
        System.out.println(list.get(3));
        System.out.println(list.get(5));
        System.out.println(list.get(7));
        // System.out.println(list.get(-1));
        System.out.println("============================================");
        // 修改数据
        Object oldElementData = list.set(1, "彭于晏");
        System.out.println(oldElementData);
        System.out.println(list);
    }
}
```

#### ArrayList集合中使用泛型约束添加数据的类型

```java
public class ListGenericsDemo {
    public static void main(String[] args) {
        // 集合中的泛型
        // 在未使用泛型时向集合添加数据,由于数据的不统一,造成无法准确调用集合中元素的私有方法
        List list = new ArrayList();
        // 添加数据
        list.add(new Person("彭于晏1", "男", 18));
        list.add(new Person("彭于晏2", "男", 19));
        list.add(new Person("彭于晏3", "男", 20));
        // 添加其他数据
        list.add("String类型数据");
        list.add("String类型数据");
        list.add("String类型数据");
        // 取出数据
        Person person = (Person) list.get(2);
        System.out.println(person.getName());
        // 获取下标为3的数据,下标为3的是String类型,转换对象时不属于同一个类型,会出现类型转换异常
        // Person person1 = (Person) list.get(3);
        // System.out.println(person1.getName());
        // 不转
        // Object o = list.get(0);
        // 只能调用Object类的方法

        System.out.println("===============================================");
        // 所以集合中有一个泛型的概念,泛型的作用就是用于约束类中的参数(参数可以代表:属性,形参,实参)传输-->泛型用于约束/规则
        // 使用泛型的集合
        // 格式例如:List<泛型类型(任意类型)> list = new ArrayList<>();
        // 定义一个List集合,泛型类型为Person,代表着只能向集合中添加Person对象数据
        List<Person> persons = new ArrayList<>();
        // 添加数据,如果添加的数据不是泛型类型则会出现编译异常
        // Required type:Person-->需要类型是Person
        // Provided:String-->但是提供的是String类型
        // persons.add("String类型数据");
        // 添加准确数据
        persons.add(new Person("彭于晏1", "男", 18));
        persons.add(new Person("彭于晏2", "男", 19));
        persons.add(new Person("彭于晏3", "男", 20));
        // 获取数据
        Person person2 = persons.get(0);
        System.out.println(person2);
        System.out.println(person2.getName() + "-" + person2.getSex());
    }
}
```

**注意:**集合中使用泛型的目的是为了约束添加数据时的类型-->统一数据类型,统一数据类型的目的使用使用时不用强转,同样也不会因为强转而发生类型转换异常-->后期所有的集合都需要指定泛型类型

### List接口之实现类LinkedList

#### LinkedList概述

LinkedList底层是使用链表存储数据,使用链表存储数据是为了解决ArrayList使用数据存储数据的问题-->数据存储查询速度快,但是插入删除速度慢,LinkedList链表结构是查询速度慢,插入删除速度快-->LinkedList底层的存储方式就是使用双链表结构存储,在其中有一个内部类Node-->节点,存储数据

#### LinkedList中的链表是什么?

链表存储时数据不连续(属于数据之间是非连续空间),空间不连续也就意味着无法通过下标指向某一个元素,所以就出现了一个问题,只要查找数据就必须遍历所有的集合数据进行查找

#### 链表分为4种类型

##### 单链表

单链表结构:单链表存储使用节点方式(节点就是一个对象),

存储数据时有两个属性：

- data：存放数据

- next：指针，指向下一个节点地址，最后一个节点存储null

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/12.png)

##### 双链表

双链表结构中使用双链表存储使用节点方式(节点就是一个对象)

存储数据时有三个属性

- prev-->存储上一个节点地址,如果是第一个则存储null
- data-->存储数据
- next-->指针,存储下一个节点地址,如果是最后一个则存储null

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/13.png)

##### 环形单链表

单链表结构:单链表存储使用节点方式(节点就是一个对象)

存储数据时有两个属性

- data(存放数据)
- next(指针,指向下一个节点地址),最后一个节点的next存储的是第一个节点的地址

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/14.png)

##### 环形双链表

双链表结构:双链表存储使用节点方式(节点就是一个对象)

存储数据时有三个属性

- prev-->存储上一个节点地址,如果是第一个则存储最后一个节点的地址
- data-->存储数据
- next-->指针,存储下一个节点地址,如果是最后一个则存储第一个节点的地址

![](https://raw.githubusercontent.com/i-xiaoxin/image/master/15.png)

#### LinkedList私有的方法(自己新增针对于增加和删除方法)

- `public void addFirst(E e)`:向集合的开头插入数据
- `public void addLast(E e)`:向集合的末尾插入数据
- `public E getFirst()`:获取集合中的第一个元素(如果第一个元素是null则抛异常)
- `public E getLast()`:获取集合中的最后一个元素(如果最后一个元素是null则抛异常)
- `public E peekFirst()`:获取集合中的第一个元素但是不删除(如果第一个元素是null则返回null)
- `public E peekLast()`:获取集合中的最后一个元素但是不删除(如果最后一个元素是null则返回null)
- `public E pollFirst()`:获取集合中的第一个元素,获取后直接删除
- `public E pollLast()`:获取集合中的最后一个元素,获取后直接删除
- `public E removeFirst()`:移除集合中的第一个元素
- `public E removeLast()`:移除集合中的最后一个元素

LinkedList私有的方法代码案例

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        // 如果使用多态方式创建则无法使用LinkedList的私有方法
        // List<String> list = new LinkedList<>();
        // 所以创建时只能指定自己的对象
        LinkedList<String> linkedList = new LinkedList<>();
        // 1. `public void addFirst(E e)`:向集合的开头插入数据
        linkedList.addFirst("彭于晏");
        System.out.println(linkedList);
        linkedList.addFirst("梅超风");
        System.out.println(linkedList);
        // 2. `public void addLast(E e)`:向集合的末尾插入数据
        linkedList.addLast("岳不群");
        System.out.println(linkedList);
        linkedList.addLast("林平之");
        System.out.println(linkedList);
        // 3. `public E getFirst()`:获取集合中的第一个元素
        String first = linkedList.getFirst();
        System.out.println(first);
        // 4. `public E getLast()`:获取集合中的最后一个元素
        String last = linkedList.getLast();
        System.out.println(last);
        // 5. `public E peekFirst()`:获取集合中的第一个元素但是不删除
        String peekFirst = linkedList.peekFirst();
        System.out.println(peekFirst);
        // 6. `public E peekLast()`:获取集合中的最后一个元素但是不删除
        String peekLast = linkedList.peekLast();
        System.out.println(peekLast);
        System.out.println(linkedList);
        // 7. `public E pollFirst()`:获取集合中的第一个元素,获取后直接删除
        String pollFirst = linkedList.pollFirst();
        System.out.println(pollFirst);
        // 8. `public E pollLast()`:获取集合中的最后一个元素,获取后直接删除
        String pollLast = linkedList.pollLast();
        System.out.println(pollLast);
        System.out.println(linkedList);
        // 9. `public E removeFirst()`:移除集合中的第一个元素
        linkedList.addFirst("梅超风");
        System.out.println(linkedList);
        linkedList.removeFirst();
        System.out.println(linkedList);
        // 10. `public E removeLast()`:移除集合中的最后一个元素
        linkedList.removeLast();
        System.out.println(linkedList);
        System.out.println("===============================================");
        System.out.println("===============================================");
        System.out.println("===============================================");
        System.out.println("===============================================");
        System.out.println("===============================================");
        System.out.println("===============================================");
        // 都是使用多态方式创建时就不存在以上的私有定义的方法
        List<String> list = new LinkedList<>();
        // 1. `boolean add(E e)`:向List集合中添加元素`常用`
        list.add("彭于晏");
        System.out.println(list);
        // 2. `void add(int index, E element)`:向List集合中指定位置添加元素
        list.add(0, "梅超风");
        System.out.println(list);
        // 3. `boolean addAll(Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的末尾
        list.addAll(linkedList);
        System.out.println(list);
        // 4. `boolean addAll(int index, Collection<? extends E> c)`:将指定集合中的所有元素追加到此集合的指定位置
        // 5. `boolean contains(Object o)`:检测此集合中是否包含指定元素`常用`
        boolean isContains = list.contains("彭于晏");
        System.out.println(isContains);
        // 6. `boolean containsAll(Collection<?> c)`:检测集合中是否包含另一个集合中的元素
        // 7. `E get(int index)`:获取集合中指定下标索引的元素并返回`常用`
        String get = list.get(0);
        System.out.println(get);
        // 8. `E set(int index, E element)`:修改集合中指定下标索引的元素
        list.set(0, "东方不败");
        System.out.println(list);
        // 9. `int indexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回第一次发现的下标索引)
        int indexOf = list.indexOf("彭于晏");
        System.out.println(indexOf);
        // 10. `int lastIndexOf(Object o)`:在集合中查找元素如果存在则返回此元素的下标索引(返回最后一次发现的下标索引)
        int lastIndexOf = list.lastIndexOf("彭于晏");
        System.out.println(lastIndexOf);
        // 11. `List<E> subList(int fromIndex, int toIndex)`:切割集合,指定起始位置和结束位置
        // 12. `ListIterator<E> listIterator()`:返回List接口迭代器
        // 13. `int size()`:返回集合中的元素数量`常用`
        int size = list.size();
        System.out.println(size);
        // 14. `boolean isEmpty()`:判断集合是否为空`常用`
        System.out.println(list.isEmpty());
        // 15. `Object[] toArray()`:将集合转为数组
        // 16. `void clear()`:清空集合内元素
        // 17. `Iterator<E> iterator()`:返回集合迭代器元素,用于遍历集合中的所有元素
        // ListIterator迭代器
        ListIterator<String> listIterator = list.listIterator();
        while (listIterator.hasNext()) {
            System.out.println(listIterator.next());
        }
        // Iterator迭代器
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        // 清空clear
        list.clear();
        System.out.println(list.isEmpty());
        System.out.println(list.size());
    }
}
```

##### 自定义实现LinkedList功能

```java
public class LinkedList {
    // 用于存放首节点的属性-->只记录集合中的第一个节点
    private Node firstNode;
    // 用于存放尾结点的属性-->只记录集合中的最后一个节点
    private Node lastNode;
    // 用于存放集合中的元素数量
    private int size;
    public boolean add(Object element) {
        // 获取最后一个节点赋值给临时对象存放
        Node tempNode = lastNode;
        // 创建新节点,每次创建都是最后一条所以next指向直接赋值null
        Node newNode = new Node(tempNode, element, null);
        // 将新创建新节点当做是集合中的最后一个节点
        lastNode = newNode;
        // 验证临时节点是否为空
        if (tempNode == null) {
            // 将集合中的第一个节点的指向,指向到newNode节点上,代表是集合中的第一个节点
            firstNode = newNode;
        } else {
            // 临时存储节点不为空则将临时节点的next指向下一个节点,下一个节点就是上面新增的
            tempNode.next = newNode;
        }
        // 将集合中的size++代表元素数量
        size++;
        return true;
    }
    // 获取集合中的第一个节点
    public Object getFirstNode() {
        return firstNode.elementData;
    }
    // 获取集合中的最后一个节点
    public Object getLastNode() {
        return lastNode.elementData;
    }
    // size方法,获取长度
    public int size() {
        return size;
    }
    // 创建私有内部类-->节点类
    private static class Node {
        // 前一个节点
        Node prev;
        // 数据
        Object elementData;
        // 下一个节点
        Node next;
        // 全参构造方法
        public Node(Node prev, Object elementData, Node next) {
            this.prev = prev;
            this.elementData = elementData;
            this.next = next;
        }
    }
}
```

```java
public class MyLinkedListDemo {
    public static void main(String[] args) {
        // 创建自定义LinkedList集合对象
        LinkedList linkedList = new LinkedList();
        // 添加数据
        linkedList.add("彭于晏");
        linkedList.add("梅超风");
        linkedList.add("东方不败");
        linkedList.add("岳不群");
        linkedList.add("林平之");
        linkedList.add("鞠婧祎");
        // 获取集合中第一个元素
        System.out.println(linkedList.getFirstNode());
        // 获取集合中最后一个元素
        System.out.println(linkedList.getLastNode());
    }
}
```

### List接口之实现类Vector

#### Vector实现类概述

Vector与ArrayList一模一样的,唯一的区别在于Vector是线程安全的而ArrayList是线程不安全

#### List接口之Vector添加方法

- `public synchronized boolean add(E e)`:向集合中追加元素`常用`
- `public void add(int index, E element)`:向集合的指定下标索引插入元素
- `public synchronized boolean addAll(Collection<? extends E> c)`:将指定 Collection 中的所有元素附加到此 Vector 的末尾
- `public synchronized boolean addAll(int index, Collection<? extends E> c)`:将指定 Collection 中的所有元素插入到此 Vector 的指定位置。将当前位于该位置的元素（如果有）和任何后续元素向右移动（增加它们的索引）
- `public synchronized void addElement(E obj)`:向集合中追加元素

#### List接口之Vector删除方法

- `public synchronized E remove(int index)`:指定下标索引删除集合中元素
- `public boolean remove(Object o)`:指定元素对象删除集合中的元素`常用`
- `public synchronized boolean removeAll(Collection<?> c)`:从此 Vector 中删除包含在指定 Collection 中的所有元素
- `public synchronized void removeAllElements()`:从此Vector中删除所有元素并将其大小设置为零。
- `public synchronized boolean removeElement(Object obj)`:从此Vector中删除参数的第一个出现的元素。

#### List接口之Vector修改方法

- `public synchronized E set(int index, E element)`:修改指定下标索引的元素`常用`
- `public synchronized void setElementAt(E obj, int index)`:修改指定下标索引的元素
- `public synchronized void setSize(int newSize)`:设置集合中数组的大小

#### List接口之Vector查询方法

- `public synchronized E get(int index)`:通过下标索引获取集合中的数据`常用`
- `public int indexOf(Object o)`:指定元素获取集合中是否存在如果存在则返回第一次找到的下标索引否则返回-1
- `public synchronized int lastIndexOf(Object o)`:指定元素获取集合中是否存在如果存在则返回最后一次找到的下标索引否则返回-1

#### List接口之Vector其他方法

- `public synchronized int size()`:获取集合中的元素数量-->集合长度
- `public synchronized boolean isEmpty()`:判断集合是否为空
- `public void clear()`:清空集合中的所有数据

#### List接口之Vector代码案例

```java
public class VectorDemo {
    public static void main(String[] args) {
        // 创建Vector对象
        Vector<String> vector = new Vector<>();
        // #### 3. List接口之Vector添加方法
        // 1. `public synchronized boolean add(E e)`:向集合中追加元素`常用`
        vector.add("彭于晏");
        System.out.println(vector);
        // 2. `public void add(int index, E element)`:向集合的指定下标索引插入元素
        vector.add(0, "梅超风");
        System.out.println(vector);
        // 3. `public synchronized boolean addAll(Collection<? extends E> c)`:将指定 Collection 中的所有元素附加到此 Vector 的末尾
        // 4. `public synchronized boolean addAll(int index, Collection<? extends E> c)`:将指定 Collection 中的所有元素插入到此 Vector 的指定位置。将当前位于该位置的元素（如果有）和任何后续元素向右移动（增加它们的索引）
        // 5. `public synchronized void addElement(E obj)`:向集合中追加元素
        vector.addElement("吴彦祖");
        System.out.println(vector);
        // #### 4. List接口之Vector删除方法
        // 1. `public synchronized E remove(int index)`:指定下标索引删除集合中元素
        vector.remove(0);
        System.out.println(vector);
        // 2. `public boolean remove(Object o)`:指定元素对象删除集合中的元素`常用`
        // 3. `public synchronized boolean removeAll(Collection<?> c)`:从此 Vector 中删除包含在指定 Collection 中的所有元素
        // 4. `public synchronized void removeAllElements()`:从此Vector中删除所有元素并将其大小设置为零。
        // 5. `public synchronized boolean removeElement(Object obj)`:从此Vector中删除参数的第一个出现的元素。
        // #### 5. List接口之Vector修改方法
        // 1. `public synchronized E set(int index, E element)`:修改指定下标索引的元素`常用`
        vector.set(1, "陈冠希");
        System.out.println(vector);
        // 2. `public synchronized void setElementAt(E obj, int index)`:修改指定下标索引的元素
        // 3. `public synchronized void setSize(int newSize)`:设置集合中数组的大小
        // #### 6. List接口之Vector查询方法
        // 1. `public synchronized E get(int index)`:通过下标索引获取集合中的数据`常用`
        String element = vector.get(0);
        System.out.println(element);
        // 2. `public int indexOf(Object o)`:指定元素获取集合中是否存在如果存在则返回第一次找到的下标索引否则返回-1
        int indexOf = vector.indexOf("陈冠希");
        System.out.println(indexOf);
        // 3. `public synchronized int lastIndexOf(Object o)`:指定元素获取集合中是否存在如果存在则返回最后一次找到的下标索引否则返回-1
        // #### 7. List接口之Vector其他方法
        // 1. `public synchronized int size()`:获取集合中的元素数量-->集合长度
        int size = vector.size();
        System.out.println(size);
        // 2. `public synchronized boolean isEmpty()`:判断集合是否为空
        System.out.println(vector.isEmpty());
        // 3. `public void clear()`:清空集合中的所有数据
        vector.clear();
        System.out.println(vector.isEmpty());
        System.out.println(vector.size());
        System.out.println("================================================");
        System.out.println("================================================");
        System.out.println("================================================");
        System.out.println("================================================");
        // 使用常用方式创建Vector对象-->多态创建
        List<String> list = new Vector<>();
        // 增加
        list.add("舒克");
        list.add("贝塔");
        list.addAll(vector);
        System.out.println(list);
        // 删除
        list.remove("陈冠希");
        System.out.println(list);
        // 修改
        list.set(0, "开飞机的舒克");
        list.set(1, "开坦克的贝塔");
        // 获取数据
        System.out.println(list.get(0));
        System.out.println(list.get(1));
        System.out.println(list);
    }
}
```

### List接口的三个实现类对比

##### ArrayList

- 底层数组实现,查询速度快,增删速度慢,速度慢的原因是:每次如果插入数据则需要将插入位置之后的数据都向后移一位

- 线程不安全的

##### LinkedList

- 底层双链表实现,增删速度快,查询速度慢,速度慢的原因是:不管查询的是第几个都需要从头到尾查一遍

- 线程不安全的

##### Vector

- 底层数组实现,查询速度快,增删速度慢,速度慢的原因是:每次如果插入数据则需要将插入位置之后的数据都向后移一位
- 线程安全的

> ArrayList和Vector实现都是相同,包括方法都是相同,唯一的区别就是ArrayList是线程不安全的,Vector是线程安全

TO DO loading
