---
layout: post
title: 캡슐화를 돕는 DTO 사용기
subtitle: 우테코 6기 프리코스 3주차
author: Metishonora
categories: 우테코
tags: [우테코프리코스, MVC, DTO, 클린아키텍처]
---

## 코드의 의존성 낮추기
재사용이 가능한 코드를 만들기 위해선 코드 간 **의존성**을 낮춰야 한다.
의존성을 낮추기 위한 방법으로 [MVC 패턴](/우테코/2023/11/07/1-woowacourse-pre3-mvc)이 사용된다.

MVC 패턴에서는 모델과 뷰가 서로의 존재를 모른다.
모델의 내부 구조가 바뀌든, 뷰의 표현 방식이 바뀌든 서로 영향을 주지 않아야 하기 때문이다.

하지만 데이터를 사용자에게 보여주기 위해선 결국 어떤 방식으로든
도메인이 가진 내용물을 뷰에게 전달해야 한다.

## 모델을 그대로 뷰에 넘겼을 때
```java
class User {
	private final String name;
	private final String password;
	private final int age;

	public User() {...}
	public String getPassword() {...}
	public someOtherLogic() {...}
	...
}

// controller.java
void printUser() {
	User user = new User();
	...
	outputView.print(user);  // 뷰에 User 객체를 그대로 전달한다.
}
```
위 코드에서는 뷰에서 *password*필드에 getter를 통해 접근할 수 있다.
또한 뷰에서 사용하지 않을 다른 로직 메서드에도 접근할 수 있다.
캡슐화가 잘 이루어지지 않아, 모델과 뷰 간의 결합(coupling)이 생길 수 있다 [[2]].
즉 모델이나 뷰의 요구사항이 변했을 때, 많은 코드를 변경해야 할 수 있을 것이다.

## DTO의 등장
이렇게 캡슐화가 깨지는 상황을 방지하기 위해 **DTO(Data Transfer Object)**를 활용할 수 있다 [[1]].
DTO는 원래 객체에서 원하는 데이터를 담아 전달할 수 있다 [[3]].
```java
class UserNameDto {
	public final String name;
	private UserNameDto(String name) {...}
	public static UserNameDto from(User user) {
		return new UserNameDto(user.name);
	}
}

void printUser() {
	...
	outputView.print(UserNameDto.from(user));
}
```
위 코드는 User 객체에서 *name*필드만을 DTO에 담아 뷰에 넘겨주는 모습이다.
이렇게 넘겨준다면 뷰에서는 더 이상 User 객체 자체에 접근할 수 없고,
캡슐화가 깨지지 않는 모습을 확인할 수 있다.

## 과제에 적용한 모습
이번 과제에 DTO를 적용할 때는:
- 뷰와 모델의 의존도 약화하기
- 모델의 캡슐화
- 코드 재사용성

이 세 가지 사항에 대해서 집중하였다.
```java
// Lotto.java
public class Lotto {
	private final List<Integer> numbers;
	...
}

// OutputView.java
public class OutputView {
	public void printObject(Object arg) {
		System.out.println(arg);
	}
}
```
고민했던 점은 private 선언된 *numbers*필드의 정보를 Dto로 건네주는 방법이다.
Getter 메서드를 추가하여 Dto에 전달해줄 수 있지만(실제로 처음 Dto를 공부했던 글[[1]], [[2]]에서는 이 방법을 권장하는 것 같다),
OutputView에 만들어둔 코드를 재사용하고 싶어 toString()을 오버라이딩하여 사용하였다.
```java
public class ResultDto {
	private final String stat;
	...
	@Override
	public String toString() {
		return stat;
	}
}

public class LottoResult {
	@Override
	public String toString() {
		// 복잡한 String 변환 코드...
	}
}
```
문제는 이렇게 완성하고 보니, 뷰의 책임은 굉장히 가벼워진 대신 출력을 구성하는 책임이 모델에게 넘어오게 되었다.

DTO를 사용해 정보를 전달할 때,
도메인 객체의 toString()을 활용하여 String으로 파싱한 후 전달하였다.
즉 뷰에서 담당해야 하는 역할(출력을 어떻게 구성할지)을 도메인에서 담당하였다.
뷰 요구사항이 바뀔 경우, 도메인을 변경해야 하는 문제가 있을 것 같다.

뷰와 모델 간 어느 정도의 결합까지 허용해야 좋은 것인지,
그리고 클래스 책임 분배를 어떻게 해야 하는지는 언제나 어려운 문제인 것 같다.

[프로젝트 전체 코드](https://github.com/metishonora/java-lotto-6)

## References
\[1] [DTO vs VO vs Entity](https://tecoble.techcourse.co.kr/post/2021-05-16-dto-vs-vo-vs-entity)

\[2] [DTO의 사용 범위에 대하여](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope)

\[3] [DTO, Domain의 변환 위치에 대해서](https://www.mainfn.dev/DTO,%20DOMAIN%20%EB%B3%80%ED%99%98%20%EC%9C%84%EC%B9%98%20%EB%B0%8F%20%EA%B0%81%20%EB%B0%A9%EB%B2%95%EC%9D%98%20%EC%9E%A5%EB%8B%A8%EC%A0%90%EC%97%90%20%EB%8C%80%ED%95%9C%20%EC%83%9D%EA%B0%81)

[1]: https://tecoble.techcourse.co.kr/post/2021-05-16-dto-vs-vo-vs-entity
[2]: https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope
[3]: https://www.mainfn.dev/DTO,%20DOMAIN%20%EB%B3%80%ED%99%98%20%EC%9C%84%EC%B9%98%20%EB%B0%8F%20%EA%B0%81%20%EB%B0%A9%EB%B2%95%EC%9D%98%20%EC%9E%A5%EB%8B%A8%EC%A0%90%EC%97%90%20%EB%8C%80%ED%95%9C%20%EC%83%9D%EA%B0%81