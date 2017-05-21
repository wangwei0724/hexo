---
 title: javascript中对象封装
 categories:
	- javascript学习
---
# javascript中对象创建方法

我们知道，JS是面向对象的。谈到面向对象，就不可避免的要涉及类的概念。不同于java这些强类型语言都有固定的定义类的语法，JS的能使用各种方法实现类和对象的封装。总结了目前想到的几种，后期有补充会继续更新：
## 1.工厂函数
>工厂函数是一个返回特定对象的函数；每次调用这个函数都会生成一个新的对象，所以每个对象用的都有自己的方法版本（比如下面的showinfo），但其实所有对象都是共享的一个方法；所以这里可以进行优化,可以在工厂函数外定义该对象的方法，这样就只需要生成一遍对象方法。其次，工厂函数只解决了对象的创建问题，没能解决对象的识别问题。

```
function createPerson(name,age){
   var temp=new Object;
   temp.name=name;
   temp.age=age;
   temp.showInfo=function(){
       console.log(this.name+' is '+this.age); 
   }
   return temp;//函数最后返回对象
}
var person1=createPerson('wang',23);
var person2=createPerson("he",25);
person1.showInfo();
person2.showInfo();
```
>优化方法:

```
function createPerson(name,age){
   var temp=new Object;
   temp.name=name;
   temp.age=age;
   temp.showInfo=showInfo;
   return temp;
}
function showInfo(){
   console.log(this.name+' is '+this.age); 
}
```
## 2.构造函数
>构造函数是指构造一个函数对象，每次需要用是需要先实例化。不过很显然，每次实例化一个对象都会创建自己的showInfo函数版本。这个问题同样可以和工厂函数采取一样的方法，将showInfo直接在person外定义。这种优化方法虽然能带来一些便利，但还是滋生了别的问题，比如showInfo是一个全局函数，但是一个全局函数只能被某个对象使用，显得有点名不符其实；更严重的是，如果定义了很多个全局函数，那这个自定义的引用类型就丝毫没有封装性了。好在这些问题可以通过原型模式解决。

```
function Person(name,age){
   this.name=name;
   this.age=age;
   this.showInfo=function(){
       console.log(this.name+' is '+this.age); 
   }
}
var person1=new Person('wang',23);
var person2=new Person("he",25);
person1.showInfo();
person2.showInfo();
```
## 3.原型方法
>该方式利用了对象的prototype属性，可把它看成创建新对象所依赖的原型。这里，用空构造函数来设置类名。然后把所有的方法和属性都直接赋予prototype属性。原型方法其实还有一个特点，我们给实例化出来的对象添加属性，当我们使用该属性时，搜索首先会是在这个在这个实例本身搜索，如果搜索不到，会从原型上搜索，直到得到结果。所以若实例和原型中有属性或方法重名，实例中的属性会屏蔽原型中的属性。

>原型方式只能直接赋值，而不能通过给构造函数传递参数初始化属性的值。在用这种方式时，会遇到两个非常讨厌的问题。第一问题是采用这种方式必须创建每个对象后才能改变属性的默认值。而不能在创建每个对象时都会直接有自己所需要的属性值。这点很讨厌。第二个问题在于属性所指的是对象的时候。函数共享不会出现任何问题，但是对象共享却会出现问题（如下文的hobbies）。

```
function Person(){
}
Person.prototype.name="wang";
Person.prototype.age=23;
Person.prototype.hobbies=new Array("basketball","shoes");
Person.prototype.showInfo=function(){
   console.log(this.name+' is '+this.age);
}
var person1=new Person();
var person2=new Person();
person1.hobbies.push("code");
console.log(person1.hobbies);//输出"basketball,shoes,code"
console.log(person1.hobbies);//输出"basketball,shoes,code"

```
## 4.构造函数+原型方式
>这种方式的思想是将上两种思想结合一下，用构造函数定义对象的所有属性（包括普通属性和指向对象的属性），用原型方式定义对象的方法。这样每个实例都会有一份属于自己的实力属性的副本，但同时又共享着对方法的引用，极大地节省了内存，可谓是集两家之长。

```
function Person(name,age){
   this.name=name;
   this.age=age;
   this.hobbies=new Array("basketball","shoes");
}
Person.prototype.showInfo=function(){
   console.log(this.name+' is '+this.age); 
}
var person1=new Person('wang',23);
var person2=new Person("he",25);
person1.hobbies.push("code");
console.log(person1.hobbies);//输出"basketball,shoes,code"
console.log(person2.hobbies);//输出"basketball,shoes"

```
## 5.动态原型模式
> 这种方法在if处判断了是否初次运行构造函数。初次运行时，this指向的实例对象并没有showInfo这个方法，因此会进入if方法体内进行原型方法的定义。第二次运行时，this指向的实例对象的原型上已经有了showInfo方法，就不会进行二次构造，避免了原型方法的重复定义。除此之外，如果一个方法没有初始化，那代表所有属性和方法都没有初始化，所以只需要对其中一个方法进行判断就好。

>其实一开始也没能发现这种方法比上一种方法的好处，然后查阅发现这是为了解决有其他OO语言开发经验人的困惑，他们对独立开的构造函数和原型感到很困惑，动态原型就可以将所有信息都封装在函数中了。顺便说一句，这种模式产生的对象，可以用instanceof来确定它的类型。

```
function person(name,age){
  this.name=name;
  this.age= age;
  if (typeof this.showInfo=='undefined') {
    person.prototype.showInfo=function(){
      return this.name+' is '+this.age;
    } 
  }
}
var person1 = new person('wang',23);
console.log(person1.showInfo());//wang is 23
```
***
>update at 2017.04.09
## 6.寄生构造函数
> 先不解释这种方法的，先看一个例子。

```
function createPerson(name,age){   
    var person = {};  
    person.name = name;
    person.age = age; 
    person.showInfo = function(){  
        return this.name+' is '+this.age;
    };  
    return person;  
}   
var person1 = new createPerson("wang",23);  
console.log(person1.showInfo());//wang is 23
```

>我自己在学习的时候不够仔细，刚接触这种方法时觉得这不就是工厂函数方法吗？在函数中创建一个新的对象，最后将对象当做返回值，直接看来工厂函数也是这么做的。那么在看一下下一个例子(此例子网上直接拷贝，非原创)。

```
function SpecialArray(){
    var values = new Array();
    values.push.apply(values, arguments);
    values.toPipedString = function(){
        return this.join("|");
    }
    return values;
}
var a = new SpecialArray(2,6,8);
a.toPipedString();//2|6|8
```
>确实工厂构造方法和寄生构造方法没有本质区别，区别就在实例化的时候，工厂函数直接用了函数返回的对象，而寄生方法用了new关键字。new关键字会先新建一个空对象，这个对象会继承该函数的原型，最后this会绑定到新的对象上。所以这里的toPipedString方法会在原型上，也就是说普通的array也可以用toPipedString方法。总结一下就是寄生构造方法可以js原生的构造函数增加一些新的方法。

>这种方法返回的独享和构造函数或构造函数原型之间没什么关系，构造函数产生的对象和构造函数外部创建的对象没什么不同，也就是说不能通过instanceof来确定其对象类型，基于这一点，个人建议还是不要用这种方法了。

## 7.稳妥构造函数模式
>稳妥对象指的是没有公共属性和方法，看下面的代码可以知道，构造过程中没有引用this，生成实例的时候也没有用new方法，只有showInfo方法可以访问到name和age值，并且无论接下来的代码如何给person1加属性，name和age两个值也是不可能改变的。稳妥构造函数模式提供的这种安全性，使得他非常适合在某些安全的环境（可惜没有实战使用过。。。汗）。
>其实个人觉得稳妥构造和工厂还是有点像，都是返回一个对象，不过看下面的例子，可以很直观的说明。

```
function createPerson(name,age){  
    var person = new Object();   
    person.showInfo = function(){  
        console.log(name+' is '+age);
    };   
    return person;  
}   
var person1 = createPerson('wangwei',23);  
person1.showInfo();  //wangwei is 23
console.log(person1.name);//undefined 
person1.name = 'wangxiaowei';
console.log(person1.name);//wangxiaowei
person1.showInfo();  //wangwei is 23

//---------以上为稳妥构造，下面是工厂函数构造------------

function createPerson(name,age){
   var temp=new Object;
   temp.name=name;
   temp.age=age;
   temp.showInfo=function(){
       console.log(this.name+' is '+this.age); 
   }
   return temp;//函数最后返回对象
}
var person1=createPerson('wang',23);
person1.name='wangxiaowei';
person1.showInfo();//wangxiaowei is 23
```