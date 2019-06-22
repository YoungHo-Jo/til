# this에 대해 알아보자
#ProgrammingLanguage/js


[this는 어렵지 않습니다](https://blueshw.github.io/2018/03/12/this/)
[this - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)


자바스크립트의 `this`는 다른 OOP 언어(C++, Java 등)과 다른 특성을 지닌다.

> this는 현재 실행 문맥이다.  


[javascript this example](https://codepen.io/youngho-jo/pen/dBvQEM)

```javascript
alert(this === window); // true
// window.alert(this === window)와 동일

const caller = {
	f: function() {
		alert(this === window)
	}
};

caller.f(); // false

```

`caller`안에 `f` 함수의 body안에서 `this`는 caller 객체의 실행 문맥이다.

```javascript
function nonStrictMode() {
	return this;
}

function strictMode() {
	'use strict';
	return this;
}

nonStrictMode(); // window
strictMode(); // undefiend
window.strictMode(); // window
```

**strict-mode**는 전역 객체이냐 일반 객체냐에 따라 함수 내부 this의 결과가 다르다.
 
~생성자 함수 / 객체에서의 this~
```javascript
function NewObject(name, color) {
	this.name = name;
	this.color = color;
	this.isWindow = function() {
		return this === window;
	}
}

const obj1 = new NewObject("nana", "yellow");
console.log(obj1.name); // nana
console.log(Obj1.color);	// yellow
console.log(obj1.isWindow()); // false
```
`new`를 통해 새로운 객체를 생성하였기 때문에 `NewObject`를 통해 새로운 실행 문맥이 생성된다고 보면된다. 내부의 this에 값을 할당하였기 때문에 출력을 하면 출력되는 것을 확인할 수 있다.

반면 `new`를 이용하지 않고 부르면 어떻게 될까?
```javascript
const withoutNew = NewObject("nana", "yellow");
console.log(withoutNew.name); // error
console.log(withoutNew.color); // error
console.log(withoutNew.isWindow()); // error
```
`new`가 없기 떄문에 일반적인 함수 호출과 같다. return 값이 없기 때문에 `withoutNew`는 **undefiend**이다.

일반 객체의 경우에는 어떻게 될까?
```javascript
const person = {
	name: "nana",
	age: 15,
	nickname: "yellow",
	getName: function() {
		return this.name;
	}
};

console.log(person.getName()); // nana

const other = person;
other.name = "didi"
console.log(other.getName()); // didi
console.log(person.getName()); // didi
```
`other`에 `person`의 레퍼런스를 할당한 것이기 때문에 같은 객체를 가리키게 된다. 그러므로 `other`의 `name`을 바꾸면 `person`의  `name`바꾼것과 동일하다.

그러므로 복사하여 사용해야 한다. ES6의 `Object.assign()`을 활용한다.


~bind, arrow~
그렇다면 함수 내부에 외부의 this를 사용하고 싶으면 어떻게 해야할까?
```javascript
function Family(firstName) {
	this.firstName = firstName;
	const names = ['bill', 'mark', 'steve'];
	names.map(function(lastName, index) {
		console.log(lastName + " " + this.firstName);
		// this === window => true
	});
}

const kims = new Family("kim");
// bill undefiend
// mark undefiend
// steve undefiend
```
`map()`의 파라미터로 서브루틴 함수를 만들어 사용하였다. 새로운 함수의 바디는 새로운 **실행 문맥**이다. 그렇기 때문에 **non strict**에서는 `window`가 `this`이다. 

> ~서브루틴~  
> 메소드와 구분을 위해 명명  


그렇기 때문에 우리의 의도대로 `Family`의 `this`에 접근하기 위해서는 변수를 따로 할당해주어 `this`를 매핑하여 사용한다.
```javascript
function Family(firstName) {
	this.firstName = firstName;
	const names = ['bill', 'mark', 'steve'];

	const that = this;
	names.map(function(lastName, index) {
		console.log(lastName + " " + that.firstName);
	});
}

const kims = new Family("kim");
// bill kim
// mark kim
// steve kim
```


매번 이런식으로 할당해서 사용할 수 없으니 `bind()`함수를 이용한다.
[Function.prototype.bind() - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
```javascript
function Family(firstName) {
	this.firstName = firstName;
	const names = ['bill', 'mark', 'steve'];

	names.map(function(lastName, index) {
		console.log(lastName + " " + this.firstName);
	}.bind(this));
}

const kims = new Family("kim");
// bill kim
// mark kim
// steve kim
```


ES6로 오면서 **arrow function**을 이용해 더 간편하고 직관적으로 작성할 수 있다.
```javascript
function Family(firstName) {
	this.firstName = firstName;
	const names = ['bill', 'mark', 'steve'];

	names.map((lastName, index) => {
		console.log(lastName + " " + this.firstName);
	});
}

const kims = new Family("kim");
// bill kim
// mark kim
// steve kim
```
ES6는 babel을 통해 일반 javascript으로 변환한다. 변환의 결과는 아래와 같다.
```javascript
‘use strict’

function Family(firstName) {
  var _this = this

  this.firstName = firstName
  var names = [‘bill’, ‘mark’, ‘steve’]
  names.map(function(value, index) {
    console.log(value + ‘ ‘ + _this.firstName)
  })
}
var kims = new Family(‘kim’)
```

