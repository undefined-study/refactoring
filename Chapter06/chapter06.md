# [6장] 기본적인 리팩토링

## 함수 추출하기

코드 조각을 찾아 무슨 일을 하는지 파악한 다음, 독립된 함수로 추출하고 목적에 맞는 이름을 붙인다.

코드를 언제 독립된 함수로 묶어야 할지에 관한 의견은 많다.

길이, 반복되는 횟수 등등 많지만 ‘목적과 구현을 분리’하는 방식이 가장 합리적으로 보인다.

코드를 보고 무슨 일을 하는지 파악하는 데 한참이 걸리면 함수로 추출한 뒤 걸맞는 이름을 짓는다.

1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다(’어떻게’가 아닌 ‘무엇을’ 하는지)
2. 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
4. 변수를 다 처리했다면 컴파일한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.
6. 테스트한다.
7. 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다.

```jsx
function before() {
  console.log(`이름: ${이름}`);
  console.log(`나이: ${나이}`);
  console.log(`성별: ${성별}`);
}

function after() {
  // 함수를 새로 만들고
  // 목적을 잘 드러내는 이름을 붙인다.
  function printProfile() {
    console.log(`이름: ${이름}`);
    console.log(`나이: ${나이}`);
    console.log(`성별: ${성별}`);
  }
}
```

```jsx
function before() {
  let outstanding = 0;

  printBanner();

  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  recordDueDate(invoice);
  printDetails(invoice, outstanding);
}

function after1() {
  printBanner();

  //  1. 사용하는 곳 가까이로 옮긴다.
  let outstanding = 0;
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  recordDueDate(invoice);
  printDetails(invoice, outstanding);
}

function after2() {
  printBanner();

  recordDueDate(invoice);
  printDetails(invoice, outstanding);

  // 2. 추출할 부분을 새로운 함수로 만든다.
  function calculateOutstanding(invoice) {
    let outstanding = 0;
    for (const o of invoice.orders) {
      outstanding += o.amount;
    }
    return outstanding;
  }
}

function after3() {
  printBanner();
  // 3. 추출한 함수를 원래 변수에 저장
  let outstanding = calculateOutstanding(invoice);

  recordDueDate(invoice);
  printDetails(invoice, outstanding);

  function calculateOutstanding(invoice) {
    let outstanding = 0;
    for (const o of invoice.orders) {
      outstanding += o.amount;
    }
    return outstanding;
  }
}
```

## 함수 인라인하기

때로는 함수 본문이 이름만큼 명확한 경우도 있다. 그럴 때는 함수를 제거한다.

1. 인라인할 함수를 호출하는 곳을 모두 찾는다.
2. 각 호출문을 함수 본문으로 교체한다.
3. 하나씩 교체할 때마다 테스트한다.
4. 함수 정의를 삭제한다.

```jsx
function before() {
  return isSuHyeokChoi(name) ? "yes" : "no";
}

function isSuHyeokChoi(name) {
  return name === "choisuhyeok";
}

function after() {
  return name === "choisuhyeok" ? "yes" : "no";
}

// 삭제한다.
function isSuHyeokChoi(name) {
  return name === "choisuhyeok";
}
```

## 변수 추출하기

지역 변수를 활용하면 표현식을 쪼개 관리하기 더 쉽게 만들 수 있다.

복잡한 로직을 구성하는 단계마다 이름을 붙일 수 있어서 코드의 목적을 더 알기 쉽다.

```jsx
function before(order) {
  // 가격 === 기본 가격  - 등급 할인 + 배송비
  return order.count * order.price - order.grade * 0.05 + order.price * 0.1;
}

function after(order) {
  const basePrice = order.count * order.price;
  const gradeDiscount = order.grage * 0.05;
  const deliveryFee = order.price * 0.1;

  return basePrice - gradeDiscount + deliveryFee;
}
```

## 변수 인라인하기

변수는 함수 안에서 표현식을 가리키는 이름으로 쓰이며, 대체로 좋은 효과가 있지만 그 이름이 원래 표현식과 다를바가 없을 때도 있다. 이럴 때는 그냥 인라인하는것이 좋다.

```jsx
// before
const basePrice = order.basePrice;
return basePrice > 10000;

// after
return order.basePrice > 10000;
```

## 함수 선언 바꾸기 (함수 이름 바꾸기)

함수는 시스템의 구성 요소를 조립하는 연결부 역할을 한다.

연결부에서 가장 중요한 요소는 함수의 이름이다.

```jsx
// 매개변수를 속성으로 바꾸기
function isEven(number) {
  return number % 2 === 0;
}
```

## 변수 캡슐화하기

접근할 수 있는 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화 하는 것이 가장 좋다.

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수를 만든다.
2. 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다.
3. 변수의 접근 범위를 제한한다.

```jsx
// before
let owner = { name: "최수혁", age: "28" };
macbook.owner = owner;
owner = { name: "수혁최", age: "27" };

// after
let owner = { name: "최수혁", age: "28" };
function getOwner() {
  return owner;
}
function setOwner(data) {
  owner = data;
}

macbook.owner = getOwner();
setOwner({ name: "수혁최", age: "27" });
```

## 매개변수 객체 만들기

데이터 항목 여러 개가 이 함수 저 함수에서 함께 몰려다니는 경우 데이터 구조 하나로 모아준다.

데이터 사이의 관계가 명확해진다.

매개변수 수가 줄어든다.

같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 이름이 같기 때문에 일관성도 높여준다.

```jsx
const grade = {
	subject: "math",
	scores: [
						{score: 50, date: "2022-10-20"},
						{score: 50, date: "2022-10-20"}
						{score: 50, date: "2022-10-20"}
					]
	};
}

// before
function findOutScopeScore(grade, min, max) {
	return grade.scores.filter(item => item.score < min || item.score > max);
}

const res = findOutScopeScore(grade, subject.minScore, subject.maxScore);

// after
class ScoreRange {
	constructor(min, max){
		this._data = {min, max};
	}

	get min() {return this._data.min}
	get max() {return this._data.max}
}

const range = new ScoreRange(subject.minScore, subject.maxScore);

const res = findOutScopeScore(range);

function findOutScopeScore(grade, range) {
	return grade.scores.filter(item => item.score < range.min || item.score > range.max);
}

```
