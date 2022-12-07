# [11장] API 리팩토링

## 질의 함수와 변경 함수 분리하기

외부에서 관찰할 수 있는 겉보기 부수효과가 전혀 없이 값을 반환해주는 함수를 추구해야 한다.

겉보기 부수효과가 있는 함수와 없는 함수를 명확히 구분하는 것이 좋다.

```jsx
// before
// 아래의 함수는 이름처럼 2가지 일을한다. (부수효과는 아니다)
function totalOutstandingAndSendBill() {
  const result = customer.invoices.reduce(
    (total, each) => each.amount + total,
    0
  );
  sendBill();
  return result;
}
```

```jsx
// after
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
  sendBill();
}
```

## 함수 매개변수화하기

두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값을 매개변수로 받아 함수를 합쳐 중복을 없앨 수 있다.

```jsx
// before
function tenPercentRaise(person) {
  person.salary = person.salary.multiply(1.1);
}

function fivePercentRaise(person) {
  person.salary = person.salary.multiply(1.05);
}
```

```jsx
// after
function raise(person, factor) {
  person.salary = person.salary.multiply(1 + factor);
}
```

## 플래그 인수 제거하기

플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다.

플래그 인수가 있으면 함수들의 기능 차이가 잘 드러나지 않는다.

이보다는 특정한 기능 하나만 수행하는 명시적인 함수를 제공하는 편이 훨씬 깔끔하다.

함수 하나에서 플래그 인수를 두 개 이상 사용 → 플래그 인수를 써야하는 근거

플래그 인수가 둘 이상 → 함수 하나가 너무 많은 일을 처리하고 있다는 신호

```jsx
// before
// 하나의 함수에서 2가지 일을 하지말고 그냥 함수를 쪼개라 (명령형으로)
function setDimension(name, value) {
  if (name === "height") {
    this.height = value;
    return;
  }
  if (name === "width") {
    this.width = value;
    return;
  }
}
```

```jsx
// after
function setHeight(value) {
  this.height = value;
}
function setWidth(value) {
  this.width = value;
}
```

만약 메서드내에서 공통으로 사용되는 코드가 있다면 private한 함수를 만들어 중복을 해결 할 수 있다. 중복 코드를 없애고 외부에서는 접근하지 못하도록 한다.

## 객체 통째로 넘기기

레코드를 통째로 넘기면 변화에 대응하기 쉽다.

하지만 함수가 레코드에 의존하기를 원치 않을 때는 이 리팩토링을 하지 않는다.

필요 없는 데이터를 가진 객체를 넘기게되면 해당 객체에 대한 함수의 의존성이 높아지기 때문이다.

```jsx
// before
const low = room.daysTempRange.low;
const high = room.daysTempRange.high;

if(!isWithInRange(low, high)){
	alert("방 온도가 지정 범위를 벗어났습니다.");
}

isWithInRange(bottom, top){
	return (bottom >= temperatureRange.low) && (top <= temperatureRange.high);
}
```

```jsx
// after
if(!isWithInRange(room.daysTempRange)){
	alert("방 온도가 지정 범위를 벗어났습니다.");
}

isWithInRange(range){
	return (range.bottom >= temperatureRange.low)
					&& (range.top <= temperatureRange.high);
}
```

## 매개변수를 질의 함수로 바꾸기

매개변수가 있다면 결정 주체가 호출자가 된다. 호출하는 쪽을 간소하게 만드는 것이다.

아래의 예시는 클래스 내부에서 복잡한 로직들을 게터로 추출하면 클래스 내에서 재사용성, 가독성이 좋아진다.

```jsx
// before
	get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    let discountLevel;
    if (this.quantity > 100) discountLevel = 2;
    else discountLevel = 1;
    return this.discountedPrice(basePrice, discountLevel);
  }

  discountedPrice(basePrice, discountLevel) {
    switch (discountLevel) {
      case 1:
        return basePrice * 0.95;
      case 2:
        return basePrice * 0.9;
    }
  }
```

```jsx
	get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    return this.discountedPrice(basePrice, discountLevel);
  }

	get discountLevel() {
    return this.quantity > 100 ? 2 : 1;
  }

  discountedPrice(basePrice) {
    switch (this.discountLevel) {
      case 1:
        return basePrice * 0.95;
      case 2:
        return basePrice * 0.9;
    }
  }
```

## 질의 함수를 매개변수로 바꾸기

대상 함수가 더 이상 특정 원소에 의존하길 원치 않을 때 일어난다.

똑같은 값을 넣으면 매번 똑같은 결과를 내는 함수는 다루기 쉽다.

```jsx
// before
targetTemperature(aPlan);

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
}
```

```jsx
// after
targetTemperature(aPlan, thermostat.currentTemperature);

function targetTemperature(aPlan, currentTemperature) {
  currentTemperature = currentTemperature;
}
```

## 세터 제거하기

세터 메서드가 있다고 함은 필드가 수정될 수 있다는 뜻 → 수정되지 않길 원하는 필드라면 세터를 제공하지 않았을 것

세터 제거하기가 필요한 상황

1. 무조건 접근자 메서드를 통해서만 필드를 다루려 할 때
2. 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때

## 생성자를 팩토리 함수로 바꾸기

클래스의 인스턴스를 만들 때 생성자를 호출하는 방법이 조금 복잡하다면 이 리팩토링을 사용할 수 있다.

아래의 코드에서는 typeCode에 어떠한 값이 들어가야되는지 알기 어렵다.

```jsx
//before
constructor(name, typeCode) {
  this._name = name;
  this._typeCode = typeCode;
}

get name() {
  return this._name;
}

get type() {
  return Employee.legalTypeCodes[this._typeCode];
}

static get legalTypeCodes() {
  return { E: 'Engineer', M: 'Manager', S: 'Salesman' };
}
```

```jsx
//after
constructor(name, typeCode) {
  this._name = name;
  this._typeCode = typeCode;
}

get name() {
  return this._name;
}

get type() {
  return Employee.legalTypeCodes[this._typeCode];
}

static get legalTypeCodes() {
  return { E: 'Engineer', M: 'Manager', S: 'Salesman' };
}

static createEngineer(name) {
	return new Employee(name, 'E');
}
```

## 명령을 함수로 바꾸기

명령, 명령 객체(command객체)는 대부분 메서드 하나로 구성되며, 이 메서드를 요청해 실행하는 것이 이 객체의 목적이다.

아래의 클래스처럼 로직이 크게 복잡하지 않다면 장점보단 단점이 크니 평범한 함수로 바꾸는게 낫다.

```jsx
// before
class ChargeCalculator {
  constructor(customer, usage, provider) {
    this._customer = customer;
    this._usage = usage;
    this._provider = provider;
  }
  get baseCharge() {
    return this._customer.baseRate * this._usage;
  }
  get charge() {
    return this.baseCharge + this._provider.connectionCharge;
  }
}
```

```jsx
// after
function charge(customer, usage, provide) {
  const baseCharge = customer.baseRate * usage;
  return baseCharge + provider.connectionCharge;
}
```

## 수정된 값 반환하기

데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 중요하다.

데이터가 수정됨을 알려주기 좋은 방법으로 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다.

이 리팩토링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에게 유요하며, 반대로 값 여러 개를 갱신하는 함수에는 효과적이지 않다.

```jsx
// before
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? verticalChange : 0;
  }
}
```

```jsx
// after
const totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    result += verticalChange > 0 ? verticalChange : 0;
  }
  return result;
}
```
