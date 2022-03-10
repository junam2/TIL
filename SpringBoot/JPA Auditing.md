JPA Auditing

* JPA Auditing 이란
  * JPA를 이용하여 Entity에 시간에 대해서 자동으로 넣어주는 기능
  * 여러 Entity에 생성 시간, 수정 시간 등이 들어가는데 이런 중복되는 상황을 막아준다.

* 적용

  * BaseTimeEntity 추가

    * ~~~java
      @Getter
      @MappedSuperclass // JPA Entity 들이 해당 클래스를 상속할 경우 필드들도 칼럼으로 인식된다.
      @EntityListeners(AuditingEntityListener.class) // Auditing 기능 포함
      public abstract class BaseTimeEntity {
      
          // Entity 가 생성되어 저장될 때 시간이 자동 저장된다.
          @CreatedDate
          private LocalDateTime createDate;
      
          // Entity 가 변경될 때 시간이 자동 저장된다.
          @LastModifiedDate
          private  LocalDateTime modifiedDate;
      }
      ~~~

  * JPA Aiditing 활성화

    * ~~~java
      @EnableJpaAuditing 
      @SpringBootApplication
      public class Application {
          public static void main(String[] args) {
              SpringApplication.run(Application.class, args);
          }
      }
      ~~~

      

