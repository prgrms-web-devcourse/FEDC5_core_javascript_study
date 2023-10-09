# 1. Java의 this


## 👉 Java의 this는 항상 객체 자신을 가리킨다

```Java
public Class Person { 

	private String name; 
	
	public Person(String name) { 
	
		this.name = name; 
	
	} 
}
```

> Person에 의해 생성된 인스턴스로 받은 인수 name에 의해 Person의 name이 교체됩니다.


<br/>

# 2. JavaScript의 this

자바스크립트에서 this는 실행 컨텍스트가 생성될 때 함께 결정됩니다. <br/> 그말인 즉 `자바스크립트의 this는 함수의 호출 방식에 따라 달라진다`는 것입니다.

<br/>

# 3. 함수의 호출 방식에 따른 this 바인딩

<br/>

## 1. 일반 함수 호출

![Pasted image 20231009190452](https://github.com/prgrms-web-devcourse/FEDC5_core_javascript_study/assets/128919388/49eb4dc2-bf4f-497c-b4ce-ef8e4652f05f)

<br/>

### 1-1. 전역공간에서의 this

```js
var foo = function () { 
	console.dir(this); 
};

foo();   // 브라우저: window     // node.js: global
```

<br/>
브라우저 환경에서는 this가 window, node.js 환경에서는 global 전역객체를 가리킵니다. 


---

<br/>

### 1-2. 함수 안에서의 함수 호출

```js
var foo = function () { 

	let a = 1;
	
	var bar = function () {
	
		console.log(this.a);
		
	}
	
	bar();
};

foo();   // undefined
```

<br/>

일반적으로 bar 함수의 상위 스코프인 foo 함수에 있는 a가 출력될거라 생각되지만 bar 함수를 일반 함수로 호출하고 있기 때문에 this는 **전역객체를 가리키게 되어 this.a는 undefined를 출력**하게 됩니다.


<br/>

### ✔ **일반 함수 호출에서의 this는 `전역 객체`를 가리킨다!**

<br/>

---


## 2. 메소드 호출

![Pasted image 20231009190614](https://github.com/prgrms-web-devcourse/FEDC5_core_javascript_study/assets/128919388/5632630a-15a6-4b22-81b6-45ddb4462bcf)

<br/>

### 2-1. 일반적인 메소드 호출

```js
var obj = { 
	value: 100, 
	foo: function() {
		console.log(this.value);
	}
}

obj.foo()   // 100
```

객체에 속한 함수(메소드)는 java의 class와 똑같이 작동합니다. foo라는 메소드의 this는 foo가 속한 객체인 obj를 가리키기 때문에 출력값은 100이 됩니다. 

다만 주의할 점은 foo라는 함수를 메소드가 아닌 다른 방식의 함수 호출을 하게 되면 foo가 가리키는 this는 그 함수 호출 방식을 따릅니다.

<br/>

#### ⚠ 주의) 메소드가 아닌 일반 함수로 호출할 경우
```js
const foo = function () {
	console.log(this.value);
}

obj.foo = foo;

obj.foo();    // 100
foo();        // undefined
```


obj에 할당된 foo와 일반함수 foo는 this가 서로 다르게 바인딩됩니다.

<br/>

---

### 2-2. 메소드의 내부함수

```js
var obj = { 
	value: 100, 
	foo: function() {
		function bar() {
			console.log(this.value);
		}
		bar();
	}
}

obj.foo();   // undefined
```

<br/>

메소드 내에서 사용되는 this는 항상 메소드가 가리키는 객체를 바인딩하지는 않습니다. <br/> 위와 같이 메소드 안에 내부함수가 this를 호출하게 되면 일반함수의 호출방식을 따르게 됩니다.  

<br/>

![Pasted image 20231009194347](https://github.com/prgrms-web-devcourse/FEDC5_core_javascript_study/assets/128919388/efd1e0a1-6744-4ad3-8840-ced7c025757a)

<br/>

---

### 2-3. 메소드 내부함수의 this를 객체로 바인딩하는 방법

- **화살표 함수** (선호!)
```js
var obj = { 
	value: 100, 
	foo: function() {
		const bar = () => {
			console.log(this.value);
		}
		bar();
	}
}

obj.foo();   // 100
```

<br/>

화살표 함수는 es6에서 추가된 문법으로 this가 상위 스코프를 가리킵니다. 따라서 위에서 bar가 호출될 때 bar의 상위 스코프는 foo 이므로 foo의 this가 가리키는 obj로 bar의 this가 바인딩 됩니다.

<br/>


- **변수에 this 할당**
```js
var obj = { 
	value: 100, 
	foo: function() {
		const that = this;
		
		const bar = function() {
			console.log(that.value);
		}
		bar();
	}
}

obj.foo();   // 100
```

<br/>

foo가 실행되는 시점에 this는 obj를 가리키기 때문에 that 변수에 할당되는 this는 obj를 가리키고 bar가 호출하는 that이 obj를 식별하여 정상적인 출력이 가능하게 합니다.

<br/>

### ✔ **메소드 호출에서의 this는 `자신이 속한 객체`를 가리킨다!**

<br/>

---

## 3. 생성자 함수 호출

- 생성자 함수란?
> 함수명 앞에 new 키워드를 붙여서 호출하여 인스턴스를 생성하는 함수

```js
var Person = function(name, age) { 
	let name = this.name;
	let age = this.age;
}

const me = new Person( "동건", 20 );

console.log(me.name)  // 동건
console.log(me.age)  // 20
```

<br/>

Java의 클래스와 동일, 생성자 함수로 생성된 인스턴스의 this는 인스턴스 자신을 가리킵니다.

<br/>

---

## 4. 콜백 함수 호출

### 4-1. 일반 함수로써의 콜백 함수

```js
function foo() {
	console.log(this);
}

setTimeout(foo, 1000);   // window
```

<br/>

일반함수를 콜백함수로 넘길 경우 기대하던 결과가 출력됩니다. 다만 이 현상에 문제가 없다고 생각하고 넘어갈 경우 이후에 나올 문제에서 굉장히 머리가 아플겁니다. 

<br/>

---

### 4-2. 메소드로써의 콜백 함수

```js
let user = {
  firstName: "John",
  sayHi() {
    console.log(`Hello, ${this.firstName}!`);
  }
};

setTimeout(user.sayHi, 1000); // Hello, undefined!
```

<br/>

user.sayHi 라는 메소드를 콜백 함수로 넘겨주고 있습니다. 이 경우 일반적으로 1초 후에 `Hello, John`이 출력될 것이라 생각하기 쉽지만 예상과 달리 `Hello, undefined`가 출력됩니다. 

이는 this가 **함수가 호출되는 방식에 따라** 결정된다는 규칙을 생각해보면 답이 나옵니다.

**콜백 함수의 인자로 넘겨지는 함수가 메소드인 것이지 호출되는 방식은 콜백 함수인 것입니다.**

그리고 콜백 함수는 따로 this가 지정되지 않는 이상 일반 함수처럼 호출됩니다.
