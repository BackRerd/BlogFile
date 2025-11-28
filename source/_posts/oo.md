---
title: 面向对象
tags:
  - Java
abbrlink: 28239a8d
date: 2025-05-04 13:18:40
categories: ["Java"]
---

## 函数与类

### 1.1 什么是函数？

在 Java 中，**函数被称为“方法（Method）”**。它是一段可以重复使用的代码，用来完成特定任务。

### 1.2 方法的结构

```java
返回类型 方法名(参数列表) {
    // 方法体
    return 返回值;
}
```

### 1.3 示例：一个简单的方法

```java
public class MyClass {
    // 一个返回两个整数和的方法
    public int add(int a, int b) {
        return a + b;
    }
}
```

### 1.4 方法说明

- `public`：访问修饰符，表示这个方法可以被其他类访问。
- `int`：返回类型，表示该方法返回一个整数。
- `add`：方法名。
- `(int a, int b)`：参数列表，接受两个整数。
- `return a + b;`：返回两数之和。

### 1.5 方法的调用

```java
public class Main {
    public static void main(String[] args) {
        MyClass obj = new MyClass();     // 创建对象
        int result = obj.add(5, 3);      // 调用方法
        System.out.println(result);      // 输出结果：8
    }
}
```

### 2.1 什么是类？

类是 Java 中的**基本构建块**，是对象的模板或蓝图。它定义了对象的属性（变量）和行为（方法）。

### 2.2 类的结构

```java
public class 类名 {
    // 属性（字段）
    数据类型 属性名;

    // 构造方法（可选）
    public 类名() {
        // 初始化
    }

    // 方法
    返回类型 方法名() {
        // 方法体
    }
}
```

### 2.3 示例：一个表示学生的类

```java
public class Student {
    // 属性
    String name;
    int age;

    // 构造方法
    public Student(String n, int a) {
        name = n;
        age = a;
    }

    // 方法
    public void introduce() {
        System.out.println("我是 " + name + "，今年 " + age + " 岁。");
    }
}
```

### 2.4 使用类

```java
public class Main {
    public static void main(String[] args) {
        Student stu = new Student("小明", 18);  // 创建对象
        stu.introduce();                       // 调用方法
    }
}
```

**输出结果：**

```
我是 小明，今年 18 岁。
```

------

## 三、类与方法的关系

- 类是“容器”，方法是“行为”。
- 方法通常定义在类中，描述类对象可以执行的操作。

------

## 四、总结

| 项目     | 方法（函数）           | 类                          |
| -------- | ---------------------- | --------------------------- |
| 定义     | 实现特定功能的代码块   | 对象的模板                  |
| 包含内容 | 参数、返回值、代码逻辑 | 属性、构造方法、方法        |
| 用途     | 执行动作               | 创建对象、组织代码          |
| 使用方式 | `对象.方法名(参数)`    | `类名 对象名 = new 类名();` |

### 类比理解：

- **类就像是一个人**：他有名字（属性）、年龄（属性）、以及能做的事情（方法）。
- **方法就像人的器官**：比如嘴巴可以说话，手可以写字，眼睛可以看东西。每个器官（方法）完成一个具体任务。
- 创建对象就像是“生出一个人”，而方法则定义了这个人能干什么。

> 💡 总结一句话：**类是对现实世界的抽象，而方法是对象的行为实现。**

## 面向对象

**面向对象编程（OOP，Object-Oriented Programming）** 是一种编程范式，它基于“对象”来组织代码，而不是仅仅依靠函数和逻辑。Java 是一种典型的面向对象语言，整个语言设计都围绕这一思想展开。

### 1. **封装（Encapsulation）**

- 把数据（属性）和行为（方法）封装在一个对象中。

- 使用 `private` 修饰属性，只允许通过 `public` 的 getter/setter 方法访问。

- 好处：隐藏实现细节，提高安全性和可维护性。

- ```java
  public class Person {
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  }
  ```

  

### 2. **继承（Inheritance）**

- 子类继承父类的属性和方法，使用关键字 `extends`。

- 提高代码复用性。

- Java 是单继承（一个类只能有一个直接父类），但支持多层继承。

- ```java
  class Animal {
      void eat() {
          System.out.println("This animal eats food.");
      }
  }
  
  class Dog extends Animal {
      void bark() {
          System.out.println("Dog barks.");
      }
  }
  ```



### 3. **多态（Polymorphism）**

- 同一个方法可以有多种表现形式。

- 分为**编译时多态**（方法重载）和**运行时多态**（方法重写 + 父类引用指向子类对象）。

- 提高代码灵活性和可扩展性。

- ```java
  class Animal {
      void speak() {
          System.out.println("Animal speaks");
      }
  }
  
  class Cat extends Animal {
      @Override
      void speak() {
          System.out.println("Cat meows");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Animal a = new Cat(); // 多态
          a.speak();  // 输出: Cat meows
      }
  }
  ```

  

### 4. **抽象（Abstraction）**

- 提取对象的共有特征，忽略细节，强调“做什么”，而不是“怎么做”。

- 使用 `abstract` 类或 `interface`。

- 抽象类可以包含实现的方法；接口是完全抽象的（Java 8 以后接口可以有默认方法）。

- ```java
  abstract class Shape {
      abstract void draw();
  }
  
  class Circle extends Shape {
      void draw() {
          System.out.println("Drawing a circle");
      }
  }
  ```

### 5.实践

#### 5.1 当前要求你开发一个游戏系统中的职业

需求:每个职业都要名称、血量、攻击力、防御力、以及三个技能。

示例职业 1：战士 Warrior

示例职业 2：法师 Mage

示例职业 3：刺客 Assassin

示例职业 4：牧师 Priest

待续....
