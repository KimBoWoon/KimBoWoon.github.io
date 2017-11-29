---
title: "[Effective Java] 정적 팩토리 메소드"
date: 2017-09-12 16:30:00
categories:
- Effective Java
---

# 정적 팩토리 메소드
클래스를 통해 객체를 만드는 방법은 public으로 선언된 생성자를 사용하는 것이다. 또 다른 방법은 클래스의 public으로 선언된 정적 팩토리 메소드(Static Factory Method)를 사용하는 것이다. 이 메소드는 static로 선언되어 있어서 외부에서 바로 접근할 수 있다.

## 정적 팩토리 메소드의 장점
1. 생성자와는 달리 이름이 있다.
생정자의 전달되는 인자들은 어떤 객체가 생성되는지를 설명하지 못하지만, 정적 팩토리 메소드는 이름만 잘 짓는다면 어떤 객체가 만들어 지는지 설명할 수 있다.
2. 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요가 없다.
변경 불가능 클래스라면 이미 만들어둔 객체를 활용할 수도 있고, 만든 객체를 캐시 해놓고 재사용하여 같은 객체가 불필요하게 거듭 생성되는 일을 피할 수도 있다. 동일한 객체가 요청되는 일이 잦고, 특히 객체를 만드는 비용이 클 때 적용하면 성능을 크게 개선할 수 있다. 다음과 같은 코드가 정적 팩토리 메소드를 적용한 것이다.
	```java
	public static Boolean valueOf(boolean b) {
		    return b ? Boolean.TRUE : Boolean.FALSE;
		}
	```
정적 팩토리 메소드를 사용하면 같은 객체를 반복해서 반환할 수 있으므로 어떤 시점에 어떤 객체가 얼마나 존재할지를 정밀하게 제어할 수 있다. 이런 기능을 갖춘 클래스를 개체 통제 클래스(instance-controlled class)라고 부른다. 개체 수를 제어하면 싱글톤 패턴을 따를 수 있고, 객체 생성이 불가능한 클래스를 만들 수도 있다. 변경 불가능의 클래스의 경우 두 개의 같은 객체가 존재하지 못하도록 할 수도 있다.
3. 생성자와는 달리 반환값 자료형의 하위 자료형 객체를 반환할 수 있다는 것이다.
public으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다. 그러면 구현 세부사항을 감출 수 있으므로 아주 간결한 API가 가능하다. 인터페이스 기반 프레인워크(interface-based framework)구현에 적합한데, 이 프레임워크에서는 정적 팩토리 메소드의 반환값 자료형으로 이용된다. 인터페이스는 정적 메소드를 가질 수 없으므로, 관습상 반환값 자료형이 Type이라는 이름의 인터페이스인 정적 팩토리 메소드는 Type라는 이름의 객체 생성 불가능 클래스안에 둔다.
	```java
	// 서비스 인터페이스
	public interface ServiceInterface {
		int add(int x, int y);
		int mul(int x, int y);
		... // 서비스에 고유한 메소드들이 이 자리에 온다
	}
	```

	```java
	public class ServiceImplement implements ServiceInterface {
		@Override
		public int add(int x, int y) {
			return x + y;
		}

		@Override
		public int mul(int x, int y) {
			return x * y;
		}
	}
	```

	```java
	//서비스 제공자 인터페이스
	public interface ProviderInterface {
		ServiceInterface newService();
	}
	```

	```java
	public class ProviderImplement implements ProviderInterface {
		@Override
		public ServiceInterface newService() {
			return new ServiceImplement();
		}
	}
	```

	```java
	// 서비스 등록과 접근에 사용되는 객체 생성 불가능 클래스
	public class ServiceProvider {
		private ServiceProvider() {} // 인스턴스 생성 X
	
		private static final Map<String, ProviderInterface> providers =
			new ConcurrentHashMap<String, ProviderInterface>();
		public static final String DEFAULT_PROVIDER_NAME = "<def>";
	
		// 제공자 등록 API
		public static void registerDefaultProvider(ProviderInterface p) {
			registerProvider(DEFAULT_PROVIDER_NAME, p);
		}
	
		public static void registerProvider(String name, ProviderInterface p) {
			providers.put(name, p);
		}
	
		// 서비스 접근 API
		public static ServiceInterface newInstance() {
			return newInstance(DEFAULT_PROVIDER_NAME);
		}
	
		public static ServiceInterface newInstance(String name) {
			ProviderInterface p = providers.get(name);
			if (p == null) {
				throw new IllegalArgumentException(
					"No provider registered with name: " + name);
			}
			return p.newService();
		}
	}
	```

	```java
	public class ServiceMain {
		public static void main(String[] args) {
			ServiceProvider.registerDefaultProvider(new ProviderImplement());
			ServiceProvider.registerProvider("BoWoon", new ProviderImplement());
	
			System.out.println(ServiceProvider.newInstance().add(1, 4));
			System.out.println(ServiceProvider.newInstance().mul(2, 2));
			System.out.println(ServiceProvider.newInstance("BoWoon").add(5, 5));
		}
	}
	```

4. 형인자 자료형(parameterized type) 객체를 만들 때 편하다
	```java
	Map<String, List<String>> m = new HashMap<String, List<String>>();
	```
자료형 명세를 중복하면 형인자가 늘어남에 따라 길고 복잡한 코드가 만들어진다. 하지만 정적 팩토리 메소드를 사용하면 컴파일러가 형인자를 스스로 알아내도록 할 수 있다. 이런 기법을 자료형 유추(type inference)라고 부른다.
	```java
	public static <K, V> HashMap<K, V> newInstance() {
		return new HashMap<K, V>();
	}
	```
이런 메소드가 있으면 아까와 같은 코드를 간결하게 작성할 수 있다.
	```java
	Map <String, List<String>> m = HashMap.newInstance();
	```

## 정적 팩토리 메소드의 단점
1. public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다
2. 정적 팩토리 메소드가 다른 정적 메소드와 확연히 구분되지 않는다는 것
