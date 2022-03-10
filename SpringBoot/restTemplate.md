RestTemplate

* RestTemplate 이란
  * 스프링에서 제공하는 http 통신에 유용하게 사용되는 템플릿
  * http 서버와의 통신을 단순화하고 RESTful 원칙을 치킨다.

* 주요 사용 메소드

  * ![image](https://media.vlpt.us/images/soosungp33/post/d02efacc-56e6-4abc-a390-12ecbb565c6b/image.png)

  * [Method]For[Object/Entity] 차이

    * get/postForObject는 객체를 결과로 반환받는다.

    * ~~~java
      @Test
      public void test_getForObject() {
        // employee 객체 자체를 변환받는다.
         Employee employee = restTemplate.getForObject(BASE_URL + "/{id}", Employee.class, 25);
         log.info("employee: {}", employee);
      }
      ~~~

    * get/postForEntity는 ResponseEntity를 반환받아 HTTP 응답에 대한 추가 정보를 담고 있다. 

    * Get - 추가적으로 여러 값을 담은 params 을 넘겨줄 수 있다. (Map 방식으로)'

    * Post - 헤더에 객체를 넘겨줄 수 있다.

    * ~~~java
      ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);
      
      assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
      assertThat(responseEntity.getBody()).isGreaterThan(0L);
      
      List<Posts> allList = postsRepository.findAll();
      assertThat(allList.get(0).getTitle()).isEqualTo(title);
      assertThat(allList.get(0).getContent()).isEqualTo(content);    
      ~~~

  * exchange

    * Get/post/put/delete 등이 모두 처리 가능한 하나의 Exchange API

    * HttpEntity 를 넘겨줘야해서 DTO의 변환이 필요하다.

    * ~~~java
      exchange(String url, HttpMethod method, @Nullable HttpEntity<?> requestEntity,
               Class<T> responseType, Object... uriVariables)
        
      // example
      String url = "http://localhost:" + port + "/api/v1/posts/" + updateId;
      
      HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);
      
      ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.POST, requestEntity, Long.class);
      
      assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
      assertThat(responseEntity.getBody()).isGreaterThan(0L);
      ~~~

      
