---
title: Java 笔试题一套
date: 2016-05-05 16:14:27
tags: [java, interview]
---

最近项目组招新人（一年经验），让我帮忙出了套笔试题。要求题目主要考察Java基础知识。  
这套题目内容覆盖了继承、异常、线程、同步锁、正则表达式、集合类、数据结构、简单算法以及少量Java8新特性。  
虽然东西都很基础，但想全部答对还是有一定的难度的。

<!-- more -->

## 选择题

1. 以下哪组方法签名构成了方法的重载 ( )  
	A. `void foo(List<String> list);` 与 `void foo(List<Integer> list);`  
	B. `void foo(List<String> list);` 与 `String foo(List<String> list);`  
	C. `void foo(String[] arr);` 与 `void foo(Integer[] arr);`  
	D. `void foo(String[] arr);` 与 `String foo(String[] arr);`  

2. 以下说法，正确的是 ( )  
	A. 父类方法中抛出受检异常，子类重写该方法时，必须抛出该异常或该异常的超类。  
	B. 父类方法中抛出受检异常，子类重写该方法时，必须抛出该异常或该异常的子类。  
	C. 父类方法中抛出非受检异常，子类重写该方法时，必须抛出该异常或该异常的超类。  
	D. 父类方法中抛出非受检异常，子类重写该方法时，必须抛出该异常或该异常的子类。

3. 以下哪项可能是程序运行后的结果 ( )  
	A. `0 1 2 0 1 2 0 1 2 `  
	B. `0 0 0 1 1 1 2 2 2 `  
	C. `0 0 1 1 2 2 0 1 2 `  
	D. `throw exception`  

	```java
	public class Question03 {
	    public static void main(String[] args) {
	        for (int i = 0; i < 3; i++) {
	            new Printer(String.valueOf(i)).start();
	        }
	    }
	}
	class Printer extends Thread {
	    private String message;
	    public Printer(String message) {
	        this.message = message;
	    }
	    @Override
	    public void run() {
	        synchronized (this) {
	            for (int i = 0; i < 3; i++) {
	                System.out.print(message + " ");
	                try {
	                    Thread.sleep(1000);
	                } catch (InterruptedException ignored) {}
	            }
	        }
	    }
	}
	```

4. 以下程序的运行结果为 ( )  
	A. `<html><body></body></html>; `  
	B. `html><body></body></html; `  
	C. `<html>; <body>; </body>; </html>;`  
	D. `html; body; /body; /html; `  

	```java
	public class Question04 {
	    public static void main(String[] args) {
	        Pattern pattern = Pattern.compile("<(.*)>");
	        Matcher matcher = pattern.matcher("<html><body></body></html>");
	        while (matcher.find()) {
	            System.out.print(matcher.group(1) + “; ”);
	        }
	    }
	}
	```

## 填空题

1. 以下程序将输出\_\_\_\_\_\_\_\_。
	`IntStream.range(0, 10).filter(i -> i % 2 != 0).map(i -> i * 2).sum();`  
	
2. 现有`listA`与`listB`两个`List<String>`类型的列表，要得到所有在`listA`且不在`listB`中的字符串的集合，应该调用`listA.`\_\_\_\_\_\_\_\_。

3. 对于`List`的而言，`ArrayList`适用于\_\_\_\_\_\_\_\_操作较频繁的场景，`LinkedList`适用于\_\_\_\_\_\_\_\_操作较频繁的场景。

4.  以下程序将输出\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_。

	```java
	class Foo {
	    private static int a = 0;
	    private int b = 1;
	    static {
	        System.out.println("静态代码块: a = " + a);
	    }
	    {
	        System.out.println("初始化代码块: b = " + b);
	    }
	    public Foo() {
	        System.out.println("构造方法: b = " + b);
	    }
	    public static void main(String[] args) {
	        new Foo();
	    }
	}
	```

## 编程题

1.  实现如下签名的方法:  

    `public static int fact(int n);`

    方法返回n的阶乘，要求分别用迭代与递归两种方式实现该方法。如果可能，请将递归优化为尾递归实现。  

    测试用例：

	```java
	assert fact(5) == 120;
	assert fact(6) == 720;
	```

2.  实现一个简化版的`List<E>`类，需要实现以下方法：  

    ```java
    int size();                  // 返回列表长度
    boolean isEmpty();           // 判断列表是否为空
    boolean contains(Object o);  // 判断列表是否包含元素 o
    boolean add(E e);            // 向列表中添加元素 e，返回操作是否成功
    E remove(int index);         // 移除下标为 index 的元素，并返回该元素
    E get(int index);            // 获取下标为 index 的元素
    E set(int index, E e);       // 将下标为 index 的元素设为 e，并返回原位置的元素
    ```

    要求：不允许使用 Java 自带的类库，底层用数组实现。可以向 List 中无限制添加数据。

3.  实现如下签名的方法:  
    
    `public static List<int[]> permutations(int[] src);`

    要求输入一个数组,输出这个数组中元素的全排列构成的列表
  
    测试用例：

    ```java
    int[] src = new int[]{1, 2, 3};
    List<int[]> result = permutations(src);
    assert result.size() == 6;
    result.forEach(arr -> {
        for (int i : arr) {
            System.out.print(i + " ")
        }
        System.out.println();
    });
    // 1 2 3
    // 2 1 3
    // 2 3 1
    // 1 3 2
    // 3 1 2
    // 3 2 1
    ```

<!--
## 解答
### 选择题:  
1~4: CBAB
	
### 填空题:  
	
1. `50`
2. `removeAll(listB);`
3. 查改、增删
4. 静态代码块: a = 0  
	初始化代码块: b = 0  
	构造方法: b = 1
	   
### 编程题:  
略
-->