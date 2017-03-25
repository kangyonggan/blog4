---
title: java手札-基础部分
date: 2017-03-25 19:16:49
categories: Java后台
tags:
- Java
---

1. 一个java源文件中是否可以包含多个类（不含内部类）？可以，但只能有一个public，且必须和文件名一致。
2. goto？goto是保留字。
3. &和&&？&是按位与，&&是逻辑与，&&会短路。
4. 跳出多重循环？可以用标志但不推荐，推荐使用变量来标识。
<!-- more -->
5. switch表达式可以有哪些类型？String呢？数字+枚举。jdk1.7之后可以有String，但底层没变，只是在编译时把String转hash
6. short s1=1;s1+=1;s1=s1+1;哪错？前对后错，类型转换错误，丢失精度，编译不过。
7. char能存储汉字？可以，char是Unicode编码，占2字节。
8. 快速乘8？左移3位。
9. final修饰一个变量时，是引用不能变?还是对象不能变？引用不能变，对象内容可以变。
10. ==和equals区别？equals不能用于比较8大基本数据类型, 如果比较的是基本类型的封装类型，会转化为==来比较他们的值，如果是String，会逐个比较char，如果是其他引用对象，会转化为==来比较他们的引用地址，但是你可以覆写equals来给两个对象是否一样来定指标。==如果比较的是基本数据类型或他们的封装类，是比较他们的值，如果比较的是引用对象，比较的是他们的引用地址。
11. static？类属性，类方法。
12. int和Integer？默认值不一样，vo层最后使用封装类，比如MBG逆向生成的model就是用的封装类型。
	- 原因一：在展现层，int显示是0，Integer显示空串。
	- 原因二：在tk.mybatis中，updateSelectiveByPrimaryKey(T t)，会更新不为null的字段，如果使用Integer，就不会更新，如果使用int，可能会误更新为0。
13. Math.round(-11.5)=-11，加0.5向下取整。
14. private、friendly、protected、public，当前类，同包，子孙类，所有类。
15. override和overload？overload重载，比如Math.abs()。override覆写，修饰符范围可小不可大，异常可少可子不可扩。
16. 构造方法可被覆写？不可。覆写要求方法名一样，但子类和父类的构造方法的方法名显然不一样。
17. 面向对象特征？封装、继承、多态、抽象。
	- 封装（高内聚，低耦合）：比如冰箱，按开关即可，内部封装，暴露接口即可。
	- 继承（复用性，扩展性）：比如，BaseService中提供了基础的curd。
	- 多态：编译期间不知道要掉哪个方法，运行期间才知道，比如：UserDao接口，有两个实现，UserMyBatisDao和UserHibernateDao这两个实现。
	- 抽象：抓住主要矛盾，忽略次要矛盾。
18. 抽象类abstract和接口interface？抽象方法所在的类必须是抽象类，抽象方法不能有方法体，抽象类的子类必须实现抽象方法，抽象类中可以有普通的方法,抽象类中可以有static方法，但是抽象方法不能同时是static,抽象方法不能有synchronized，可以在子类中覆写后加synchronized关键字。接口中所有的方法都是public，所有的属性都是public static final,所有的方法不能有方法体，子类必须实现所有方法(抽象类实现接口后可以不去覆写，但继承此抽象类的子类还是需要去实现的), 抽象类中不能有static方法。
19. 逗号分隔字符串，如果最后一个符号是逗号，数组长度是不对的。
20. String str="a"+"b"+"c"+"d";值创建了一个对象，javac编译时，对它进行了优化，+号两边的字符串直接合并。
21. 一般异常必须捕获或抛出，运行时异常不必须，比如：空指针异常，数组越界异常，类型转换异常。
22. sleep不释放锁，wait会释放锁。
23. 临界资源需要同步处理（原子化操作）。
24. ArrayList和Vector？都实现了List接口，List继承Collection，有序，允许重复，底层是数组。Vector线程安全，ArrayList不是线程安全，效率高。扩增时，Vector增2倍，ArrayList增1.5倍。
25. HashMap和HashTable？都实现了Map接口，都能存储名值对，HashMap允许null键值，干掉了contains()方法，改成了containsKey()和containsValue()。HashTable线程安全，并继承了Dictionary。
26. ArrayList和LinkedList？都是集合，前者读快写慢，后者读慢写快。前者是数组，后者是链表。
27. Collection和Collections？前者是集合的顶级接口，后者是集合的帮助类。