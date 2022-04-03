Stream

* Stream 이란
  
  * 자바 8부터 등장한 람다을 이용한 기술
  
  * 데이터에 익명 함수를 조합하여 원하는 결과를 필터링 하고 가공할 수 있다.
  
  * 병렬 처리가 가능하다.

* 과정
  
  * 생성하기
  
    * 배열에서 스트림 사용
      
      * ```java
        String[] arr = new String[]{"a", "b", "c"};
        Stream<String> stream = Arrays.stream(arr);
        Stream<String> streamOfArrayPart = 
          Arrays.stream(arr, 1, 3); // 1~2 요소 [b, c]
        ```
  
    * 컬렉션에서 스트림 사용
      
      * ```java
        List<String> list = List.of("mad", "play");
        Stream<String> stream = list.stream();
        ```
  
    * 기본 타입 특화 스트림 생성
      
      * ```java
        // 0, 1, 2
        IntStream intStream = IntStream.range(0, 3);
        
        // 0, 1, 2, 3
        IntStream closedIntStream = IntStream.rangeClosed(0, 3);
        
        // 0, 1, 2
        LongStream longStream = LongStream.range(0, 3);
        
        // 0.0, 0.3
        DoubleStream doubleStream = DoubleStream.of(0, 3);
        ```
  
    * Stream.iterate() 생성
      
      * ```java
        // 0, 1, 2
        Stream<Integer> stream = Stream.iterate(0, x -> x + 1).limit(3);
        ```
  
    * Stream.concat() 생성
      
      * ```
        List<String> list1 = List.of("mad", "play");
        List<String> list2 = List.of("mad", "life");
        Stream<String> stream = Stream.concat(list1.stream(), list2.stream());
        // mad, play, mad, life
        ```
  
  * 가공하기
  
    * 스트림을 필터링 하거나 원하는 형태에 맞게 가공하는 연산
  
    * 반환 값으로 다시 스트림을 반환하기 때문에 이어서 가공하려면 메서드 체이닝이 필요하다.
  
    * filter
      
      * 스트림 내 요소들을 조건에 맞게 필터링 할 수 있다.
      
      * ```java
        List<String> list = List.of("kim", "tng");
        list.stream().filter(s -> s.length() == 5);
        // "taeng"
        ```
  
    * map
      
      * 스트림 내 요소를 원하는 형태로 반환
      
      * ```java
        List<String> list = List.of("mad", "play");
        list.stream().map(s -> s.toLowerCase());
        // "MAD", "PLAY"
        ```
  
    * 기본 타입에 특화된 스트림 변환 (mapToInt, mapToDouble)
      
      * 박싱 비용을 피하기 위해 기본 데이터 타입에 특화된 스트림으로 변환 가능
      
      * ```java
        List<String> list = List.of("0", "1");
        IntStream intStream = list.stream()
                .mapToInt(value -> Integer.parseInt(value));
        intStream.forEach(System.out::println);
        ```
      
      * sum을 구할 때 reduce 등을 사용하는 것보다 훨씬 빠른 속도를 보여줄 수 있다. 
        reduce는 기본 타입 <-> wrapper 클래스를 왔다 갔다 하는 비용이 추가적으로 발생하기 때문.
        
        * ```java
              Map<String, Integer> hashmap = new HashMap<>();
          
              for (int i = 0; i < 10000000; i++) {
                  hashmap.put(i + "", i);
              }
          
              // for-loop 시간측정
              long start1 = System.currentTimeMillis();
              int sum1 = 0;
              for (String key : hashmap.keySet()) {
                  sum1 += hashmap.get(key);
              }
              long end1 = System.currentTimeMillis();
          
              // stream mapToInt 시간측정
              long start2 = System.currentTimeMillis();
              int sum2 = hashmap.values().stream().reduce(0, Integer::sum);
              long end2 = System.currentTimeMillis();
          
              // stream reduce 시간측정
              long start3 = System.currentTimeMillis();
              int sum3 = hashmap.values().stream().mapToInt(i -> i).sum();
              long end3 = System.currentTimeMillis();
          
              System.out.println("for-loop: " + (end1 - start1));
              System.out.println("int reduce: " + (end2 - start2));
              System.out.println("mapToInt: " + (end3 - start3));
              
              
              /*
              ================== RESULT ==============
              for-loop: 303
              reduce: 368
              mapToInt: 310
              */
          ```
  
    * flatmap
      
      * 중첩된 구조를 한 단계 없애고 단일 원소 스트림으로 만들어준다.
      
      * ```java
        List<String> list1 = List.of("mad", "play");
        List<String> list2 = List.of("kim", "taeng");
        List<List<String>> combinedList = List.of(list1, list2);
        
        List<String> streamByList = combinedList.stream()
                .flatMap(list -> list.stream())
                .collect(Collectors.toList());
        
        // mad, play, kim, taeng
        System.out.println(streamByList);
        
        // 2차원 배열
        String[][] arrs = new String[][]{
                {"mad", "play"}, {"kim", "taeng"}
        };
        
        List<String> streamByArr = Arrays.stream(arrs)
                .flatMap(arr -> Arrays.stream(arr))
                .collect(Collectors.toList());
                
        // mad, play, kim, taeng
        System.out.println(streamByArr);
        ```
  
    * distinct
      
      * 스트림 내 요소 중복 제거
      
      * ```java
        IntStream stream = Arrays.stream(
                    new int[]{1, 2, 2, 3, 3});
        
            // 1, 2, 3
            stream.distinct()
                    .forEach(System.out::println);
        ```
  
    * sorted 
      
      * 스트림 내 요소 정렬
      
      * 람다식을 이용하는 것보단 Comparator 을 사용하는 것이 권장됨
      
      * ```java
        List<Human> sortedHumans = humans.stream()
                    .sorted(Comparator.comparingInt(Human::getMoney).reversed())
                    .collect(Collectors.toList());
        
        // 역순
        List<Human> sortedHumans = humans.stream()
                    .sorted(Comparator.comparingInt(Human::getMoney).reversed())
                    .collect(Collectors.toList());
        
        ```
  
    * limit
      
      * 스트림 내 특정 요소 개수를 제한
      
      * ```java
        List<String> list = List.of("a", "b", "c").stream()
                .limit(2).collect(Collectors.toList());
        
        // a, b
        System.out.println(list);
        ```
  
    * skip
      
      * 스트림 내 첫 번째 요소부터 전달된 수 만큼 제외
      
      * ```java
        List<String> list = Arrays.stream(new String[]{"a", "b", "c"})
                .skip(2).collect(Collectors.toList());
        
        // c
        System.out.println(list);
        ```
  
    * boxed
      
      * 기본 특화 스트림을 일반 스트림으로 변환 가능
      
      * ```java
        IntStream intStream = IntStream.range(0, 3);
        
        // 객체 타입의 일반 스트림으로 변환
        Stream<Integer> boxedStream = intStream.boxed();
        ```
  
  * 결과 연산
  
    * 순회 (forEach)
  
      * ```java
        List<Integer> list = List.of(3, 2, 1, 5, 7);
        list.stream().forEach(System.out::println);
        ```
  
      * 병렬 스트림 사용 시에는 순서 보장을 위해 forEachOrdered() 를 사용해야 한다.
  
    * 계산하기
  
      * min() / max() / count() / sum() / average() 등
  
    * 결과 모으기 (collect)
      * 스트림을 List, Set, Map 과 같은 형태로 변환시켜준다.// 리스트로 변환
      * ```java
        List<String> nameList = list.stream()
                .map(Food::getName) // name 얻기
                .collect(Collectors.toList());
        
        // 하나의 문자열로 합치기
        // joining 파라미터로 구분자 / prefix / suffix 사용 가능
        String defaultJoining = list.stream()
                .map(Food::getName).collect(Collectors.joining());
        
        // burgerchipscokesoda
        System.out.println(defaultJoining);
        
        // 특정 조건으로 그룹핑
        Map<Integer, List<Food>> calMap = list.stream()
                .collect(Collectors.groupingBy(Food::getCal));
        
        // { 230=[name: chips, cal: 230],
        //   520=[name: burger, cal: 520],
        //   143=[name: coke, cal: 143, name: soda, cal: 143]}
        System.out.println(calMap);
        
        // 참, 거짓으로 그룹핑
        Map<Boolean, List<Food>> partitionMap = list.stream()
                .collect(Collectors.partitioningBy(o -> o.getCal() > 200));
        
        // { false=[name: coke, cal: 143, name: soda, cal: 143],
        //   true=[name: burger, cal: 520, name: chips, cal: 230]}
        System.out.println(partitionMap);
        
        // Map으로 결과 받기
        // 동일한 키가 있는 경우 새 값으로 대체한다.
        Map<Integer, String> map = list.stream()
                .collect(Collectors.toMap(
                        o -> o.getCal(),
                        o -> o.getName(),
                        (oldValue, newValue) -> newValue));
        
        // {230=chips, 520=burger, 143=soda}
        System.out.println(map);
        ```
  
  * 조건 체크
  
    * anyMatch , allMatch, noneMatch
  
    * ```java
      // 300 칼로리가 넘는 것이 하나라도 있는가?
      boolean anyMatch = list.stream()
              .anyMatch(food -> food.getCal() > 300);
              
      // 모두 100 칼로리가 넘는가?
      boolean allMatch = list.stream()
              .allMatch(food -> food.getCal() > 100);
      
      // 모두 1000 칼로리가 넘지 않는가?
      boolean noneMatch = list.stream()
              .noneMatch(food -> food.getCal() < 1000);
      ```
  
      
