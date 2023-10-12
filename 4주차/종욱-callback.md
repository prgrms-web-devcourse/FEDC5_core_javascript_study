## 1. 콜백 함수란?

다른 코드의 인자로 넘겨주는 함수

정의 : 함수의 파라미터로 들어가는 함수
용도 : 순차적으로 실행하고 싶을 때 씀
다른데서 만든 함수도 콜백함수로 넣을 수 있음

B는 알람 시계를 세팅합니다.
6시가 되자 시계의 알람소리를 듣고 일어납니다.

B는 시계의 알람을 설정하는 함수를 호출하고, 해당 함수는 호출 당시에는 아무것도 하지 않다가 B가 정해준 시각이 됐을 때 비로소 ‘알람을 울리는’ 결과를 반환했습니다.

B는 시계 함수에게 요청을 하면서 알람을 울리는 명령에 대한 **제어권**을 시계에게 넘겨줬습니다. 이처럼 콜백 함수는 **제어권**과 관련이 깊습니다.

위의 예시를 간단하게 코드로 구현하면 다음과 같습니다.

```jsx

const 시계 = 시계의 알람을 설정하는 함수(시간){
	console.log('6시가 되면 알람을 울려줘')
}

시계(6시)

// 시계의 알람을 설정하는 함수 : 콜백함수를 받은 함수,
// 시간 : 콜백함수
```

callback은 되돌아 호출해달라 는 뜻으로, 어떤 함수 X를 **`호출`** 하면서 특정 조건일 때 함수 Y를 실행해서 결과를 나에게 **`되돌려줘`** 라는 명령입니다.

이처럼 다른 코드(함수 또는 메서드)에게 인자로 함수(콜백 함수)를 넘겨줌으로써 콜백 함수에 대한 제어권도 넘겨줍니다. 콜백 함수를 받은 코드는 자체적인 내부 로직에 의해 이 콜백 함수를 적절한 시점에 실행할 것입니다.

## 2. 제어권

몇 가지 예제를 통해 구체적으로 살펴보겠습니다.

### 2-1 호출 시점

```jsx
var count = 0;

var cbFunc = function () {
    console.log(count);
    if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);

// setInterval의 인자로 넘겨주는 함수 === 콜백 함수
// 콜백 함수 === cbFunc

// setInterval 함수는 첫번째 인자로 받은 값을
// 두번째 인자 (밀리초(ms)) 마다 실행합니다.

// -- 실행 결과 --
// 0 (0.3초)
// 1 (0.6초)
// 2 (0.9초)
// 3 (1.2초)
// 4 (1.5초)
```

이처럼 setInterval 이라고 하는 **`다른 코드`** 에 첫 번째 인자로 cbFunc 함수를 넘겨주자 **`제어권을 넘겨받은 setInterval`** 이 내부 로직을 통해 적절한 시점에(0.3초마다) 이 cbFunc 함수를 실행했습니다.

이처럼 콜백함수의 제어권을 넘겨받은 코드는 **`콜백함수 호출 시점에 대한 제어권`** 을 가집니다.

### 2-1 인자

먼저 map에 대해 간략하게 알아보겠습니다.

```jsx
// map 메서드는 메서드의 대상이 되는 배열의 모든 요소들을 처음부터 끝까지 하나씩 꺼내어
// 콜백 함수를 반복 호출하고, 콜백 함수의 실행 결과들을 모아 새로운 배열을 만듭니다.
// 콜백 함수의 첫 번째 인자에는 배열의 요소 중 현재값
// 두 번째 인자에는 현재값의 인덱스
// 세 번째 인자에는 map 메서드의 대상이 되는 배열 자체가 담깁니다.
Array.prototype.map(callback[ , thisArg])
callback: function(currentValue, index, array)
```

예시를 통해 구체적으로 살펴보겠습니다.

```jsx
// map 메서드는 첫 번째 인자로 콜백 함수를 받고,
// 생략 가능한 두 번째 인자로 콜백 함수 내부에서 this로 인식할 대상을 특정할 수 있습니다.
// thisArg를 생략할 경우 일반적인 함수와 마찬가지로 전역객체가 바인딩됩니다.
// 이는 뒤에서 자세하게 알아보겠습니다.

var newArr = [10, 20, 30].map(function (currentValue, index) {
    console.log(currentValue, index);
    return currentValue + 5;
});
console.log(newArr);
// -- 실행 결과 --
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

map 메서드를 호출해서 원하는 배열을 얻으려면 map 메서드에 정의된 규칙에 따라 함수를 작성해야 합니다.
콜백 함수를 호출하는 주체가 map 메서드이므로 map 메서드가 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 **`순서`** 로 넘길 것인지가 전적으로 map 메서드에 달린 것입니다.

이처럼 콜백 함수의 제어권을 넘겨받은 코드는 **`콜백 함수의 인자에 어떤 값들을 어떤 순서로 넘길 것`** 인지에 대한 제어권을 가집니다.

### 2-2 this

콜백 함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만, 별도의 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 됩니다. 별도의 this를 지정하는 방식을 map 예시를 통해 살펴보겠습니다.

```jsx
Array.prototype.map = function (callback, thisArg) {
    var mappedArr = [];
    for (var i = 0; i < this.length; i++) {
        var mappedValue = callback.call(thisArg || window, this[i], i, this);
        mappedArr[i] = mappedValue;
    }
    return mappedArr;
};
// 메서드 구현의 핵심은 call/apply 메서드에 있습니다.
// this에는 thisArg 값이 있을 경우에는 그 값을, 없을 경우에는 전역객체를 지정하고,
// 첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로 배열의 i번째 요소 값을,
// 두 번째 인자에는 i 값을,
// 세 번째 인자에는 배열 자체를 지정해 호출합니다.
// 그 결과가 변수 mappedValue에 담겨 mappedArr의 i번째 인자에 할당됩니다.
```

간단한 예시를 살펴보겠습니다.

```jsx
// thisArg 설정
var newArr2 = [10, 20, 30].map(function (idx, currentValue) {
    console.log(idx, currentValue, this);
    return currentValue + 5;
}, "this");
console.log(newArr2);
// -- 실행 결과 --
// 10 0 [String: 'this']
// 20 1 [String: 'this']
// 30 2 [String: 'this']
// [15, 25, 35]

// thisArg 생략 -> this에 전역 객체
var newArr2 = [10, 20, 30].map(function (idx, currentValue) {
    console.log(idx, currentValue, this);
    return currentValue + 5;
});
console.log(newArr2);
// -- 실행 결과 --
// 10 0 <ref *1> Object [global] { ... }
// 20 1 <ref *1> Object [global] { ... }
// 30 2 <ref *1> Object [global] { ... }
// [15, 25, 35]
```

이처럼 제어권을 넘겨받을 코드에서 call/ apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩 하기 때문에 this에 다른 값이 담길 수 있습니다.

정리하자면

다른 코드에 인자로 콜백 함수를 넘겨줌으로써 콜백 함수에 대한 제어권도 넘겨줍니다.

제어권에는 호출 시점, 인자, this 가 있습니다.

콜백 함수를 인자로 받은 코드는 **`자체적인 내부 로직`** 에 의해 이 콜백 함수를 적절한 시점에 실행할 것입니다.

여기서 문제는 수많은 코드 (setInterval, forEach, addEventListener 등) 가 있고 코드마다 내부 로직이 다 다르기 때문에 제어권도 다 다릅니다.

제어권이 다르다는 말은 호출시점이 언제인지, 콜백 함수의 인자에 어떤 값들을 어떤 순서로 넘길지, this를 명시적으로 설정해야 넘어 가는지 아니면 암묵적으로 넘어가는지 다 다르다는 말 입니다.

그러므로 메서드 마다 내부 로직이 다르다는 사실을 인지하고 쓸 때마다 유의하며 써야 한다고 생각합니다.

## 3. 콜백 함수는 함수다

**`콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출됩니다.`**

저번 시간에 this의 예시 3-28, 90쪽 에서 헷갈렸던 부분을 이 파트에서 해결할 수 있었습니다.

잠깐 this에 대해 간략하게 복습하자면

this는 실행 컨텍스트가 생성될 때 정해집니다.
실행 컨텍스트는 함수가 호출될 때 생성되므로
쉽게 얘기하자면 **`this는 함수가 호출될 때 생성된다`** 입니다.

그리고 this는 **`함수로 호출되면 전역객체`** 를, **`메서드로 호출되면 메서드로 호출한 객체`** 를 가리킵니다.

```jsx
let obj = {
    logThis: function () {
        console.log(this);
    },
    logThisLater1: function () {
        setTimeout(this.logThis, 500);
    },
    logThisLater2: function () {
        setTimeout(this.logThis.bind(this), 1000);
    },
};
obj.logThisLater1(); // window { ... }
obj.logThisLater2(); // obj { logThis: f, ... }
```

obj.logThisLater1() 을 하면 setTimeout안의 this.logThis 는 obj.logThis입니다.

저는 여기서 logThis 가 메서드로 호출되었기 때문에 logThis 내부의 this가 obj를 가리키는 줄 알았는데

전역객체가 콘솔에 찍혀 당황했었습니다.

그 이유는 setTimeout 이라는 코드의 콜백 함수로 obj.logThis 라는 메서드가 전달되어도, 이 메서드는 메서드가 아닌 **`함수로서 호출`** 되기 때문에 logThis의 this가 전역 객체를 가리키게 됩니다.

어떤 함수의 인자에 **`객체의 메서드를 전달하더라도 이는 메서드가 아닌 함수`** 라는 점이 중요한 포인트라고 생각합니다.

## 4. 콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메서드를 콜백 함수로 전달하면서 해당 객체를 this로 바라보려면 어떻게 해야할까요?

앞의 예제에 정답이 있었는데 정답은 클로저와 bind 등이 있습니다.

먼저 클로저를 사용한다면 메모리를 많이 사용한다는 단점이 있지만 다른 객체에도 재사용할 수 있다는 장점이 있습니다.

```jsx
const obj1 = {
    // 클로저 생성
    // 클로저는 함수와 그 함수가 선언됐을 때의 렉시컬 환경과의 조합이다.
    name: "obj1",
    func: function () {
        var self = this;
        return function () {
            console.log(self.name);
        };
    },
};

// callback 에는 function(){console.log(self.name)}
var callback = obj1.func(); // this에는 obj1이 담김
setTimeout(callback, 1000); // obj1

const obj2 = {
    name: "obj2",
    func: obj1.func, // obj1.func를 실행한 게 아니고 복사한 것
};

// callback2 에는 function(){console.log(self.name)}
var callback2 = obj2.func(); // this에는 obj2가 담김
setTimeout(callback2, 1000); // obj2
```

마지막으로 bind 메서드를 이용하는 방법입니다.

```jsx
const obj1 = {
    name: "obj1",
    func: function () {
        console.log(this.name);
    },
};
setTimeout(obj1.func, 1000); // undefined
setTimeout(obj1.func.bind(obj1), 1000); // obj1

// obj1.func를 실행하면 콜백함수로 객체의 메서드가 전달되어도 함수로 호출되기 때문에 this에는 전역객체가 담깁니다.
// obj1.func.bind(obj1)을 하면 obj1이라는 객체를 func의 this로 삼는 새로운 함수를 만듭니다.
// 새롭게 만들어진 함수를 setTimeout이 실행하면 this에는 obj1이 담겨 있기에 obj1이 출력됩니다.

const obj2 = {
    name: "obj2",
};

setTimeout(obj1.func.bind(obj2), 1000); // obj2
```

이상입니다
