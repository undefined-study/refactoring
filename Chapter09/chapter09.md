# [9장] 데이터 조직화

이번 장은 데이터 구조에 집중한 리팩터링이다.

## 변수 쪼개기

역할이 둘 이상인 변수는 쪼개야 한다.

역할 하나당 변수 하나다.

```jsx
function distanceTravelled(scenario, time) {
  let result;
  let acc = scenario.primaryForce / scenario.mass; // 가속도(a) = 힘(F) / 질량(m)
  let primaryTime = Math.main(time, scenario.delay);
  result = 0.5 * acc * primaryTime * primaryTime; // 전파된 거리
  let secondaryTime = time - scenario.delay;
  if (secondaryTime > 0) {
    // 두 번째 힘을 반영해 다시 계산
    let primaryVelocity = acc * scenario.delay;
    acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
    result +=
      primaryVelocity * secondaryTime +
      0.5 * acc * secondaryTime * secondaryTime;
  }
  return result;
}
```

위의 코드는 acc 변수에 값이 두 번 대입되는 것이다.

역할이 두 개라는 것이다. 쪼개야 할 변수다.

1. 변수에 새로운 이름을 지어준다.
2. 선언 시 const를 붙여 불변으로 만든다.
3. 두 번째 대입 전까지 모든 참조를 새로운 이름으로 바꾼다.
4. 두 번째로 대입할 때 변수를 새로 선언한다.

```jsx
function distanceTravelled(scenario, time) {
  let result;
  const primaryAcceleration = scenario.primaryForce / scenario.mass; // 1,2
  let primaryTime = Math.main(time, scenario.delay);
  result = 0.5 * primaryAcceleration * primaryTime * primaryTime; // 3
  let secondaryTime = time - scenario.delay;
  if (secondaryTime > 0) {
    // 두 번째 힘을 반영해 다시 계산
    let primaryVelocity = primaryAcceleration * scenario.delay; // 3
    let acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; // 4
    result +=
      primaryVelocity * secondaryTime +
      0.5 * acc * secondaryTime * secondaryTime;
  }
  return result;
}
```

1. 두 번째 용도에 적합한 이름으로 수정

```jsx
function distanceTravelled(scenario, time) {
  let result;
  const primaryAcceleration = scenario.primaryForce / scenario.mass;
  let primaryTime = Math.main(time, scenario.delay);
  result = 0.5 * primaryAcceleration * primaryTime * primaryTime;
  let secondaryTime = time - scenario.delay;
  if (secondaryTime > 0) {
    // 두 번째 힘을 반영해 다시 계산
    let primaryVelocity = primaryAcceleration * scenario.delay;
    const secondaryAcceleration =
      (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; // 5
    result +=
      primaryVelocity * secondaryTime +
      0.5 * secondaryAcceleration * secondaryTime * secondaryTime;
  }
  return result;
}
```

## 파생 변수를 질의 함수로 바꾸기

가변 데이터는 문제를 일으키는 가장 큰 골칫거리로 완전히 배제하기란 현실적으로 불가능해 유효 범위를 최대한 좁혀야 한다.

한 쪽 코드에서 수정한 값이 연쇄 효과를 일으켜 다른 쪽 코드에 원인을 찾기 어려운 문제를 야기하기도 한다.

```jsx
// before
class ProductionPlan {
  get production() {
    return this._production;
  }
  applyAdjustment(adjustment) {
    this._adjustments.push(adjustment);
    this._production += adjustment.amount;
  }
}
```

adjustment 값을 적용할 때 production까지 갱신하는 문제가 있다. (필요없는 동작)

여러 곳에서 하나의 데이터를 수정하게 된다면 실수 할 가능성이 높아진다.

아래 get production처럼 필요할 때 adjustments를 통해 값을 구하도록 수정해야 한다.

```jsx
// after
class ProductionPlan {
  get production() {
    return this._adjustments.reduce((sum, a) => sum + a.mount, 0);
  }
  applyAdjustment(adjustment) {
    this._adjustments.push(adjustment);
  }
}
```

## 참조를 값으로 바꾸기

참조로 다루는 경우 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하고, 값으로 다루는 경우에는 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다.

참조값을 사용하게 되면 내부 데이터를 수정 할 수 있기 때문에 좋지않다.

```jsx
// before
class Person {
  constructor() {
    this._telephoneNumber = new TelephoneNumber();
  }

  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }

  set officeAreaCode(arg) {
    this._telephoneNumber.areaCode = arg;
  }

  get officeNumber() {
    return this._telephoneNumber.number;
  }

  set officeNumber(arg) {
    this._telephoneNumber.number = arg;
  }
}

class TelephoneNumber {
  get areaCode() {
    return this._areaCode;
  }

  set areaCode(arg) {
    this._areaCode = arg;
  }

  get number() {
    return this._number;
  }

  set number(arg) {
    this._number = arg;
  }
}
```

```jsx
// after
class Person {
  constructor() {
    this._telephoneNumber = new TelephoneNumber();
  }

  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }

  set officeAreaCode(arg) {
    this._telephoneNumber = new TelephoneNumber(arg, this.officeNumber);
  }

  get officeNumber() {
    return this._telephoneNumber.number;
  }

  set officeNumber(arg) {
    this._telephoneNumber.number = new TelephoneNumber(this.officeAreaCode, arg);
  }
}

class TelephoneNumber {
  get areaCode(areaCode, number) {
		this._areaCode = areaCode;
		this._number = number;
  }

  get number() {
    return this._number;
  }
}
```

## 매직 리터럴 바꾸기

매직 리터럴이란 소스 코드에 등장하는 일반적인 리터럴 값

코드를 읽는사람이 값의 의미를 모른다면 해석하기 어렵기 때문에 코드 자체가 뜻을 분명하게 드러내는 게 좋다.

```jsx
// before
function areaOfCircle(r) {
  return r * r * 3.14;
}

// after
const PERIMETER = 3.14;

function areaOfCircle(r) {
  return r * r * PERIMETER;
}
```
