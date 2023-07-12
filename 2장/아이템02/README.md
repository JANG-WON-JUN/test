# 이펙티브 자바 정리

# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라.

## 정적 팩터리 메서드의 장점

### **생성을 해주는 메서드가 이름을 가질 수 있다.**

1. 생성자보다 직관적으로 사용할 수 있다.

### **호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**

1. FlyWeight 패턴에서 유용하다. (팩토리 클래스가 따로 있음)
2. 매번 같은 객체를 반환하지 않아 메모리를 아낄 수 있다.
3. 싱글톤과 다른점?
    1. 기본 아이디어는 같다.
    2. FlyWeight는 특정 클래스의 “속성”이 다르면 다른 클래스를 생성해서 준다.
    3. 싱글톤은 무조건 1개의 클래스를 반환한다.
    
    ```java
    public class TreeFactory {
    
    	public static final HashMap<String, Tree> treeMap = new HashMap<>();
    
    	public static Tree of(String treeColor) {
    		return treeMap.getOrDefault(treeColor, new Tree(treeColor));
    	}
    }
    ```
    

### **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

1. 자바 8부터 인터페이스에 static 메소드를 선언할 수 있게 되어서 보다 직관적이고, 다형성을 사용할 수 있게 되었다.

### **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**

1. 대표 예시는 EnumSet
2. 내부적으로 어떤 클래스를 반환하는지 클라에게 공개하지 않아도 된다.
3. 즉 상황에 따라 효율적으로 클래스를 선택하여 반환할 수 있다는 것이다.

```java
public class MyEnumSet {

	public static void main(String[] args) {
		// EnumSet은 enum값을 관리하기 위한 집합이다.
		EnumSet<Color> colors = EnumSet.allOf(Color.class);

		colors.forEach(System.out::println);
	}

	enum Color {
		RED, YELLOW, GREEN, BLUE, BLACK, WHITE
	}
}
```

### **정적 팩터리를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

### 서비스 제공자 프레임워크의 3 + 1개의 핵심 컴포넌트

1. **서비스 인터페이스** → 구현체의 동작을 정의한다.
2. **제공자 등록 API** → 제공자가 구현체를 등록할 때 사용한다. (쉽게 말해 등록한다는 것)
3. **서비스 접근 API** → 클라가 서비스의 인스턴스를 얻을 때 사용한다. (실제 인스턴스를 생성하는 역할)
4. **서비스 제공자 API** → 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다. (쉽게 말해 뭘 생성해야 될지 알려줌)

### JDBC를 예로 들면..

1. Connection → 서비스 인터페이스
2. DriverManager.registerDriver → 제공자 등록 API
3. DriverManager.getConnection → 서비스 접근 API
4. Driver → 서비스 제공자 인터페이스 (여기서는 mysql 드라이버를 의미)

```java
public static void main(String[] args) {
		String driverName = "com.mysql.jdbc.Driver";
		String url = "jdbc:mysql://localhost:3306/test";
		String user = "root";
		String password = "soyeon";

		try {
			// 드라이버 이름으로 된 클래스가 없으면 ClassNotFoundException 발생
			Class.forName(driverName);

			// 서비스 접근 API인 DriverManager.getConnection가 서비스 구현체(서비스 인터페이스)인 Connection 반환
			Connection connection = DriverManager.getConnection(url, user, password);

		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
```

### 간단히 정리하면..

1. 팩터리 메서드에 생성 로직을 작성한다.
2. 어떤 구현체를 생성할지 결정하는 건 외부에서 결정 후에 팩터리 메서드에 넘겨준다.
3. 이렇게 하면 내부 로직은 어떠한 구현체라도 받아 들일 수 있으며, 마치 DI 같은 느낌이다.

## 정적 팩터리 메서드의 단점

### 상속하려면 public 이나 protected 생성자가 필요하니까 정적 메서드만 있으면 하위 클래스를 만들 수 없다.

1. 오히려 불변이라는 장점을 가져갈 수도 있음

### 정적 팩터리 메서드는 프로그래머가 직접 찾으러 다녀야 한다.

1. 자바 규약 상 생성자는 무조건 있으니까 굳이 찾지 않아도 있다는 것을 알 수 있다.
2. 정적 팩터리 메서드가 있는지 없는지 직접 찾아야 한다.

## 정적 팩터리 메서드 명명규칙

1. from → 매개 변수 1개로 생성함
2. of → 매개 변수 여러 개로 생성함
3. valueOf → 매개 변수 1개인데 좀 더 자세하게 설명하는 이름을 지어줌
4. instance, getInstance → 매개 변수의 조건에 따라 인스턴스를 반환해준다.
    1. 새로운 것일수도, 아닐 수도 있음
5. create, newInstance → 매개 변수를 가지고 매번 새로운 인스턴스를 생성해준다.
6. getType → 팩터리 메서드를 가진 클래스가 아닌 다른 클래스를 만들어서 반환할 때 사용
    
    ```java
    FileStore fs = Files.getFileStore(path);
    ```
    
7. newType → 팩터리 메서드를 가진 클래스가 아닌 다른 클래스를 매번 새롭게 만들어서 반환할 때 사용
    
    ```java
    BufferedReader br = Files.newBufferedReader(path);
    ```
    
8. type → getType, newType의 간결한 버
    
    ```java
    List<String> list = Collections.list(..);
    ```
    

---

# 아이템 2. 생성자에 매개 변수가 많다면 빌더를 고려하라.

## 클래스를 생성할 때 매개 변수가 너무 많다!

### 생성자, 정적 팩터리를 사용하는 경우

### 아이디어

1. 점층적 생성자 패턴으로 해결할 수 있다.
    1. 점층적 생성자 패턴이란 간단히 말해 메서드 오버로딩을 사용하여 생성자를 많이 만들어두는 것이다.

### 단점

1. 가독성이 떨어진다.
2. 원하지 않는 매개변수도 지정해야 하는 경우가 있다.
3. 다른 사람이 전달하는 인수의 순서를 바꿔도 타입이 같으면 알아채기가 어렵다.

### 자바 빈즈 패턴을 사용하는 경우

### 아이디어

1. setter 메소드를 만들어서 각 필드에 값을 넣는

### 단점

1. 필요한 필드의 setter 메소드가 전부 필요하다.
2. 누군가 값을 임의로 바꿀 수 있다.
3. 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.
    1. 예를 들어 필수 값이 들어있어야 하는데 필수 값이 없는 시점이 생겨버린다.
4. 디버깅이 어렵다.

### 빌더 패턴을 사용하는 경우

### 아이디어

1. 클래스 안에 static Builder 클래스를 내부 클래스로 생성한다.
2. 내부 클래스에는 builder를 사용하여 필드에 값을 넣어준다.
3. build 메서드를 호출하면 클래스의 인스턴스를 생성한다. 

### 장점

1. 필요한 매개변수만 골라서 인수를 전달 해 줄 수 있다.
2. build 메소드를 호출하여 인스턴스를 생성하기 전까지는 일관성이 유지된다.
3. 어떤 매개변수에 어떤 인수를 넣는지 확인하기가 쉬워 가독성이 높아진다.
4. 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.

### 단점

1. 복잡한 빌더 클래스를 매번 작성해줘야 한다. → Lombok으로 해결가능!

## 빌더 패턴을 사용한 예제

### 클래스 구조

1. Pizza 클래스의 자식 클래스로 NyPizza와 Calzone 가 있다.

### Pizza

```java
public abstract class Pizza {
	public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

	final Set<Topping> toppings;

	Pizza(Builder<?> builder) {
		// toppings = builder.toppings.clone();
		// 책에서는 clone을 사용했지만 EnumSet.copyOf가 있으며
		// EnumSet.copyOf는 내부적으로 clone을 사용한다.

		// copyOf를 쓰는 이유는 Pizza 인스턴스 생성 후
		// builder의 토핑이 변경되면 Pizza의 토핑도 변경되는 걸 막기 위해서이다.
		toppings = EnumSet.copyOf(builder.toppings);
	}

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstract Pizza build();

		// 하위 클래스는 이 메서드를 재정의하여
		// "this"를 반환하도록 해야 한다.
		protected abstract T self();
	}
}
```

### NyPizza

```java
public class NyPizza extends Pizza {

	public enum Size {SMALL, MEDIUM, LARGE,}

	private final Size size;

	// 자식 클래스의 생성자는 private로 숨기고
	// 빌더 패턴만 이용할 수 있도록 제한한다.
	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}

	// builder 클래스를 new하여 만들 수 있게 한다.
	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}
}
```

### Calzone

```java
public class Calzone extends Pizza {
	private final boolean sauceInside;

	// 자식 클래스의 생성자는 private로 숨기고
	// 빌더 패턴만 이용할 수 있도록 제한한다.
	private Calzone(Builder builder) {
		super(builder);
		sauceInside = builder.sauceInside;
	}

	// builder 클래스를 new하여 만들 수 있게 한다.
	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInside = false;

		// 빌더를 사용하여 Calzone 클래스의 필드에 값을 넣어주는 메소드
		public Builder sauceInside() {
			sauceInside = true;
			return this;
		}

		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}
}
```

### 실행 코드

```java
public class PizzaMain {

    public static void main(String[] args) {
       // 1. 빌더 패턴으로 피자 생성하기
       NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.SMALL)
          .addTopping(Pizza.Topping.SAUSAGE)
          .addTopping(Pizza.Topping.ONION)
          .build();

       Calzone calzone = new Calzone.Builder()
          .addTopping(Pizza.Topping.HAM)
          .sauceInside()
          .build();

       // 2. EnumSet인 피자 토핑을 copyOf로 처리하는 이유
       NyPizza.Builder builder = new NyPizza.Builder(NyPizza.Size.SMALL)
          .addTopping(Pizza.Topping.SAUSAGE)
          .addTopping(Pizza.Topping.ONION);

       NyPizza myPizza = builder.build();

       // 피자 생성 후 빌더의 토핑을 변경하면 피자의 토핑도 바뀐다.
       builder.addTopping(Pizza.Topping.HAM);

       System.out.println(myPizza.toppings);
    }
}
```

## 추가 학습

## Lombok에서 빌더 사용하기

### @Builder

### 빌더 선언하기

```java
@Getter
public class Member {
	/*
		Lombok의 Builder 어노테이션의 속성들
		1. String builderMethodName() default "builder"
			-> 빌더에 접근하는 static 메소드의 이름을 변경할 수 있다.
		2. String buildMethodName() default "build"
			-> 빌더의 빌드 메소드의 이름을 변경할 수 있다.
		3. String builderClassName() default ""
			-> new Class.builderClassName() 으로 접근 가능한데 이를 커스텀 한다.
		4. boolean toBuilder() default false
			-> 이미 생성한 인스턴스를 가지고 toBuilder를 사용해서 또 다른 인스턴스를 만들게 할지 여부
		5. AccessLevel access() default lombok.AccessLevel.PUBLIC
			-> 빌더의 접근 레벨을 제한
			-> 컴파일 오류가 아닌 런타임 오류가 나버린다.
		6. String setterPrefix() default ""
			-> 빌더에서 파라미터 입력 시 setter 할 때 메소드에 접두어 붙이기
			-> ex) setterPrefix = "set" 일 때
				Member.builder().setName().setAge().build();
				이렇게 쓸 수 있다.
	 */

	private String name;
	private int age;
	private boolean isActive = true;

	@Builder(toBuilder = true)
	public Member(String name, int age, boolean isActive) {
		this.name = name;
		this.age = age;
		this.isActive = isActive;
	}

	@Builder(builderMethodName = "customBuilder")
	public Member(String name, int age) {
		this.name = name;
		this.age = age;
	}

	@Builder(builderClassName = "ByName", builderMethodName = "byName")
	public Member(String name) {
		this.name = name;
		this.age = age;
	}
}
```

### 테스트 코드

```java
class MemberTest {

	@Test
	@DisplayName("Lombok의 Builder를 사용하여 인스턴스 생성하기")
	void newInstanceWithBuilderTest() {
		Member member = Member.builder()
			.name("admin")
			.age(10)
			.isActive(true)
			.build();

		assertThat(member.getName()).isEqualTo("admin");
		assertThat(member.getAge()).isEqualTo(10);
		assertThat(member.isActive()).isTrue();
	}

	@Test
	@DisplayName("Lombok의 customBuilderMethod를 사용하여 인스턴스 생성하기")
	void newInstanceWithCustomBuilderMethodTest() {
		Member member = Member.customBuilder()
			.name("admin")
			.age(10)
			.build();

		assertThat(member.getName()).isEqualTo("admin");
		assertThat(member.getAge()).isEqualTo(10);
		assertThat(member.isActive()).isFalse();
	}

	@Test
	@DisplayName("Lombok의 customBuilderClassName를 사용하여 인스턴스 생성하기")
	void newInstanceWithCustomBuilderClassTest() {
		Member member = new Member.ByName()
			.name("admin")
			.build();

		assertThat(member.getName()).isEqualTo("admin");
	}

	@Test
	@DisplayName("Lombok의 toBuilder 사용하여 기존 인스턴스를 활용해 새로운 인스턴스 생성하기")
	void newInstanceWithToBuilderTest() {
		Member admin = new Member.ByName()
			.name("admin")
			.build();

		Member userFromAdmin = admin.toBuilder()
			.name("user")
			.build();

		assertThat(admin.getName()).isEqualTo("admin");
		assertThat(userFromAdmin.getName()).isEqualTo("user");
	}
}
```

### @Builder.Default, @Builder.ObtainVia

### 코드 선언하기

```java
@Getter
@Builder(toBuilder = true)
public class Car {
	/*
		@Builder.Default를 사용하기 위한 조건
			1. 클래스 레벨에 적용되어야 한다.
			2. 이것 때문에 오류가 발생한다면 런타임 시점에 확인된다.

		@Builder.ObtainVia(field = "name")
		빌더를 사용할 떄 특정 필드의 값을 다른 field나 메소드로 만들어서 값을 주입해준다.
			1. toBuilder = true 설정을 해 줘야 한다.
			2. 빌더 생성 시 toBuilder 메소드를 거쳐야 한다.
				Car car = Car.builder()
				.build()
				.toBuilder()
				.build();
	 */

	@Builder.Default
	private String brand = "benz";

	@Builder.Default
	private String color = "black";

	@Builder.Default
	private String name = "myCar";

	@Builder.ObtainVia(field = "name")
	private String obtainViaField;

	@Builder.ObtainVia(method = "generate")
	private String obtainViaMethod;

	private String generate() {
		return brand + "&" + color;
	}
}
```

### 테스트 코드

```java
class CarTest {

	@Test
	@DisplayName("@Builder.Default를 사용하여 기본값 설정하기")
	void builderDefaultTest() {
		Car car = Car.builder()
			.build();

		assertThat(car.getBrand()).isEqualTo("benz");
		assertThat(car.getColor()).isEqualTo("black");
		assertThat(car.getName()).isEqualTo("myCar");
	}

	@Test
	@DisplayName("@Builder.ObtainVia를 사용하여 값 가져오를 ")
	void builderObtainVia를Test() {
		Car car = Car.builder()
			.build()
			.toBuilder()
			.build();

		assertThat(car.getBrand()).isEqualTo("benz");
		assertThat(car.getColor()).isEqualTo("black");
		assertThat(car.getName()).isEqualTo("myCar");
		assertThat(car.getObtainViaField()).isEqualTo("myCar");
		assertThat(car.getObtainViaMethod()).isEqualTo("benz&black");
	}
}
```

[lombok (Lombok)](https://projectlombok.org/api/lombok/package-summary)

## Objects와 Assert 클래스

1. Objects는 자바에서 지원하는 유효성 체크 클래스이다.
2. Assert 클래스는 스프링에서 지원하는 유효성 체크 클래스이다.

### 사용하는 이유?

1. 유효하지 않은 데이터가 입력되었을 때 빠르게 예외처리할 수 있다.
2. 그러므로 어디서 에러가 났는지 디버깅하기가 쉽다.

## final과 불변

### final이란?

1. final은 값을 재할당 하는 것을 금지하는 것이다.
    
    ```java
    class MyClass {
    	// names 필드는 불변이 아니다!
    	// list 내부가 바뀔 수 있기 때문이다.
    	private final List<String> names;
    }
    ```
    

### 불변객체와 불변식

1. 불변객체란 한번 만들어지면 절대 값을 바꿀 수 없는 객체를 의미한다.
    1. 대표적인 예로 String 클래스가 있다.
2. 불변식이란 변경을 할 수는 있으나, 정해진 조건을 만족해야 한다는 것을 의미한다.
    1. 예를들어 리스트의 크기는 반드시 0 이상이어야 한다.

### 불변의 장점

[Immutable in Java](https://www.leadingagile.com/2018/03/immutable-in-java/)

### 자바에서 불변 컬렉션으로 만드는 법

1. List 예시
    
    ```java
    List<Integer> immutableList1 = List.of(1, 2, 3, 4);
    ```
    
    ```java
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    
    List<Integer> integers = Collections.unmodifiableList(list);
    ```
    
2. 이외에도 다양한 방법이 있으니 아래 링크를 참고하자.
    
    [immutable collection in java and Java immutable](https://javagoal.com/immutable-collection-in-java/)
    

## 빌더를 이용한 엔티티 수정

### 아이디어

1. 프론트에서 “수정한 데이터만” 보내주고 나머지는 null로 보내주는 경우를 해결하고 싶다.
2. 엔티티로부터 빌더를 만들어서 값을 수정한다.

### 프로세스

1. EditDTO로 클라이언트로부터 수정 할 데이터를 받아온다.
2. 엔티티를 조회 후 엔티티를 EditorBuilder로 만든다.
3. EditerBuilder에 수정한 값을 넣는다.
4. 엔티티에 Editor를 넘겨서 값을 갱신해준다.

### 장점

1. 프론트에서 “수정한 데이터만” 보내주는 경우를 처리할 수 있다.
2. 만약 그렇지 않으면 갱신되면 안되는 값이 null로 바뀌는 경우가 있을 수 있는데 그것을 해결할 수 있다.
3. 그리고 변경되면 안되는 데이터에 대한 null 체크 같은 유효성 체크를 EditorBuilder에 위임할 수 있다.

### 단점

1. 설계하기 복잡하다.
2. 변경해야 될 필드가 추가되면 EditDTO, Editor, EditorBuilder에 코드를 추가 작성해야 하므로 귀찮다.
3. 롬복의 @Builder를 쓰는 것이 아니므로 롬복의 이점이 사라진다.

### 코드

### Board (엔티티)

1. DB의 테이블과 연관되는 엔티티이다.
    1. 엔티티를 EditorBuilder로 변환하는 메소드 선언
    2. Editor를 받아 엔티티의 내용을 변경하는 메소드 선언
    
    ```java
    @Getter
    @Builder // 테스트를 위해 빌더 선언하였고 실제로는 없어도 됨
    public class Board {
    
    	private String title;
    	private String content;
    
    	public BoardEditor.BoardEditorBuilder toBuilder() {
    		return BoardEditor.builder()
    			.title(title)
    			.content(content);
    	}
    
    	public void edit(BoardEditor boardEditor) {
    		this.title = boardEditor.getTitle();
    		this.content = boardEditor.getContent();
    	}
    }
    ```
    

### BoardEdit (수정 DTO)

1. 프론트에서 수정할 값을 받는 DTO이다.
    
    ```java
    @Getter
    @Builder // 테스트를 위해 빌더 선언하였고 실제로는 없어도 됨
    public class BoardEdit {
    
    	private String title;
    	private String content;
    }
    ```
    

### BoardEditor (수정을 위한 DTO)

1. 엔티티의 값을 변경하기 위해 중간에서 역할을 하는 Editor이다.
2. BoardEditorBuilder를 선언하고 각 필드를 변경하는 메소드에 null 체크같은 유효성 체크를 넣어준다.
    
    ```java
    @Getter
    public class BoardEditor {
    
    	private String title;
    	private String content;
    
    	public BoardEditor(String title, String content) {
    		this.title = title;
    		this.content = content;
    	}
    
    	public static BoardEditorBuilder builder() {
    		return new BoardEditorBuilder();
    	}
    
    	public static class BoardEditorBuilder {
    		private String title;
    		private String content;
    
    		BoardEditorBuilder() {
    		}
    
    		public BoardEditorBuilder title(String title) {
    			if (title != null) {
    				this.title = title;
    			}
    			return this;
    		}
    
    		public BoardEditorBuilder content(String content) {
    			if (content != null) {
    				this.content = content;
    			}
    			return this;
    		}
    
    		public BoardEditor build() {
    			return new BoardEditor(this.title, this.content);
    		}
    
    		public String toString() {
    			return "BoardEditor.BoardEditorBuilder(title=" + this.title + ", content=" + this.content + ")";
    		}
    	}
    }
    ```
    

### 테스트

```java
class BoardTest {

	private Board board;
	private final String ORIGINAL_TITLE = "변경 전 제목";
	private final String ORIGINAL_CONTENT = "변경 전 제목";
	private final String EDITED_TITLE = "변경 후 제목";
	private final String EDITED_CONTENT = "변경 후 제목";

	@BeforeEach
	void init() {
		board = Board.builder()
			.title(ORIGINAL_TITLE)
			.content(ORIGINAL_CONTENT)
			.build();
	}

	@Test
	@DisplayName("제목, 글 내용을 모두 수정할 때")
	void editTitleAndContentTest() {
		// given
		BoardEdit boardEdit = BoardEdit.builder()
			.title(EDITED_TITLE)
			.content(EDITED_CONTENT)
			.build();

		BoardEditor.BoardEditorBuilder builder = board.toBuilder();

		// when
		BoardEditor boardEditor = builder.title(boardEdit.getTitle())
			.content(boardEdit.getContent())
			.build();

		board.edit(boardEditor);

		// then
		assertThat(board.getTitle()).isEqualTo(EDITED_TITLE);
		assertThat(board.getContent()).isEqualTo(EDITED_CONTENT);
	}

	@Test
	@DisplayName("제목만 수정할 때")
	void editTitleTest() {
		// given
		BoardEdit boardEdit = BoardEdit.builder()
			.title(null)
			.content(EDITED_CONTENT)
			.build();

		BoardEditor.BoardEditorBuilder builder = board.toBuilder();

		// when
		BoardEditor boardEditor = builder.title(boardEdit.getTitle())
			.content(boardEdit.getContent())
			.build();

		board.edit(boardEditor);

		// then
		assertThat(board.getTitle()).isEqualTo(ORIGINAL_TITLE);
		assertThat(board.getContent()).isEqualTo(EDITED_CONTENT);
	}

	@Test
	@DisplayName("글 내용만 수정할 때")
	void editContentTest() {
		// given
		BoardEdit boardEdit = BoardEdit.builder()
			.title(EDITED_CONTENT)
			.content(null)
			.build();

		BoardEditor.BoardEditorBuilder builder = board.toBuilder();

		// when
		BoardEditor boardEditor = builder.title(boardEdit.getTitle())
			.content(boardEdit.getContent())
			.build();

		board.edit(boardEditor);

		// then
		assertThat(board.getTitle()).isEqualTo(EDITED_TITLE);
		assertThat(board.getContent()).isEqualTo(ORIGINAL_CONTENT);
	}

	@Test
	@DisplayName("제목, 글 내용 수정하지 않을 때")
	void doseNotEditTest() {
		// given
		BoardEdit boardEdit = BoardEdit.builder()
			.title(null)
			.content(null)
			.build();

		BoardEditor.BoardEditorBuilder builder = board.toBuilder();

		// when
		BoardEditor boardEditor = builder.title(boardEdit.getTitle())
			.content(boardEdit.getContent())
			.build();

		board.edit(boardEditor);

		// then
		assertThat(board.getTitle()).isEqualTo(ORIGINAL_CONTENT);
		assertThat(board.getContent()).isEqualTo(ORIGINAL_CONTENT);
	}
}
```

## 생각할 거리

1. 스태틱으로 만들어두면 메모리는?
    1. 그만큼 차지하는가..??

---