SecurityConfig.java

* 기본적인 springboot security 를 사용하기 위해 설정하는 config 파일 정리

* ~~~java
  @RequiredArgsConstructor
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      private final CustomOAuth2UserService customOAuth2UserService;
  
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http
                  .csrf().disable()
                  .headers().frameOptions().disable()
                  .and()
                  .authorizeRequests()
                  .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                  .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                  .anyRequest().authenticated()
                  .and()
                  .logout().logoutSuccessUrl("/")
                  .and()
                  .oauth2Login()
                  .userInfoEndpoint()
                  .userService(customOAuth2UserService);
      }
  }
  ~~~

* @EnableWebSecurity

  * Spring Security 설정들을 활성화 시켜준다.

* csrf().disable().headers().frameOptions().disable()

  * H2-console 을 사용하기 위해 해당 옵션들을 disable 해준다.
  * csrf().disable()
    * Cross site Request forgery로 사이즈간 위조 요청인데, 즉 정상적인 사용자가 의도치 않은 위조요청을 보내는 것을 의미한다.
    * 해당 기능을 꺼준다. (rest api로 안전한 api만 일단 쓸거니까)
  * headers().frameOptions().disable()
    * Iframe 을 띄우는 방식을 spring security에서 막는데 해당 부분을 disable 해준다.
    * X-Frame-Options click jacking 공격 방지를 위한 기본 설정

* authorizeRequests()

  * URL 별 권한 관리를 설정하는 옵션의 시작점
  * 해당 옵션이 선언되어야 antMatchers 옵션을 사용할 수 있다.

* antMathcer()

  * 권한 관리 대상을 지정하는 옵션
  * URL, HTTP 메소드별 관리 가능
  * permitAll() 을 통해 일반적인 url은 전체 열람 권한 가능
  * 로그인이 필요한 api들을 hasRole() 을 통해 권한 가진 사람만 이용 가능

* anyRequest()

  * 설정된 값들 이외의 나머지 URL

* authenticated()

  * 인증된 사용자에게만 허용 (로그인한 사용자)

* logout().logoutSucessUrl("/")

  * 로그아웃과 로그아웃 성공시 이동 URL 설정

* oauth2Login

  * Oauth2 진입 시작점

* userInfoEndPoint

  * Oauth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정 담당

* userService

  * 소셜 로그인 성공 후 후속 조치를 진행할 서비스 인터페이스 등록
  * 추가로 진행하고자 하는 기능 명시 가능
