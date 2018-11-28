# 剑指offer 题解

## 2. 实现单例模式

### 什么是单例模式？

**单例模式**，顾名思义就是整个系统只能有1个实例存在，不能再多了。

### 单例模式的好处

单例模式适用于应用中频繁创建的对象，尤其是频繁创建的重量级对象。
例如统一的配置文件。
使用单例模式能够提升性能，减小内存开销。

### 单例模式的实现

#### 0. 概述

如何实现一个单例模式呢？我们讨论一下这个过程。

首先我们搞一个类出来：

```java
public class Singleton {
    
}
```

空空如也，非常好，但是有个问题，大家可以在外面随便`new Singleton()`,那系统里就不止一个Singleton实例了，因此需要我们阻挡`new Singleton()`。那就不写构造方法呗？当然不行。因为如果不写构造方法，系统会默认分配无参构造器。那就直接将构造方法改为`private`呗。

```java
public class Singleton {
    private Singleton() {}
}
```

好，既然外面造不出Singleton实例了，那就自己造呗。

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
}
```

`private`关键字保证了Singleton实例的私有性，不可见性，不可访问性；`static`关键字保证了Singleton实例的静态性，与类同在，不依赖于类的实例化就在内存中永生；`final`关键字保证这个实例是常量，不可修改。

那么既然外部无法创造这个实例，如何获取这个实例呢？整个`getInstance()`静态方法在外部获取这个实例，而且这个方法必须是`public`的。

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return instance;
    } 
}
```

到此，外部就可以通过`Singleton.getInstance()`获取这个单例。

这就是一个简单的**单例模式**的雏形（这其实是一个饿汉式单例模式）。以下介绍几种面试中常见的单例模式实现。

#### 1.饿汉式
饿汉式加载类的时候，就会创建类的实例对象，那么如果不用这个单例的话，就会消耗内存，浪费性能。
因此，需要懒加载（lazy-load），即延迟加载。

```java
public class Singleton {
    // instance必须是static变量（类变量）
    private static Singleton instance = new Singleton();

    // private无参构造函数: Java会自动为没有明确声明构造函数的类，
    // 定义一个public的无参构造函数，若没有这个private无参构造函数，
    // 外部可以随意new Singleton
    private Singleton() {}

    // getInstance必须是static方法（类方法）
    public static Singleton getInstance() {
        return instance;
    }
}
```

#### 2.懒汉式
懒汉式改进了饿汉式的性能问题，但是又带来了一个问题，即线程安全问题。
在多线程的情况下，会出现多个实例，违背了单例模式的原则。

```java
public class Singleton {
    private static Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

#### 3.双重校验（双检锁）
双重校验能够保证单例模式的实例唯一性，同时能够兼顾效率和线程安全。

```java
public class Singleton {
    // volatile提供可见性（工作内存的值能立即在主内存中可见），
    // 保证getInstance返回的是初始化完全的对象
    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        // 同步之前进行null检查，避免进入代价相对昂贵的同步块
        if (instance == null) {
            synchronized (Singleton.class) {
                // 持有锁之后进行null检查，避免上一个线程已经初始化变量
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 4.静态内部类
静态内部类的方式实现，可以保证多线程对象的唯一性，同时可以降低双重校验中同步锁机制的代价。
这种方式还有个好处，就是静态内部类不会在单例类加载的时候就加载，而是在调用`getInstance()`
方法时才加载。

```java
public class Singleton {
    private static class SingletonHolder {
        public static Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

####  5.枚举方法 
枚举方法是由《Effective Java》的作者Josh Bloch提出，其好处在于：
* 自由序列化
* 保证只有一个实例
* 线程安全

```java
public enum Singleton {
    INSTANCE;

    private int id;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public static void main(String[] args) {
        //调用
        Singleton.INSTANCE.setId(1);
        Singleton.INSTANCE.getId();
    }
}
```



## 3. 数组中重复的数字

### 3.1 找出数组中重复的数字

在一个长度为n的数组里的所有的数字都在0~n-1的范围内。数组中某些数字是重复的，但不知道有几个数字重复，也不知道每个数字重复了几次。请找出数组中任意的一个数字。

例如，输入长度为7的数组{2,3,1,0,2,5,3}，那么对应输出的是重复数字2或者3。

[nowcoder](https://www.nowcoder.com/practice/623a5ac0ea5b4e5f95552655361ae0a8?tpId=13&tqId=11203&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

**Solution**

1.遍历数组，同时使用HashMap记录数组中每个元素出现的个数，当某个元素个数大于1时，输出该元素即可。

时间复杂度O(n)，空间复杂度O(n)。

实现如下：

```java
public boolean duplicate(int numbers[], int [] duplication) {
        if (numbers == null || numbers.length < 2) {
            return false;
        }

        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < numbers.length; i++) {
            if (!map.containsKey(numbers[i])) {
                map.put(numbers[i], 1);
            } else {
                duplication[0] = numbers[i];
                return true;
            }
        }
        return false;
}
```

2.方法1并未使用题目中数组(numbers)元素都在0~n-1这个条件，如果使用这个条件的话，会发现若数组中没有重复元素的话，那么长度为n的数组，排序之后，应该是{0,1,2，...，n-1}，即k位置的元素为k。若有重复元素的话，那么k位置元素为k，其他位置也可能有k。利用这个特征遍历数组，若k位置的值为k，则遍历下一个元素；若k位置的元素的值不是k，则将k位置元素与numbers[k]位置元素交换，直到k位置的元素为k，然后遍历下一个元素，若在交换的过程中，发现k位置的元素等于numbers[k]位置的值，则k就是所求重复的值，结束遍历。

时间复杂度O(n)，空间复杂度O(1)。

实现如下：

```java
public boolean duplicate(int numbers[], int [] duplication) {
        if (numbers == null || numbers.length < 2) {
            return false;
        }

        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] == i) {
                i++;
            } else {
                int m = numbers[i];
                if (numbers[m] == m) {
                    duplication[0] = m;
                    return true;
                } else {
                    int tmp = numbers[i];
                    numbers[i] = numbers[m];
                    numbers[m] = tmp;
                }
            }
        }
        return false;
}
```

### 3.2 不修改数组找出重复的数字

在一个长度为n+1的数组里的所有的数字都在1~n的范围内，所以数组中至少有一个数字是重复的，请在不修改数组的情形下，找出数组中任意一个重复数字。

例如，输入长度为8的数组{2,3,5,4,3,2,6,7}，那么对应输出的是重复数字2或者3。

