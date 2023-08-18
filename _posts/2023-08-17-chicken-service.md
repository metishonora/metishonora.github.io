---
layout: post
title: TDD로 리포지토리와 서비스 개발하기
subtitle: 오늘의 치킨 프로젝트
author: Metishonora
categories: 프로젝트
tags: [Java, Spring, TDD]
---
[이전 포스팅][linkToPrev]에 이어서, TDD를 활용하여 repository와 service 부분을 구현하였다.
이번 글에서는 어떤 과정을 거쳐서 구현에 성공하였는지 차례대로 소개하겠다.

TDD를 진행하는 과정은 이렇다:
1. 테스트 코드를 먼저 작성한다.
2. 실패하는 것을 확인한다.
3. 테스트 통과를 위해 약간 수정한다.
4. 리팩토링한다.

## 테스트 목록 작성하기
우선 작성했던 [시퀀스 다이어그램][linkToPrev]에서,
3) 메뉴 등록부터 5) 결과 반환까지를 작업하기로 한다. 그다음에 할 일 목록을 생각나는 대로 적었다.

|--|-------------------------------------------|
|**_1_**| **_메모리에 메뉴 등록하기_**|
|**_2_**| **_무작위 선택하기_**|
|3| 특정 카테고리에서 선택하기|
|4| 특정 가게에서 선택하기|
|5| 선택 시 점수 제한 옵션 주기|
|6| 메뉴 등록 페이지 생성하기|
|7| 메뉴 선택 페이지 생성하기|
|8| 선정 결과 페이지 생성하기|

이렇게 작성한 할 일 목록이 곧 테스트 목록이 된다.

굵게 표시한 1, 2번 테스트를 먼저 작성하기로 하였다.

## 리포지토리 구현하기
이전에 Menu 클래스는 이렇게 구현한 상태였다.
```java
// main/java/domain/Menu.java
public class Menu {
    private Long id;
    private String name;
    private String category;
    private Integer price;
    private List<Review> reviews;
}
```
이 클래스를 메모리에 저장하기 위한 <span style="color:orangered">**테스트 클래스를 먼저 작성**</span>한다.
### 클래스 생성
```java
// test/java/repository/MemoryMenuRepositoryTest.java
class MemoryMenuRepositoryTest {
	private final MemoryMenuRepository memoryMenuRepository;

	public MemoryMenuRepositoryTest(MemoryMenuRepository memoryMenuRepository) {
		this.memoryMenuRepository = memoryMenuRepository;
	}

}
```
당연하지만, 이 상태에서 테스트를 실행하면 Fail한다.
애초에 리포지토리 클래스를 작성하지 않았기 때문에 컴파일 에러가 발생한다.
![230817-1](/assets/posts/230817-1.png)

다음 과정은 <span style="color:orangered">**테스트를 통과하게 만들기**</span>이다.
이때 완벽한 코드를 구현해도 되지만, 빠른 통과를 위해서 [가짜 구현][stub] 등을 활용할 수도 있다.
```java
// main/java/repository/MemoryMenuRepository.java
public class MemoryMenuRepository {
	public void clear() {
		// 아무 것도 안 함
	}
}
```
이렇게만 하면 일단 Fail은 사라진다.
![230817-2](/assets/posts/230817-2.png)

다음 과정은 <span style="color:orangered">**리팩토링**</span>인데, 아직은 적은 코드가 별로 없어서 넘어간다. 그 대신 할 일 목록을 업데이트하였다.

|-----------------------|
|~~클래스 생성~~ (완료)|
|메모리에 메뉴 등록하기|
|clear 구현하기 _(가짜 구현을 해뒀으므로 할일로 추가하였음)_|
|무작위 선택하기|

### 메뉴 등록
이번엔 정말로 메뉴를 등록하는 테스트 케이스를 작성하였다.
```java
// test/java/repository/MemoryMenuRepositoryTest.java
@Test
public void 메뉴등록하기() {
	// given
	Menu menu = new Menu();
	menu.setID(memoryMenuRepository.size() + 1);
	menu.setName("마법클");
	menu.setCategory("시즈닝");
	menu.setPrice(20000);
	menu.setStore("BHC");
	menu.setScore(90);

	// when
	memoryMenuRepository.save(menu);

	// then
	Menu result = memoryMenuRepository.get(1);
	assertThat(result.getId()).isEqualTo(1);
	assertThat(result.getName()).isEqualTo("마법클");
	assertThat(result.getCategory()).isEqualTo("시즈닝");
	assertThat(result.getPrice()).isEqualTo(20000);
	assertThat(result.getStore()).isEqualTo("BHC");
	assertThat(result.getScore()).isEqualTo(90);
}
```
테스트는 실패한다. 그리고 사실 Menu 클래스 안에 store 필드는 구현해 두지 않았지만,
테스트를 작성하면서 어떤 필드가 있으면 좋을지 생각하다가 포함하게 되었다. 어차피 다음 과정에서 고치게 될 것이다.
```java
// main/java/domain/Menu.java
public class Menu {
	...
    private String store;
    private Integer score;
	...
	// Getter, Setter 생략
}

// main/java/repository/MemoryMenuRepository.java
public class MemoryMenuRepository {
	private static List<Menu> store = new ArrayList<>();
	public void clear() {
		// 아무 것도 안 함
	}
	public Long size() {
		return (long) store.size();
	}

	public void save(Menu menu) {
		store.add(menu);
	}

	public Menu get(Integer index) {
		return store.get(index - 1);
	}
}
```
우선은 생각나는 대로 ArrayList를 사용해 메모리에 메뉴 정보를 담기로 한다.
그리고 컴파일 에러를 없애기 위한 클래스를 작성한다.

추가로 [의존성 주입][DI]을 위해 Config 파일도 작성해야 한다.[3]
그래야 테스트 클래스의 repository 변수에 미리 생성된 repository가 잘 연결된다.
```java
// main/java/SpringConfig.java
@Configuration
public class SpringConfig {
    @Bean
    public MemoryMenuRepository memoryMenuRepository() {
        return new MemoryMenuRepository();
    }
}

// test/java/repository/MemoryMenuRepositoryTest.java
@SpringBootTest
public class MemoryMenuRepositoryTest {
	private final MemoryMenuRepository memoryMenuRepository;

	@AutoWired
	public MemoryMenuRepositoryTest(MemoryMenuRepository memoryMenuRepository) {
		this.memoryMenuRepository = memoryMenuRepository;
	}
	...
}
```
테스트를 통과하는 것을 볼 수 있다. 그런데 이게 정말 된 건지 의심스러워서 <span style="color:orangered">**Triangulation**</span>[1]을
사용해 보기로 했다. 간단히 말하면, Assertion을 하나 더 추가하고 코드를 일반화시키는 것이다.
```java
public void 메뉴등록하기() {
	// given
	Menu menu = new Menu();
	menu.setId(memoryMenuRepository.size() + 1);
	...

	Menu menu2 = new Menu();
	menu2.setId(memoryMenuRepository.size() + 1);
	menu2.setName("황금올리브");
	menu2.setCategory("후라이드");
	menu2.setPrice(22000);
	menu2.setStore("BBQ");
	menu2.setScore(95);

	// when
	memoryMenuRepository.save(menu);
	memoryMenuRepository.save(menu2);

	// then
	Menu result = memoryMenuRepository.get(1);
	assertThat(result.getId()).isEqualTo(1);
	...

	Menu result2 = memoryMenuRepository.get(2);
	assertThat(result2.getId()).isEqualTo(2);
	assertThat(result2.getName()).isEqualTo("황금올리브");
}
```
![230817-3](/assets/posts/230817-3.png)

아니나 다를까, 새로운 에러를 발견했다. 원인을 찾아보니 메뉴1과 메뉴2의 Id 값이 모두 1로 저장되어 있다.
잘 보면 메뉴의 Id를 설정할 때 (현재 리포지토리에 저장된 메뉴 수) + 1로 설정하였다. 그런데 save(menu)가 실행되기 전에
menu2의 Id 값을 정하니 동일하게 1이 되었다. 다시 말해 오퍼레이션 순서에 의존적으로 설계된 것이다.

Id(sequence) 값 설정을 repository에서 관리하게 하고, store를 Id 값으로 조회할 수 있도록 Map으로 변경하였다.
테스트 코드에서도 setId를 제거하였다.
```java
// main/java/repository/MemoryMenuRepository.java
public class MemoryMenuRepository {
    private static Map<Long, Menu> store = new LinkedHashMap<>();
    private static Long sequence = 0L;

    public void clear() {

    }
    public Long size() {
        return sequence;
    }

    public void save(Menu menu) {
        menu.setId(++sequence);
        store.put(sequence, menu);
    }

    public Menu get(Long id) {
        return store.get(id);
    }
}

// test/java/repository/MemoryMenuRepositoryTest.java
@Test
public void 메뉴등록하기() {
	// given
	Menu menu = new Menu();
	...

	Menu menu2 = new Menu();
	...

	// when
	memoryMenuRepository.save(menu);
	memoryMenuRepository.save(menu2);

	// then
	Menu result = memoryMenuRepository.get(1L);
	assertThat(result.getId()).isEqualTo(1);
	assertThat(result.getName()).isEqualTo("마법클");
	...

	Menu result2 = memoryMenuRepository.get(2L);
	assertThat(result2.getId()).isEqualTo(2);
	assertThat(result2.getName()).isEqualTo("황금올리브");
}
```
다시 테스트를 통과할 수 있다. 이제 리팩토링을 진행하면서 깔끔한 코드를 만들면 된다.
마침 Getter/Setter를 몇 줄씩 반복해서 적는 것이 귀찮아서 위 코드에서 생략된 상태인데, 아예 생성자로 처리해 버리는 것도 좋겠다.
그리고 테스트를 완성하고 보니, 처음에 구상했던 '리뷰 리스트 필드'를 사용하지 않게 되었다.

생각해 보면, 처음 구상한 대로 List\<Review\> review 필드로 리뷰를 관리할 경우, 리뷰 하나가 생길 때마다 menu 인스턴스를 수정해야 할 것이다.
의존성을 제거하는 측면에서도 아예 없애버리는 편이 나을 것 같다.
```java
// main/java/domain/Menu.java
public class Menu {
    private Long id;
    private String name;
    private String category;
    private Integer price;
    private String store;
    private Integer score;

    public Menu(String name, String category, Integer price, String store) {
        this.name = name;
        this.category = category;
        this.price = price;
        this.store = store;
    }

	// Getter, Setter 생략
}
```
테스트 부분도 생성자를 사용하도록 변경하였다.

|-----------------------|
|~~클래스 생성~~|
|~~메모리에 메뉴 등록하기~~|
|clear 구현하기|
|무작위 선택하기|

### clear()
이런 과정을 반복하면서 개발을 진행하면 된다.
```java
// main/java/repository/MemoryMenuRepository.java
public class MemoryMenuRepository {
    public void clear() {
        store.clear();
        sequence = 0L;
    }
}

// test/java/repository/MemoryMenuRepositoryTest.java
@Test
public void 치우기() {
	// given
	Menu menu1 = new Menu("마법클", "시즈닝", 20000, "BHC");

	// when
	memoryMenuRepository.save(menu1);
	memoryMenuRepository.clear();

	// then
	assertThat(memoryMenuRepository.size()).isEqualTo(0);
}
```

|-----------------------|
|~~클래스 생성~~|
|~~메모리에 메뉴 등록하기~~|
|~~clear 구현하기~~|
|무작위 선택하기|

## 서비스 구현하기
드디어 마지막 작업에 도착했다.
리포지토리에 저장된 메뉴 중에서 무작위 하나를 선택하여 반환하는 <span style="color:orangered">**테스트를 작성**</span>해야 한다.
```java
// test/java/service/MenuServiceTest.java
@SpringBootTest
public class MenuServiceTest {
    private final MemoryMenuRepository memoryMenuRepository;
    private final MenuService menuService;

	@Autowired
    public MenuServiceTest(MenuService menuService, MemoryMenuRepository memoryMenuRepository) {
        this.memoryMenuRepository = memoryMenuRepository;
        this.menuService = menuService;
    }

    @Test
    public void 무작위추천() {
        // given
        Menu menu1 = new Menu("마법클", "시즈닝", 20000, "BHC");
        Menu menu2 = new Menu("황금올리브", "후라이드", 22000, "BBQ");
        Menu menu3 = new Menu("블랙라벨치킨", "후라이드", 23000, "KFC");

        // when
        menuService.addMenu(menu1);
        menuService.addMenu(menu2);
        menuService.addMenu(menu3);
        int[] seen = {0, 0, 0, 0};
        for (int i = 0; i < 100; i++) {
            Menu menu = menuService.getRandom();
            seen[Math.toIntExact(menu.getId())]++;
        }

        // then
        assertThat(seen[1]).isGreaterThanOrEqualTo(20);
        assertThat(seen[2]).isGreaterThanOrEqualTo(20);
        assertThat(seen[3]).isGreaterThanOrEqualTo(20);
    }
}
```
3개의 메뉴를 리포지토리에 생성한 뒤, 100번 정도 무작위 추천을 실행해서 각 20번 정도만 나오면
테스트 성공으로 설정하였다. 의존성 주입 부분은 이미 했던 내용과 유사하므로 함께 작성하였다.

<span style="color:orangered">**테스트 실패를 확인**</span>하고,
![230817-4.png](/assets/posts/230817-4.png)

<span style="color:orangered">**통과하도록 코드를 작성**</span>한다. 의존성 주입을 위한
Config 파일 작성도 잊지 않아야 한다.

```java
// main/java/service/MenuService.test
public class MenuService {
    MemoryMenuRepository memoryMenuRepository;
    public MenuService(MemoryMenuRepository memoryMenuRepository) {
        this.memoryMenuRepository = memoryMenuRepository;
    }

    public void addMenu(Menu menu) {
        memoryMenuRepository.save(menu);
    }

    public Menu getRandom() {
        long index = new Random().nextLong(1, memoryMenuRepository.size() + 1);
        return memoryMenuRepository.get(index);
    }
}

// main/java/SpringConfig.java
@Configuration
public class SpringConfig {
	...
    @Bean
    public MenuService menuService() {
        return new MenuService(memoryMenuRepository());
    }
}
```
이상한 현상이 발생한다. 테스트를 통과하기도 했다가, 실패하기도 한다.
![230817-5](/assets/posts/230817-5.png)

```java
seen[Math.toIntExact(menu.getId())]++;
```

바로 이 라인이다.
분명히 테스트에서 3개의 메뉴만 삽입했는데, getId()의 결과가 4가 나왔다고 한다.

현재 전체 테스트를 실행하면 MenuServiceTest, MemoryMenuRepositoryTest의 테스트가 무작위 순서로 실행되는데,
메뉴등록하기()에서 2개 메뉴를 리포지토리에 등록한 것이 지워지지 않고 남은 것이다. 그다음 무작위추천()을 실행하면 오류가 발생한다.
반대로 실행될 경우에는 잘 작동하는 모습이다.

<div style="display:flex; flex-wrap:wrap; align-items:flex-start;">
<img src="/assets/posts/230817-6.png" width="300px" height="200px"/>
<img src="/assets/posts/230817-7.png" width="300px" height="200px"/>
</div>


테스트가 실행 순서에 영향을 받지 않고, 독립적으로 실행되도록 바꿔주면 될 것 같다.
다른 테스트를 추가하기 전까지는, 아래처럼 바꿔서 빠르게 해결할 수 있을 것이다.
```java
// test/java/service/MenuServiceTest.java
@BeforeEach
public void beforeEach() {
	memoryMenuRepository.clear();
}
```
이제 테스트를 여러 번 실행해도 모두 통과하는 모습을 볼 수 있고, 마지막 할 일도 완료할 수 있다.

|-----------------------|
|~~클래스 생성~~|
|~~메모리에 메뉴 등록하기~~|
|~~clear 구현하기~~|
|~~무작위 선택하기~~|

## 마치며
TDD 방법론을 공부하고 실제로 프로젝트에 사용하는 것은 처음인데, 확실히 장점이 보이는 것 같다.

store를 ArrayList -> HashMap으로 변경하는 부분을 비롯해, 코드 구현을 변경하더라도 미리 작성한 테스트를 통해
잘 변경되었는지 빠르게 확인할 수가 있었다. 어떤 메서드를 구현하고 싶은지 미리 테스트로 확실히 하는 점도,
개발 목표가 확실해지는 면에서 굉장히 좋았다.

프로젝트 전체 코드는 [Github](https://github.com/metishonora/todaychicken/tree/post)에 게시하였다.

### 어려웠던 점
이번에는 Spring 및 TDD를 활용하는 것이 처음이라 많은 글을 참고하였는데, 익숙해지는 것에 시간이
약간 걸린 부분은 있었다.

이번 포스트에서는 TDD 도서의 기초적인 흐름을 따라가긴 했지만, 아직 초반부 내용밖에 학습하지 못한
상황으로, 아직도 갈 길이 멀다.

어렵더라도 여러 번 반복하면서 Spring과 TDD를 익혀 봐야 하겠다.

## References
[1] [Kent Beck, Test-Driven Development By Example, 2014.](https://www.google.co.kr/books/edition/Test_driven_Development/CUlsAQAAQBAJ?hl=ko&gbpv=0)

[2] [망나니개발자, TDD로 멤버십 등록 API 구현 예제, 2021.](https://mangkyu.tistory.com/184)

[3] [nathan, JUnit 5 Test가 생성자 의존성 주입을 하는 방법, 2022.](https://velog.io/@nathan29849/JUnit-Test-%EA%B5%AC%EC%A1%B0)

[DI]: https://en.wikipedia.org/wiki/Dependency_injection
[linkToPrev]: /%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/2023/08/15/chicken-domain-modelling
[stub]: https://en.wikipedia.org/wiki/Test_stub