# 2-1. 실행 컨텍스트와 콜 스택
## 변수 a는 함수 내부에서 var a = 3으로 재정의됨.
## 함수 실행 시 실행 컨텍스트가 콜 스택에 쌓이고, 가장 위의 컨텍스트부터 실행됨.
## 호이스팅으로 인해 inner 함수의 a는 선언만 먼저 되며 값은 undefined인 상태.
```javascript
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
  }
  inner();
  console.log(a); // 1
}
outer();
console.log(a); // 1

```
# 2-2. 매개변수와 변수의 호이스팅 (1)
## 매개변수도 내부에서 선언된 변수처럼 간주되어 호이스팅됨.
## 실제 동작은 2-3과 동일.
```javascript
function a(x) {
  console.log(x); // 1
  var x;
  console.log(x); // 1
  var x = 2;
  console.log(x); // 2
}
a(1);

```

# 2-3. 매개변수를 변수 선언처럼 변환한 코드
## 매개변수를 변수로 선언한 것처럼 취급하여 설명.
```javascript
function a() {
  var x = 1;
  console.log(x); // 1
  var x;
  console.log(x); // 1
  var x = 2;
  console.log(x); // 2
}
a();

```

# 2-4. 호이스팅 후의 실제 실행 구조
## 선언은 위로, 할당은 원래 위치에 남아 있는 전형적인 호이스팅 구조.
```javascript
function a() {
  var x;
  var x;
  var x;

  x = 1;
  console.log(x); // 1
  console.log(x); // 1
  x = 2;
  console.log(x); // 2
}
a();

```

# 2-5. 함수 선언의 호이스팅 (1)
## 함수 선언은 전체가 호이스팅됨. 변수는 선언만 위로 올라감.
```javascript
function a() {
  console.log(b); // [Function: b]
  var b = 'bbb';
  console.log(b); // 'bbb'
  function b() {}
  console.log(b); // 'bbb'
}
a();
```

# 2-6. 함수 선언의 호이스팅 구조 설명
```javascript
function a() {
  var b;
  function b() {}

  console.log(b); // [Function: b]
  b = 'bbb';
  console.log(b); // 'bbb'
  console.log(b); // 'bbb'
}
a();

```

# 2-7. 함수 선언문 → 함수 표현식으로 변환
## 동일 동작. 함수 표현식으로 구조를 바꿔 설명.
```javascript
function a() {
  var b;
  var b = function b() {}

  console.log(b);
  b = 'bbb';
  console.log(b);
  console.log(b);
}
a();
```

# 2-8. 함수 정의 방식 3가지
## 함수 표현식에서 내부 이름(d)은 외부에서 접근 불가.
```javascript
function a() {}
a();

var b = function () {}
b();

var c = function d() {}
c();
d(); // 에러 발생

```

# 2-9. 선언문 vs 표현식 차이 (1)
## sum은 정상 작동. multiply는 할당 전에 호출되어 에러 발생.
```javascript
console.log(sum(1, 2));
console.log(multiply(3, 4));

function sum(a, b) {
  return a + b;
}

var multiply = function (a, b) {
  return a * b;
};

```


# 2-10. 위 코드를 호이스팅한 상태로 표현
```javascript
var sum = function sum(a, b) {
  return a + b;
};

var multiply;

console.log(sum(1, 2));
console.log(multiply(3, 4)); // TypeError

multiply = function (a, b) {
  return a * b;
};

```


# 2-11. 함수 선언의 위험성
## 동일 이름의 함수가 여러 번 선언되면 마지막 선언이 적용됨.
```javascript
console.log(sum(3, 4));

function sum(x, y) {
  return x + y;
}

var a = sum(1, 2);

function sum(x, y) {
  return x + '+' + y + '=' + (x + y);
}

var c = sum(1, 2);
console.log(c);

```

# 2-12. 함수 표현식의 안전성
## 함수 표현식을 사용하면 의도치 않은 덮어쓰기 방지 가능.
```javascript
console.log(sum(3, 4)); // TypeError

var sum = function (x, y) {
  return x + y;
};

var a = sum(1, 2);

var sum = function (x, y) {
  return x + '+' + y + '=' + (x + y);
};

var c = sum(1, 2);
console.log(c);

```

# 2-13. 스코프 체인
## 스코프 체인 탐색 시, 가장 가까운 식별자를 먼저 찾음.
## inner 함수 내 a는 호이스팅되어 undefined 출력.
```javascript
var a = 1;
var outer = function () {
  var inner = function () {
    console.log(a);
    var a = 3;
  };
  inner();
  console.log(a);
};
outer();
console.log(a);

```

# 2.14 스코프 체인 확인(1) - 크롬 전용

```javascript
var a = 1;
var outer = function() {
  var b = 2;
  var inner = function() {
    console.dir(inner);
  };
  inner();
};
outer();

```

# 2.15 스코프 체인 확인(2) - 크롬 전용
```javascript
var a = 1;
var outer = function() {
  var b = 2;
  var inner = function() {
    console.log(b);
    console.dir(inner);
  };
  inner();
};
outer();
```

# 2.16 스코프 체인 확인(3) 
```javascript
var a = 1;
var outer = function() {
  var b = 2;
  var inner = function() {
    console.log(b);
    debugger;
  };
  inner();
};
outer();
```