---
title: TypeScrip-学习（三）
date: 2023-04-25 16:45:59
cover: https://www.foodiesfeed.com/wp-content/uploads/2023/04/summer-grapefruit-cocktail.jpg
tags:
  - JavaScript
  - TypeScript
---

# Class类

## 定义类

```ts
class Person {
    //在TypeScript是不允许直接在constructor 定义变量的 需要在constructor上面先声明
    name: string
    age: number
    constructor(name: string, age: number = 11) {
        //你如果了定义了变量不用 也会报错 通常是给个默认值 或者 进行赋值
       this.name=name
       this.age=age;
    }
    run() { }
}
let item=new Person('张三')
```

## 类的修饰符

1. `访问修饰符`总共有三个 `public`、`private`、`protected`
    - `public` :公共的,没有访问限制
    - `protected` : 受保护的,只能自身和子类访问，实例不能访问
    - `pravite`  :私人的，只能自身访问

2. `只读修饰符` **readonly**

    ```ts
    class  F {
    readonly a =444;
    }
    class S extends F {
    }
    let  s = new S;

    console.log(s.a);
    s.a = 999 ;//无法分配到 "a" ，因为它是只读属性。
    ```

3. `静态修饰符` **static**

    ```ts
    class Person {
    name: string
    age: number
    static sex: string = 'man'
    constructor(name: string, age: number = 11) {
        //你如果了定义了变量不用 也会报错 通常是给个默认值 或者 进行赋值
        this.name = name
        this.age = age;
        //static 定义的属性 不可以通过this 去访问 只能通过类名去调用
        //this.sex  this.run()
    }
    static run() {
        // console.log(this.name); //this.name 会报错  因为在静态函数中this只能访问同是this的值
        console.log(this.sex)
    }

    }
    let item = new Person('张三')
    //item.run() item.sex //实例无法调用
    Person.run()
    Person.sex
    ```

## interface 定义类

和对象一样我们同样可以使用`interface` 定义类,但是需要使用关键字`implements`,后面跟`interface`的名字多个用逗号隔开继承还是用`extends`

```ts
interface PersonClass {
    get(type: boolean): boolean
}
interface PersonClass2{
    set():void,
    strName:string
}
class A {
    name: string
    constructor() {
        this.name = "123"
    }
}
class Person extends A implements PersonClass,PersonClass2 {
    strName: string
    constructor() {
        //继承需要执行super()加载父类，且放在必须放在第一位
        super();
        this.strName = '123'
    }
    get(type:boolean) {
        return type
    }
    set () {
 
    }
}
```

## 抽象类

应用场景如果你写的类实例化之后毫无用处此时我可以把他定义为抽象类
或者你也可以把他作为一个基类-> 通过继承一个派生类去实现基类的一些方法
> `基类`:父类，被继承的类
  `派生类`:子类，继承的类

1. 通过`abstract` 去定义的类为抽象类,该类无法被实例化
2. 通过`abstract` 去定义的方法为抽象方法,该类无法被实现
3. 通过`abstract` 去定义的方法想要实现必须在派生类中去实现

```ts
abstract class A {
    name: string
    constructor(name: string) {
       this.name = name;
    }
    print(): string {
       return this.name
    }
    //抽象方法不能具有实现
    abstract getName():string
 }
// new A() --无法创建抽象类的实例
 class B extends A {
    constructor() {
       super('小满')
    }
    getName(): string {
       return this.name
    }
 }
 let b = new B();
 console.log(b.getName());
```
