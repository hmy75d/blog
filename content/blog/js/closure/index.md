---
title: 코어 자바스크립트, 클로저 이해하기
date: '2024-05-11T03:55:40'
# description: ''
---

# 클로저의 의미 및 원리 이해

일반적으로 내부 함수는 외부 함수의 변수를 참조할 수 있습니다.

```jsx
var outer = function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner()
}

var outer2 = outer()

console.log(outer2)
```

위 코드처럼 내부에 선언된 inner가 LexicalEnvironment의 outerEnvironmentReference에 접근하여 외부 함수 변수a를 참조합니다.

클로저는 내부 함수가 외부 함수로 전달되어질 때, 외부 함수의 실행 컨텍스트가 종료된 뒤에도 내부 함수가 외부 함수의 참조한 변수를 유지하는 현상을 말합니다.

```jsx
var outer = function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner
}

var outer2 = outer()

console.log(outer2()) // 2
console.log(outer2()) // 3
```

위 코드를 예시로 보면 outer를 호출하고 inner함수를 반환함으로서, outer함수에 대한 실행 컨텍스트는 종료되지만 변수 a에 접근하여 값을 증가 시키고 유지합니다.

이는 가비지 컬렉팅의 방식때문인데, 하나라도 참조하고 있는 변수가 있다면 추후에 사용될 수 있기 때문에 수거되지 않습니다.

outer()에서 inner가 반환되고 inner가 outer2를 통해 참조되기 때문에 언제라도 호출되고 이용될 가능성이 있어서 수거되지 않고, a도 같이 수거되지 않아 값을 유지하게 됩니다.

가비지 컬렉팅 방식 때문에 클로저가 발생하게 됩니다.

꼭 return으로 외부에 함수를 전달하지 않아도 콜백함수로 전달해도 클로저가 발생합니다.

```jsx
;(function () {
  var a = 0
  var intervalId = null

  var inner = function () {
    if (++a > 10) {
      clearInterval(intervalId)
    }

    console.log(a)
  }

  intervalId = setInterval(inner, 1000)
})()
```

# 클로저와 메모리 관리

클로저는 객체 지향 프로그래밍의 맥락으로 공통적인 부분을 연결시키는 특성이 있습니다. 다만 참조 카운트가 0이 되지 않아 메모리 누수가 발생할 위험이 있어 관리가 필요합니다.

관리 방법은 참조형이 아닌 기본형 데이터를 할당하면 됩니다.

```jsx
var outer = (function () {
  var a = 1

  var inner = function () {
    return ++a
  }

  return inner
})()

console.log(outer())
outer = null
;(function () {
  var a = 0
  var intervalId = null
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId)
      inner = null
    }

    console.log(a)
  }

  intervalId = setInterval(inner, 1000)
})()
```

# 클로저 활용 사례

## 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

일반적인 콜백 함수 코드

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', function () {
    alert('your choice is ' + fruit)
  })
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruit = function (fruit) {
  alert('your choice is ' + fruit)
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruit)
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

위 코드는 반복을 피해 alertFruit함수를 따로 선언해서 사용했지만, addEventListener의 기본적으로 첫번째인자를 이벤트 객체를 활용하기 때문에 문제가 발생합니다.

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruit = function (fruit) {
  alert('your choice is ' + fruit)
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruit.bind(null, fruit))
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

bind를 통해서 첫번째 인자에 원하는 fruit를 전달하지만, this가 null로 바인딩이 됩니다.

```jsx
var fruits = ['apple', 'banana', 'peach']
var $ul = document.createElement('ul')

var alertFruitBuilder = function (fruit) {
  return function () {
    alert('your choice is ' + fruit)
  }
}

fruits.forEach(function (fruit) {
  var $li = document.createElement('li')
  $li.innerText = fruit
  $li.addEventListener('click', alertFruitBuilder(fruit))
  $ul.appendChild($li)
})

document.body.appendChild($ul)
```

위 문제점들을 모두 개선하기 위해 alertFruitBuilder를 선언하여 함수를 반환하며 이 반환된 함수는 클로저가 형성되어 추후에 fruit를 참조하여 활용하게 됩니다.

## 접근 권한 제어

자바스크립트에서는 다른 언어와 달리 변수에 대해 접근 권한 기능이 따로 없습니다. 접근 권한을 구현하기 위해 클로저를 이용하여 함수 차원에서 변수에 대한 접근을 차단할 수 있습니다.

클로저를 통해서 외부 함수로 반환할 때 반환한 변수는 공개, 반환하지 않은 변수는 접근을 제한할 수 있습니다.

```jsx
var car = {
  fuel: Math.ceil(Math.random() * 10 + 10),
  power: Math.ceil(Math.random() * 3 + 2),
  moved: 0,
  run: function () {
    var km = Math.ceil(Math.random() * 6)
    var wasteFuel = km / this.power

    if (this.fuel < wasteFuel) {
      console.log('이동불가')
      return
    }

    this.fuel -= wasteFuel
    this.moved += km
    console.log(km + 'km dlehd (총' + this.moved + 'km')
  },
}
```

예를들어 일반적인 객체는 fuel속성에 접근하여 변경이 가능하게 됩니다.

```jsx
var createCar = function () {
  var fuel = Math.ceil(Math.random() * 10 + 10)
  var power = Math.ceil(Math.random() * 3 + 2)
  var moved = 0

  return {
    get moved() {
      return moved
    },
    run: function () {
      var km = Math.ceil(Math.random() * 6)
      var wasteFuel = km / power

      if (fuel < wasteFuel) {
        console.log('이동불가')
        return
      }

      fuel -= wasteFuel
      moved += km
      console.log(km + 'km dlehd (총' + moved + 'km')
    },
  }
}

var car = createCar()
```

fuel와 같은 속성들은 외부로 노출하지 않으므로서 비공개로 하고, moved는 getter을 통해서 읽기 전용으로 설정했습니다.

접근은 막았지만 car.fuel =1000;와 같은 재할당으로 변경이 가능하게 됩니다.

```jsx
var createCar = function () {
  // ...

  var publicMembers = {
    fuel: Math.ceil(Math.random() * 10 + 10),
    power: Math.ceil(Math.random() * 3 + 2),
  }

  Object.freeze(publicMembers)

  return publicMembers
}
```

공개는 하지만 변경을 피하고 싶은 변수들은 위 코드처럼 Object.freeze를 통해서 변경을 막은뒤 반환하여 접근하게 할수 있습니다.

## 부분 적용 함수

```jsx
var add = function () {
  var result = 0

  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i]
  }

  return result
}

var addPartial = add.bind(null, 1, 2, 3, 4, 5)
console.log(addPartial(6, 7, 8, 9, 10))
```

위와 같이 bind를 이용하여 부분 적용함수를 만들 수 있습니다. 그러나 문제는 this를 원하는 값으로 만들지 못합니다.

```jsx
var partial = function () {
  var originalPartialArgs = arguments
  var func = originalPartialArgs[0]

  if (typeof func !== 'function') {
    throw new Error('is not function')
  }

  return function () {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1)
    var restArgs = Array.prototype.slice.call(arguments)

    return func.apply(this, partialArgs.concat(restArgs))
  }
}

var add = function () {
  var result = 0

  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i]
  }

  return result
}

var addPartial = partial(add, 1, 2, 3, 4, 5)
console.log(addPartial(6, 7, 8, 9, 10))
```

첫번째 인자 함수를 반환하며 나머지 인자를 concat을 이용해서 apply로 전달하고 바로 실행시점에 this를 바인딩하게 합니다.

```jsx
var debounce = function (eventName, func, wait) {
  var timeoutId = null

  return function (event) {
    var self = this
    console.log(eventName, 'event 발생')
    clearTimeout(timeoutId)
    timeoutId = setTimeout(func.bind(self, event), wait)
  }
}

var moveHandler = function (e) {
  console.log('move event')
}

document.body.addEventListener('mousemove', debounce('move', moveHandler, 500))
```

debounce를 예로들면 setTimeout에 this를 실행 시점을 기준으로 적용하기 위해 self에 담아서 바인딩했습니다.

클로저를 통해서 함수를 반환하며 이벤트 발생후 timeoutId를 참조하며 유지하던중에 clearTimout이 발생하면 이전에 대기큐를 정리하게 만듭니다.

## 커링 함수커링함수는 여러 개의 인자를 받는 함수를, 하나의 인자만 받는 함수로 나눠서 순차적으로 호출되게 하는 함수 입니다.

부분적용 함수와 다른점은 먼저 최종적으로 한개의 인자만 받아서 호출하는 형태입니다. 그리고 중간 과정에서 마지막 인자가 전달되기 까지 실행하지 않습니다.

그래서 원본함수를 재실행 하더라도 최종인자가 실행 되기 전까지는 원본함수가 실행되지 않습니다.

```jsx
var curry3 = function (func) {
  return function (a) {
    return function (b) {
      return func(a, b)
    }
  }
}

var getMaxWith10 = curry3(Math.max)(10)
console.log(getMaxWith10(8))
console.log(getMaxWith10(25))

var getMaxWith10 = curry3(Math.min)(10)
console.log(getMaxWith10(8))
console.log(getMaxWith10(25))
```

```jsx
var curry5 = function (func) {
  return function (a) {
    return function (b) {
      return function (c) {
        return function (d) {
          return function (e) {
            return func(a, b, c, d, e)
          }
        }
      }
    }
  }
}

var getMax = curry5(Math.max)
console.log(getMax(1)(2)(3)(4)(5))
```

위에는 중첩이 되면서 가독성이 떨어지게 됩니다.

```jsx
var curry5 = func => a => b => c => d => e => func(a, b, c, d, e)
```

이를 ES6에 있는 화살표 함수를 통해서 간단하게 표현이 가능합니다.

각 단계에서 받은 인자들은 모두 마지막 단계에서 참조할 것이므로 GC되지 않고 메모리에 저장했다가 마지막 호출시 실행 컨텍스트가 종료된후에 GC에서 수거 됩니다.
커링방식은 마지막 인자가 넘어갈 때가지 함수실행을 미루는 셈인데 이를 함수 프로그래밍에서는 지연실행(lazy execution)이라고 합니다.

원하는 시점까지 지연시켰다가 실행하려고 할때 요긴합니다.

```jsx
var getInformation = function (baseUrl) {
  return function (path) {
    return function (id) {
      return fetch(baseUrl + parh + '/' + id)
    }
  }
}

var getInformation = baseUrl => path => id => fetch(baseUrl + path + '/' + id)
```

여기서는 보통 baseUrl은 고정되고 나머지 정보는 유동적일 수 있는데 이럴때 커링을 활용하여 baseUrl과 같은 정보는 미리 넘겨 고정시켜두면 유용하게 사용할 수 있습니다.

```jsx
var imageUrl = 'http://...'

var getImage = getInformation(imageUrl)
var getEmotion = getImage('emotion')
var getIcon = getImage('icon')
```

```jsx
const logger = store => next => action => {
  console.log('dispatching', action)
  console.log('next state', store.getState())

  return next(action)
}

const thunk = store => next => action => {
  return typeof action === 'function'
    ? action(dispatch, store.getState)
    : next(action)
}
```

이 코드에서도 store, next는 한번 생성이후 변하지 않으므로 미리 인자를 넘겨 고정시켜두고, 변경하는 action만을 받아 사용하게 됩니다.
