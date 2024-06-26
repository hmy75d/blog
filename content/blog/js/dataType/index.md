---
title: 코어 자바스크립트, 데이터 타입 이해하기
date: '2024-03-11T08:16:40.250109Z'
# description: ''
---

# 메모리, 데이터와 변수, 식별자

## 메모리, 데이터

컴퓨터에서는 모든 데이터는 0 또는 1로 저장합니다. 저장되어 있는 값들을 메모리라하고 하나의 메모리조각을 1bit라고 하며, 8bit의 묶음을 1byte 단위로 봅니다.
각 메모리들은 고유의 식별자, 즉 메모리 주소값을 가지고 있으며 이 주소값을 통해 구분되고 연결하여 데이터를 이루게 됩니다.
자바스크립트에서는 숫자의 경우 정수형, 부동소수형인지를 구분하지 않고 64비트 메모리를 넉넉하게 확보해 형변환에 대해 더 자유롭습니다.

## 식별자와 변수

변수(variable), 식별자(identifier)는 문맥에 따라서 혼용될 수 있습니다. 하지만 정확히 구별하자면 변수는 변할수 있는 숫자, 문자역, 객체 등등의 데이터 이며, 식별자는 그러한 데이터를 식별하는 이름 변수명입니다.

# 변수 선언과 데이터 할당

## 변수 선언과정

변수 선언과 데이터 할당을 통해 데터를 변수에 담아 사용하게 됩니다.
변수는 변경가능한 데이터가 담긴 공간으로서 변수선언 과정에서 변수가 담길 메모리 공간을 확보하고 식별자를 지정하여 저장합니다. 이러한 과정을 통해 특정 식별자를 가진 변수를 선언하게 됩니다.

## 데이터 할당과정

데이터를 변수에 담는 과정입니다. 이때 데이터 영역의 메모리 공간을 따로 확보하여 데이터 값을 저장하고, 그 메모리 주소값을 변수영역 메모리에 참조 시킵니다.
이렇게 따로 변수 영역과 데이터 영역을 할당하는 이유는 데이터를 쉽게 변경하고 메모리를 효율적으로 관리하기 위함입니다.
데이터 영역의 메모리 공간을 별도로 하면 메모리 공간의 변경에서 더 자유로울수 있습니다. 또 중복된 데이터를 할당하는 경우 하나의 데이터로 처리가 가능하여 더 효율적입니다.

# 기본형 데이터와 참조형 데이터

## 불변성

변수와 상수를 구분하는 변경 가능성 대상은 변수 영역 메모리 이고, 보통 흔히 말하는 불변성 여부를 구분할 때의 변경 가능성의 대상은 데이터 영역 메모리 입니다.

왜냐하면 변수에 새로운 데이터를 할당할 때 데이터 자체를 변경하지 않고 새로 만드는 동작을 통해서만 할당하기 때문에 불변합니다.

## 가변성, 참조형 데이터

정확히는 기본형 데이터와 마찬가지로 참조형 데이터도 불변합니다. 다만 참조형 데이터는 프로퍼티의 변수를 할당하는 과정에서 차이점을 보입니다.

### 참조형 데이터 할당과정

```
var obj1 = {
  a: '1',
  b: 'bbb'
}
```

위와 같은 참조형데이터 obj1을 선언하고 데이터를 할당하는 과정은 아래와 같습니다.

- 먼저 변수 영역에 메모리공간을 확보하고 obj1이라는 식별자로 이름을 지정합니다.
- 데이터 영역에 obj1의 값을 넣는데, 이때 그릅형 데이터로 프로퍼티의 변수 영역 메모리 주소값을 넣는다.
- 이 프로퍼티의 변수 영역 메모리의 각 식별자 이름 'a', 'b'를 지정한다
- 데이터 영역에서 각 프로퍼티의 데이터를 검색하고 없다면 새로 메모리를 확보하고 값을 지정한뒤 각 메모리의 주소값을 프로퍼티 변수 영역에 저장한다.

여기서 기본형과의 차이는 객체 프로퍼티의 변수영역이 별도로 존재한다는 점입니다.

| 프로퍼티 재할당

```
var obj1 = {
  a: '1',
  b: 'bbb'
};

obj1.a = 2;
```

객체의 프로퍼티에 값을 재할당하게 되면 데이터 자체는 불변으로 변하지 않습니다. 다만 프로퍼티 변수 데이터가 가리키는 값이 바뀌고 2를 재할당하게 됩니다.

| 중첩된 참조형 데이터 할당

```
var obj = {
  x: 3,
  arr: [3, 4, 5]
};
```

- 변수영역에 빈공간을 확보하고 obj로 이름을 지정합니다.
- 그룹형 데이터로 데이터 영역에 프로퍼티의 변수 영역 메모리를 확보하여 만들고, 데이터 영역에 주소값을 저장합니다.
- 프로퍼티 변수영역에 이름값을 x, arr로 지정합니다.
- 먼저 x의 경우 3의 데이터값을 겁색하고 없다면 데이터 영역에 메모리를 확보하여 3을 저장후 주소값을 x의 데이터에 저장합니다.
- arr의 경우 그룹형이므로 새로운 프로퍼티 변수영역을 만듭니다.
- 각 프로퍼티 변수영역에 데이터 값이 있다면 해당 데이터 주소값을 저장하고, 없다면 새로 확보하여 값을 저장후에 주소값을 변수영역에 저장합니다.

## 변수 복사

```
var a = 10;
var b = a;

var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;

b = 15;
obj2.c = 20;
```

위 변수 a, b, obj1, obj2모두 위에 설명한 과정으로 데이터가 할당됩니다. 단순 변수 복사과정에서는 b, obj2모두 각 a, obj1의 주소값을 저장하여 데이터가 복사됩니다.
차이점은 복사이후 변경했을때 기본형의 경우 바라보는 주소값이 달라지며, 참조형 데이터 obj2의 경우는 여전히 obj1의 데이터 값 즉 같은 객체를 바라봅니다. 따라서 obj2.c를 변경하면 같은 객체obj1을 바라보면서 속성을 변경하기 때문에 obj1의 c속성도 같이 변경되게 됩니다.

반면 내부 프로퍼티 변경이 아닌 객체 자체를 변경한 경우는 다르게 진행됩니다.

```
var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;
obj2 = { c: 20, d: 'ddd' };
```

여기서 obj2에 새로운 객체를 할당하기 때문에 기존처럼 같은 데이터가 아닌 새로운 데이터를 참조합니다.
그리고 새로운 객체는 새로운 프로퍼티 변수데이터를 생성합니다. 따라서 기존에 c프로퍼티와 다른 메모리공간의 데이터입니다. 차이점은 obj1과 다른 객체를 바라보며 obj1의 c프로퍼티 값은 10을 유지하게 됩니다.

참조형 데이터의 가변성을 설명할 때 중요한점은 객체 데이터 자체를 변경할 때가 아니라 객체 데이터의 프로퍼티를 변경할때만 성립한다는 것입니다.

# 불변 객체

```
var user = {...};

var changeName = function (user, newName) {
  var newUser = user;
  newUser.name = newName;
  return newUser;
}

changeName(user);
```

위 코드처럼 객체 복사 후 프로퍼티 변경시 데이터의 가변성으로 메서드에서 객체의 불변성을 보장하기 위해 불변객체를 만드는 방법을 활용합니다.

## 불변객체 만드는 방법

| 새로운 객체를 리턴하는 방법

```
var user = {...};

var changeName = function (user, newName) {
  return {
    name: newName,
    ...
  };
}

changeName(user);

```

새로운 객체를 리턴하므로 다른 객체를 복사하지 않는다.

| 반복문으로 얕은 복사

```
var user = {...};

var copyObject = function (target) {
  var result = {};

  for (var prop in target) {
    result[prop] = target[prop];
  }

  return result;
}

copyObject(user);

```

## 얕은복사, 깊은복사

얕은복사는 객체의 바로 아래 1depth의 속성암 복사하는 방법입니다.
보통 문제가 되지 않지만 중첩된 객체중 프로퍼티를 변경할때 참조형 데이터의 가변문제가 발생합니다.
이를 방지하려면 깊은복사가 필요합니다. 중첩된 프로퍼티를 모두 조회하며 복사하는 방법입니다.

| 재귀적으로 조회하는 방법

```
var copyObjectDeep = function(target) {
  var result = {};

  if (typeof target === 'object' && target !== null) {
    for (var prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }

  return result;
}
```

| JSON을 활용하기

```
var copyObjectJSON = function (target) {
  return JSON.parse(JSON.stringfy(target));
}
```

# undefined, null

둘다 데이터가 없다는 의미이지만 차이는 undefined의 경우에는 자바스크립트 엔진이 자동으로 부여할수 있고, null의 경우에는 사용자가 명시적으로 할당할 수 있는 값입니다.

| 메모리 엔진에서 undefined

변수 선언만 하고 데이터를 할당하지 않은 변수에 대해서는 자바스크립트 엔진이 undefined값을 자동으로 할당합니다.
다만 ES6부터 let, const에 대해서는 undefined값을 할당하지 않고 초기화를 마치며, 변수에 접근하지 못하게 합니다.
