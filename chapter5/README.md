# 5장 클로저와 고급 함수 기법 예제 모음

## 5-1 외부 함수의 변수를 참조하는 내부 함수(1)
```javascript
var outer = function() {
  var a = 1;
  var inner = function() {
    console.log(++a);
  };
  inner();
};
outer();
```
> 클로저란 "외부 함수에서 선언한 변수를 참조하는 내부 함수에서만 발생하는 현상"이다. 가장 기본적인 형태의 클로저 구조를 보여준다.

---

## 5-2 외부 함수의 변수를 참조하는 내부 함수(2)
```javascript
var outer = function () {
  var a = 1;
  var inner = function(){
    return ++a;
  };
  return inner();
};
var outer2 = outer();
console.log(outer2);
```
> 내부 함수를 반환하지 않고 실행만 했기 때문에 outer 컨텍스트가 종료되며 클로저가 유지되지 않는다.

---

## 5-3 외부 함수의 변수를 참조하는 내부 함수(3)
```javascript
var outer = function(){
  var a = 1;
  var inner = function(){
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2());
console.log(outer2());
```
> inner 함수가 반환되며 외부 변수 a를 참조하는 클로저가 유지된다.

---

## 5-4 return 없이도 클로저가 발생하는 다양한 경우
### (1) setInterval/setTimeout 예제
```javascript
// (1) setInterval/setTimeout
(function(){
    var a = 0;
    var intervalId = null;
    var inner = function(){
        if(++a>=10){
            clearInterval(intervalId);
        }
        console.log(a);
    };
    intervalId = setInterval(inner,1000);
})();

### (2) DOM 이벤트 리스너 예제
// (2) eventListener
(function(){
    var count = 0;
    var button = document.createElement('button');
    button.innerText='click';
    button.addEventListener('click',function(){
        console.log(++count,'times clicked');
    });
    document.body.appendChild(button);
})();

(1)의 경우 별도의 외부 객체인 window의 메서드에 전달할 콜백 함수 내부에서 지역변수를 참조한다.
(2)의 경우 별도의 외부객체인 DOM의 메서드에 등록할 handler 함수 내부에서 지역변수를 참조한다.
두 상황 모두 지역변수를 참조하는 내부함수를 외부에 전달했기 때문에 클로저이다.
> 반환 없이도 외부 함수 컨텍스트 내 지역변수를 참조하는 경우 클로저가 생성됨을 보여준다.
```

---

## 5-5 클로저의 메모리 관리
### 클로저의 수명과 해제 시점, 메모리 누수 방지를 위한 패턴을 설명.
```javascript
// (1) return 에 의한 클로저의 메모리 해제
var outer = (function(){
    var a = 1;
    var inner = function(){
        return ++a;
    };
    return inner;
})();
console.log(outer());
console.log(outer());
outer = null; // outer 식별자의 inner 함수 참조를 끊음

// (2) setInterval에 의한 클로저의 메모리 해제
(function(){
    var a = 0;
    var intervalId = null;
    var inner = function(){
        if (++a>=10){
            clearInterval(intervalId);
            inner = null;   // inner 식별자의 함수 참조를 끊음
        }
        console.log(a);
    };
    intervalId = setInterval(inner,1000);
})();

// (3) eventListener에 의한 클로저의 메모리 해제
(function(){
    var count = 0;
    var button = document.createElement('button');
    button.innerText='click';

    var clickHandler = function(){
        console.log(++count,'times clicked');
        if(count>=10){
            button.removeEventListener('click',clickHandler);
            clickHandler=null; // clickHandler 식별자의 함수 참조를 끊음
        }
    }
})

클로저의 특성을 정확히 이해해야 메모리 누수등의 위험을 최소화 할 수 있다.
하지만, 최근의 자바스크립트 엔진에서는 위와 같은 위험이 크게 줄어들었고, 개발자의 의도하에 생기는 메모리 소비에 대한 관리법만 잘 파악해서 적용하면 충분하다.
```

---

## 5-6 콜백 함수와 클로저(1)
```javascript
- 이벤트 리스너 내 클로저 생성 방식
- `bind`를 이용한 고정 인자 전달
- 고차 함수를 통한 클로저 활용
var fruits = ['apple','banana','peach'];
var $ul = document.createElement('ul');

fruits.forEach(function(fruit){
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click',function(){
        alert('your choice is '+fruit);
    });
    $ul.appendChild($li);
});
document.body.appendChild($ul);

대표적인 콜백 함수 중 하나인 이벤트 리스너에 관한 예시이다.
클로저의 '외부 데이터'에 주목하면서 흐름을 따라가자.
```

---
## 5-7 콜백 함수와 클로저(2)
```javascript
var fruits = ['apple','banana','peach'];
var $ul = document.createElement('ul');

var alertFruit = function (fruit){
    alert('your choice is '+fruit);
};

fruits.forEach(function(fruit){
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click',alertFruit);
    $ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);

공통 함수로 쓰고자 콜백 함수를 외부로 꺼내어 alertFruit라는 변수에 담았다.
이제 alertFruit을 직접 실행할 수 있다.
그런데 각 li를 클릭하면 클릭한 대상의 과일명이 아닌 [object MouseEvent]라는 값이 출력된다.
콜백 함수의 인자에 대한 제어권을 addEventListener가 가진 상태이며, add EventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문이다.
```
---

## 5-8 콜백 함수와 클로저(3)
```javascript
var fruits = ['apple','banana','peach'];
var $ul = document.createElement('ul');

var alertFruit = function (fruit){
    alert('your choice is '+fruit);
};

fruits.forEach(function(fruit){
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click',alertFruit.bind(null,fruit));
    $ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);

예제 5-7의 [object MouseEvent]라는 값이 출력되는 문제를 해결했다.
하지만 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점 및 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야 한다.
이런 변경사항이 발생하지 않게끔 하면서 이슈를 해결하기 위해 bind 메서드가 아닌 고차함수를 활용한다.
```
---

## 5-9 콜백 함수와 클로저(4)
```javascript
var fruits = ['apple','banana','peach'];
var $ul = document.createElement('ul');

var alertFruitBuilder = function (fruit){
    return function(){
        alert('your choice is '+fruit);
    }
};

fruits.forEach(function(fruit){
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click',alertFruitBuilder(fruit));
    $ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);

alertFruit 함수 대신 alertFruitBuilder라는 이름의 함수를 작성하였다.
이 함수 내부에서는 다시 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit 함수이다.
이렇게 작성하면 함수의 실행 결과가 다시 함수가 되며, 반환된 함수를 리스너에 콜백 함수로써 전달할 것이다.
이후 언젠가 클릭 이벤트가 발생하면 비로소 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있게된다.
즉 alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재한다.
```

---

## 5-10 간단한 자동차 객체
> 정보 은닉이 적용되지 않은 구조로, 클로저 도입의 필요성을 보여줌.

---

## 5-11 클로저로 변수를 보호한 자동차 객체(1)
> 클로저를 활용한 은닉 변수 구성, getter만 제공하는 구조.

---

## 5-12 클로저로 변수를 보호한 자동차 객체(2)
> `Object.freeze`로 외부 수정도 차단한 구조.

---

## 5-13 bind 메서드를 활용한 부분 적용 함수
```javascript
- `bind`를 통한 부분 적용
- 직접 구현한 `partial` 함수
- 자리 지정자 `_` 활용하여 인자 순서 유연화

var add = function(){
    var result = 0;
    for (var i = 0 ; i< arguments.length;i++){
        result += arguments[i];
    }
    return result;
};
var addPartial = add.bind(null,1,2,3,4,5);
console.log(addPartial(6,7,8,9,10)); // 55

부분 적용함수란 n개의 인자를 받는 함수에 미리 n개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 한 함수이다.
this를 바인딩해야 하는 점을 제외하면 앞서 살펴본 bind 메서드의 실행결과가 바로 부분 적용 함수이다.
위 add 함수는 this의 값을 변경할 수 밖에 없기 때문에 메서드에는 사용할 수 없다.
```
---

## 5-14 부분 적용 함수 구현(1)
```javascript
var partial = function(){
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if(typeof func !=='function'){
        throw new Error('첫 번째 인자가 함수가 아닙니다.');
    }
    return function(){
        var partialArgs = Array.prototype.slice.call(originalPartialArgs,1);
        var restArgs = Array.prototype.slice.call(arguments);
        return func.apply(this,partialArgs.concat(restArgs));
    };
};

var add = function(){
    var result = 0;
    for (var i = 0 ; i < arguments.length;i++){
        result += arguments[i];
    }
    return result;
};
var addPartial = partial(add,1,2,3,4,5);
console.log(addPartial(6,7,8,9,10)); // 55

var dog = {
    name : '강아지',
    greet: partial(function(prefix,suffix){
        return prefix+this.name+suffix;
    }, '왈왈, ')
};
console.log(dog.greet('입니다.')); // 왈왈, 강아지 입니다.

this에 관여하지 않는 별도의 부분 적용 함수는 범용성 측면에서 더욱 좋다.
```
---

## 5-15 부분 적용 함수 구현(2)
```javascript
Object.defineProperty(window,'_',{
    value: 'EMPTY_SPACE',
    writable: false,
    configurable: false,
    enumerable: false
});
var partial2 = function(){
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if (typeof func != 'function'){
        throw new Error('첫 번째 인자가 함수가 아닙니다.');
    }
    return function(){
        var partialArgs = Array.prototype.slice.call(originalPartialArgs,1);
        var restArgs = Array.prototype.slice.call(arguments);
        for (var i=0;i<partialArgs.length; i++){
            if(partialArgs[i] === _){
                partialArgs[i] = restArgs.shift();
            }
        }
        return func.apply(this, partialArgs.concat(restArgs));
    };
};
var add = function(){
    var result = 0;
    for (var i = 0;i<arguments.length;i++){
        result+= arguments[i];
    }
    return result;
};
var addPartial = partial2(add,1,2,_,4,5,_,_,8,9);
console.log(addPartial(3,6,7,10)); // 55
var dog = {
    name:'강아지',
    greet: partial2(function(prefix,suffix){
        return prefix+this.name+suffix;
    }, '왈왈, ')
};
console.log(dog.greet("배고파요!")); // 왈왈, 강아지 배고파요!

5-14 예제에서 추가로 원하는 위치에 미리 넣어놓고 나중에 빈 자리에 인자를 채워넣어 실행할 수 있도록 하였다.
'비워놓음'을 표시학 ㅣ위해 미리 전역객체에 _라는 프로퍼티를 준비하면서 삭제 변경 등의 접근에 대한 방어 차원에서 여러 가지 프로퍼티 속성을 설정하였다.
```
---
## 5-16 디바운스 구현
> 실무에서 사용되는 디바운스 로직 구현 예제.

---

## 5-17 커링 함수(1)
> 2단계 커링 함수 구조, `Math.max`에 적용한 사례.

---

## 5-18 커링 함수(2)
> 5단계 커링 구조 및 화살표 함수 적용 예제.

```javascript
const curry5 = func => a => b => c => d => e => func(a, b, c, d, e);
```
> 커링 함수는 인자들을 나눠서 순차적으로 받을 수 있게 하며, 함수형 프로그래밍에서 지연 실행을 가능하게 한다.

---
