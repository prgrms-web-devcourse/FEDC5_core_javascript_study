## 2. 명시적으로 this를 바인딩하는 방법

앞서 this란 무엇이며 상황마다 this가 어떻게 달라지는지에 대해 알게 되었습니다.

지금부터는 명시적으로 this를 바인딩 할 수 있는 call, apply, bind 메서드에 대해서 알아보겠습니다.

이들의 공통점은 **`첫 번째로 전달하는 객체에 this를 바인딩`** 할 수 있고, **`이후의 인자들은 함수의 매개변수`** 로 이용합니다.

<br />

### 2-1 call 메서드

함수를 **즉시 실행**하도록 하는 명령입니다.
첫 번째 인자를 this로 바인딩하고, 이후 인자들을 호출할 함수의 매개변수로 이용합니다.

```javascript
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // 출력: Window {...} 1 2 3
func.call({ x: 1 }, 4, 5, 6); // 출력: { x: 1 } 4 5 6
```

call을 사용하기 전에는 this가 전역 객체 window였지만,
call을 사용한 후에는 this에 첫 번째 인자인 { x: 1 }가 할당되었습니다.

<br />

### 2-2 apply 메서드

call의 동작 방식과 완전히 동일하지만, 다른 점은 **인자를 배열로 받아서 매개변수로 지정**한다는 점입니다.

```javascript
var obj = {
  x: 1,
  method: function (a, b, c) {
    console.log(this, a, b, c);
  },
};
obj.method(1, 2, 3); // 출력: obj { x: 1, method: f } 1 2 3
obj.method.apply({ y: 2 }, [4, 5, 6]); // 출력: { y: 2 } 4 5 6
```

apply를 사용하기 전에는 this가 상위 스코프인 obj였지만, apply 사용 후에는 첫 번째 인자인 { y: 2 } 객체가 this에 바인딩 된 것을 볼 수 있습니다.

<br />

### 💡 생성자 내부에서 반복 줄이기

다른 생성자와 공통된 내용이 있을 경우 call, apply를 사용해서 **다른 생성자를 호출하여 반복되는 코드를 줄일 수 있습니다.**

```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}

var 홍길동_학생 = new Student("홍길동", "남", "한국대");
```

<br />

### 2-3 bind 메서드

call, apply 메서드와의 차이점은 **`함수를 실행하지 않고 새로운 함수를 반환`** 한다는 것입니다.

그 외의 동작 과정은 이전 메서드들과 같습니다.

```javascript
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};
func(1, 2, 3, 4); // 출력: Window {...} 1 2 3 4
console.log(func.name); // 출력: func

var bindFunc = func.bind({ x: 1 }, 4, 5);
bindFunc(6, 7); // 출력: { x: 1 } 4 5 6 7
console.log(bindFunc.name); // 출력: bound func
```

특이한 점은, 함수에는 name이라는 식별자를 담는 프로퍼티를 가지고 있는데, bind 메서드를 통해 만든 함수는 **이름 앞에 bound가 붙는다**는 점입니다.

<br />

### 💡 상위 컨텍스트의 this를 바인딩하는 방법!

```javascript
// call 메서드 이용
var obj = {
  outer: function () {
    console.log(this); // outer

    var innerFunc = function () {
      console.log(this); // outer
    };

    innerFunc.call(this); // 할당된 함수를 실행
  },
};
obj.outer();
```

```javascript
// bind 메서드 이용
var obj = {
  outer: function () {
    console.log(this); // outer

    var innerFunc = function () {
      console.log(this); // outer
    }.bind(this);

    innerFunc(); // 반환 받은 함수를 실행
  },
};
obj.outer();
```

apply는 call의 동작과 같기 때문에 생략했습니다.

차이점이 있다면 메서드가 호출되는 시점이 조금 다른데, 그 이유는 **`bind의 경우 함수를 즉시 실행하지 않고 새로운 함수를 반환`** 하기 때문에 반환 받은 함수를 실행한다는 차이에서 비롯됩니다.

<br />

### 콜백 함수 내부에서의 this

#### addEventListener

리스너가 가리키는 target이 this가 됨

```javascript
document.body.onclick = function () {
  console.log(this); // <body>
};
```

#### ❗️ 헷갈리는 사례

```javascript
document.body.onclick = function () {
  console.log(this); // <body>

  function inner() {
    console.log(this); // window
  }
  inner();
};
```

일반 함수의 this는 기본적으로 전역 객체에 바인딩 되기 때문

#### ✅ 해결법

this를 **변수에 저장**해서 사용하거나, **화살표 함수** 사용

```javascript
document.body.onclick = function () {
  console.log(this); // <body>

  const that = this;

  function inner() {
    console.log(that); // <body>
  }
  inner();
};

document.body.onclick = function () {
  console.log(this); // <body>

  const inner = () => {
    console.log(this); // <body>
  };

  inner();
};
```

#### forEach 등 배열 메서드

기본적으로 this가 전역 객체로 바인딩 되지만 원하는 this로 바인딩을 할 수 있도록 thisArg 매개변수를 가지고 있음

```javascript
Array.prototype.forEach(callback[, thisArg]);

var report = {
    sum: 0,
    add: function(){
        var args = Array.prototype.slice.call(arguments);

        args.forEach(function(entry){
            // this가 window가 아닌 report로 바인딩 됨
            this.sum += entry;
        }, this);
    }
}

```

<br />

## 정리

- call, apply 메서드는 this를 명시적으로 지정하면서 **함수 또는 메서드를 호출**합니다.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 **새로운 함수를 만듭니다.**
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 **별도의 인자로 this를 받기도** 합니다.
