---
title: javascript继承方法
date: 2017-04-07 17:10:09
categories:
  - javascript学习
---
# javascript 继承
继承是很多面向对象语言中让人津津乐道的概念，ECMAScript中的继承主要是靠原型链来实现。
## 1.原型链方法
>原型链实现继承的基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法（其实个人认为继承还是主要以方法为主）。简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象中包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。所以如果让原型对象等于另一个类型的实例，此时的原型对象就会包含一个指向另一个原型的指针，另一个原型中也包含着一个指向另一个构造函数的指针。如此层层推进，就构成了实例和原型之间的链条。下面提供一个原型链继承的例子。

```
function SuperType(){
  this.property = true;
}
SuperType.prototype.getValue=function(){
  return this.property;
}
function SubType(){
  this.subProperty = true;
}
SubType.prototype = new SuperType();
SuperType.prototype.getSubValue=function(){
  return this.subProperty;
}
var instance = new SubType();
console.log(instance.getValue());//true
```
>上面代码其实定义了两个类型：SuperType和SubType，每个类型都属于自己的一个属性和方法。主要在于SubType继承了SuperType，继承的方法是通过创建SuperType的实例并将实例赋给了SubType.prototype，本质上是重写了原型对象。所以结果就是instance指向SubType的原型，SubType又指向了SuperType的原型。getValue方法仍然在SuperType.prototype中，但是property则位于SubType.prototype中。这是因为property是一个实例属性，而getValue是一个原型方法。不过在这里要注意instance.constructer现在是指向SuperType，这是因为SubType.prototype中的constructer被重写了。

>使用原型链方法其实有一个小的注意点，我们在创建原型方法时不能使用对象字面量，因为这种方法会重写原型链,如下例：

```
function SuperType(){
  this.property = true;
}
SuperType.prototype.getValue=function(){
  return this.property;
}
function SubType(){
  this.subProperty = true;
}
SubType.prototype = new SuperType();
SuperType.prototype={
  getSubValue:function(){
     return this.subProperty;
  }
}
var instance = new SubType();
console.log(instance.getValue());//error
```
>除此之外，原型链继承也还是有一些问题的，最主要的是包含引用类型值的原型，如下例：

```
function Super(){
    this.val = 1;
    this.arr = [1];
}
function Sub(){
    // ...
}
Sub.prototype = new Super();    var sub1 = new Sub();
var sub2 = new Sub();
sub1.val = 2;
sub1.arr.push(2);
alert(sub1.val);    // 2
alert(sub2.val);    // 1

alert(sub1.arr);    // 1, 2
alert(sub2.arr);    // 1, 2
```

>原型链的第二个问题就是在创建子类的实例时，不能向超类型的构造函数中传递参数。其实按照我的理解应该说是无法再不影响实例对象的情况下，给超类型的构造函数传递参数，所以平时我一般不考虑用这种方法。

## 2.借用构造函数

>这种方法的思想很简单，就是直接在子类构造函数的内部调用超类构造函数，用call或者apply来改变this的指向，其实就是在我们未来要新创建的Sub实例的环境下调用Super的方法，如下例：

```
function Super(val){
    this.val = val;
    this.arr = [1];

    this.fun = function(){
        // ...
    }
}
function Sub(val){
    Super.call(this, val);
    this.name = 'sub';      
}
var sub1 = new Sub(1);
var sub2 = new Sub(2);
sub1.arr.push(2);
console.log(sub1.val);    // 1
console.log(sub2.val);    // 2
console.log(sub2.name);    // sub
console.log(sub1.arr);    // 1, 2
console.log(sub2.arr);    // 1
console.log(sub1.fun === sub2.fun);   // false
```
>从上面的例子可以看出来，借用构造函数是可以向超类构造函数中传递参数的，如例子中的name属性。为了确保Super构造函数不会重写子类的属性，可以在调用超类构造函数后，再添加相应的子类。

>借用构造函数也是有一定问题的，所有的方法都是在构造函数中定义，函数复用就无从谈起了。而且超类原型中定义的方法对子类是不可见的，结果所有的类型就只能使用构造函数模式了，所以平时我也是很少用这个方法。

## 3.组合继承

>组合继承指的是将原型链和借用构造函数组合到一起的一种继承模式，主要思路就是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样既通过在原型上定义方法实现了函数复用，又能保证每个实例都有自己的属性。

```
function Super(val){
    // 只在此处声明基本属性和引用属性
    this.val = val;
    this.arr = [1];
}
//  在此处声明函数，把实例函数都放在原型对象上，以实现函数复用
Super.prototype.fun = function(){};
//Super.prototype.fun3...
function Sub(val,name){
    Super.call(this,val); 
    this.name = name; 
}
Sub.prototype.sayName = function(){
	 console.log(this.name);
}
Sub.prototype = new Super(); 

var sub1 = new Sub(1);
var sub2 = new Sub(2);
alert(sub1.fun === sub2.fun);   // true
```

>上面的例子中，Super构造函数有arr和value两个属性，原型上有fun这个方法。Sub构造函数在调用Super构造函数时传入了val参数，然后有定义了属于自己的name属性。最后将Super的实例赋给Sub的原型，然后在该新原型上定义了sayName方法。这样一来可以让两个不同的Sub实例既拥有属于自己的属性，又可以使用相同的方法。

>但是这种方法也不是完全没有缺点的，可以看出在这个过程中超类使用了两次，一次在创建子类原型的时候，另一次是在子类构造函数内部，但是使用过程中，子类的实例属性会屏蔽原型属性，也就是说某些原型属性其实是用不上的，这造成了内存的浪费。

## 4.原型式继承
>这种方法其实没有严格意义上的构造函数，主体思想是基于已有的对象并借助原型构建新的对象。

```
function inherit(obj){
  function temp(){};
  temp.prototype = obj;
  return new temp;
}
```

>参照上述主题思想，在inherit函数内部先创建一个临时的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回这个临时原型的新实例。本质上讲，这就是对传入对象进行了一次浅复制。这种方法一开始是一个叫克劳克福德的老外提出来的，现在es5中有Object.create()方法，这个方法接收两个参数，一个用作新对象原型的对象，一个为新对象增加额外属性属性的对象。在只传入一个参数的情况下，上述inherit方法和Object.create()方法效果一样。

```
var person = {
  name:'wang',
  hobbies:['shoes','ballgames']
};
var person1 = Object.create(person);
person1.hobbies.push('code');
console.log(person.hobbies);//'shoes','ballgames','code'
```

>包含引用类型值的属性始终会共享相应的值，就像原型模式一样。如果不想构建创造函数，只是想让一个对象和另一个对象保持类似的情况下，可以考虑使用原型书继承，别的情况下不建议使用。

***
update at 2017.04.15

## 5.寄生式继承

>这种方法的思路和寄生构造函数和工厂模式类似，就是创建一个仅用于封装继承过程的函数，该函数的内部以别的方法来增强对象，最后再返回对象。

```
function createAnother(original){
  var clone = Object.create(original);
  clone.show = function(){
     //
  }
  return clone;
}
```

>上述例子中，接收的参宿就是将要作为新对象基础的对象，最后再给clone对象增加新的方法。新的对象不仅有original所有的属性和方法，还有自己的show方法。使用这种继承方法给对象添加函数，会由于不能做到函数复用而降低效率。

## 6.寄生组合式继承

>之前说过组合继承中超类使用了两次从而造成内存浪费，浪费的原因主要是在原型和实例中都会出现重名的属性。寄生组合式继承是通过借用构造函数来继承属性，通过原型链的混成形式来继承方法；基本思路就是不必为了指定子类型的原型而调用超类的构造函数，我们所需要的其实就是一个超类型的副本。简单说就是使用寄生式来继承超类的原型，再将结果给子类的原型。

```
function inheritPrototype(Sub,Super){
  var prototype = Object.create(Super.prototype);//创建对象
  prototype.constructor = Sub;//增强对象
  Sub.prototype = prototype;//指定对象
}
```

>上述函数接收两个参数，分别为子类构造函数和超类构造函数。函数内部第一步常见超类原型的一个副本，第二步为创建的副本添加constructer属性，弥补一下因为重写原型而失去的默认的constructer属性，第三部将新创建的对象赋给子类的原型。

```
function Super(val){
    this.val = val;
    this.arr = [1];
}
Super.prototype.fun = function(){
	 alert(this.val);
};
function Sub(val,name){
    Super.call(this，val); 
    this.name = name; 
}
inheritPrototype(Sub,Super);
Sub.prototype.sayName = function(){
	 console.log(this.name);
}

var sub1 = new Sub(1,'wang');
var sub2 = new Sub(2,'wei');
alert(sub1.fun === sub2.fun);   // true
sub1.sayName();//wang
sub1.arr.push(111);
console.log(sub1.arr);//[1,111]
console.log(sub2.arr);//[1]
```

>相对于组合继承，这个方法最大的优势就在于他只调用了一席Super构造函数，因此避免了在Sub.prototype上创建不必要的属性，与系统是，原型链还能保持不变，简直完美！！！