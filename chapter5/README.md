위에 잘했어 밑에 5장의 README도 너가 했던것처럼 하나하나 다른사람이 적은 것처럼 한번 바
꿔줘봐

Explations on the examples in chapter 5
5-1 외부 함수의 변수를 참조하는 내부 함수(1)
var outer = function() {
    var a = 1;
    var inner = function() {
      console.log(++a);
    };
    inner();
  };
  outer();

클로저란 "어떤 함수 에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상" 이다.
해당 예제는 외부 함수의 변수를 참조하는 내부 함수에 관한 예제이다.
5-2 외부 함수의 변수를 참조하는 내부 함수(2)
var outer = function () {
    var a = 1;
    var inner = function(){
        return ++a;
    };
    return inner();
};
var outer2 = outer();
console.log(outer2)

inner함수 내부에서 외부 변수인 a를 사용했지만, inner함수를 실행한 결과를 리턴하고 있기 때문에, outer 함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 사라진다. 
a와 inner변수의 값들은 가비지 컬렉터에 의해 소멸하게 된다.
예제 5-3 외부 함수의 변수를 참조하는 내부 함수(3)
var outer = function(){
    var a = 1;
    var inner = function(){
        return ++a;
    };
    return inner;
};
var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3

가비지 컬렉터의 동작방식 때문에 inner함수의 실행 시점에 outer함수가 종료 상태이지만 out함수의 Lexicalenviroment에 접근할 수 있게 된다.
가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않는다.
5-4 return 없이도 클로저가 발생하는 다양한 경우
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
5-5 클로저의 메모리 관리
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
5-6 콜백 함수와 클로저(1)
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
5-7 콜백 함수와 클로저(2)
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
5-8 콜백 함수와 클로저(3)
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
5-9 콜백 함수와 클로저(4)
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
5-10 간단한 자동차 객체
var car = {
    fuel: Math.ceil(Math.random()*10+10), // 연료(L)
    power: Math.ceil(Math.random()*3+2), // 연비(km/L)
    moved:0,
    run: function(){
        var km = Math.ceil(Math.random()*6);
        var wasteFuel = km / this.power;
        if(this.fuel < wasteFuel){
            console.log('이동불가');
            return;
        }
        this.fuel -= wasteFuel;
        this.moved += km;
        console.log(km+'km 이동 (총 '+this.moved+'km)');
    }
};

정보 은닉은 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 중요한 개념이다.
자바스크립트의 경우 클로저를 이용한다.
위 예제의 경우 멤버 변수 변경에 취약하다.
5-11 클로저로 변수를 보호한 자동차 객체(1)
var createCar = function(){
    var fuel = Math.ceil(Math.random()*10+10); // 연료(L)
    var power = Math.ceil(Math.random()*3+2); // 연비(km/L)
    var moved = 0;
    return {
        get moved(){
            return moved;
        },
        run: function(){
            var km = Math.ceil(Math.random()*6);
            var wasteFuel = km / this.power;
            if(this.fuel < wasteFuel){
                console.log('이동불가');
                return;
            }
            fuel -= wasteFuel;
            moved += km;
            console.log(km+'km 이동 (총 '+this.moved+'km). 남은 연료: '+fuel);
        }
    };
};

var car = createCar()

car.run(); // 3km 이동(총 3km). 남은 연료: 17.4
console.log(car.moved); // 3
console.log(car.fuel) // undeifined
console.log(car.power); // undeifined

car.fuel = 1000;
console.log(car.fuel); // 1000
car.run(); //1km 이동(총 4km). 남은 연료: 17.2

car.power = 100;
console.log(car.power); // 100
car.run(); //4km 이동(총 8km). 남은 연료: 16.4

car.moved = 1000;
console.log(car.moved); // 8
car.run(); //2km 이동(총 10km). 남은 연료: 16

이번에는 createCar라는 함수를 실행함으로써 객체를 생성하게 했다.
fuel,power변수를 비공개 멤버로 지정해 외부에서의 접근을 제한했고, moved 변수는 getter만을 부여함으로써 읽기 전용 속성을 부여하였다.
이제 외부에서 오직 run 메서드를 실행하는 것과 현재의 moved 값 확인하는 두가지 동작만 할 수 있다.
하지만 run 메서드를 다른 내용으로 덮어씌우는 등의 어뷰징이 가능한 상태이다.
5-12 클로저로 변수를 보호한 자동차 객체(2)
var createCar = function(){
    var publicMembers={
        get moved(){
            return moved;
        },
        run: function(){
            var km = Math.ceil(Math.random()*6);
            var wasteFuel = km / this.power;
            if(this.fuel < wasteFuel){
                console.log('이동불가');
                return;
            }
            fuel -= wasteFuel;
            moved += km;
            console.log(km+'km 이동 (총 '+this.moved+'km). 남은 연료: '+fuel);
        }
    };
    Object.freeze(publicMembers);
    return publicMembers
};
var car = createCar()

객체를 return 하기 전에 미리 변경할 수 없게 조치를 취하였다.
5-13 bind 메서드를 활용한 부분 적용 함수
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
5-14 부분 적용 함수 구현(1)
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
5-15 부분 적용 함수 구현(2)
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
5-16 부분 적용 함수 - 디바운스
var debounce = function(eventName,func,wait){
    var timeoutId = null;
    return function (event){
        var self = this;
        console.log(eventName,'event 발생');
        clearTimeout(timeoutId);
        timeoutId = setTimeout(func.bind(self,event),wait);
    };
};

var moveHandler = function(e){
    console.log('move event 처리');
};
var wheelHandler = function (e){
    console.log('wheel evnet 처리')
}

document.body.addEventListener('mouseover',debounce('move',moveHandler,500));
document.body.addEventListener('mousewheel',debounce('wheel',wheelHandler,700));


실무에서 부분 함수를 사용하기에 적합한 예시이다.
디바운스는 짧은 시간 동안 동일한 이벤트가 많이 발생할 경우 이를 전부 처리허지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한번만 처리를 하는 것이다.
5-17 커링 함수(1)
var curry3 = function(func){
    return function (a){
        return function(b){
            return func(a,b)
        };
    };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8));   //10
console.log(getMaxWith10(25));  //25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8));  //8
console.log(getMinWith10(25));  //10


커링 함수란 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것이다.
5-18 커링 함수(2)
var curry5 = function (func){
    return function (a){
        return function (b){
            return function (c){
                return function (d){
                    return function (e){
                        return func(a,b,c,d,e)
                    }
        
                }
            }
        }
    }
}
var getMax = curry5(Math.max);
console.log(getMax(1)(2)(3)(4)(5));

인자가 많아지면 가독성이 떨어지는 단점이 있다.
ES 6에서 화살표 함수를 써서 다음과 같이 한 줄로 표기 가능 하다.


var curry5 = func => a=>b=>c=>d=>e=>func(a,b,c,d,e);

당장 필요한 정보만 받아서 전달하고 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 떄까지 함수 실행을 미루는 셈이다.
이를 함수형 프로그래밍에서는 지연실행이라고 한다.