# 1-1. 변수선언

```javascript
var a; # 식별자, 변수명 variable a 선언
```

# 1-2. 변수 선언과 할당
## 식별자 = 데이터 형태로 바로 메모리에 할당된다.
## variable : 변경가능 , constant : 변경 불가
## primitive type : 불변, reference type : 가변
```javascript
var a; # 변수 a 선언 
a = 'abc';  #변수 a에 대해 'abc' 데이터 할당 
var a = 'abc'; # 변수 a를 선언과 동시에 데이터 할당  
```

# 1-3. 불변성
## 기본형은 모두 불변
## 값이 바뀔 경우 기존 데이터는 유지, 새로운 데이터가 다른 메모리 주소에 생성됨
```javascript
# 해당 과정에서 데이터는 사라지지 않고 추가적인 주소에 할당이 된다.
var a = 'abc';
a = a + 'def';

var b = 5;
var c = 5;
b = 7;
```

# 1-4. 참조형 데이터의 할당
## key-value 형식
## obj1, a, b 모두 식별자
## 식별자들은 identifier area에 값은 data area에 저장됨
```javascript
var obj1 = { a: 1, b: 'bbb' };
```

# 1-5. 참조형 데이터의 프로퍼티 재할당
## obj1은 참조형 데이터
## 기존 값 1을 바꾸는 게 아니라, 새로운 값 2의 주소를 할당
```javascript
var obj1 = { a: 1, b: 'bbb' };
obj1.a = 2;
```

# 1-6. 중첩된 참조형 데이터(객체)의 프로퍼티 할당
## 배열 내부 값들은 인덱스를 통해 접근
## x와 arr[0]은 같은 3 값을 가리킬 수 있음(중복 참조 가능)
```javascript
var obj = { x: 3, arr: [3, 4, 5] };
```

# 1-7. 변수 복사
##  기본형은 값 복사, 참조형은 주소 복사
```javascript
var a = 10;
var b = a;
var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;
```

# 1-8. 변수 복사 이후 값 변경 결과 비교 (1) - 객체의 프로퍼티 변경 시
## b는 별개지만 obj1.c 도 같이 바뀜(obj1과 obj2가 같은 객체 참조)

```javascript
var a = 10;
var b = a;
var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;

b = 15;
obj2.c = 20;
```

# 1-9. 변수 복사 이후 값 변경 결과 비교 (2) - 객체 자체를 변경했을 때
## obj2는 새 객체를 참조하게 되어 obj1과의 연결이 끊김

```javascript
var a = 10;
var b = a;
var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;

b = 15;
obj2 = { c: 20, d: 'ddd' };
```

# 1-10. 객체의 가변성에 따른 문제점
## user와 user2가 같은 객체를 참조 → 둘 다 값이 변경됨
```javascript
var user = { name: "Jaenam", gender: "male" };
var changeName = function (user, newName) {
  var newUser = user;
  newUser.name = newName;
  return newUser;
};
var user2 = changeName(user, "Jung");
```

# 1-11. 객체의 가변성에 따른 문제 해결 방법
## 새로운 객체를 반환해 불변성을 유지함
```javascript
var changeName = function(user, newName) {
  return {
    name: newName,
    gender: user.gender
  };
};
```


# 1-12. 기존 정보를 복사해서 새로운 객체를 반환하는 함수 (얕은 복사)
```javascript
var copyObject = function (target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
};
```

# 1-13. copyObject를 이용한 객체 복사
## user와 user2는 서로 다른 객체
```javascript
var user = { name: "Jaenam", gender: "male" };
var user2 = copyObject(user);
user2.name = "Jung";
```

# 1-14. 중첩된 객체에 대한 얕은 복사
## urls처럼 중첩 객체는 얕은 복사로 해결되지 않음
```javascript
var user = {
  name: 'Jaenam',
  urls: {
    portfolio: 'http://github.com/abc',
    blog: 'http://blog.com',
    facebook: 'http://facebook.com/abc'
  }
};
var user2 = copyObject(user);
user2.name = 'Jung';
user.urls.portfolio = 'http://portfolio.com';
```

# 1-15. 중첩된 객체에 대한 깊은 복사
## 중첩 객체도 각각 복사하여 참조 분리
```javascript
var user2 = copyObject(user);
user2.urls = copyObject(user.urls);
```

# 1-16. 객체의 깊은 복사를 수행하는 범용 함수
## 재귀 방식으로 깊은 복사 구현
```javascript
var copyObjectDeep = function(target) {
  var result = {};
  if (typeof target === 'object' && target !== null) {
    for (var prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
};
```

# 1-17. 깊은 복사 결과 확인
## obj와 obj2는 완전히 분리된 객체
```javascript
var obj = {
  a: 1,
  b: {
    c: null,
    d: [1, 2]
  }
};
var obj2 = copyObjectDeep(obj);
```

# 1-18. JSON을 활용한 간단한 깊은 복사
## 함수는 복사되지 않음 → 순수 데이터 복사에만 적합
```javascript
var copyObjectViaJSON = function(target) {
  return JSON.parse(JSON.stringify(target));
};
```

# 1-19. 자동으로 undefined를 부여하는 경우
## 명시적 undefined 대신 null 사용 권장
```javascript
var a;
console.log(a); // undefined

var obj = { a: 1 };
console.log(obj.a); // 1
console.log(obj.b); // undefined

var func = function() {};
var c = func();
console.log(c); // undefined
```

# 1-20. undefined와 배열
## undefined와 비어있는 항목은 구분됨
```javascript
var arr1 = [];
arr1.length = 3;
console.log(arr1); // [ <3 empty items> ]

var arr3 = [undefined, undefined, undefined];
console.log(arr3); // [undefined, undefined, undefined]
```

# 1-21. 빈 요소와 배열의 순회
## forEach, map, filter, reduce는 undefined와 빈 값에서 다른 결과를 보여줌
```javascript
var arr1 = [undefined, 1];
var arr2 = [];
arr2[1] = 1;
```

# 1-22. undefined와 null의 비교
## typeof null === 'object'는 JS 버그
## === 사용으로 정확히 구별 가능

```javascript
var n = null;
console.log(typeof n); // object
console.log(n == undefined); // true
console.log(n === undefined); // false
```