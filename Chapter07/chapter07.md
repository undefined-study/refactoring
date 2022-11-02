# [7장] 캡슐화

## 레코드 캡슐화하기

레코드는 보통 key, value로 이루어진 것

```jsx
// before
const organization = { name: "Acme Gooseberries", country: "GB" };

organization.name = "Dream Coding";
console.log(organization.name);
console.log(organization.country);
```

```jsx
// after
class Organization {
  constructor(data) {
    this.data = data; // 원본 데이터 보존
    this.name = data.name;
    this.country = data.country;
  }

  get name() {
    return this.name;
  }
  set name(name) {
    this.name = name;
  }
  get country() {
    return this.country;
  }
  set country(country) {
    this.country = country;
  }

  // 수정된 데이터가 아닌 원본 데이터를 get하는 함수
  get rawData() {
    return cloneDeep(this.data);
  } // 깊은복사
}
```

## 컬렉션 캡슐화하기

```jsx
// before
class Person {
  constructor(name) {
    this._name = name;
    this._courses = [];
  }

  get name() {
    return this._name;
  }
  get courses() {
    return this._courses;
  }
  set courses(list) {
    this._courses = list;
  }
}

class Course {
  constructor(name, isAdvanced) {
    this._name = name;
    this._isAdvanced = isAdvanced;
  }

  get name() {
    return this._name;
  }
  get isAdvanced() {
    return this._isAdvanced;
  }
}
```

```jsx
// after
class Person {
  constructor(name) {
    this._name = name;
    this._courses = [];
  }

  get name() {
    return this._name;
  }
  get courses() {
    return this._courses;
  }

  // 아래 addCourse, removeCourse를 생성함으로서 set courses를 제거
  set courses(list) {
    this._courses = list;
  }

  addCourse(course) {
    this._courses.push(course);
  }
  removeCourse(course) {
    const index = this._courses.indexOf(course);
    if (index === -1) return;
    this._courses.splice(index, 1);
  }
}
```

## 기본형을 객체로 바꾸기

처음에는 단순한 문자열로 시작해서 나중에는 포맷팅, 특별한 동작이 추가 되는 등 중복 코드가 늘어나고 사용할 때마다 드는 노력도 늘어난다.

출력 이상의 기능이 필요해지는 순간 클래스를 정의하면 좋다.

클래스로 집중시켜 놓으면 관련된 로직을 한 곳에서 관리 할 수 있으면 중복된 코드를 처리 할 수 있다.

```jsx
// before
class Order {
  constructor(data) {
    this.priority = data.priority;
  }
}

highPriorityCount = orders.filter(
  (o) => "high" === o.priority || "rush" === o.priority
).length;
```

```jsx
// after
class Order {
	constructor(data){
		this.priority = data.priority;
	}

	get priorityString() {return this._priority.toString()}
	set priority(string) {this._priority. new Priority(string)}
}

class Priority {
	constructor(value) {this._value = value}
	toString() {return this._value}
}
```

## 임시 변수를 질의 함수로 바꾸기

함수 안에서 어떤 코드의 결괏값을 다시 참조할 목적으로 임시 변수를 사용한다.

임시 변수를 사용하면 계산하는 중복 코드를 줄이고 값의 의미를 설명할 수 있어 유용하다.

그런데 더 나아가 아예 함수로 만들어 사용하는 편이 나을 때가 많다.

```jsx
// before
class Order {
  constructor(quantity, item) {
    this._quantity = quantity;
    this._item = item;
  }

  get price() {
    const basePrice = this._quantity * this._item.price;
    const discountFactor = 0.98;

    if (base > 1000) discountFactor -= 0.03;
    return basePrice * discountFactor;
  }
}
```

```jsx
// before
class Order {
  constructor(quantity, item) {
    this._quantity = quantity;
    this._item = item;
  }

  get basePrice() {
    return this._quantity * this._item.price;
  }

  get discountFactor() {
    return base > 1000 ? 0.95 : 0.98;
  }

  get price() {
    return this.basePrice * this.discountFactor;
  }
}
```

## 클래스 추출하기

기존 클래스를 굳이 쪼갤 필요까지 없다고 생각해서 새로운 역할을 덧씌우며 점점 복잡해는데 이해하기 어려우니 적절히 분리하는 것이 좋다.

```jsx
// before
class Person{
	conststructor(name,countryCode, phoneNumber){
		this.name = name
		this.countryCode = countryCode
		this.phoneNumber = phoneNumber
	}

	get name() {return this.name}
	set name(name) {this.name = name}
	get telephoneNumber() {return `(${this.countryCode} ${this.phoneNumber})`}
	get countryCode = {return this.countryCode}
}
```

```jsx
// after
class Person{
	constructor(name,){
		this.name = name
		this.telephone = new Telephone();
	}

	get name() {return this.name}
	set name(name) {this.name = name}
	get telephoneNumber() {return `(${this.telephone.countryCode} ${this.telephone.phoneNumber})`}
	get countryCode = {return this.telephone.countryCode}

}

class Telephone {
	constructor(countryCode, phoneNumber){
		this.countryCode = countryCode
		this.phoneNumber = phoneNumber
	}

	get telephoneNumber() {return `(${this.countryCode} ${this.phoneNumber})`}
	get countryCode = {return this.countryCode}
}
```

## 위임 숨기기

```jsx
// before
class Person {
  constructor(name, phone) {
    this.name = name;
    this.phone = phone;
  }
  get phone() {
    return this.phone;
  }
}

class Phone {
  construcktor(phoneNumber) {
    this.phoneNumber = phoneNumber;
  }
}

const person = new Person("suhyeok", new Phone("010-1234-1234"));
console.log(person.phone.phoneNumber);
```

```jsx
// after
class Person {
  constructor(name, phone) {
    this.name = name;
    this.phone = phone;
  }
  get phoneNumber() {
    return this.phone.phoneNumber;
  }
}

class Phone {
  construcktor(phoneNumber) {
    this.phoneNumber = phoneNumber;
  }
}

const person = new Person("suhyeok", new Phone("010-1234-1234"));
console.log(person.phoneNumber);
```
