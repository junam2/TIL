JWT (Json Web Token)

* 정의

  * 인증에 필요한 정보들을 json 에 '서명된' 토큰 형식으로 담아 사용하는 표준 인증 방식 

* 구조

  * Header.Payload.Signature (XXXX.YYYY.ZZZZ) 의 형식으로 되어 있다.

  * Header

    * ```json
      {
      	"typ": "JWT",
      	"alg": "HS512"
      }
      ```

    * 토큰의 타입과, 서명 생성에 어떤 알고리즘이 사용되었는지를 저장

  * Payload

    * ```json
      {
      	"sub": "1", // 토큰 제목
      	"iss": "junam", // 토큰 발급자
      	"exp": 160000000, // 만료 시간
      	"iat": 160010000 // 발급 시간
      }
      ```

    * 표준으로는 3글자의 key를 사용하여 Claim 이라는 사용자에 대한 정보를 key-value 형태로 저장한다.

    * 추가적으로 원하는 커스텀 정보를 넣어도 된다. 다만 특별한 암호화가 걸려있는 것은 아니기 때문에 민감한 정보를 넣어서는 안된다.

  * Signature

    * ```
      HS512(
      	base64UriEncode(header)+ "." +
      	base63UriEncode(payload),
      	'server-private-key'
      )
      ```

    * header + payload 디코딩 값에 서버가 갖고 있는 개인키로 암호화 되어 있는 상태

* 인증 방법

  * 로그인 시, 서버가 생선한 JWT 토큰을 사용자에게 전달한다.
  * 사용자는 로그인 후 권한이 필요한 작업 요청 시, 서버로부터 발급받은 JWT 를 함께 전달한다.
  * 서버에서 개인키를 이용해 signature를 복호화 한 후 header와 payload가 각각 서버가 갖고 있는 정보와 일치하는지 확인하여 인증 여부를 확인한다.

* 사용 이유

  * 기존에는 세션과 쿠키를 사용하여 인증을 관리 했다.
  * 쿠키는 민감 정보까지 노출이 될 수 있으며, 웹 브라우저마다 쿠키가 다르기 때문에 브라우저 간 공유가 불가능하다.
  * 쿠키의 보안적인 단점을 해결하기 위해 나온 것이 세션이다. 인증 정보 자체를 세션 저장소에 저장하여 이후에는 id,pw를 넘기지 않고 세션 id를 넘겨 저장소에 있는 세션과 일치하는지 확인하여 인증을 했다.
  * 세션의 단점으로는 stateless를 위반한다. 세션 저장소에 세션 id를 저장하는 상황이기 때문에 stateful 한 상황이 된다.
  * 만약 서버가 여러 대라면 각각의 서버마다 세션 id가 필요할 것이고 이를 통합하기 위해 통합 세션 저장소가 추가적으로 필요하다. (추가 비용 발생 / 메모리 차지 / 매번 요청 시 마다 세션 DB 저장소 조회 필요)
  * 이러한 단점들을 극복하기 위해 JWT 가 나오게 되었다.

* 장점

  * 토큰 자체가 인증된 정보이기 때문에 별도의 저장소가 필요하지 않다.
  * Stateless 하여 확장성이 높다.
  * 개인키 암호화를 사용하여 보안성이 높다.

* 구현

  * SecurityConfig.java

    * ```java
      @Configuration
      @EnableWebSecurity
      @RequiredArgsConstructor
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
      
          private final JwtTokenProvider jwtTokenProvider;
      
          // AuthenticationManager를 Bean으로 등록합니다.
          @Bean
          @Override
          public AuthenticationManager authenticationManagerBean() throws Exception {
              return super.authenticationManagerBean();
          }
      
          @Override
          protected void configure(HttpSecurity http) throws Exception {
              http
                      .csrf().disable()
                      .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 토큰 기반 인증이므로 세션 없이 stateless 를 사용한다.
                      .and()
                      .authorizeRequests()
                      .antMatchers("/join", "/login").permitAll()
                      .antMatchers("/admin/**").hasRole("ADMIN")
                      .antMatchers("/user/**").hasRole("USER")
                      .anyRequest().authenticated()
                      .and()
                      .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                      UsernamePasswordAuthenticationFilter.class);
              // JwtAuthenticationFilter를 UsernamePasswordAuthenticationFilter 전에 넣는다
          }
      }
      ```

  * JwtTokenProvider.java

    * ```java
      @RequiredArgsConstructor
      @Component
      public class JwtTokenProvider {
      
          // 키
          private String secretKey = "secret";
      
          // 토큰 유효시간 | 30min
          private long tokenValidTime = 30 * 60 * 1000L;
      
          private final CustomUserDetailService customUserDetailService;
      
          // 의존성 주입 후, 초기화를 수행
          // 객체 초기화, secretKey를 Base64로 인코딩한다.
          @PostConstruct
          protected void init() {
              secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
          }
      
          // JWT Token 생성.
          public String createToken(String user, List<String> roles){
              Claims claims = Jwts.claims().setSubject(user); // claims 생성 및 payload 설정
              claims.put("roles", roles); // 권한 설정, key/ value 쌍으로 저장
      
              Date date = new Date();
              return Jwts.builder()
                      .setClaims(claims) // 발행 유저 정보 저장
                      .setIssuedAt(date) // 발행 시간 저장
                      .setExpiration(new Date(date.getTime() + tokenValidTime)) // 토큰 유효 시간 저장
                      .signWith(SignatureAlgorithm.HS256, secretKey) // 해싱 알고리즘 및 키 설정
                      .compact(); // 생성
          }
      
          // JWT 토큰에서 인증 정보 조회
          public Authentication getAuthentication(String token) {
              UserDetails userDetails = customUserDetailService.loadUserByUsername(this.getUserPK(token));
              return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
          }
      
          // 토큰에서 회원 정보 추출
          public String getUserPK(String token) {
              return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
          }
      
          public boolean validateToken(String token) {
              try {
                  Jwts.parser().setSigningKey(secret).parse(token);
                  return true;
              } catch (MalformedJwtException | SignatureException e) {
                  //log.error("잘못된 JWT 서명입니다.");
              } catch (UnsupportedJwtException e) {
                  //log.error("지원되지 않는 JWT 토큰입니다.");
              } catch (IllegalArgumentException e) {
                  //log.error("JWT 토큰이 잘못되었습니다.");
              }
              return false;
          }
      
          public String resolveToken(HttpServletRequest request) {
              String bearerToken = Arrays.stream(request.getCookies()).filter(cookie -> "Bearer".equals(cookie.getName())).findFirst().get().getValue();
              return bearerToken;
          }
      }
      ```

      