# 콜백 함수

## 4. 콜백 함수 내부의 this에 다른 값 바인딩하기
```javascript
var obj = {
  vals: [1, 2, 3],
  logValues: function(v, i) {
    console.log(this, v, i);
  }
};
obj.logValues(1, 2); // { val: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues);
// window { ... } 4 0
// window { ... } 5 1
// window { ... } 6 1
```
위 코드를 통해 객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없게 된다는 점을 알게 되었습니다.
그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고 싶다면 어떻게 해야 할까요?

별도의 인자로 this를 받는 함수의 경우에는 여기에 원하는 값을 넘겨주면 되지만 그렇지 않은 경우에는 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀 수 없습니다.

그래서 전통적으로는 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용하게 하고, 이를 클로저로 만드는 방식이 많이 쓰였습니다.

```javascript
var obj1 = {
  name: 'obj1',
  func: function() {
    var self = this; // 미리 obj1.func() 메서드를 통해 클로저로 self에 this(obj1) 값 저장
    return function() {
      console.log(self.name);
    };
  }
};
var callback = obj1.func(); // 여기서 obj1.func()로 호출 했을 때 func 함수 내부의 this는 obj1이다.
setTimeout(callback, 1000); // 1초 후에 기존에 저장 된 self.name 값을 호출한다 -> 'obj1'
```



#### Bind를 이용한 this에 다른 값 바인딩하는 방법
```javascript
var obj1 = {
  name: 'obj1',
  func: function() {
    console.log(this.name);
  }
};
setTimeout(obj1.func.bind(obj1), 1000); // bind를 통해 this의 값을 명시적으로 obj1로 설정하여 새로운 함수로 반환해줍니다.
// 1초 후에 bind로 반환 받은 함수가 실행 됩니다. 그렇게 obj1.name이 콘솔에 출력 됩니다.

var obj2 = { name: 'obj2' };
setTimeout(obj1.func.bind(obj2), 1500); // bind를 통해 this의 값을 명시적으로 obj2로 설정하여 새로운 함수로 반환해줍니다.
// 1.5초 후에 bind로 반환 받은 함수가 실행 됩니다. 그렇게 obj2.name이 콘솔에 출력 됩니다.
```

<br/>

## 5. 콜백 지옥과 비동기 제어

### 콜백 지옥 (callback hell)
콜백 지옥은 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상을 의미합니다.

콜백 지옥의 가장 큰 문제는 **가독성이 떨어질뿐더러 코드를 수정하기도 어렵다는 것 입니다.**

![callbackhell](https://github.com/prgrms-web-devcourse/FEDC5_core_javascript_study/assets/114329713/9252c61f-15ef-4f09-9a8b-458138d5a170)

아래 코드를 통해 콜백 지옥의 예시 코드를 살펴보도록 하겠습니다.
```javascript
setTimeout(function(name) {
    var coffeeList = name;
    console.log(coffeeList);
    setTimeout(function(name) {
        coffeeList += ', ' + name;
        console.log(coffeeList);
        setTimeout(function(name) {
            coffeeList += ', ' + name;
            console.log(coffeeList);
            setTimeout(function(name) {
                coffeeList += ', ' + name;
                console.log(coffeeList);
            }, 500, '카페라떼')
        }, 500, '카페모카')
    }, 500, '아메리카노')
}, 500, '에스프레소');
```

위 코드를 보면 어떤 생각이 드시나요?

저는 이런 가독성이 떨어지는 코드를 보면 코드를 읽기가 싫어지고.. 읽다가 어디까지 읽었는지 놓치는 경우도 많은거 같습니다.

어떻게 코드가 동작할 지 예상하기가 힘든 코드 입니다.

위와 같은 가독성 문제를 해결하는 방법으로 익명의 콜백 함수를 모두 기명함수로 전환하는 것이 있습니다.

``` javascript
var coffeeList = '';

var addEspresso = function(name) {
    coffeeList = name;
    console.log(coffeeList);
    setTimeout(addAmericano, 500, '아메리카노');
};

var addAmericano = function(name) {
    coffeeList += ', ' + name;
    console.log(coffeeList);
    setTimeout(addMocha, 500, '카페모카');
};

var addMocha = function(name) {
    coffeeList += ', ' + name;
    console.log(coffeeList);
    setTimeout(addLatte, 500, '카페라떼');
};

var addLatte = function(name) {
    coffeeList += ', ' + name;
    console.log(coffeeList);
};

setTimeout(addEspresso, 500, '에스프레소');
```
어떠신가요? 이 방식은 코드의 가독성을 높일뿐 아니라 함수 선언과 호출만 구분할 수 있다면,
위에서부터 아래로 순서대로 읽어내려가는 데 어려움이 없습니다.

하지만, 일회성 함수를 전부 변수에 할당하는 것이 메모리 측면에서 좋지 않을 것입니다.
또한 변수명을 일일이 따라다녀야 하므로 오히려 헷갈릴 소지가 있기도 합니다.

이러한 비동기적인 일련의 작업을 동기적으로, 혹은 동기적인 것처럼 보이게끔 처리해주는 장치가 있습니다.

바로 ES6에서 나온 **`Promise/Generator`** 와 ES2017에서 나온 **`async/await`**  입니다.

<br/>

> ✅ **우리는 이러한 Promise, async/await를 알아보기 전 간단하게 동기와 비동기가 무엇인지 살펴보도록 하겠습니다.**

<br/>
<br/>

![hh3Mawr](https://github.com/prgrms-web-devcourse/FEDC5_core_javascript_study/assets/114329713/0eb6358f-d975-461c-b9dd-44ae33653a06)

### 동기적 (Synchronous)
먼저 시작된 하나의 작업이 끝날 때까지 다른 작업을 시작하지 않고 기다렸다가 다 끝나면 새로운 작업을 시작하는 방식입니다. 

위 그림과 같이 작업이 직렬로 배치되어 실행되며 작업 실행의 순서가 확실히 정해져 있는 것을 동기식 처리라 부릅니다.

<br/>

### 비동기적 (Asynchronous)
동기식 방식과는 다르게 먼저 시작된 작업의 완료 여부와는 상관없이 새로운 작업을 시작하는 방식입니다.

위 그림과 같이 작업이 병렬로 배치되어 실행되며 작업의 순서가 확실하지 않아 나중에 시작된 작업이 먼저 끝나는 경우도 발생합니다.

이와 같은 방식을 비동기식 처리라 부릅니다.

대표적으로 **`SetTimeout`** , **`addEventListener`** , **`fetch`** , **`XMLHttpRequest`** 등 **별도의 요청**, **실행 대기**, **보류** 등과 관련된 코드는 비동기적인 코드 입니다.

<br/>

### new Promise(executor)
`Promise`는 ES6에서 추가된 자바스크립트 내장 객체입니다.

Promise는 생성할 때 매개변수로 `executor`를 전달받게 됩니다.

이러한 executor는 (resolve, reject)라는 두 가지 매개변수를 전달받습니다.

**resolve**는 비동기작업이 성공했을 때 호출되게 되고, **reject**는 작업이 실패했을 때 호출되게 됩니다.

```javascript
new Promise((resolve, reject) => {
    setTimeout(() => {
        var name = '에스프레소';
        console.log(name)
        resolve(name)
    }, 500)
}).then(prevName => {
    return new Promise(resolve => {
        setTimeout(() => {
            var name = prevName + ', 아메리카노';
            console.log(name);
            resolve(name);
        }, 500)
    })
}).then(prevName => {
    return new Promise(resolve => {
        setTimeout(() => {
            var name = prevName + ', 카페모카';
            console.log(name);
            resolve(name);
        }, 500)
    })
}).then(prevName => {
    return new Promise(resolve => {
        setTimeout(() => {
            var name = prevName + ', 카페라떼';
            console.log(name);
            resolve(name);
        }, 500)
    })
})
```
new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만 내부 resolve, reject 함수를 호출하는 구문이 있을 경우 둘 중 하나가 실행되기 전까지는
다음(then) 또는 오류(catch) 구문으로 넘어가지 않습니다.

여기서 then을 붙여 사용하는건 개발자 마음입니다. 하루 있다 사용해도 되고 한시간 있다가 사용해도 됩니다.

이제 Generator를 활용하여 비동기 작업을 처리해 보겠습니다.

<br/>

### 제네레이터(Generator)란?
제네레이터(Generator)는 중간에 원하는 부분에서 멈추었다가, 그 부분부터 다시 실행할 수 있는 능력을 가진 함수입니다.

제네레이터 함수는 function * 키워드를 사용하며, 아래와 같이 작성하면 됩니다.
```javascript
function* generator() {...}
```

```javascript
function* infinity(i = 0) {
    while (true) yield i++;
}
const a = infinity();
console.log(a.next()); // {value: 0, done: false}
console.log(a.next()); // {value: 1, done: false}
console.log(a.next()); // {value: 2, done: false}
console.log(a.next()); // {value: 3, done: false}
```
`yield`는 제네레이터 함수를 멈추거나 다시 시작하는데 사용하는 키워드 입니다.
`next` 메서드를 사용하면 `yield` 키워드가 사용된 표현식까지 실행되고 함수가 일시 중단됩니다.
바로 이때 **함수의 제어권이 함수 호출자에게 양도 됩니다.**

```javascript
var addCoffee = function(prevName, name) {
    setTimeout(function() {
        coffeeMaker.next(prevName ? prevName + ', ' + name : name)
    }, 500);

};

var coffeeGenerator = function*() {
    var espresso = yield addCoffee('', '에스프레소');
    console.log(espresso);
    var americano = yield addCoffee(espresso, '아메리카노');
    console.log(americano);
    var mocha = yield addCoffee(americano, '카페모카');
    console.log(mocha);
    var latte = yield addCoffee(mocha, '카페라떼');
    console.log(latte);
}

var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```
- ‘*’가 붙은 coffeeGenerator 함수가 바로 Generator 함수입니다.
- Generator 함수를 실행하면 Iterator가 반환되는데, Iterator는 next라는 메서드를 가집니다.
- 이 next 메서드를 호출하면 Generator 함수 내부에서 가장 먼저 등장하는 yield에서 함수의 실행을 멈춥니다.
- 이후 다시 next 메서드를 호출하면 앞서 멈췄던 부분부터 시작해서 다음 yield에서 함수의 실행을 멈춥니다. (반복)
- 따라서, 비동기 작업이 완료되는 시점마다 next 메서드를 호출해준다면 Generator 함수 내부의 소스가 위에서부터 아래로 순차적으로 진행되는 것을 알 수 있습니다.

<br/>

### async / await
async와 await는 자바스크립트의 비동기 처리 패턴 중 가장 최근에 나온 문법입니다.
기존의 비동기 처리 방식인 콜백 함수와 프로미스의 단점을 보완하고 개발자가 읽기 좋은 코드를 작성할 수 있게 도와줍니다.
특히, 복잡했던 Promise를 조금 더 편하게 사용할 수 있습니다.

```javascript
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(name);
    }, 500);
  });
};
var coffeeMaker = async function () {
  var coffeeList = "";
  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? "," : "") + (await addCoffee(name));
  };

  await _addCoffee("아메리카노");
  console.log(coffeeList);
  await _addCoffee("카페모카");
  console.log(coffeeList);
  await _addCoffee("카페라떼");
  console.log(coffeeList);
  await _addCoffee("에스프레소");
  console.log(coffeeList);
};

coffeeMaker();
```
- ES2017에서는 가독성도 뛰어나고, 작성법도 간단한 async/await 기능이 추가되었습니다.
- 비동기 작업을 수행하고자 하는 함수 앞에 async를 표기하고, 함수 내부에서 실질적인 비동기 작업이 필요한 위치마다 await를 표기하면 뒤의 내용을 Promise로 자동 전환하고, 해당 내용이 resolve 된 이후 다음으로 진행합니다.
- async/await를 사용하면 Promise의 then과 흡사한 효과 얻을 수 있습니다.

<br/>

**then과의 차이점은 async/await는 호출된 곳에서 비동기 작업이 종료될 때까지 기다렸다가 완료되면 다음 작업으로 넘어갑니다.**
- 비동기 작업을 동기처럼 사용하는 경우에는 **`async/await`** 를 사용하면 됩니다.
- `Promise`가 실행되는 시점과 후속 작업을 하는 시점을 달리해야한다면 `then`을 사용하면 됩니다.

<br/>

## 6. 정리
- 콜백 함수는 다른 코드에 인자로 넘겨줌으로써 그 **제어권**도 함께 위임한 함수입니다.
- 제어권을 넘겨받은 코드는 다음과 같은 제어권을 가집니다.
1) 콜백 함수를 호출하는 시점을 스스로 판단해서 실행합니다.
2) 콜백 함수를 호출할 때 인자로 넘겨줄 값들의 순서가 정해져 있습니다. ex ) array.map(function(value, index, array){)
   이 순서를 따르지 않고 코드를 작성하면 엉뚱한 결과를 얻게 됩니다.
3) 콜백 함수의 **this를 바꾸고 싶을 경우 bind 메서드를 활용**하면 됩니다. (this가 무엇을 바라보도록 할지 정해져 있는 경우도 있지만, 없다면 전역객체를 바라보게 됩니다.)
- 어떤 함수에 인자로 메서드를 전달하더라도 이는 결국 함수로서 실행됩니다.