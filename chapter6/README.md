# 6장 Prototype, Constructor, and Inheritance in JavaScript

## 6-1 Person.prototype
### 생성자 함수 Person의 prototype에 메서드를 정의하면, 인스턴스인 suzi도 해당 메서드에 접근할 수 있습니다.
### 단, __proto__.getName()처럼 직접 접근할 경우, this 바인딩 문제로 원하는 값이 출력되지 않을 수 있습니다.
```javascript
var Person = function(name){
    this._name = name;
}
Person.prototype.getName = function(){
    return this._name
}
var suzi = new Person("Suzi")
console.log(suzi.__proto__.getName())             // undefined
console.log(Person.prototype === suzi.__proto__)  // true
```


## 6-2 prototype과 __proto__
### 생성자 함수의 prototype에 정의된 속성과 메서드는 인스턴스를 통해 마치 자신의 것처럼 접근할 수 있습니다.
### 콘솔에서 prototype과 __proto__의 구조를 비교해보며 차이를 이해할 수 있습니다.

```javascript
var Constructor = function (name){
    this.name = name;
};
Constructor.prototype.method1 = function(){};
Constructor.prototype.property1 = "Constructor Prototype Property";

var instance = new Constructor("Instance");
console.dir(Constructor);
console.dir(instance);
```

## 6-3 constructor 프로퍼티
### 대부분의 프로토타입 객체에는 constructor라는 프로퍼티가 존재하며, 이를 통해 원래의 생성자 함수를 참조할 수 있습니다.

```javascript
var arr = [1,2];
console.log(Array.prototype.constructor === Array)  // true
console.log(arr.__proto__.constructor === Array)    // true
console.log(arr.constructor === Array)              // true

var arr2 = new arr.constructor(3,4);
console.log(arr2);                                  // [3,4]

```

## 6-4 constructor 변경
### 대부분의 값에서 constructor는 변경 가능하지만, 이는 실제 인스턴스의 내부 동작이나 원형에는 영향을 주지 않습니다.

### 단지 constructor 프로퍼티만 바꾸는 것이며, 내부 타입이나 동작은 그대로 유지됩니다.

```javascript
var NewConstructor = function(){
    console.log("this is new constructor!");
};
var dataType = [1, 'test', true, {}, [], function(){}, /test/, new Number(), new String(), new Boolean(), new Object(), new Array(), new Function(), new RegExp(), new Date(), new Error()];

dataType.forEach(function(d){
    d.constructor = NewConstructor;
    console.log(d.constructor.name, '&', d instanceof NewConstructor)
})

```

## 6-5 다양한 constructor 접근 방법
### 다양한 방식으로 constructor를 통해 인스턴스를 생성할 수 있음을 보여주는 예제입니다.

### 프로토타입 체인을 통해 간접적으로 생성자 함수에 접근하는 여러 방법을 실습해볼 수 있습니다.

```javascript
var Person = function(name){
    this.name = name;
}
var p1 = new Person("사람1");
var p1Proto = Object.getPrototypeOf(p1);
var p2 = new Person.prototype.constructor('사람2');
var p3 = new p1Proto.constructor('사람3');
var p4 = new p1.__proto__.constructor('사람4');
var p5 = new p1.constructor('사람5');

[p1,p2,p3,p4,p5].forEach( function(p) {
    console.log(p, p instanceof Person);
});

```

## 6-6 메서드 오버라이드
### 인스턴스에 동일한 이름의 메서드를 정의하면, 프로토타입의 메서드는 가려지며 인스턴스의 것이 우선됩니다. 이를 메서드 오버라이드라고 합니다.
```javascript
var Person = function (name) {
    this.name = name;
};
Person.prototype.getName = function(){
    return this.name;
};
var iu = new Person('지금');
iu.getName = function () {
    return '바로 ' + this.name;
};
console.log(iu.getName()); // 바로 지금

```

## 6-7 배열에서 배열 메서드 및 객체 메서드 실행
### 배열에서 메서드를 실행할 때, 자바스크립트는 __proto__를 따라 올라가며 해당 메서드를 찾습니다.

### 이를 프로토타입 체이닝이라고 하며, 메서드 오버라이드 시에도 같은 규칙이 적용됩니다.
```javascript
var arr = [1,2];
arr.push(3);
console.log(arr.hasOwnProperty(2)); // true

```

## 6-8 메서드 오버라이드와 프로토타입 체이닝
### 프로토타입 체이닝을 통해 메서드를 찾고 실행합니다.

### 동일한 이름의 메서드를 인스턴스에 정의하면 체이닝 순서를 우회하여 오버라이드하게 됩니다.
```javascript
var arr = [1,2];
Array.prototype.toString.call(arr); // "1,2"
Object.prototype.toString.call(arr); // "[object Array]"

arr.toString = function(){
    return this.join('_')
};
console.log(arr.toString()); // "1_2"

```

## 6-9 Object.prototype에 추가한 메서드에의 접근
### 모든 객체의 최상단인 Object.prototype에 메서드를 추가하면, 대부분의 데이터 타입에서 접근할 수 있게 됩니다.

### 하지만 비객체 타입에 대해선 기대한 대로 동작하지 않을 수 있으니 주의해야 합니다.

```javascript
Object.prototype.getEntries = function(){
    var res = [];
    for (var prop in this){
        if(this.hasOwnProperty(prop)){
            res.push([prop,this[prop]]);
        }
    }
    return res;
};

var data = [
    ['object', { a: 1, b: 2, c: 3 }],
    ['number', 345],
    ['string', 'abc'],
    ['boolean', false],
    ['func', function() {}],
    ['array', [1, 2, 3]],
];

data.forEach(function(datum){
    console.log(datum[1].getEntries())
});
```

## 6-10 Grade 생성자 함수와 인스턴스
### 인자 목록을 배열처럼 다뤄 인스턴스를 배열 형태로 구성하는 패턴입니다.

### 생성자 간의 상속 구조를 구성할 때 __proto__를 조작하거나 prototype을 연결해 체인을 구성할 수 있습니다.

```javascript
var Grade = function () {
    var args = Array.prototype.slice.call(arguments);
    for(var i=0;i<args.length;i++){
        this[i] = args[i];
    }
    this.length = args.length;
};
var g = new Grade(100,80);

```

