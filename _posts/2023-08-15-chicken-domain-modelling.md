---
layout: post
title: 도메인 모델링
subtitle: 오늘의 치킨 프로젝트
author: Metishonora
categories: 프로젝트
tags: [Java, Spring]
---

개발에 앞서, 처음 생각했던 서비스의 방향성을 시퀀스 다이어그램으로 표시해 보았다.
```plantuml!
skinparam responseMessageBelowArrow true
autonumber

actor User
participant Service #42d4f5
participant API #99FF99

User -> Service : 메뉴 등록 요청
Service -> Service : 리뷰 수집
Service -> API : 분석 요청
API -> Service : 결과 전달
Service -> User : 메뉴 등록 완료
User -> Service : 메뉴 추천 요청
Service -> User : 결과 반환
```

최종적으로는 이런 방향으로 만들고 싶지만, 한 번에 개발하기에는 어려워 보여 아래와 같이 어드민 역할을 분리해 보았다.

```plantuml!
skinparam responseMessageBelowArrow true
autonumber

actor User
participant Service #42d4f5
actor Admin
participant API #99FF99

Admin -> API : 등록할 메뉴의 리뷰 분석
API -> Admin : 감정분석 결과 전달
Admin -> Service : 메뉴 등록
User -> Service : 메뉴 추천 요청
Service -> User : 결과 반환
```
이렇게 접근하니 단숨에 프로젝트가 쉬워지는 느낌이 들었다.
빠르게 기능을 개발하려면 1~3 부분을 하드코딩하고, 4~5 부분을 먼저 만드는 것도 괜찮을 것 같다.

## 클래스 다이어그램

Get, Set 메서드는 생략하였다.

```plantuml!
class Store {
	id : Integer
	name : String
	menus : List<Menu>
}
class Menu {
	id : Integer
	name : String
	category : String
	price : Integer
	reviews : List<Reviews>
}
class Review {
	id : Integer
	text : String
	positiveScore : Double
	image : Image
}

Store *-- Menu
Menu *-- Review
```

## 어려웠던 점
[TDD에 대한 책](https://www.google.co.kr/books/edition/Test_driven_Development/CUlsAQAAQBAJ)을 읽으면서 프로젝트에 적용하고 싶었는데, domain 설계에는 어떻게 적용하는 게 좋을지 쉽게 떠오르지 않았다.

오히려 생각나는 대로 클래스 다이어그램을 작성하고, 그대로 코드로 옮기는 것이 직관적이었던 것 같다.
TDD에 익숙해지려면 테스트를 작성하는 연습이 더 필요할 수 있겠다.

## 다음 작업
이렇게 구상한 도메인 클래스를 기반으로 리포지토리, 서비스, 컨트롤러 계층의 개발을 계속 이어갈 예정이다.
그러나 아직은 각 계층이 어떤 역할을 담당해야 하는지, 어떤 기능이 필요한지 확실하지 않다.
좀 더 공부해 봐야 프로젝트 진행 속도가 붙을 것 같다.