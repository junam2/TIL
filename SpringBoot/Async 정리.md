Spring boot Async 정리

* node js와 같은 Async Await 같은 함수를 Springboot 에서도 사용할 수 있다.



#### @Async 를 이용한 비동기 사용

* AsyncConfig.java

  * ```java
    @Configuration
    @EnableAsync
    public class AsyncConfig extends AsyncConfigurerSupport {
    
        @Override
        public Executor getAsyncExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(2);
            executor.setMaxPoolSize(10);
            executor.setQueueCapacity(500);
            executor.setThreadNamePrefix("goneda-async-");
            executor.initialize();
    
            return executor;
        }
    }
    ```

  * @EnableAsync

    * spring의 메소드 비동기 기능 활성화

  * ThreadPoolTaskExecutor 설정

    * CorePoolSize

      * 기본 적으로 실행 대기 중인 Thread 개수

    * MaxPoolSize

      * 동시 시작하는 최대 Thread 개수

    * QueueCapacity

      * MaxPoolSize 이상 요청 시, Queue 에서 대기하는 최대 요청 개수

        

* Service 에서의 사용

  * ~~~java
    @Service
    public class AsyncService{  
        @Async
        public void onAsync() {
            try {
                // Async 동작
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
      
    }
    ~~~

  * private 메소드에는 @Async 가 적용되지 않으며, 반드시 public 메소드로 사용해야 한다.

  * 같은 클래스 내부의 메소드에서 해당 함수를 호출하면 비동기로 실행되지 않는다.

    

* 이후 Controller 에서의 사용은 동일하다.