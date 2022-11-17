# [9장] 데이터 조직화

## 조건문 분해하기

거대한 코드 블록이 주어지면 코드를 부위별로 분해한 다음 의도를 살린 이름의 함수 호출로 바꾸자.

```jsx
// before
if (!date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd)) {
  charge = quantity * plan.summerRate;
} else {
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
}
return charge;
```

```jsx
// after
if (isSummer()) {
    charge = summerCharge();
  } else {
    charge = regularChage();
  }
  return charge;

  function isSummer() {
    return !date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd);
  }

  function summerCharge() {
    return quantity * plan.summerRate;
  }

  function regularChage() {
    return quantity * plan.regularRate + plan.regularServiceCharge;
  }
}

// after2
charge = isSummer() ? summerCharge() : regularCharge();
```

## 조건식 통합하기

비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드들을 하나로 통합하는 것이다.

&&, || 연산자를 사용해서 하나로 합칠 수 있다.

```jsx
// before
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

```jsx
// after
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
  return (
    anEmployee.seniority < 2 ||
    anEmployee.monthsDisabled ||
    anEmployee.isPartTime
  );
}
```

## 중첩 조건문을 보호 구문으로 바꾸기

조건문의 두가지 형태

1. 참, 거짓 경로 모두 정상 동작하는 경우
2. 참 또는 거짓 한쪽만 정상 동작하는 경우

한쪽만 정상이라면 비정상 조건을 if에서 검사한다음, 조건이 참(거짓)이면 함수에서 빠져나온다. 흔히 보호 구문이라고 한다.

중첩 조건문을 보호 구문으로 바꾸는 핵심은 의도를 부각하는 것이다.

if else문을 사용할 땐 if,else 절에 똑같은 무게를 두어 똑같이 중요하다는 것을 나타내는데 보호 구문은 이건 핵심이 아니기 때문에 조치를 취한후 함수를 빠져나가도록 한다.

```jsx
// before
export function payAmount(employee) {
  let result;
  if (employee.isSeparated) {
    // 퇴사한 직원
    result = { amount: 0, reasonCode: "SEP" };
  } else {
    if (employee.isRetired) {
      // 은퇴한 직원
      result = { amount: 0, reasonCode: "RET" };
    } else {
      // 급여 계산 로직
      lorem.ipsum(dolor.sitAmet);
      consectetur(adipiscing).elit();
      sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
      ut.enim.ad(minim.veniam);
      result = someFinalComputation();
      return result;
    }
  }
  return result;
}
```

```jsx
// after
// 조건 중 가장 바깥 것을 보호 구문으로 바꾼다.
export function payAmount(employee) {
  let result;
  if (employee.isSeparated) {
    // 퇴사한 직원
    result = { amount: 0, reasonCode: "SEP" };
  }

  if (employee.isRetired) {
    // 은퇴한 직원
    result = { amount: 0, reasonCode: "RET" };
  }

  // 급여 계산 로직
  lorem.ipsum(dolor.sitAmet);
  consectetur(adipiscing).elit();
  sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
  ut.enim.ad(minim.veniam);
  result = someFinalComputation();
  return result;
}
```

## 제어 플래그를 탈출문으로 바꾸기

제어 플래그란 코드의 동작을 변경하는 데 사용되는 변수

리팩토링으로 충분히 간소화 할 수 있음에도 복잡하게 작성된 코드에서 흔히 나타나기 때문이다.

```jsx
// before
let found = false;
for (const p fo people) {
	if (!found) {
		if (p === "조커"){
			sendAlert();
			found = true;
		}
		if (p === "사루만"){
			sendAlert();
			found = true;
		}
	}
}
```

```jsx
// after
let found = false;
for (const p fo people) {
	if (!found) {
		if (p === "조커"){
			sendAlert();
			return; // found가 true가 되면 종료하려고 했으니 return 으로 종료시킨다.
		}
		if (p === "사루만"){
			sendAlert();
			return;
		}
	}
}
```

```jsx
// after2
// 제어 플래그를 참조하는 모든 코드를 제거한다.
for (const p fo people) {
	if (p === "조커"){
		sendAlert();
		return;
	}
	if (p === "사루만"){
		sendAlert();
	  return;
	}
}
```
