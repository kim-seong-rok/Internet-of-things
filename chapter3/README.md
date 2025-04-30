# 3-1 전역 공간에서의 this(브라우저 환경)
## 브라우저 환경에서 전역 컨텍스트의 this는 window임
## this === window는 항상 true
```javascript
console.log(this); // { alert: f(), atob: f(), blur: f(), btoa: f(), ... }
console.log(window); // { alert: f(), atob: f(), blur: f(), btoa: f(), ... }
console.log(this === window); // true
```

# 3-2 전역 공간에서의 this(Node.js 환경)
## Node.js에서는 전역 객체가 global임
## 전역 컨텍스트에서 this === global이 성립
```javascript
console.log(this); // { process: { title: 'node', version: 'v10.13.0',... } }
console.log(global); // { process: { title: 'node', version: 'v10.13.0',... } }
console.log(this === global); // true
```

# 3-3 전역변수와 전역객체(1)
## var로 선언한 변수는 window 객체의 프로퍼티가 됨
```javascript
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

# 3-4 전역변수와 전역객체(2)
## var a = 1은 window.a와 동일하며, b = 2도 암묵적으로 전역 객체에 등록
```javascript
var a = 1;
window.b = 2;
console.log(a, window.a, this.a); // 1 1 1
console.log(b, window.b, this.b); // 2 2 2

window.a = 3;
b = 4;
console.log(a, window.a, this.a); // 3 3 3
console.log(b, window.b, this.b); // 4 4 4
```

# 3-5 전역변수와 전역객체(3)
## var로 선언된 변수는 delete로 삭제되지 않음
## 직접 프로퍼티로 추가한 것은 삭제 가능
```javascript
var a = 1;
delete window.a; // false
console.log(a, window.a, this.a); // 1 1 1

var b = 2;
delete b; // false
console.log(b, window.b, this.b); // 2 2 2

window.c = 3;
delete window.c; // true
console.log(c, window.c, this.c); // Uncaught ReferenceError: c is not defined

window.d = 4;
delete d; // true
console.log(d, window.d, this.d); // Uncaught ReferenceError: d is not defined
```

# 3-6 함수로서 호출, 메서드로서 호출
## 일반 함수 호출에서는 this === window
## 객체 메서드로 호출 시 this === 그 객체
```javascript
var func = function(x) {
  console.log(this, x);
};
func(1); // Window { ... } 1

var obj = {
  method: func,
};
obj.method(2); // { method: f } 2
```

# 3-7 메서드로서 호출 - 점 표기법, 대괄호 표기법
## 점(.)이나 괄호([]) 접근 모두 객체 메서드 호출로 this는 객체를 참조
```javascript
var obj = {
  method: function(x) {
    console.log(this, x);
  },
};
obj.method(1); // { method: f } 1
obj['method'](2); // { method: f } 2
```

# 3-8 메서드 내부에서의 this
## 중첩된 객체에서 메서드를 호출하면 this는 해당 메서드가 속한 객체를 참조
```javascript
var obj = {
  methodA: function() {
    console.log(this);
  },
  inner: {
    methodB: function() {
      console.log(this);
    },
  },
};
obj.methodA(); // { methodA: f, inner: {...} }    ( === obj)
obj['methodA'](); // { methodA: f, inner: {...} } ( === obj)

obj.inner.methodB(); // { methodB: f }            ( === obj.inner)
obj.inner['methodB'](); // { methodB: f }         ( === obj.inner)
obj['inner'].methodB(); // { methodB: f }         ( === obj.inner)
obj['inner']['methodB'](); // { methodB: f }      ( === obj.inner)
```

# 3-9 내부함수에서의 this
## outer()는 obj1을 this로 가짐
```javascript
var obj1 = {
  outer: function() {
    console.log(this); // (1)
    var innerFunc = function() {
      console.log(this); // (2) (3)
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

# 3-10 내부함수에서의 this를 우회하는 방법
## innerFunc1은 일반 함수 호출이므로 this === window
```javascript
var obj = {
  outer: function() {
    console.log(this); // (1) { outer: f }
    var innerFunc1 = function() {
      console.log(this); // (2) Window { ... }
    };
    innerFunc1();

    var self = this;
    var innerFunc2 = function() {
      console.log(self); // (3) { outer: f }
    };
    innerFunc2();
  },
};
obj.outer();
```

# 3-11 this를 바인딩하지 않는 함수(화살표 함수)
## 화살표 함수는 this를 바깥에서 가져와서 obj를 가리킴
```javascript
var obj = {
  outer: function() {
    console.log(this); // (1) { outer: f }
    var innerFunc = () => {
      console.log(this); // (2) { outer: f }
    };
    innerFunc();
  },
};
obj.outer();
```

# 3-12 콜백 함수 내부에서의 this
## setTimeout 안은 일반 함수라서 this === window
## forEach 콜백도 마찬가지로 this === window
## 이벤트 리스너 안에서 this는 해당 DOM 요소 (#a 버튼)
```javascript
setTimeout(function() {
  console.log(this);
}, 300); // (1)

[1, 2, 3, 4, 5].forEach(function(x) {
  // (2)
  console.log(this, x);
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector('#a').addEventListener('click', function(e) {
  // (3)
  console.log(this, e);
});
```

# 3-13 생성자 함수
## Cat은 생성자 함수고, new로 객체 만들면 this는 새 인스턴스를 가리킴
## 각각 고양이 객체가 생기고 bark, name, age가 들어감
```javascript
var Cat = function(name, age) {
  this.bark = '야옹';
  this.name = name;
  this.age = age;
};
var choco = new Cat('초코', 7);
var nabi = new Cat('나비', 5);
console.log(choco, nabi);

/* 결과
Cat { bark: '야옹', name: '초코', age: 7 }
Cat { bark: '야옹', name: '나비', age: 5 }
*/
```

# 3-14 call 메서드(1)
## call은 첫 번째 인자로 this를 바꿔서 실행시킴
```javascript
var func = function(a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // Window{ ... } 1 2 3
func.call({ x: 1 }, 4, 5, 6); // { x: 1 } 4 5 6
```

# 3-15 call 메서드(2)
## obj.method는 원래는 obj가 this
## call 쓰면 this를 { a: 4 }로 바꿀 수 있음
```javascript
var obj = {
  a: 1,
  method: function(x, y) {
    console.log(this.a, x, y);
  },
};

obj.method(2, 3); // 1 2 3
obj.method.call({ a: 4 }, 5, 6); // 4 5 6
```

# 3-16 apply 메서드
## apply는 call이랑 거의 같은데, 인자를 배열로 줌
## apply({ x: 1 }, [4, 5, 6]) 식으로 쓰면 됨
```javascript
var func = function(a, b, c) {
  console.log(this, a, b, c);
};
func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6

var obj = {
  a: 1,
  method: function(x, y) {
    console.log(this.a, x, y);
  },
};
obj.method.apply({ a: 4 }, [5, 6]); // 4 5 6
```

# 3-17 call/apply 메서드의 활용 1-1 유사배열객체에 배열 메서드를 적용
## 객체에 length랑 인덱스 있으면 배열 비슷한 객체로 취급 가능
## push.call()로 요소 추가되고, slice.call()로 배열로 변환 가능

```javascript
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
};
Array.prototype.push.call(obj, 'd');
console.log(obj); // { 0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4 }

var arr = Array.prototype.slice.call(obj);
console.log(arr); // [ 'a', 'b', 'c', 'd' ]
```

# 3-18 call/apply 메서드의 활용 1-2 arguments, NodeList에 배열 메서드 적용
## arguments도 배열 아님. 그래서 slice.call(arguments)로 배열처럼 다룰 수 있음
##  querySelectorAll도 NodeList라 같은 방식으로 배열 변환함
```javascript
function a() {
  var argv = Array.prototype.slice.call(arguments);
  argv.forEach(function(arg) {
    console.log(arg);
  });
}
a(1, 2, 3);

document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';
var nodeList = document.querySelectorAll('div');
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function(node) {
  console.log(node);
});
```

# 3-19 call/apply 메서드의 활용 1-3 문자열에 배열 메서드 적용 예시
## push는 문자열에 안 됨
## map, reduce, some, every 같은 메서드는 문자열도 가능
```javascript
var str = 'abc def';

Array.prototype.push.call(str, ', pushed string');
// Error: Cannot assign to read only property 'length' of object [object String]

Array.prototype.concat.call(str, 'string'); // [String {"abc def"}, "string"]

Array.prototype.every.call(str, function(char) {
  return char !== ' ';
}); // false

Array.prototype.some.call(str, function(char) {
  return char === ' ';
}); // true

var newArr = Array.prototype.map.call(str, function(char) {
  return char + '!';
});
console.log(newArr); // ['a!', 'b!', 'c!', ' !', 'd!', 'e!', 'f!']

var newStr = Array.prototype.reduce.apply(str, [
  function(string, char, i) {
    return string + char + i;
  },
  '',
]);
console.log(newStr); // "a0b1c2 3d4e5f6"
```

# 3-20 call/apply 메서드의 활용 1-4 ES6의 Array.from 메서드
## Array.from은 유사 배열 객체를 진짜 배열로 만들어줌
## length랑 인덱스 값만 있으면 배열처럼 변환 가능함
```javascript
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
};
var arr = Array.from(obj);
console.log(arr); // ['a', 'b', 'c']
```

# 3-21 call/apply 메서드의 활용 2 생성자 내부에서 다른 생성자를 호출
## Person.call이나 apply로 상속처럼 생성자 내부 속성 복사
## Student랑 Employee가 각각 Person의 속성 가져옴
## new로 호출하면 this는 각 인스턴스를 가리킴
```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}
function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}
function Employee(name, gender, company) {
  Person.apply(this, [name, gender]);
  this.company = company;
}
var by = new Student('보영', 'female', '단국대');
var jn = new Employee('재난', 'male', '구골');
```

# 3-22 call/apply 메서드의 활용 3-1 최대/최솟값을 구하는 코드를 직접 구현
## forEach로 배열 순회하면서 직접 최대값, 최소값 찾는 방식
## 변수 max, min에 비교하면서 업데이트
## 한 줄에 var max = (min = numbers[0])로 동시에 초기화
```javascript
var numbers = [10, 20, 3, 16, 45];
var max = (min = numbers[0]);
numbers.forEach(function(number) {
  if (number > max) {
    max = number;
  }
  if (number < min) {
    min = number;
  }
});
console.log(max, min); // 45 3
```

# 3-23 call/apply 메서드의 활용 3-2 여러 인수를 받는 메세드(Math.max/Math.min)에 apply 적용
## Math.max.apply(null, 배열)로 배열 최대값 구함
## apply는 인자 배열로 넘겨줘야 함
## min도 똑같이 Math.min.apply로 구함
```javascript
var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);
console.log(max, min); // 45 3
```

# 3-24 callk/apply 메서드의 활용 3-3 ES6의 펼치기 연산자 활용
## 전개 연산자 ...로 배열을 개별 인자로 펼침
## Math.max(...numbers)는 배열 요소를 하나씩 넘긴 것처럼 작동
## apply보다 짧고 간편한 문법
```javascript
const numbers = [10, 20, 3, 16, 45];
const max = Math.max(...numbers);
const min = Math.min(...numbers);
console.log(max, min); // 45 3
```

# 3-25 bind 메서드 - this 지정과 부분 적용 함수 구현
## bind는 this 고정하고 함수 새로 만듦
## bind(thisArg, ...args)로 미리 인자도 고정 가능
## 나중에 실행할 때 남은 인자만 추가로 넘기면 됨
```javascript
var func = function(a, b, c, d) {
  console.log(this, a, b, c, d);
};
func(1, 2, 3, 4); // Window{ ... } 1 2 3 4

var bindFunc1 = func.bind({ x: 1 });
bindFunc1(5, 6, 7, 8); // { x: 1 } 5 6 7 8

var bindFunc2 = func.bind({ x: 1 }, 4, 5);
bindFunc2(6, 7); // { x: 1 } 4 5 6 7
bindFunc2(8, 9); // { x: 1 } 4 5 8 9
```

# 3-26 bind 메서드 - name 프로퍼티
## bind로 만든 함수는 이름이 bound로 시작함
## 원래 함수 이름은 그대로 유지됨
## 함수 이름 확인할 땐 .name 속성 쓰면 됨
```javascript
var func = function(a, b, c, d) {
  console.log(this, a, b, c, d);
};
var bindFunc = func.bind({ x: 1 }, 4, 5);
console.log(func.name); // func
console.log(bindFunc.name); // bound func
```

# 3-27 내부함수에 this 전달-call vs bind
## innerFunc.call(this)로 this를 명시적으로 넘김
## 또는 bind(this)로 고정해서 나중에 호출
## 일반 함수는 바깥 this를 직접 넘겨줘야 같은 객체 가리킴
```javascript
var obj = {
  outer: function() {
    console.log(this);
    var innerFunc = function() {
      console.log(this);
    };
    innerFunc.call(this);
  },
};
obj.outer();

var obj = {
  outer: function() {
    console.log(this);
    var innerFunc = function() {
      console.log(this);
    }.bind(this);
    innerFunc();
  },
};
obj.outer();
```

# 3-28 bind 메서드-내부함수에 this 전달
## setTimeout(this.logThis)는 this가 window로 바뀜
## bind(this)로 넘기면 this가 유지됨
## 타이머 안에서는 함수 참조만 넘기면 this가 달라짐
```javascript
var obj = {
  logThis: function() {
    console.log(this);
  },
  logThisLater1: function() {
    setTimeout(this.logThis, 500);
  },
  logThisLater2: function() {
    setTimeout(this.logThis.bind(this), 1000);
  },
};
obj.logThisLater1(); // Window { ... }
obj.logThisLater2(); // obj { logThis: f, ... }
```

# 3-29 화살표 함수 내부에서의 this
## 화살표 함수는 this를 외부 스코프에서 가져옴
## outer의 this가 그대로 innerFunc에도 적용됨
## obj.outer() 호출 시 this === obj 유지됨
```javascript
var obj = {
  outer: function() {
    console.log(this);
    var innerFunc = () => {
      console.log(this);
    };
    innerFunc();
  },
};
obj.outer();
```

# 3-30 thisArg를 받는 경우 예시-forEach 메서드
## forEach 두 번째 인자로 this 넘기면 콜백에서 this 유지됨
## arguments는 유사 배열이라 slice.call로 배열화
```javascript
var report = {
  sum: 0,
  count: 0,
  add: function() {
    var args = Array.prototype.slice.call(arguments);
    args.forEach(function(entry) {
      this.sum += entry;
      ++this.count;
    }, this);
  },
  average: function() {
    return this.sum / this.count;
  },
};
report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); // 240 3 80
```

# 3-31 콜백 함수와 함께 thisArg를 인자로 받는 메서드
## 배열 메서드 대부분 callback[, thisArg] 형태
## thisArg 주면 콜백 내부 this를 원하는 값으로 설정 가능
## Set, Map도 forEach는 thisArg 쓸 수 있음
```javascript
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(arrayLike[, callback[, thisArg]])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

