# 7장 Chapter 7 - Class, Prototype, Inheritance

## 7-1 스태틱 메서드, 프로토타입 메서드
### 인스턴스에서 직접 호출할 수 없는 메서드는 스태틱 메서드, 직접 호출 가능한 것은 프로토타입 메서드라 한다.

```javascript
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
Rectangle.isRectangle = function(instance) {
  return (
    instance instanceof Rectangle && instance.width > 0 && instance.height > 0
  );
};

var rect1 = new Rectangle(3, 4);
console.log(rect1.getArea()); // 12 (O)
console.log(rect1.isRectangle(rect1)); // Error (X)
console.log(Rectangle.isRectangle(rect1)); // true
```

## 7-2 6-2-4절의 Grade 생성자 함수 및 인스턴스
### 생성자 함수에 정의된 isRectangle은 클래스 자체에서 호출하는 정적(static) 메서드이다.
### 반면 인스턴스 메서드 getArea는 prototype에 정의되어 인스턴스를 통해 접근할 수 있다.
### 생성자 함수와 배열 기반 상속 구조
### Grade.prototype = []을 통해 배열 메서드를 상속하는 구조를 갖는다.
### 기본적으로는 배열처럼 동작하지만 완전한 서브클래스 구조는 아니며, length와 같은 속성은 여전히 문제가 된다.

```javascript
var Grade = function() {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);
```


## 7-3 length 프로퍼티를 삭제한 경우
### Grade 인스턴스는 일반 객체이므로 length 속성을 삭제할 수 있다.
### 이로 인해 배열 메서드 사용 시 동작이 달라지거나 비정상 동작이 발생할 수 있다.

```javascript
var Grade = function() {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);

g.push(90);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }
delete g.length;
g.push(70);
console.log(g); // Grade { 0: 70, 1: 80, 2: 90, length: 1 }
```


## 7-4 요소가 있는 배열을 prototype에 매칭한 경우
### Grade.prototype = ['a', 'b', 'c']처럼 값을 가진 배열을 prototype으로 설정하면 인스턴스에 예기치 않은 영향을 줄 수 있다.
### 클래스는 메서드만 포함된 추상적 틀로 유지되어야 하며, 구체적인 데이터를 가지면 안 된다.

```javascript
var Grade = function() {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = ['a', 'b', 'c', 'd'];
var g = new Grade(100, 80);

g.push(90);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }

delete g.length;
g.push(70);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, ___ 4: 70, length: 5 }
```


## 7-5 Recctangle.Square 클래스
### 두 클래스 모두 width를 가지고 있고, getArea의 동작도 유사하다.
### Square에서 height를 생략하고 width만 사용하면 두 클래스의 구조를 통일시킬 수 있다.

```javascript
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var rect = new Rectangle(3, 4);
console.log(rect.getArea()); // 12

var Square = function(width) {
  this.width = width;
};
Square.prototype.getArea = function() {
  return this.width * this.width;
};
var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-6 Square 클래스 변형
### height에 width 값을 복사함으로써 Rectangle과 동일한 인터페이스를 제공하게 된다.
### 이를 통해 getArea는 공통 메서드로 활용될 수 있다.
```javascript
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var rect = new Rectangle(3, 4);
console.log(rect.getArea()); // 12

var Square = function(width) {
  this.width = width;
  this.height = width;
};
Square.prototype.getArea = function() {
  return this.width * this.height;
};

var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-7 Rectangle을 상속하는 Square 클래스
### Rectangle.call(this, width, width)를 통해 부모 생성자를 호출하고,
### prototype을 new Rectangle()로 설정하지만 이는 데이터까지 상속되어 구조적으로 적절하지 않다.
```javascript
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var rect = new Rectangle(3, 4);
console.log(rect.getArea()); // 12

var Square = function(width) {
  Rectangle.call(this, width, width);
};
Square.prototype = new Rectangle();

var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-8 클래스 상속 및 추상화 방법(1) - 인스턴스 생성 후 프로퍼티 제거
### new SuperClass()로 프로토타입 설정 후, 모든 데이터를 삭제해 메서드만 남김.
### 이후 필요한 메서드를 subMethods로 명시적으로 추가함.
```javascript
var extendClass1 = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = new SuperClass();
  for (var prop in SubClass.prototype) {
    if (SubClass.prototype.hasOwnProperty(prop)) {
      delete SubClass.prototype[prop];
    }
  }
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};

var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = extendClass1(Rectangle, function(width) {
  Rectangle.call(this, width, width);
});
var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-9 클래스 상속 및 추상화 방법(2) - 빈 함수를 활용
### 중간 다리 역할의 빈 함수 Bridge를 사용하여 인스턴스를 생성하지 않고 프로토타입만 연결함.
### 클로저로 함수 생성을 캡슐화하여 메모리 낭비도 줄임.

```javascript
var extendClass2 = (function() {
  var Bridge = function() {};
  return function(SuperClass, SubClass, subMethods) {
    Bridge.prototype = SuperClass.prototype;
    SubClass.prototype = new Bridge();
    if (subMethods) {
      for (var method in subMethods) {
        SubClass.prototype[method] = subMethods[method];
      }
    }
    Object.freeze(SubClass.prototype);
    return SubClass;
  };
})();

var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = extendClass2(Rectangle, function(width) {
  Rectangle.call(this, width, width);
});
var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-10 클래스 상속 및 추상화 방법(3) - Object.create 활용
### Object.create를 활용하여 prototype 체인을 구성하면서도 인스턴스를 직접 생성하지 않음.
### 다른 방식보다 더 안전하고 간결하게 클래스 상속 구조를 설계할 수 있다.

```javascript
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = function(width) {
  Rectangle.call(this, width, width);
};
Square.prototype = Object.create(Rectangle.prototype);
Object.freeze(Square.prototype);

var sq = new Square(5);
console.log(sq.getArea()); // 25
```


## 7-11 클래스 상속 및 추상화 방법 - 완성본(1) - 인스턴스 생성 후 프로퍼티 제거
### SubClass.prototype.constructor를 SubClass로 명시적으로 재지정함으로써 생성자 정보 보완.

```javascript
var extendClass1 = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = new SuperClass();
  for (var prop in SubClass.prototype) {
    if (SubClass.prototype.hasOwnProperty(prop)) {
      delete SubClass.prototype[prop];
    }
  }
  SubClass.prototype.consturctor = SubClass;
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};
```


## 7-12 클래스 상속 및 추상화 방법 - 완성본(2) - 빈 함수를 활용
### 7-9의 방식에서 constructor 속성을 보완하여 SubClass를 정확히 가리키게 함.
```javascript
var extendClass2 = (function() {
  var Bridge = function() {};
  return function(SuperClass, SubClass, subMethods) {
    Bridge.prototype = SuperClass.prototype;
    SubClass.prototype = new Bridge();
    SubClass.prototype.consturctor = SubClass;
    Bridge.prototype.constructor = SuperClass;
    if (subMethods) {
      for (var method in subMethods) {
        SubClass.prototype[method] = subMethods[method];
      }
    }
    Object.freeze(SubClass.prototype);
    return SubClass;
  };
})();
```


## 7-13 클래스 상속 및 추상화 방법 - 완성본(3) - Object.create 활용
### Object.create를 통한 프로토타입 연결과 함께 constructor를 SubClass로 수정하여 완성도 향상.
```javascript
var extendClass3 = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};
```


## 7-14 상위 클래스 접근 수단인 super 메서드 추가
### 하위 클래스에서 상위 클래스의 생성자 또는 메서드를 호출할 수 있도록 super 메서드를 구현.
### this.super() 또는 this.super('methodName')() 방식으로 상위 기능에 접근 가능.
```javascript
var extendClass = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  SubClass.prototype.super = function(propName) {
    // 추가된 부분 시작
    var self = this;
    if (!propName)
      return function() {
        SuperClass.apply(self, arguments);
      };
    var prop = SuperClass.prototype[propName];
    if (typeof prop !== 'function') return prop;
    return function() {
      return prop.apply(self, arguments);
    };
  }; // 추가된 부분 끝
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};

var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = extendClass(
  Rectangle,
  function(width) {
    this.super()(width, width); // super 사용 (1)
  },
  {
    getArea: function() {
      console.log('size is :', this.super('getArea')()); // super 사용 (2)
    },
  }
);
var sq = new Square(10);
sq.getArea(); // size is : 100
console.log(sq.super('getArea')()); // 100
```


## 7-15 ES5와 ES6의 클래스 문법 비교
### ES5에서는 함수 기반 생성자와 prototype을 활용.
### ES6에서는 class, constructor, static, extends 등의 키워드로 더 명시적이고 간결한 문법을 제공함.

```javascript
var ES5 = function(name) {
  this.name = name;
};
ES5.staticMethod = function() {
  return this.name + ' staticMethod';
};
ES5.prototype.method = function() {
  return this.name + ' method';
};
var es5Instance = new ES5('es5');
console.log(ES5.staticMethod()); // es5 staticMethod
console.log(es5Instance.method()); // es5 method

var ES6 = class {
  constructor(name) {
    this.name = name;
  }
  static staticMethod() {
    return this.name + ' staticMethod';
  }
  method() {
    return this.name + ' method';
  }
};
var es6Instance = new ES6('es6');
console.log(ES6.staticMethod()); // es6 staticMethod
console.log(es6Instance.method()); // es6 method
```


## 7-16 ES6의 클래스 상속
### class A extends B 구조를 통해 상속을 표현.
### super() 호출로 부모 클래스 생성자 및 메서드를 호출 가능.
### 7-14의 커스텀 super 메서드와 비교해볼 수 있다.
```javascript
var Rectangle = class {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
};
var Square = class extends Rectangle {
  constructor(width) {
    super(width, width);
  }
  getArea() {
    console.log('size is :', super.getArea());
  }
};
```
