# [8장] 기능 이동

## 함수 옮기기

좋은 소프트웨어 설계의 핵심은 모듈성이다.

모듈성이란 프로그램의 어딘가를 수정할 때 해당 기능과 깊이 관련된 작은 일부만 이해해도 가능하게 해주는 능력이다.

만약 어떤 함수가 자신이 속한 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 B로 옮기는게 마땅하다.

```jsx
// before
function trackSummary(points) {
  const totalTime = calcultaeTime();
  const totlaDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;

  return {
    time: totalTime,
    distance: totalDistance,
    pace: pace,
  };

  function calculateDistance() {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }
    return result;
  }
}
```

```jsx
// after
function trackSummary(points){
	const totalTime = calcultaeTime(point);
	const totlaDistance = calculateDistance();
	const pace = totalTime / 60 / totalDistance;

	return {
		time: totalTime,
		distance: totalDistance,
		pace: pace
	}
}

function calculateDistance(points){
		let result = 0;
		for (let i = 1; i < points.length; i++) {
			result += distance(points[i-1], points[i]);
		}
		return result;
	}
}
```

trackSummary과 관련이 적은 calculateDistance는 함수를 분리해서 trackSummary의 응집도를 높혀준다.

## 필드 옮기기

```jsx
// before
class Customer {
  constructor(name, discountRate) {
    this.name = name;
    this.discountRate = discountRate;
  }

  get discountRate() {
    return this.discountRete;
  }
  becomePreferred() {
    this.discountRate += 0.03;
  }
  applyDiscount(amount) {
    return amount.subtract(amount.multiply(this.discountRate));
  }
}

class CustomerContract {
  constructor(startDate) {
    this.startDate = startDate;
  }
}
```

```jsx
// after
// discountRate를 Customer -> CustomerContract으로 옮겨보자
class Customer {
  constructor(name, discountRate) {
    this.name = name;
    this.contract = new CustomerContract(dateToday());
    this.setDiscountRate(discountRate);
  }

  get discountRate() {
    return this.contract.discountRete;
  }
  becomePreferred() {
    this.discountRate += 0.03;
  }
  applyDiscount(amount) {
    return amount.subtract(amount.multiply(this.discountRate));
  }
}

class CustomerContract {
  constructor(startDate, discountRate) {
    this.startDate = startDate;
    this.discountRate = discountRate;
  }

  get discountRate() {
    return this.discountRate;
  }
  set discountRate(arg) {
    this.discountRate = arg;
  }
}
```

## 문장을 함수로 옮기기

중복 제거는 코드를 건강하게 관리하는 가장 효과적인 방법 중 하나다.

```jsx
// before
function renderPerson(outStream, person) {
  const result = [];
  result.push(`<p>${person.name}</p>`);
  result.push(renderPhoto(person.photo));
  result.push(`<p>제목: ${person.photo.title}</p>`);
  result.push(emitPhotoData(person.photo));
  return result.join("\n");
}

function photoDiv(p) {
  return ["<div>", `<p>제목: ${p.title}</p>`, emitPhotoData(p), "</div>"].join(
    "\n"
  );
}

function emitPhotoData(photo) {
  const result = [];
  result.push(`<p>위치: ${photo.location}</p>`);
  result.push(`<p>날짜: ${photo.date.toDateString()}</p>`);
  return result.join("\n");
}
```

```jsx
// after
function renderPerson(outStream, person) {
  const result = [];
  result.push(`<p>${person.name}</p>`);
  result.push(renderPhoto(person.photo));
  result.push(emitPhotoData(person.photo));
  return result.join("\n");
}

function photoDiv(p) {
  return ["<div>", emitPhotoData(p), "</div>"].join("\n");
}

function emitPhotoData(photo) {
  const result = [];
  result.push(`<p>제목: ${person.photo.title}</p>`);
  result.push(`<p>위치: ${photo.location}</p>`);
  result.push(`<p>날짜: ${photo.date.toDateString()}</p>`);
  return result.join("\n");
}
```

## 문장을 호출한 곳으로 옮기기

초기에는 응집도 높고 한 가지 일만 수행하던 함수가 어느새 둘 이상의 다른 일을 수행하게 바뀔 수 있다.

공통으로 사용하는 부분은 함수로 옮겨 재사용성을 높이고 사용하는 곳 마다 조금씩 다르게 사용 된다면 그 부분을 밖으로 빼준다.

1. 기존 함수에서 남길 코드를 함수로 추출한다.
2. 기존 함수에서 추출한 함수를 호출한다.
3. 기존 함수를 호출하는 곳에 옮길 코드를 인라인화 한다.

## 인라인 코드를 함수 호출로 바꾸기

함수는 여러 동작을 하나로 묶어준다. 함수의 이름이 코드의 목적을 말하기 때문 이해력이 높아진다.

중복을 없애는 데도 효과적이다.

## 문장 슬라이드하기

관련된 코드들이 가까이 모여 있다면 이해하기가 쉽다.

가장 흔한 사례는 변수를 선언하고 사용할 때다. 모든 변수 선언을 함수 첫머리에 모아두는 경향이 있는데 그것 보단 처음 사용할 때 선언하는 스타일을 선호한다.

```jsx
// before
let result;
if (availableResources.length === 0) {
  result = createResource();
  allocatedResources.push(result);
} else {
  result = availableResources.pop();
  allocatedResources.push(result);
}
return result;
```

```jsx
// after
const result =
  availableResources.length === 0 ? createResource() : availableResources.pop();

allocatedResources.push(result);

return result;
```

## 반복문 쪼개기

종종 반복문 하나에서 두 가지 일을 수행하는 모습을 보게 된다. 반복문을 수정해야 할 때마다 두 가지 일 모두를 잘 이해하고 진행해야 한다.

```jsx
// before
function reportYoungestAgeAndTotalSalary(people) {
  let youngest = people[0] ? people[0].age : Infinity;
  let totalSalary = 0;
  for (const p of people) {
    if (p.age < youngest) youngest = p.age;
    totalSalary += p.salary;
  }

  return `최연소: ${youngest}, 총 급여: ${totalSalary}`;
}
```

1. 사용하는 변수 가까이 이동
2. 로직 함수화
3. 내장 함수, 라이브러리 등으로 코드 간결화

```jsx
// after
function reportYoungestAgeAndTotalSalary(people) {
  return `최연소: ${youngest}, 총 급여: ${totalSalary}`;

  function totalSalary() {
    return people.reduce((total, p) => total + p.salary, 0);
  }

  function youngestAge() {
    return Math.min(...people.map((p) => p.age));
  }
}
```

## 반복문을 파이프라인으로 바꾸기

파이프라인을 이용하면 처리 과정을 일련의 연산으로 표현할 수 있다.

```jsx
// before

// 데이터
office, country, telephone
Chicago, USA, +1 312 373 1000
Beijing, China, +86 4008 900 505

function acquireData(input) {
  const lines = input.split('\n');
  let firstLine = true;
  const result = [];
  for (const line of lines) {
    if (firstLine) {
      firstLine = false;
      continue;
    }
    if (line.trim() === '') continue;
    const record = line.split(',');
    if (record[1].trim() === 'India') {
      result.push({ city: record[0].trim(), phone: record[2].trim() });
    }
  }
  return result;
}
```

```jsx
// after
function acquireData(input) {
  return input
    .split("\n")
    .slice(1)
    .filter((line) => line.trim() !== "")
    .map((line) => line.split(","))
    .filter((fields) => fields[1].trim() === "India")
    .map((fields) => ({ city: fields[0].trim(), phone: fields[2].trim() }));
}
```
