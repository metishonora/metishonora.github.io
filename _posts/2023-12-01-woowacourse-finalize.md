---
layout: post
title: 클린 코드 작성하기 - 우테코를 마치며
subtitle: 우테코 6기 프리코스 마무리
author: Metishonora
categories: 우테코
tags: [우테코프리코스, 클린코드]
---

## 왜 클린코드가 중요할까
프로젝트를 진행하다 보면 다른 사람의 코드를 읽거나, 요구사항이 변하는 경우가 굉장히 많았다.
4주간의 우테코 프리코스를 진행하는 중에도 *요구사항 변화에 대응할 수 있어야 한다*는 피드백이 있었다.

하지만 코드를 기간에 쫓겨 작성하다 보면 깔끔한 코드에서 멀어지기도 하고,
코드를 작성한 직후에는 어느 부분이 깔끔하지 않았는지 파악하기도 쉽지 않다.
그래서 평소에 클린 코드를 작성하려고 노력해야 할 것이다.

이번 우테코 과정 중에는 다른 사람들과 코드 리뷰를 하면서
고칠 부분을 쉽게 찾을 수 있었고,
덕분에 짧은 4주 동안 코드 스타일을 많이 변화시킬 수 있었던 것 같다.

## 문제의 부분들
1주 차에 작성했던 코드를 다시 보면서 부족했던 부분들을 찾아봤다.

### 주석보다 코드로 표현하기
코드만으로 나타내기 어려운 *작성한 의도*나 *주의할 부분*을 표현하기 위해서라도,
주석을 작성하는 것은 중요한 일이라고 생각한다.
라이브러리처럼 재사용 가능성이 높은 코드일수록 자세한 주석이 도움 되는 경우가 많을 것이다.

하지만 주석에만 의존하는 대신, 우선 **코드 자체를 알기 쉽게** 작성하는 것을 우선시해야 한다.

아래는 1주 차, 야구 게임 과제에서 작성했던 코드이다.
```java
public class Player {
	// 플레이어가 작성한 숫자 (예시: 123)
	private final List<Integer> numbers;

	/**
	 * 자신과 another 객체의 숫자값을 비교합니다.
	 * 예시: 123과 192를 비교하면 1스트라이크 1볼을 반환합니다.
	 *
	 * @param another 비교할 대상
	 * @return [strike 개수, ball 개수]를 나타내는 Result 객체
	 */
	public Result compareWith(Player another) {
		int strike = 0;
		int ball = 0;

		for (int i = 1; i <= numbers.size(); i++) {  // i번째 자리 숫자 확인
			int myNumber = numbers.get(i - 1);
			int anotherNumberAt = another.containsNumber(myNumber);
			if (anotherNumberAt == i) {  // strike
				strike++;
			} else if (anotherNumberAt > 0) {  // ball
				ball++;
			}
		}
		return new Result(strike, ball);
	}
}
```
그리고 이쪽은 4주 차, 크리스마스 프로모션 과제에서 작성했던 코드이다.
```java
// 현재 이벤트의 혜택을 받을 수 있는지 판단합니다.
public boolean isEligible(EntireOrder orders, Day day) {
	return Event.isEligibleForEntireEvent(orders) &&
			day.isDayAfter(DAY_WHEN_EVENT_ENDS);
}

// 전체 이벤트에 참여하기 위한 최소 조건
static boolean isEligibleForEntireEvent(EntireOrder orders) {
	return orders.calculateEntirePrice() >= MINIMUM_PURCHASE_FOR_ENTIRE_EVENT;
}
```
위쪽 코드도 주석이 있다면 이해하기 어려운 코드는 아니라고 생각하지만,
아래쪽 코드가 직관적으로 빠르게 파악하기 쉽다.
메서드를 분리하여 최대한 작은 책임을 지게 하였고, 자세하게 이름을 달아 줬다.
덕분에 단위 별로 테스트하기도 편해졌고, 이후 변경 사항에도 대처하기 쉬울 것이다.

### else 제거하기
무분별한 else 구문을 지양해야 하는 이유 역시
코드를 읽기 어렵게 만들고,
디버깅을 어렵게 만들어 유지보수가 힘들어지기 때문이다.

다시 한번 1주 차 코드를 소개한다.
```java
/**
 * 투수 객체와 타사 객체를 비교한 결과를 출력합니다.
 *
 * @param result 스트라이크 수, 볼 수를 담은 Result 레코드
 */
public static void printResult(Result result) {
	if (result.strike() > 0 && result.ball() > 0) {
		System.out.printf("%d볼 %d스트라이크\n", result.strike(), result.ball());
	} else if (result.strike() > 0) {
		System.out.printf("%d스트라이크\n", result.strike());
	} else if (result.ball() > 0) {
		System.out.printf("%d볼\n", result.ball());
	} else {
		System.out.println("낫싱");
	}
}
```
반복적인 부분이 많아서, 출력 요구사항이 변하거나 조건이 변한다면
여러 부분을 변경할 가능성이 높다.
조건이 많아질수록 분기는 복잡해지고, 실수하기 쉬워질 것이다.

4주 차 코드에서는 if-else를 나열하여 입력값을 검증하는 대신,
별도의 검증 메서드로 분리하였다.
```java
public static EntireOrder readOrders(String line) {
	validateRegex(line);

	List<Order> orders = Arrays.stream(line.split(ORDER_SEPARATOR))
			.map(MenuReader::readSingleOrder)
			.toList();

	validateOrdersAreDistinct(orders);
	validateOrdersSize(orders);
	validateOrdersHaveNonDrinkMenu(orders);

	return new EntireOrder(orders);
}

// 예시
private static void validatePattern(String line) {
	if (!line.matches("^(.+-\\d+,)*.+-\\d+$")) {
		throw new IllegalArgumentException(MENU_EXCEPTION);
	}
}
```
다른 부분에서도 early return을 사용하고,
메서드를 분리하여 가독성을 높였다.
```java
// 전
public Integer getEventBenefitAmount(EntireOrder orders, Day day) {
	if (Event.isEligibleForEntireEvent(orders)) {
		if (orders.calculateEntirePrice() >= MINIMUM_PURCHASE) {
			return Drink.CHAMPAGNE.getPrice();
		} else {
			return 0;
		}
	} else {
		return 0;
	}
}
```
```java
// 후
public Integer getEventBenefitAmount(EntireOrder orders, Day day) {
	if (!isEligible(orders, day)) {
		return 0;
	}
	return Drink.CHAMPAGNE.getPrice();
}

public boolean isEligible(EntireOrder orders, Day day) {
	return Event.isEligibleForEntireEvent(orders) &&
		orders.calculateEntirePrice() >= MINIMUM_PURCHASE;
}
```
덕분에 각 조건이 무슨 의미를 가지는지 파악하기 쉬워졌고,
단위 테스트를 쉽게 작성할 수 있었다.

## 무리한 리팩토링도 문제다
한편 너무 리팩토링에만 집중한 나머지, 오히려 알기 어려워진 부분도 있었다.
예를 들자면, 3주 차에는 [DTO 패턴](/우테코/2023/11/07/2-woowacourse-pre3-dto)을 배워 사용했다.
```java
public class LottoDto {
    private final String numbers;

    private LottoDto(Lotto lotto) {
        numbers = lotto.toString();
    }

    public static LottoDto from(Lotto lotto) {
        return new LottoDto(lotto);
    }

    @Override
    public String toString() {
        return numbers;
    }
}
```
DTO는 **모델과 뷰의 결합을 줄이기 위해** 사용하는 것이지만,
내가 작성한 코드에서는 그렇지 않았다.
사용하는 이유를 잘 모르고 단순히 편하게 toString을 정의한 뒤
내부 로직, 테스팅, 출력 등 많은 부분에서 사용했다.

겉보기에는 코드가 단순했지만, 한 곳에서의 변경이 여러 부분에 영향을 주게 되었다.
4주 차 피드백에서도 이 내용을 확인할 수 있었다.
바로 *단일 책임의 원칙*을 지키고,
비즈니스 로직과 UI 로직을 하나의 클래스/메서드가 담당하지 않도록 하라는 것이었다.

즉 왜 리팩토링이 필요한지 이유를 알고 필요한 장소를 잘 파악하는 것이 중요할 것이다.

## 마치며
지금까지 우테코 프리코스를 진행하면서 배운 내용들 중에,
클린 코드 관련해 기억에 남는 부분을 정리했다.

어떻게 좋은 코드를 만들 수 있는지
위 내용 외에도 소통할 기회가 있었고,
그중 편하게 중복을 제거하고 객체 지향적인 코드를 짤 수 있게 도와주는
스프링에 대해 소개받을 수 있었다.

우테코를 더 길게 진행할 수 있었다면 좋았겠지만,
그렇지 않더라도 스프링을 공부하면서
클린 코드를 만들기 위해 노력해 보려 한다.