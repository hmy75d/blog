---
title: 코어 자바스크립트, 콜백함수 이해하기
date: '2024-05-06T03:57:40'
# description: ''
---

# 콜백함수?

---

콜백 함수는 다른 함수 또는 메서드에게 인자로 넘겨지며, **제어권**이 함께 위임된 함수를 말합니다.

# 제어권

---

## 호출 시점

```jsx
var count = 0

var cbFunc = function () {
  console.log(count)

  if (++count > 4) clearInterval(timer)
}

var timer = setInterval(cbFunc, 300)
```

cbFunc 콜백함수는 setInterval에 제어권을 넘겨 0.3초라는 호출시점에 대한 제어권을 넘겨 받습니다

## 인자

```jsx
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
});

var newArr = [10, 20, 30].map(function (index, currentValue) {
  console.log(index, currentValue);
});

Array.prototype.map(callback[, thisArg]);
callback: function (currentValue, index, array);
```

콜백함수는 인자에 대한 제어권도 넘깁니다. 위 코드처럼 콜백함수내에서 currentValue, index를 바꿔도 값은 변하지 않으며, 제어권을 넘겨받은 함수 map에 따라서 콜백함수 인자를 제어하게 됩니다.

그래서 네이밍이 관계없이 순서에 따라서만 바뀌게 됩니다.

## this

```jsx
Array.prototype.map = function (callback, thisArg) {
  var mappedArr = []

  for (var i = 0; i < this.length; i++) {
    var newValue = callback.call(thisArg || window, this[i], i, thisArg || this)

    mappedArr[i] = newValue
  }

  return mappedArr
}
```

this에 대한 제어권도 call/apply등을 통해서 this를 명시적으로 바인딩 할수 있습니다.

위 코드는 thisArg를 어떻게 바인딩 될 수 있는지 구현한 map 메서드 입니다.

# 콜백 함수는 함수다

```jsx
var obj = {
  vals: [1, , 2, 3],
  logValues: function (value, index) {
    console.log(this, value, index)
  },
}

obj.logValues(1, 2)
;[4, 5, 6].forEach(obj.logValues)
```

logValues는 메서드로서 호출되므로 첫번째 호출에서는 obj를 this로 바인딩 하지만, 두번째 호출 forEach의 콜백함수로 호출될 때는 window를 출력합니다.

이는 obj.logValues라는 메서드로 전달한것 이 아닌, obj.logValues가 가리키는 함수만 전달한것이기 때문입니다.

객체의 메서드를 콜백함수로 전달하더라도 함수로서 전달됩니다.

# 콜백 함수 내부의 this에 다른 값 바인딩하기

---

콜백함수로 객체 메서드를 전달하면 this가 바인딩 되지 않습니다. 바인딩이 되게 하려면 전통적인 방식들이 있지만 bind를 이용해서 해결할 수 있습니다.

```jsx
var obj1 = {
  name: 'obj1',
  logName: function () {
    console.log(this.name)
  },
}

var callback = obj1.logName.bind(obj1)

setTimeout(callback, 1000)
```

# 콜백 지옥과 비동기 제어

---

콜백함수가 중첩하여 사용하면서, 들여쓰기가 이해하기 어려울정도로 깊어지는 현상을 콜백지옥이라고 합니다.

이러면 가독성이 많이 떨어져 코드의 이해와 수정이 어려워지게 됩니다.

> 콜백지옥 코드

```jsx
setTimeout(
  function (name) {
    var coffeeList = name
    console.log(coffeeList)

    setTimeout(
      function (name) {
        coffeeList += ', ' + name
        console.log(coffeeList)

        setTimeout(
          function (name) {
            coffeeList += ', ' + name
            console.log(coffeeList)

            setTimeout(
              function (name) {
                coffeeList += ', ' + name
                console.log(coffeeList)
              },
              500,
              '카페라떼',
            )
          },
          500,
          '카페모카',
        )
      },
      500,
      '아메리카노',
    )
  },
  500,
  '에스프레소',
)
```

> 해결방법 - 기명합수로 전환

익명함수들을 기명함수로 전환해서 호출하는 방법입니다.

```jsx
var coffeeList = ''

var addEspresso = function (name) {
  coffeeList = name
  console.log(coffeeList)
  setTimeout(addAmericano, 500, '아메리카노')
}

var addAmericano = function (name) {
  coffeeList += ', ' + name
  console.log(coffeeList)
  setTimeout(addMocha, 500, '카페모카')
}

var addMocha = function (name) {
  coffeeList += ', ' + name
  console.log(coffeeList)
  setTimeout(addMocha, 500, '카페라떼')
}

var addLatte = function (name) {
  coffeeList += ', ' + name
  console.log(coffeeList)
  setTimeout(addMocha, 500, '카페라떼')
}

setTimeout(addEspresso, 500, '에스프레소')
```

> Promise 이용하기

ES6에 도입된 Promise를 이용한 방식이며 resolve를 통해서 다음 구문 then으로 넘어갈 수 있습니다.

그래서 비동기 처리를 가능하게 합니다.

```jsx
new Promise(function (resolve) {
  setTimeout(function () {
    var name = '에스프레소'
    console.log(name)
    resolve(name)
  }, 500)
})
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ', 아메리카노'
        console.log(name)
        resolve(name)
      }, 500)
    })
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ', 카페모카'
        console.log(name)
        resolve(name)
      }, 500)
    })
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ', 카페라떼'
        console.log(name)
        resolve(name)
      }, 500)
    })
  })
```

```jsx
var addCoffee = function (name) {
  return function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var newName = preName ? prevName + ', ' + name : name
        console.log(newName)
        resolve(newName)
      }, 500)
    })
  }
}

addCoffee('에스프레소')()
  .then(addCoffee('아메리카노'))
  .then(addCoffee('카페모카'))
  .then(addCoffee('카페라떼'))
```

> Generator 이용하기

```jsx
var addCoffee = function (prevName, name) {
  setTimeout(function () {
    coffeeMaker.next(prevName ? prevName + ', ' + name : name)
  }, 500)
}

var cofeeGenerator = function* () {
  var espresso = yield addCoffee('', '에스프레소')
  console.log(espresso)

  var americano = yield addCoffee(espresso, '아메리카노')
  console.log(americano)

  var mocha = yield addCoffee(americano, '카페모카')
  console.log(mocha)

  var latte = yield addCoffee(mocha, '카페라떼')
  console.log(latte)
}

var coffeeMaker = cofeeGenerator()
coffeeMaker.next()
```

> async/await

```jsx
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(name)
    }, 500)
  })
}

var coffeeMaker = async function () {
  var coffeeList = ''

  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? ',' : '') + (await addCoffee(name))
  }

  await _addCoffee('에스프레소')
  console.log(coffeeList)

  await _addCoffee('아메리카노')
  console.log(coffeeList)

  await _addCoffee('카페모카')
  console.log(coffeeList)

  await _addCoffee('카페라떼')
  console.log(coffeeList)
}

coffeeMaker()
```
