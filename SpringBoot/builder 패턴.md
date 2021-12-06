Builder 패턴

* 정의
  * 인스턴스를 생성 할 때 생성자만을 통해 만드는 것에 어려움으로 인해 고안된 패턴
  * 특히 생정자 인자가 많다면 builder 패턴이 좋다.



* 생성자 패턴 / Bean 패턴 비교

  * 생성자 패턴과 달리 인자의 순서를 신경 쓸 필요가 없다.

  * ~~~java
    // 생성자 패턴은 다음과 같이 인자의 순서를 지켜야한다.
    User user = new User(1, "이름", 25, "이메일");
    ~~~

  * 자바 Bean 패턴에서 setter을 통해 NoArgsConstructor로도 할 수 있지만 이 또한 계속 호출해야 해서 불편하다.

  * ~~~java
    public static void main(String[] args) {
       User user = new User();
       user.setUserId(1);
       user.setName("이름");
       user.setAge(25);
       user.setEmail("Builder@naver.com");
    }
    ~~~



* Lombok을 이용한 builder 패턴 적용

  * User 객체

    * ~~~java
      @Data
      @NoArgsConstructor
      // getter와 setter를 한 번에 만들어준다.
      Public class User {
      	private int id;
        private String name;
        private int age;
        private String email;
        
        //@builder 을 통해 빌더 패턴을 적용한다.
        //@NonNull 을 통해 필수 입력 정보를 설정한다.
        @Builder
        pulibc User(int id, @NonNull String name, @NonNull int age, String email) {
          this.id = id;
          this.name = name;
          this.age = age;
          this.email = email;
        }
      }
      ~~~

  * Controller

    * ~~~java
      public class testController {
      	@GetMapping("/test")
        public String lombokTest() {
          // builder()를 통해 모든 인자를 쓰지 않고도 가독성 있게 인스턴스를 만들 수 있다.
          User u1 = User.builder()
            					.username("ssar")
            					.age(25)
            					.email("admin@admin.com")
            					.build();
          ...
        }
      }
      ~~~

      



