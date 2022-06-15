## 构造函数的继承
让一个构造函数继承另一个构造函数，是非常常见的需求。这可以分成两步实现。第一步是在子类的构造函数中，调用父类的构造函数。
``` javascript
function Sub(value) {
  Super.call(this);
  this.prop = value;
}
```

上面代码中，Sub是子类的构造函数，this是子类的实例。在实例上调用父类的构造函数Super，就会让子类实例具有父类实例的属性。
第二步，是让子类的原型指向父类的原型，这样子类就可以继承父类原型。
``` javascript
Sub.prototype = Object.create(Super.prototype);
```
//让子类的原型指向父类的原型，这样子类就可以继承父类原型。Object.create()用父类创造了一个新对象(子类), 并把子类对象的__proto__指向父类对象，使父成为子的原型对象，玩成继承。
``` javascript
Sub.prototype.constructor = Sub;
Sub.prototype.method = '...';
```

上面代码中，Sub.prototype是子类的原型，要将它赋值为Object.create(Super.prototype)，而不是直接等于Super.prototype。否则后面两行对Sub.prototype的操作，会连父类的原型Super.prototype一起修改掉。
另外一种写法是Sub.prototype等于一个父类实例。
``` javascript
Sub.prototype = new Super();
``` javascript

上面这种写法也有继承的效果，但是子类会具有父类实例的方法。有时，这可能不是我们需要的，所以不推荐使用这种写法。
举例来说，下面是一个Shape构造函数。
``` javascript
function Shape() {
  this.x = 0;
  this.y = 0;
}

Shape.prototype.move = function (x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};
```

我们需要让Rectangle构造函数继承Shape。
// 第一步，子类继承父类的实例
``` javascript
function Rectangle() {
  Shape.call(this); // 调用父类构造函数
}
```

// 另一种写法
``` javascript
function Rectangle() {
  this.base = Shape;
  this.base();
}
``` javascript

// 第二步，子类继承父类的原型
``` javascript
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;
```

采用这样的写法以后，instanceof运算符会对子类和父类的构造函数，都返回true。
``` javascript
var rect = new Rectangle();

rect instanceof Rectangle  // true
rect instanceof Shape  // true
```

上面代码中，子类是整体继承父类。有时只需要单个方法的继承，这时可以采用下面的写法。
``` javascript
ClassB.prototype.print = function() {
  ClassA.prototype.print.call(this);
  // some code
}
```

上面代码中，子类B的print方法先调用父类A的print方法，再部署自己的代码。这就等于继承了父类A的print方法。
