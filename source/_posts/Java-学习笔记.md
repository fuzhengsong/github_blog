---
title: Java 学习笔记
date: 2018-02-14 10:48:26
tags:
- 笔记
- java
categories:
 - java
---
# 类和对象

##   类之间的关系
1. 关联关系

``` javascript
class Car{
	
}

class Drive{
	Car car;   // drive 类中包含car类
}
```
2.聚合关系

``` stylus
class CPU{

}

class ROM {

}

class Computer{
	CPU cpu;
	ROM rom;
}
```

3. 合成关系
例如： 胳膊和腿等合成了一个人 是比聚合关系更强的关系

4. 继承关系
5. 依赖关系
6. 接口关系

<!--more-->
## 属性
1.  如果加了public 关键字 可以在类外部直接访问
``` stylus

// java 中的引号需要用双引号
public class Test {
	public static void main (String[] args){
		// code
		// 实例化
		Person tom = new Person();
		tom.Pid = 22;
		tom.name = "fuzhengsong";
		tom.age = 26;
		tom.display();
	}
}

class Person {
	// 属性
	public int Pid;   // 如果加了public 关键字 可以在类外部直接访问
	public String name;
	public int age;
	
	// 构造方法
	Person(){
		
	}
	
	// 方法
	void display(){
		String str = this.Pid + "," + this.name + "," + this.age;
		System.out.println(str);   // 22,fuzhengsong,26
	}
}

```


## 方法
1. 方法必须放到类的定义里面，不能单独定义。

### 分类
1.类方法（静态方法）
区别 : 在方法前面有static关键字,可以通过类名直接访问,不需要实例化。
``` cs
public class Test1 {
	public static void main (String[] args){
		Person1.startComputer(10);
	}
}

class Person1 {
	static void startComputer(int n){
		for(int i = 0; i<n; i++){
			System.out.println("*");
		}
	}
}

```

2.实例方法
需要通过实例化成对象，通过对象才能访问。


3. 方法的重载（overloading）
类中的方法名相同，参数类型或者数量不同，称为方法的重载。返回值不同不能构成方法的重载。
例子： System.out.println() 中的println 方法， 可以打印各种类型的值（参数类型不同）。

## 构造方法
如果我们显示定义了有参数的构造方法，那么系统不再提供默认的无参数构造方法。但是也可以手动添加无参数构造方法（重载）。

## this关键字

``` cs
// 构造方法 和 this关键字
public class Test3 {
	public static void main(String[] args){
		Person3 per = new Person3(12, "fuzhengsong", 26);
		per.displsy();  //  Person
						/// 12,fuzhengsong,26  
	}
}

class Person3 {
	int pid;
	String name;
	int age;
	
	// 无参数构造方法  （自己手动添加的无参数构造方法）
	Person3(){
		System.out.println("Person");
	}
	
	// 有参数构造方法
	public Person3(int pid, String name, int age){
		this();   				//  可以通过this()来调用无参数构造方法
		this.pid = pid;
		this.name = name;
		this.age = age;
	}
	
	void displsy(){
		String str = pid + "," + name + "," + age;
		System.out.println(str);
	}
}
```

## static关键字
1. static修饰的属性是所有实例共享的（类属性）
2. static修饰的方法可以通过类名直接访问，但是方法中不能访问实例变量和实例方法。

### 单例模式

``` cs
// 单例模式
public class Test4 {
	public static void main(String[] args){
		// Demo d = new Demo(); 		// 错误。 因为构造方法是私有的 所以new一个对象是不行的
		Demo d1 = Demo.getInstance();
		Demo d2 = Demo.getInstance();
		System.out.println(d1 == d2);  // 返回true  说明2次实例化的同一个对象
	}
}

class Demo {
	private Demo(){   // 私有的构造方法
		
	}
	
	private static Demo instance; // 申明一个私有的（外部不能访问）静态的（所有实例共享） Demo类型的变量
	
	public static Demo getInstance(){  // 实例化类的方法
		if(instance == null){
			// 自己实例化自己
			instance = new Demo();
		}
		return instance;
	}
}
```
## paceage 和 import 

``` javascript
import java.util.Date;

import com.geek99.B;

public class Test5 {
	// ctrl+shift+O ;  导入相应的包；
	// alt+/ 自动补全代码
	public static void main(String[] args) {
		B b = new B();
		Date d = new Date();
	}
}
```

##  访问控制
![enter description here][1]

1. 如果类前面没有任何的修饰符 那么只能在当前包或者类中访问。
  
  
  # 继承
  
  1. 使用extends实现
  2. 语法 class SubClassName extends SuperClassName {};
  3. java 只支持单根继承（一个父亲） 子类拥有父类的所有方法和属性（构造方法不行）。
  4. java使用接口实现多重继承的语义。
  5. 子类的构造方法自动调用父类的默认构造方法
  

``` stylus

	public Animal(){ 								// 父类构造方法
		System.out.println("Animal...");
	}
	
	public Dog(){								// 子类构造方法
		System.out.println("DOG...");
	}
	
	Dog dog = new Dog();
	输出：
	// Animal....
	// DOG....
```


  6. 如果父类没有默认的构造方法，子类必须手动调用构造方法。
  

``` stylus
	
	public Animal(int aid, String name, int age){   // 重写了构造方法
		this.aid = aid;
		this.name = name;
		this.age = age;
	}
	
	public Dog(){						//  子类手动调用super()方法
		super(1,"fu", 18);
		System.out.println("DOG...");
	}
```
## 方法覆盖
1. 方法覆盖和方法重载的区别？
	方法覆盖是继承的子类里面重写父类里的同名方法， 方法重载是同一个类里面相同方法名称，但是参数类型或者个数不同。
	
方法覆盖中子类方法的访问范围必须大于父类方法的访问范围。


## super关键字
父类的一个引用。
1. 使用super可以调用父类的构造方法
2. 使用super可以调用父类的属性和方法

## 抽象方法
abstract 关键字

``` stylus

// 类必须为抽象类
abstract class Animal{
	private String name;
	
	public abstract void run();
	// 抽象方法不能设为 private final static
	
	
}
```
不能为private： private只能在类中访问，所以设一个没有方法体的抽象方法没有意义

不能为static： 静态方法只能通过类名直接访问， 也没有方法体 ，没有yiyi

不能为final:  final方法是不能被重写的， 也没有意义。

一般来说，抽象类都是被继承的。

``` scala
class Dog extends Animal{
	public void run(){
		System.out.println("Dog run....")
	}
}
```

## 抽象类
1. 在类的前面有abstract关键字
2. 一个类里面有一个或者多个抽象方法
3. 抽象类不能被实例化
4. 子类在继承抽象类时，必须实现抽象类中的抽象方法
5. 抽象类只能是超类


## final的用法
1.类前面 阻止类的继承
2.方法前面 阻止方法的覆盖
3.属性前面 常量

# 接口
接口是一种特殊的抽象类，其中智能定义常量和方法。

语法： [public][abstract][interface] NAME {}

``` stylus

public interface USB {    //  默认是抽象的 所以可以不加abstract
	public void read();     
	public void write();
}

```
多个类可以实现同一个接口
一个类也可以实现多个接口（接口之间逗号隔开）

``` stylus

public class computer implements USB{

	@Override
	public void read() {
		// TODO Auto-generated method stub
		System.out.println("Read...");
	}

	@Override
	public void write() {
		// TODO Auto-generated method stub
		System.out.println("write...");
	}
	
}

```
## 接口的继承
接口的继承和类的继承相似 都是使用extends 


## 内部类

``` stylus

public class OuterClass {
	// 内部类
	class InnerClass{
		
	}
	
	// 静态内部类
	static class StaticInnerClass{
		
	}
	
	// 局部内部类
	
	void test(){
		class LocalClass{
			
		}
	}
}


// 并行类
class Animal{
	
}
```

内部类的实例化

``` ocaml
public class Test {
	OuterClass oc = new OuterClass();
	OuterClass.StaticInnerClass sic= new OuterClass.StaticInnerClass();
	OuterClass.InnerClass ic = oc.new InnerClass();
}
```

匿名内部类（重点）

public class Test {
	
``` nimrod
	public static void main(String[] args) {
		OuterClass oc = new OuterClass();
		OuterClass.StaticInnerClass sic= new OuterClass.StaticInnerClass();
		OuterClass.InnerClass ic = oc.new InnerClass();
		
		// 匿名内部类
		USB usb = new USB(){

			@Override
			public void read() {
				// TODO Auto-generated method stub
				
			}

			@Override
			public void write() {
				// TODO Auto-generated method stub
				
			}
			
		};
		
		Animal a = new Animal(){
			
			@Override
			 void run() {
				// TODO Auto-generated method stub
				System.out.println("run...")
			}
		};
		
		a.run(); //  run...
	}
}

interface USB{
	public void read();
	public void write();
}

//   抽象类
abstract class Animal{
	abstract void run();
}
```

# 引用类型
##  向上类型转换
无需强制转换，可以直接使用。

## 向下类型转换
需要强制转换，不一定成功。

# 异常

## 异常的处理
处理异常有2中方式：
1. 直接往外抛出异常
2. 可以使用try  catch  finally 处理异常。 无论是否有异常 finally内的代码都会执行。 如果有多个catch 那么按顺序执行，如果catch到错误，那么后面的catch不再执行。


## 自定义异常

``` stylus

public class Test {
	public static void main(String[] args) {
		Pan p = new Pan(400);
		try {
			p.use();
		} catch (PanExp e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}


class PanExp extends Exception{
	int p;
	public PanExp (int p, String name){
		super(name);
		this.p = p;
	}
}

class Pan{
	int p;
	public Pan(int p){
		this.p = p;
		
	};
	
	public void use() throws PanExp{			// 方法继续抛出异常
		if(p>500){				// 抛出异常
			throw new PanExp(p, "压力太高，危险");
		} else {
			System.out.println("运行正常。。。");
		}
	}
	
}
```

# 框架集合

## 简介
学习框架集合的整体思路
1. 如何添加元素
2. 如何获得元素
3. 如何删除元素
4. 如何遍历元素


## 常用的接口
1. collection 
2. List
3. Set
4. Map


# 输入和输出
## 文件和目录

1. 创建文件对象

``` stylus
	// 创建文件对象
	static void test1(){
		String path = "c:\\fuzhengsong\\11.txt";
		File f = new File(path);
		boolean b = f.exists();
		System.out.println(b);   // true
		
		String path1 = "c:\\fuzhengsong";
		String file1 = "22.txt";
		File f1 = new File(path1, file1);
		System.out.println(f1.exists());	// false
	}
```

2. 创建文件

``` stylus
	
	// 创建文件
	static void test2(){
		File f = new File("c:\\fuzhengsong\\44.txt");
		
		// 加catch的原因是可能创建会被拒绝
		try {
			boolean b = f.createNewFile();
			System.out.println(b);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```


3. 创建文件夹

``` stylus
	// 创建目录
	static void test3(){
		
		// 创建单个目录
		File f = new File("c:\\fuzhengsong\\a");
		boolean b = f.mkdir();
		
		// 创建多个目录
		File f1 = new File("c:\\fuzhengsong\\b\\c\\d");
		boolean b1 = f1.mkdirs();
	}
```


4. 删除文件

``` stylus
	
	// 删除文件
	static void test4(){
		File f = new File("c:\\fuzhengsong\\44.txt");
		f.delete();
	}
```


5. 获得文件信息


## 抽象流
1. InputStream

	int read() 
		从输入流读取下一个数据字节。
		返回 0 到 255 范围内的int字节值。
		如果到达流末尾没有可用的字节 返回-1

	int read(byte[] b)
		从输入流中读取一定数量的字节储存在**缓存区数组b中**
		以整数形式返回实际读取的字节数

	int read(byte[] b , int off, int len)
	偏移量 off 读取字节长度len  到缓存区b中。


2. OutputStream

	void write(int b)
	将指定的字节写入此输出流
	
	void write(byte[] b)
	将b.length 个字节从指定的字节数组写入到改字节流
	
	void write(byte[] b, int off, int len)
	将指定字节数组中从偏移量off 开始的len个字节写入
	
3. Reader

	int read()
	
	int read(char[] c)
	
	int read(char[]c, int off, int len)
4. Writer

## 输入输出流的体系结构
![enter description here][2]

  
  
## 文件流
1. FileInputStream FileOutputStream 文件字节输入输出流
2. FileReader FileWriter 文件字符流

### 文件字节输出流

``` stylus

	// 文件字节输出流
	static void test1(){
		try {
			FileOutputStream out = new FileOutputStream("src\\temp.txt");
			String str = "http://wwww.baidu.com";
			// 获取字节数组
			byte[] buffer = str.getBytes();
			out.write(buffer);
			out.close();
		} catch (FileNotFoundException e) {
			
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```

### 文件字节输入流

``` stylus
	// 文件字节输入流
	static void test2(){
		try {
			FileInputStream in = new FileInputStream("src\\temp.txt");
			int a;
			while((a = in.read()) != -1){
				System.out.print((char)a);
			}
			in.close();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```

### 文件字符输出

``` stylus
// 文件字符输出
	static void test3(){
		try {
			FileWriter out = new FileWriter("src\\temp1.txt");
			String str = "付正松";
			// 直接写入字符串
			out.write(str);
			out.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```

### 文件字符输入

``` stylus
	// 文件字符输入流
	static void test4(){
		try {
			FileReader in = new FileReader("src\\temp1.txt");
			int c;
			while((c = in.read())!= -1){
				System.out.print((char)c);
			}
			in.close();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```


  [1]: ./images/%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6.png "访问控制"
  [2]: ./images/%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E6%B5%81%E4%BD%93%E7%B3%BB.png "输入输出流体系"