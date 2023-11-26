---
layout: post
title: 클린 코드는 어떻게 작성하는가?
subtitle: 우테코 6기 프리코스 1주차 - 숫자 야구 회고
author: Metishonora
categories: 우테코
banner:
  image: /assets/images/banners/woowaheader.jpg
  opacity: 0.4
  height: 50vh
  min_height: 38vh
  heading_style: "font-size: 3em; font-weight: bold; text-decoration: underline"
  subheading_style: "font-weight: bold; color: gold"
tags: [우테코프리코스, 클린코드]
---
Github link:
[https://github.com/metishonora/java-baseball-6/tree/metishonora](https://github.com/metishonora/java-baseball-6/tree/metishonora)

## 개요
첫 인상은 굉장히 쉬워보였지만, 클린 코드에 대해 고민하면서 시간을 꽤 썼던 문제였다.

처음에는 PS 하듯이 메인 메서드에 모든 내용을 두고 구현할 수 있었다. [당시 커밋](https://github.com/metishonora/java-baseball-6/blob/f2be5569f4bdfb648d3139cf8ed4c0662622661b/src/main/java/baseball/Application.java)

이번 과제를 진행하며 깨달았던 내용을 간략하게 요약하고자 한다.
### 클린 코드 규칙
[우테코 클린 코드 체크리스트](https://github.com/woowacourse/woowacourse-docs/blob/main/cleancode/pr_checklist.md)에서 권장하는 사항은 다음과 같다.
> 1. 한 메서드에 한 단계의 indent만 허용하기
> 2. else 쓰지 않기
> 3. 모든 원시값과 문자열 포장하기
> 4. 일급 콜렉션 적용하기

이 규칙을 어떻게 적용할 수 있을지 고민하면서, 구현한 코드를 끊임없이 리팩토링을 하고 클린 코드에 대해 배울 수 있었다.

## 리팩토링
### 인덴트 줄이기

이 조건에 따르면 한 메서드 내에서 아래와 같은 코드는 사용할 수 없었다.
```java
/**
 * 컴퓨터의 숫자와 사용자의 숫자를 비교하여 출력하고, 3스트라이크일 때까지 반복한다.
 */
static void play() {                       // 0단계
	do {                                 // 1단계
		int[] result = pitcher.compareWith(hitter);
		if (result[1] > 0 && result[0] > 0) {   // 2단계, 금지.
			System.out.println(...);
		} else if (...) {
			...
		} else {
			...
		}
	} while (!foundAnswer(result));
}
```

왜 코드에서 반복문과 조건문을 동시에 사용하지 말라는 것인지 의문이 들었지만 우선 조건에 맞도록 메서드를 분리하고 나니 금세 답을 알 수 있었다.
```java
static void play() {
	do {
		int[] result = pitcher.compareWith(hitter);
		printResult(result);
	} while (!foundAnswer(result));
}
```
위쪽 코드는 주석이 있어야 무슨 일을 하는 코드인지 알 수 있었던 반면에,
아래쪽은 한눈에 흐름이 들어오는 코드로 바뀌었다.
이전에는 코드의 어느 부분을 분리하여 메서드로 만들어야 할지 몰라 절차적으로 작성했는데,
<span style="color: red">*인덴트 하나*</span>라는 명확한 기준이 생기자 쉽게 알기 쉬운 코드를 만들 수 있었다.

### 원시값과 문자열 포장하기
원시값(Primitive values)과 문자열을 함수 인자에 그대로 사용하지 말고, 의미 있는 상수에 넣어 사용하라는 내용이다.
위쪽 코드에도 *result\[1]*, *result\[0]*과 같이 무엇을 의미하는지 알기 어려운
상수가 있었다.

이런 값들을 알기 쉽고 간결하게 코드로 표현하고 싶었고, 그러면서 *Enum*과 *DTO*에 관해 자연스럽게 학습할 수 있었다.
```java
// Enum으로 정의한 경우
public enum Result {
	STRIKE(0),
	BALL(1);
	private final int index;
	Result(int index) {...}
	public int getIndex() {...}
}
// 사용 방식은 이랬다
if (result[STRIKE.getIndex()] > 0) { /* 스트라이크 개수 출력 */ }

// record를 사용한 DTO로 정의한 경우
public record Result(
	int strike,
	int ball
) {}
// 사용 방식은 이랬다
if (result.strike() > 0) { /* 스트라이크 개수 출력 */ }
```
확실히 두 경우 모두 기존의 상수를 사용할 때보단 이해하기 쉬워 보였다.

어떤 것을 사용하는 것이 좋을지 오랜 시간 고민했지만,
1. C와 다르게 Enum을 단순 정수값으로 취급할 수 없는 점
2. 이 때문에 result[STRIKE.getIndex()]처럼 더 긴 코드를 사용해야 하는 점

이런 문제 때문에 최종 코드에서는 record를 사용하였다.

### 일급 컬렉션
일급 컬렉션(First Class Collection) 부분은 이번 과제를 진행하면서 왜 사용해야 하는지 가장 많이 고민했던 부분이었다.

우선 일급 컬렉션이란, 하나의 Java Collection 변수만을 멤버로 가지는 Wrapper class를 의미한다.
```java
// 기존에 사용하던 리스트를...
List<Integer> numbers = new ArrayList<>();

// Wrapper class로 감싼다
public class Player {
	private List<Integer> numbers;
	public Player() {...}
	...
}
```
이때 중요한 것은 getter/setter를 제공하지 않는 것으로써 생성된 값을 변경하지 못하게 하는 것이다.
```java
// 컬렉션을 final로 선언하더라도...
final List<Integer> canModifyThis = new ArrayList<>();
// 값을 변경하는 것은 막을 수 없다.
canModifyThis.put(1);

// 이렇게 만든 경우...
public class Player {
	private List<Integer> numbers;
	public Player(List<Integer> numbers) {
		this.numbers = new ArrayList<>(numbers);
	}
}
Player finalList = new Player(Arrays.asList(1, 2, 3));
// getter와 setter를 아예 정의하지 않아서 수정할 수가 없다.
```
이 외에도 관련 로직을 한 곳에 묶어준다는 장점 등 여러 가지 좋은 점들이 있었다.
자세한 것은 [이전 우테코 관련 글][1]\[1]을 보면 자세하게 설명되어 있다.

## 다른 배운 점들

이렇게 클린 코드와 관련된 부분 밖에도, 최근 PS만을 진행하면서 소홀히 했던 부분들을
다시 돌아볼 수 있는 기회가 되기도 했다.
1. [try~catch문과 throws, throw의 차이][2]\[2]
2. [Java에서 메서드가 로컬 배열을 반환해도 되는가?][3]\[3]

## 아쉬운 점

### else 없애기
[다른 글][4]\[4]을 찾아보았을 땐 if-else를 나열하는 대신,
Enum과 stream을 사용해 모든 분기를 깔끔한 코드로 관리하는 모습을 볼 수 있었다.
하지만 아직 이런 API에 익숙하지 않아 잘 이해하지 못했고,
이번 결과물에는 else만 없을 뿐, if 분기의 나열이 끝까지 남게 되었다.
```java
public static void printResult(Result result) {
	if (result.strike() > 0) { /* 스트라이크 개수 출력 */ }
	if (result.ball() > 0) { /* 볼 개수 출력 */ }
}
```

### 주석 처리
클린 코드 도서[5]에 따르면 주석을 다는 것보다
코드 자체를 이해하기 쉽게 만드는 것이 바람직하다고 한다.

과제 종료 이후 적은 주석을 다시 보니, 나쁜 주석에 속하는 것들이 많은 것을 확인할 수 있었다.
- 같은 이야기를 반복하는 주석

```java
/**
 * 게임을 재시작할 것인지 입력받습니다.
 * 입력 형식에 맞지 않을 경우 IllegalArgumentException을 던질 수 있습니다.
 */
    public static boolean checkReplay() {
        String input = Console.readLine();
        if (input.equals("1")) {
            return true;
        } else if (input.equals("2")) {
            return false;
        }
        throw new IllegalArgumentException();
    }
```

- 의무적으로 다는 주석. 예전에 C 코딩을 하면서 생겼던 습관 때문에 의무적으로 작성했던 것 같다.

```java
/**
 * 투수 객체와 타자 객체를 비교한 결과를 출력합니다.
 * @param result 스트라이크 수, 볼 수를 담은 Result 레코드
 */
public static void printResult(Result result) {
	...
}
```
## 마무리
아쉬운 점도 여럿 있었지만, 다음 주차 과제에서는 개선해보려 한다.
커뮤니티가 잘 활성화된 덕분에 이런 배울 점, 아쉬운 점들을 쉽게 확인할 수 있었다.

이번 기회를 통해 다른 사람들도 이해하기 쉬운, 생산성 좋은 코드의 중요성을
실감할 수 있었다. 연습을 통해 클린한 코드를 작성할 수 있도록 노력해야겠다.

다음 주차부터는 진행하면서 고민한 내용을 차근차근 풀어보려고 한다.

## Reference
\[1] [https://tecoble.techcourse.co.kr/post/2020-05-08-First-Class-Collection/](https://tecoble.techcourse.co.kr/post/2020-05-08-First-Class-Collection/)

\[2] [https://velog.io/@mooh2jj/%EC%9E%90%EB%B0%94-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%ACtry-catch-throw-throws](https://velog.io/@mooh2jj/%EC%9E%90%EB%B0%94-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%ACtry-catch-throw-throws)

\[3] [https://www.geeksforgeeks.org/arrays-in-java/](https://www.geeksforgeeks.org/arrays-in-java/)

\[4] [https://velog.io/@lxxjn0/else-%EC%98%88%EC%95%BD%EC%96%B4%EB%A5%BC-%EC%93%B0%EC%A7%80-%EC%95%8A%EB%8A%94%EB%8B%A4](https://velog.io/@lxxjn0/else-%EC%98%88%EC%95%BD%EC%96%B4%EB%A5%BC-%EC%93%B0%EC%A7%80-%EC%95%8A%EB%8A%94%EB%8B%A4)

\[5] R. C. Martin, Clean Code, 인사이트.

[1]: https://tecoble.techcourse.co.kr/post/2020-05-08-First-Class-Collection/
[2]: https://velog.io/@mooh2jj/%EC%9E%90%EB%B0%94-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%ACtry-catch-throw-throws
[3]: https://www.geeksforgeeks.org/arrays-in-java/
[4]: https://velog.io/@lxxjn0/else-%EC%98%88%EC%95%BD%EC%96%B4%EB%A5%BC-%EC%93%B0%EC%A7%80-%EC%95%8A%EB%8A%94%EB%8B%A4