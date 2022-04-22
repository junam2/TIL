일반 로그인 / OAuth2 로그인 사용 시 Authentication 내 객체 함께 사용

* 스프링 시큐리티 세션 안에 들어갈 수 있는 객체는 Authentication 객체 밖에 없다.

  * Authentication

    * UserDetails
      * 일반적인 로그인 시 UserDetails 객체 사용
    * OAuth2User
      * Oauth2 로그인 시 해당 객체 사용

  * 일반 로그인 / Oauth2 로그인에 따른 각기 다른 객체를 사용해야 하는 불편함 발생

    * @AuthenticationPrincipal OAuth2User oauth
    * @AuthenticationPrincipal UserDetail userDetail

    

* 새로운 객체를 만들어서 UserDetail 와 Oauth2User 를 implements 하는 새로운 객체 생성

  * PrincipalDetails 에 UserDetails 와 OAuth2User 모두 상속시킨다.

    * 

    ```java
    public class PrincipalDetails implements UserDetails, OAuth2User {
    		
        // User 객체와 OAuth2 로그인 시 정보를 가져오는 attributes Map 을 선언.
        private User user;
        private Map<String, Object> attributes;
    
        // 일반 로그인
        public PrincipalDetails(User user) {
            this.user = user;
        }
    
        // OAuth 로그인
        public PrincipalDetails(User user, Map<String, Object> attributes) {
            this.user = user;
            this.attributes = attributes;
        }
    
        // UserDetails Override
        // 유저의 권한 return
        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            Collection<GrantedAuthority> collect = new ArrayList<>();
            collect.add(new GrantedAuthority() {
                @Override
                public String getAuthority() {
                    return user.getRole();
                }
            });
    
            return collect;
        }
    
        @Override
        public String getPassword() {
            return user.getPassword();
        }
    
        @Override
        public String getUsername() {
            return user.getUsername();
        }
    
        @Override
        public boolean isAccountNonExpired() {
            return true;
        }
    
        @Override
        public boolean isAccountNonLocked() {
            return false;
        }
    
        @Override
        public boolean isCredentialsNonExpired() {
            return true;
        }
    
        @Override
        public boolean isEnabled() {
            return true;
        }
    
        // OAuth2 Override
    
        @Override
        public String getName() {
            return null;
        }
    
        @Override
        public Map<String, Object> getAttributes() {
            return attributes;
        }
    }
    ```

  * 이후 구글로 받은 userRequest 데이터에 대한 후처리를 해주는 함수인 loadUser에서 PrincipalDetails 를 return 해줄 수 있다.

    * ~~~java
      @Service
      public class PrincipalOauth2UserService extends DefaultOAuth2UserService {
      
          @Autowired
          private BCryptPasswordEncoder bCryptPasswordEncoder;
      
          @Autowired
          private UserRepository userRepository;
      
          // 구글로 부터 받은 userRequest 데이터에 대한 후처리되는 함수
          // 함수 종료 시 @AuthenticationPrincipal 어노테이션이 만들어진다.
          @Override
          public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
              OAuth2User oauth2User = super.loadUser(userRequest);
      
              String provider = userRequest.getClientRegistration().getClientId();
              String providerId = oauth2User.getAttribute("sub");
              String username = provider + "_" + providerId;
              String password = bCryptPasswordEncoder.encode("getInThere");
              String email = oauth2User.getAttribute("email");
              String role = "ROLE_USER";
      
              User userEntity = userRepository.findByUsername(username);
      
              if(userEntity == null) {
                  // 회원 가입 진행
                  userEntity = User.builder()
                          .username(username)
                          .password(password)
                          .email(email)
                          .role(role)
                          .provider(provider)
                          .providerId(providerId)
                          .build();
      
                  userRepository.save(userEntity);
              }
      
              return new PrincipalDetails(userEntity, oauth2User.getAttributes());
          }
      }
      ~~~

  * 이렇게 되면 이제 일반 로그인 / 구글 로그인 시 모두 PrincipalDetails 객체로 세션 정보를 확인할 수 있다.

    * ~~~java
      @GetMapping("/test/login")
          public @ResponseBody String loginTest(@AuthenticationPrincipal PrincipalDetails principalDetails) {
              System.out.println("/test/login =============");
              System.out.println("authentication: " + principalDetails.getAttributes());
      
              return "세션 정보 확인하기";
          }
      ~~~

  * @AuthenticationPrincipal

    * 해당 어노테이션은 DefaultOAuth2UserService 의 loadUser() , UserDetailsService 의 loadUserByUsername() 함수 종료 시 해당 어노테이션이 생성된다.