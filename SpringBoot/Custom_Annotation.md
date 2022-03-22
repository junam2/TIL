커스텀 어노테이션

* 커스텀 어노테이션 만드는 법

  * ~~~java
    @Target(ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyAnnotation {
      ...
    }
    
    ~~~

  * @Target(ElementType.PARAMETER)

    * 어노테이션이 생길 수 있는 위치를 지정해준다. 
    * PARAMETER -> 파라미터로 선언된 객체에서만 사용 가능

  * @Retention(RetentionPolicy.SOURCE/CLASS/RUNTIME)

    * RetentionPolicy.SOURCE
      * 어노테이션이 소스 코드까지만 유지 된다. (컴파일 과정에서 어노테이션 정보 사라짐)
      * 컴파일 과정에서 메소드 등으로 변경되기 때문에 굳이 컴파일 이후에도 남아있을 필요가 없다.
      * Ex) @Getter / @Setter 등
    * RetentionPolicy.CLASS
      * 클래스 파일 까지만 유지 (런타임 시 어노테이션 정보 사라진다.)
      * 타입체커 등 컴파일 후에도 확인할 수 있어야 IDE나 JAR 등에서도 확인이 가능하다.
      * Ex) @NonNull
    * RetentionPolicy.RUNTIME
      * 런타임 시점까지 유지 (Reflection API로 어노테이션 정보 조회 가능)
      * 스프링이 실행되는 시점에서 컴포넌트 스캔이 가능해야 한 어노테이션을 위치
      * 대부분의 어노테이션이 위치한다.
      * Ex) @Autowired / @Service / @Controller

