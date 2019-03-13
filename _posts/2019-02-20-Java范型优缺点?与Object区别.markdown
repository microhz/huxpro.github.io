---
layout:     post
title:      "大促稳定性"

date:       2019-01-20 11:00:00
author:     "micro"
header-img: "img/in-post/post-hz-bg.jpeg"
catalog: true
tags:
    - 学习
---
- 范型是高级编程语言一般都具有的特性，在现在框架中应用广泛，但是对于一些细微的使用上的区别需要做一个记录。
最初我打算通过其他网站找到想要的答案，但是网上的文章鱼龙混杂，我一般都会对比多家进行参考，但是最有说服力的还是
talk is cheap,show me your code.
- 关于范型有几个问题也并没找到想要的答案，大多数都是简单的介绍范型的使用。极少数相关的文章也大多是互相copy。
- 本文不打算介绍范型的基础知识，所以希望学习范型基本使用的同学可以绕行了。
1. 范型产生的原因？
范型的产生根本原因就是为了编码的高效。废话，什么语言特性不是为了高效。
好吧，我们可以show code了。假如说我们没有范型，我们要写一个整数的排序算法。

```
class Sort {
    void sort(Integer[] sort) {
        // 排序
    }
}
```

如果后面需要浮点数的排序.

```
class Sort {
    void sort(Integer[] sort) {
        // 排序
    }

    void sort(Double[] sort) {
        // 排序
    }
}
```

发现排序的逻辑是一样的，就因为数据类型不同，后面还要添加各种数据类型，添加各种方法太冗余了。
我们可以传入一个通用的对象进行排序。

```
class Sort {
    void sort(Object[] sort) {
        Number[] numbers = (Number[]) sort;
        // 排序
    }
}
```

这样个问题，强转无法在编译阶段发现传入参数类型.

范型的引入一个重要原因就是解决强转的问题，采用范型写法的话。

```
class Sort {
    <T> void sort(T[] sort) {
        Number[] numbers = (Number[]) sort;
        // 排序
    }
}
```

这与前面一个示范其实是等价的，因为JVM会在编译后生成的字节码进行一个所谓的“范型擦除”，
<T> void sort(T[] sort) 。前面的<T>是范型定义，没有任何限制，因此方法签名的入参T[]会在字节码里显示为Object[].
简单来说就是范型的占位符号被真实的类给替换掉，因此范型可以看作是对JVM执行引擎透明的存在。
我们要对其进行限制，例如此处排序算法我们的设计只是限定为对Number类型的数组进行排序。因此需要
范型对我们的入参数进行校验，一定要满足Number数组才允许调用，否则编译出错。
可以如下

```
class Sort {
    <T extends Number> void sort(T[] sort) {
        Number[] numbers = (Number[]) sort;
        // 排序
    }
}
```

这样如果调用试图传入一个String[],编译就会出错。因为<T extends Number>限制了该方法只能传入一个Number类型的数组.

从此处看范型做了一个隐形的类型校验的功能。

其实范型不仅可以修饰方法签名，也可以修饰类，此处又有什么用途呢?
其实主要是看该范型的限制范围，如果是整个类的多个方法限制，定义在类，否则定义在方法签名上。
定义类的时候 class [类名]<范型定义>.多个方法可以直接使用，
例如方法可能不只排序，还有比较大小呢,这样我们定义

```
class Sort<T extends Number> {
    void sort(T[] sort) {
        Number[] numbers = (Number[]) sort;
        // 排序
    }

    boolean equals(T number1, T number2) {

        // 比较
        return false;
    }
}
```

其实同价:

```
class Sort {
    <T extends Number> void sort(T[] sort) {
        Number[] numbers = (Number[]) sort;
        // 排序
    }

    <T extends Number> boolean equals(T number1, T number2) {

        // 比较
        return false;
    }
}
```

但是在调用的地方，第一种可以指定明确的子类。

```
Sort<Integer> sort = new Sort<>();
sort.sort(new Integer[]{1,2,3});
```

这样在不同的地方调用复用了该方法，又避免了类型问题。 

这是对于参数的限制，范型也对返回参数也可以限制。但是返回值的限制    

```
class NumberService<T extends Number> {
    T getRandom() {
        return (T) new Integer(1);
    }
}
```

这里遇到一个问题，返回的类型必须强制转换成限制的类型，这里在调用的地方如果指定的范型类型无法强转将报错。
所以范型一般用在接口或则抽象类上。 子类去实现限制具体类型.
可以参考Java集合框架代码
public interface List<E> extends Collection<E> 
一般我们使用都是，数字集合就
List<Integer> list = new ArrayList<>();
如果是字符串
List<String> list = new ArrayList<>();
List与Collection等接口的定义行为规范，这个行为规范是解耦于具体指定类型的. 可以理解成，集合是支持Object类型的。
如果没有范型，集合的使用大概如下：

```
List stringList = new ArrayList();
stringList.add("1");
stringList.add(1);
stringList.add(1D);
```

String s = (String)stringList.get(0);
这个集合什么都可以保存，强转的error风险延迟到了运行阶段。
因此范型的指定保证了代码的健壮性，避免了强转的风险.

-----
- 占位符的理解
范型里面的?与限制符号有什么区别，show code
Person有两个子类Student和Teacher。代码略。

```
class PersonService {
    <T extends Person> List<T> queryList() {
        List<Person> personList = Lists.newArrayList();
        personList.add(new Student());
        personList.add(new Teacher());
        return (List<T>) personList;
    }

    List<? extends Person> queryList2() {
        List<Person> personList = Lists.newArrayList();
        personList.add(new Student());
        personList.add(new Teacher());
        return personList;
    }

    List<Person> queryList3() {
        List<Person> personList = Lists.newArrayList();
        personList.add(new Student());
        personList.add(new Teacher());
        return personList;
    }
}

```

调用如下：

```
PersonService personService = new PersonService();
List<Teacher> personList = personService.queryList();
System.out.println(personList);

List<? extends Person> personList2 = personService.queryList2();
System.out.println(personList2);
```

这并不会报错，注意personList添加了Student对象，返回值声明List<Teacher>居然程序没报错。
不要怀疑，还记得范型的类型擦除和帮我们做了强转的过程么。为什么没报错。因为其没有强转。
执行以下代码：

```
Teacher teacher = personList.get(0);
```

`xception in thread "main" java.lang.ClassCastException: Student cannot be cast to Teacher。`

结论很明显了。这里存在运行时异常的风险，这个时候建议使用queryList2的方式。
List<? extends Person> personList2的方式明确返回的值是未限定的。需要调用方识别。
queryList3的返回值也是List<Person>这与personList2有啥区别？其实这里没区别。
List<? extends Person>与List<Person>在返回值的情况我还没发现在运行时有啥区别。如果有，希望评论区指教一下。
那?存在的意义是什么。其实有的，在入参上兼容性上的区别。

```

List<Person> queryListByIds(List<? extends Number> idList) {
       // 略
}

List<Person> queryListByIds(List<Number> idList) {
       // 略
}
```

如果调用如下：

```
personService3.queryListByIds(new ArrayList<Integer>()); 
personService3.queryListByIds2(new ArrayList<Integer>());
```

第二个调用是会编译出错的.
可以理解成**List<? extends Number>为List<Integer>和List<Double>的逻辑父类**。
但是个人理解第二种方式更加“安全”，因为第一种其实只是接口兼容了入参的类型，并不方便真正去强转为子类，第二种方式就明确了类型。

