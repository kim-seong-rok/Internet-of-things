# 4장 콜백 함수 및 비동기 처리

## 4-1 콜백 함수 예제 (1-1) - setInterval
```javascript
var count = 0;
var timer = setInterval(function(){
    console.log(count);
    if (++count > 4) clearInterval(timer);
}, 300);
```
`setInterval`은 일정 시간 간격으로 함수를 반복 호출하며, 반환된 ID를 이용해 언제든 종료할 수 있습니다.

## 4-2 콜백 함수 예제 (1-2) - 기명 함수 사용
```javascript
var count = 0;
var cbFunc = function(){
    console.log(count);
    if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);
```
기명 함수로 콜백을 선언해 가독성을 높일 수 있습니다.

## 4-3 콜백 함수 예제 (2-1) - map 메서드 기본 사용
```javascript
var newArr = [10, 20, 30].map(function(currentValue, index){
    console.log(currentValue, index);
    return currentValue + 5;
});
```
`map`은 배열의 각 요소에 대해 콜백을 실행하고 결과값을 새 배열로 반환합니다.

## 4-4 콜백 함수 예제 (2-2) - 인자 순서 오류
```javascript
var newArr2 = [10, 20, 30].map(function(index, currentValue){
    console.log(index, currentValue);
    return currentValue + 5;
});
```
인자 순서를 임의로 바꾸면 의도치 않은 결과를 초래할 수 있습니다.

## 4-5 콜백 함수 예제 (2-3) - map 구현
```javascript
Array.prototype.map = function (callback, thisArg) {
    var mappedArr = [];
    for (var i = 0; i < this.length; i++) {
        var mappedValue = callback.call(thisArg || window, this[i], i, this);
        mappedArr[i] = mappedValue;
    }
    return mappedArr;
};
```
내장 `map` 메서드의 동작 원리를 이해하기 위한 구현 예제입니다.

## 4-6 콜백 함수 내부에서의 this
```javascript
setTimeout(function(){ console.log(this); }, 300);
[1, 2, 3].forEach(function(x){ console.log(this); });
```
콜백 내 `this`는 일반적으로 전역 객체를 가리킵니다.

## 4-7 콜백 함수로 객체 메서드 전달 시 this
```javascript
var obj = {
    vals: [1, 2, 3],
    logValues: function(v, i){ console.log(this, v, i); }
};
[4, 5, 6].forEach(obj.logValues);
```
객체의 메서드를 콜백으로 전달하면 `this`는 전역 객체를 가리킵니다.

## 4-8 this 바인딩 우회 방식
```javascript
var obj1 = {
    name: 'obj1',
    func: function() {
        var self = this;
        return function () {
            console.log(self.name);
        };
    }
};
setTimeout(obj1.func(), 1000);
```
`this`를 `self`에 저장하여 클로저 내부에서 참조합니다.

## 4-9 this 미사용 구조
```javascript
var obj1 = {
    name: 'obj1',
    func: function(){
        console.log(obj1.name);
    }
};
setTimeout(obj1.func, 1000);
```
객체명을 직접 참조하면 `this`를 사용할 필요가 없습니다.

## 4-10 this 재활용
```javascript
var obj1 = {
    name: 'obj1',
    func: function () {
        var self = this;
        return function () {
            console.log(self.name);
        };
    }
};
var obj2 = { name: 'obj2', func: obj1.func };
setTimeout(obj2.func(), 1500);
```
`self`를 이용하면 여러 객체에서 함수 재활용이 가능합니다.

## 4-11 bind를 이용한 this 고정
```javascript
setTimeout(obj1.func.bind(obj1), 1000);
```
`bind`는 함수 실행 시 `this`를 고정합니다.

## 4-12 콜백 지옥 예시
```javascript
setTimeout(function(name){
  var coffeeList = name;
  console.log(coffeeList);
  setTimeout(function(name){
    coffeeList += ', ' + name;
    console.log(coffeeList);
    setTimeout(function(name){
      coffeeList += ', ' + name;
      console.log(coffeeList);
    }, 500, '카페모카');
  }, 500, '아메리카노');
}, 500, '에스프레소');
```
콜백 함수가 중첩되면 가독성이 크게 떨어지는 콜백 지옥 발생.

## 4-13 기명 함수로 콜백 지옥 해결
```javascript
var addEspresso = function(name){ ... };
setTimeout(addEspresso, 500, '에스프레소');
```
콜백을 분리하여 유지보수성과 가독성 향상.

## 4-14 Promise 사용
```javascript
new Promise(function(resolve){ ... }).then(function(preName){ ... });
```
`Promise`로 콜백의 순서를 체계적으로 관리 가능.

## 4-15 Promise + 함수화
```javascript
var addCoffee = function(name) {
    return function(prevName){
        return new Promise(function(resolve){
            setTimeout(function(){
                var newName = prevName ? prevName + ', ' + name : name;
                console.log(newName);
                resolve(newName);
            }, 500);
        });
    };
};
addCoffee('에스프레소')()
.then(addCoffee('아메리카노'))
.then(addCoffee('카페모카'))
.then(addCoffee('카페라떼'));
```
중복 구조를 함수화하여 간결하게 표현.

## 4-16 Generator 이용
```javascript
function* coffeeGenerator() {
    var espresso = yield addCoffee('', '에스프레소');
    ...
}
var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```
`Generator`는 중단 가능한 함수로서 순차 제어에 유용.

## 4-17 async/await 사용
```javascript
var coffeeMaker = async function() {
    await _addCoffee('에스프레소');
    ...
};
coffeeMaker();
```
`async/await`는 비동기 흐름을 동기식처럼 표현하여 가독성이 높고 직관적입니다.
