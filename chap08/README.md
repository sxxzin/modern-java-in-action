# Chapter 8 - 컬렉션 API 개선



### 8.1 컬렉션 팩토리

**8.1.1 리스트 팩토리(List.of)**

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
//[Raphael,Olivia,Thibaut]
friends.add("Chih-Chun");
//UnsupportedOperationException 발생
```

List.of로 생성 후 add 시 UnsupportedOperationException 발생

set()메서드로 아이템을 바꾸려해도 예외 발생

이는 컬렉션이 의도치 않게 변하는 것을 막을 수 있기 떄문 

<aside>
💡 오버로딩 vs 가변인수
List.of에는 다양한 오버로드 버전이 있음 
static <E> List<E> of(E e1, E e2, E e3) 
static <E> List<E> of(E e1, E e2, E e3, E e4)

static <E> List<E> of(E… elements) 라고 하지 않은 이유 
(… : elements가 배열이거나 배열이 아닌 단일변수여도 된다는 의미)
: 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싸기 때문에 배열을 할당하고 초기화 하며 나중에 가비지 컬렉션을 하는 비용을 지불해야함. 고정된 숫자 요소 (최대 열개)를 API로 정의하므로 이런 비요을 제거 할 수 있다. 열개 이상인경우에는 가변인수를 이용하는 메소드 사용

</aside>

**데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 사용하기 간편한 팩토리 메서드를 이용**할 것을 권장. 팩토리 메서드 구현이 더 단순하고 목적을 달성하기 충분하기 때문 

**8.1.2 집합 팩토리(Set.of)**

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
//[Raphael,Olivia,Thibaut]
```

중복된 요소를 제공해 집합을 만들려고하면 Olivia라는 요소가 중복되어 있다는 설명과 함께 IllegalArgumentException 발생. 

**8.1.3 맵 팩토리(Map.of)**

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
System.out.println(ageOfFriends);
//{Olivia=25,Raphael=30,Thibaut=26}
```

열 개 이하의 키와 값 쌍을 가진 작은 맵을 생성할때는 Map.of

그 이상에서는 Map.ofEntries 메서드가 좋다. 이 메서드는 키와 값을 감쌀 추가 객체 할당을 필요로 한다.

```java
Map<String, Integer> ageOfFriends2 = Map.ofEntries(
        entry("Raphael", 30),
        entry("Olivia", 25),
        entry("Thibaut", 26));
System.out.println(ageOfFriends2);
//{Olivia=25,Raphael=30,Thibaut=26}
```

### 8.2 리스트와 집합 처리

- removeIf: 프레디케이트를 만족하는 요소 제거. List, Set에서 사용
- replaceAll: UnaryOperator함수를 이용해 요소 변경. List에서 사용
- sort: 리스트 정렬. List에서 사용

**8.2.1 removeIf 메서드**

```java
for(Iterator<Transaction> iterator=transactions.iterator();iterator.hasNext();){
            Transaction transaction = iterator.next();
            if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
                    iterator.remove();
            }
}
//복잡한 코드

transactios.removeIf(transaction->
            Character.isDigit(transaction.getReferenceCode().charAt(0)));
//removeIf로 간단하게 변경 
```

**8.2.2 replaceAll 메서드**

```java
for (ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext(); ) {
      String code = iterator.next();
      iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
//복잡한 코드 

referenceCodes.replaceAll(code -> 
            Character.toUpperCase(code.charAt(0)) + code.substring(1));
//replaceAll로 간단하게 변경
```

### 8.3 맵 처리

**8.3.1 forEach 메서드**

```java
for (Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
      String friend = entry.getKey();
      Integer age = entry.getValue();
      System.out.println(friend + " is " + age + " years old");
}

ageOfFriends.forEach((friend, age) -> 
            System.out.println(friend + " is " + age + " years old"));
```

**8.3.2 정렬 메서드** 

```java
Map<String, String> favouriteMovies = Map.ofEntries(
        entry("Raphael", "Star Wars"),
        entry("Cristina", "Matrix"),
        entry("Olivia", "James Bond"));
//map 생성 
favouriteMovies.entrySet().stream()
        .sorted(Entry.comparingByKey())
        .forEachOrdered(System.out::println);
//이름을 알파벳 순으로 처리 
//Cristina=Matrix
//Olivia=James Bond
//Raphael=Star Wars
```

**8.3.3 getOrDefault 메서드** 

첫번째 인수로 키를, 두번째 인수로 디폴트값을 받으며 맵에 키가 존재하지 않으면 두번째 인수로 반환.

```java
System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
//James Bond
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
//Matrix
```

**8.3.4 계산 패턴** 

- computeIfAbsent: 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가
- computeIfPresent: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
- compute: 제공된 키로 새 값을 계산하고 맵에 저장

```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

lines.forEach(line ->
        dataToHash.computeIfAbsent(line,//line은 맵에서 찾은 키
                this::calculateDigest));//키가 존재하지 않으면 동작 실행 

private byte[] calculateDigest(String key) {//헬퍼가 제공된 키의 해시를 계산 
    return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```

```java
Map<String, List<String>> friendsToMovies = new HashMap<>();

String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if (movies == null) {//리스트가 초기화 되었다면
       movies = new ArrayList<>();
       friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");//영화추가
System.out.println(friendsToMovies);
//복잡한 코드

friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
        .add("Star Wars");
//computeIfAbsent를 이용해 구현 
```

**8.3.5 삭제 패턴** 

```java
favouriteMovies.remove(key, value);
```

**8.3.6 교체 패턴**

```java
Map<String, String> favouriteMovies = new HashMap<>();
//replaceAll은 바꿀 수 있는 맵에 적용해야 함
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

**8.3.7 합침**

```java
Map<String, String> family = Map.ofEntries(
        entry("Teo", "Star Wars"),
        entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```

중복된 키가 없다면 잘 동작하는 코드 

```java
Map<String, String> everyone2 = new HashMap<>(family);
friends.forEach((k, v) -> 
            everyone2.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```

```java
moviesToCount.merge(movieName,1L,(key,count)->count+1L);
//영화를 몇회 시청했는지 기록하는 맵
```

### 8.4 개선된 ConcurrentHashMap

ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다 

내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다. 따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다. 표준 HashMap은 비동기 동작 

**8.4.1 리듀스와 검색**

- forEach: 각 (키,값) 쌍에 주어진 액션을 실행
- reduce: 모든 (키,값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search: 널이 아닌 값을 반환할때 까지 각 (키,값) 쌍에 함수를 적용

인수에 키, 값, (키,값), Map.Entry 객체를 받는 네가지 연산 형태를 지원

이들 연산은 상태를 잠그지 않고 수행하여 계산이 진행되는 동안 바뀔 수 잇는 객체, 값, 순서 등에 의존 하지 않아야한다. 

또한 이들 연산에 병렬성 기준값을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다. 

```java
ConcurrentHashMap<String, Long>map= new ConcurrentHashMap<>();
//여러 키와 값을 포함하도록 갱신될 ConcurrentHashMap
long parallelismThreshold=1;
Optional<Interger> maxValue=
        Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

**8.4.2 계수**

ConcurrentHashMap 클래스는 매핑 개수를 반환하는 mappingCount 메서드를 제공. 기존의 size 메서드 대신 int를 반환하는 mapingCount메서드를 사용하는 것이 좋다. 그래야 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있기 때문이다.

**8.4.3 집합뷰**

ConcurrentHashMap 클래스는 ConcurrentHashMap을 집합 뷰로 반환하는 KeySet이라는 새 메서드를 제공한다.