# 0. Contents
1. 회원가입
2. 이메일 인증

<br>

# 1. 회원가입

```java

// AuthenticationConfig.java

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class AuthenticationConfig {

    private final UserService userService;

    @Value("${jwt.secret}")
    private String secretKey;

    private static final String[] PERMIT_URL_ARRAY = {
            /* swagger v2 */
            "/swagger-resources",
            "/swagger-resources/**",
            "/configuration/ui",
            "/configuration/security",
            "/swagger-ui.html",
            "/webjars/**",
            /* swagger v3 */
            "/v3/api-docs/**",
            "/swagger-ui/**",
            /* User */
            "/api/v1/user/signup",
            "/api/v1/user/login",
            "/api/v1/user/login/social",
            "/api/v1/user/refresh",
            "/api/v1/token",
            "/api/v1/user/email-check",
            "/api/v1/user/nickname/duplicate",
            "/api/v1/user/email/duplicate",
            "/api/v1/user/email/auth/check",
            "/api/v1/user/email/issue/password",
            "/api/v1/recipe/getData2"

    };

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity
                .httpBasic().disable()
                .csrf().disable()
                .cors().and()
                .authorizeRequests()
                .antMatchers(PERMIT_URL_ARRAY).permitAll() // join, login, swagger 은 항상 허용
                .anyRequest().authenticated()
//              .antMatchers(HttpMethod.POST).authenticated()
//              .antMatchers(HttpMethod.GET).authenticated()
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .addFilterBefore(new JwtFilter(userService, secretKey), UsernamePasswordAuthenticationFilter.class)
                .build();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// JwtFilter.java

@RequiredArgsConstructor
@Slf4j
public class JwtFilter extends OncePerRequestFilter {

    private final UserService userService;
    private final String secretKey;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        final String authorization = request.getHeader(HttpHeaders.AUTHORIZATION);

        // 토큰 안보내면 block
        if (authorization == null || !authorization.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        // 토큰 꺼내기
        String token = authorization.split(" ")[1];

        try {
            //토큰이 만료되었는지 여부
            if (JwtUtil.isExpired(token, secretKey)) {
                log.error("token이 만료되었습니다.");
                filterChain.doFilter(request, response);
                return;
            }

            Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);

            // 토큰에서 유저 정보 꺼내기
            String userName = JwtUtil.getUserName(token, secretKey);// 일단 world로 가정

            //권한 부여
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(userName, null, List.of(new SimpleGrantedAuthority("USER")));
            //Detail을 넣어줍니다.
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
            filterChain.doFilter(request, response);

        } catch (ExpiredJwtException e) {
            log.error("토큰이 만료되었습니다.");
            response.setContentType("application/json;charset=UTF-8");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

            JSONObject responseJson = new JSONObject();
            responseJson.put("message", e.getMessage());
            responseJson.put("code", 401);

            response.getWriter().print(responseJson);

        } catch (UnsupportedJwtException e) {
            log.info("지원하지 않는 형식 서명입니다.");
            response.setStatus(403);
        } catch (MalformedJwtException e) {
            log.info("유효하지 않은 구성의 JWT 토큰입니다.");
            response.setStatus(403);
        } catch (SignatureException e) {
            log.info("잘못된 JWT 서명입니다.");
            response.setStatus(403);
        }catch(IllegalArgumentException e){
            log.info("JWT 토큰이 잘못되었습니다.");
            response.setStatus(403);
        }
    }

}

// JwtUtil.java

public class JwtUtil {

    public static String getUserName(String token, String secretKey) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token)
                .getBody().get("UserName",String.class);
    }

    public static boolean isExpired(String token, String secretKey){
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token)
                .getBody().getExpiration().before(new Date());
    }
    public static String createJwt(String userName, String secretKey, Long expiredMs){
        Claims claims = Jwts.claims();
        claims.put("UserName", userName);

        return Jwts.builder()
                .setHeaderParam("type","jwt")
                .setClaims(claims)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis()+expiredMs))
                .signWith(SignatureAlgorithm.HS256,secretKey)
                .compact();
    }
    public static String createRefreshToken( Long expiredMs,String secretKey){

        return Jwts.builder()
                .setHeaderParam("type","Refresh")
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis()+expiredMs))
                .signWith(SignatureAlgorithm.HS256,secretKey)
                .compact();
    }

}

// user/service/UserService.java

// 이메일 닉네임 중복체크

public boolean emailCheck (String email){
        System.out.println(userRepository.existsByEmail(email));
        return userRepository.existsByEmail(email);

    }

    public boolean nickNameCheck (String nickName){
        System.out.println(userRepository.existsByNickName(nickName));
        return userRepository.existsByNickName(nickName);

    }

// 회원가입

@Value("${jwt.secret}")
    private String secretkey;
    @Value("${app.sec}")
    private String appkey; // 회원가입, 로그인할때 외부 요청 막기위한 키
    private Long expiredMs = 1000 * 60 * 60 * 24l; //하루
//    private Long expiredMs = 1000 * 60l;    // 1분
    private Long expiredMsRe= expiredMs*24*30;

    String default_img = "https://yobee.s3.ap-northeast-2.amazonaws.com/default_profile.png";
//    private Long expiredMsRe= expiredMs*2;
    public ResponseCreateUserDto createUser (CreateUserDto createUserDto, MultipartFile profileImage) throws IOException {

        String imgPath = null;

        String email = createUserDto.getEmail();
        String password = createUserDto.getPassword();
        String fcmToken = createUserDto.getFcmToken();
        String nickName = createUserDto.getNickname();
        int type = Integer.parseInt(createUserDto.getType());

        String profileImageUrl = createUserDto.getProfileImageUrl();

        //MultipartFile profileImage = createUserDto.getProfileImage();

        System.out.println(type==0);
        //System.out.println(profileImage.isEmpty());
        System.out.println(profileImage==null);
        System.out.println(profileImageUrl==null||profileImageUrl.isEmpty());

        if (!(profileImage==null)) {
            String fileName = "profile_" + String.valueOf(System.currentTimeMillis()) + "." + profileImage.getOriginalFilename().split("\\.")[profileImage.getOriginalFilename().split("\\.").length-1];
            try {
                imgPath = s3Service.upload(profileImage, fileName);
            }
            catch (NullPointerException e) {
                System.out.println(e);
            }
        } else {
            imgPath = profileImageUrl;
        }

//        if (!createUserDto.getSecret().equals(appkey)){
//            return null;
//        }

        UserDto user = new UserDto();

        user.setEmail(email);
        user.setPassword(password);
        user.setType(type);
        user.setFcmToken(fcmToken);
        user.setNickName(nickName);
        if (imgPath == null){
            user.setProfileImage(default_img);
        }else {
            user.setProfileImage(imgPath);
        }
        user.setLevel(0);

        String token = JwtUtil.createJwt(createUserDto.getEmail(), secretkey,expiredMs);//토큰생성
        String refreshToken = JwtUtil.createRefreshToken(expiredMsRe,secretkey);

        user.setRefreshToken(refreshToken);// Refresh 토큰은 db에 저장

        userRepository.save(user.toEntity());

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(createUserDto.getEmail()));

        ResponseCreateUserDto ruser = new ResponseCreateUserDto();

        ruser.setAccessToken(token); //엑세스 토큰 응답값에 넣어주고
        ruser.setRefreshToken(optionalUser.get().getRefreshToken());
        ruser.setNickname(optionalUser.get().getNickName());
        ruser.setProfileImage(optionalUser.get().getProfileImage());
        ruser.setUserId(optionalUser.get().getUserId());
        ruser.setEmail(optionalUser.get().getEmail());
        ruser.setPassword(optionalUser.get().getPassword());
        ruser.setType(optionalUser.get().getType());
        ruser.setFcmToken(optionalUser.get().getFcmToken());
        ruser.setLevel(optionalUser.get().getLevel());

        experienceService.createExperience(optionalUser.get());

        return ruser;

    }
```

<br>

# 2. 이메일 인증
```java
// configuration/EmailConfig.java

// 메일 전송 설정

@Configuration
@PropertySource("classpath:application.properties")
public class EmailConfig {

    @Value("${spring.mail.username}")
    private String id;
    @Value("${spring.mail.password}")
    private String password;
    @Value("${spring.mail.host}")
    private String host;
    @Value("${spring.mail.port}")
    private int port;

    @Bean
    public JavaMailSender javaMailService() {
        JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();

        javaMailSender.setHost(host); // smtp 서버 주소
        javaMailSender.setUsername(id); // 설정(발신) 메일 아이디
        javaMailSender.setPassword(password); // 설정(발신) 메일 패스워드
        javaMailSender.setPort(port); //smtp port
        javaMailSender.setJavaMailProperties(getMailProperties()); // 메일 인증서버 정보 가져온다.
        javaMailSender.setDefaultEncoding("UTF-8");
        return javaMailSender;
    }

    private Properties getMailProperties() {
        Properties properties = new Properties();
        properties.setProperty("mail.transport.protocol", "smtp"); // 프로토콜 설정
        properties.setProperty("mail.smtp.auth", "true"); // smtp 인증
        properties.setProperty("mail.smtp.starttls.enable", "true"); // smtp starttls 사용
        properties.setProperty("mail.debug", "true"); // 디버그 사용
        properties.setProperty("mail.smtp.ssl.trust","smtp.naver.com"); // ssl 인증 서버 주소
        properties.setProperty("mail.smtp.ssl.enable","true"); // ssl 사용
        return properties;
    }
}

// util/RedisUtil.java

// redis 설정

@RequiredArgsConstructor
@Service
public class RedisUtil {

    private final StringRedisTemplate stringRedisTemplate;

    // key를 통해 value 리턴
    public String getData(String key) {
        ValueOperations<String, String> valueOperations = stringRedisTemplate.opsForValue();
        return valueOperations.get(key);
    }

    // 유효 시간 동안(key, value)저장
    public void setDataExpire(String key, String value, long duration) {
        ValueOperations<String, String> valueOperations = stringRedisTemplate.opsForValue();
        Duration expireDuration = Duration.ofSeconds(duration);
        valueOperations.set(key, value, expireDuration);
    }

    public void deleteData(String key) {
        stringRedisTemplate.delete(key);
    }
}

// user/service/EmailService.java

// 인증번호 생성 및 메일 전송 로직

@PropertySource("classpath:application.properties")
@Slf4j
@RequiredArgsConstructor
@Service
public class EmailService {

    private final JavaMailSender javaMailSender;

    private final RedisUtil redisUtil;

    //인증번호 생성
    //private final String ePw = createKey();

    @Value("${spring.mail.username}")
    private String id;

    public MimeMessage createMessage(String to, String ePw)throws MessagingException, UnsupportedEncodingException {

        log.info("보내는 대상 : "+ to);
        log.info("인증 번호 : " + ePw);
        MimeMessage  message = javaMailSender.createMimeMessage();

        message.addRecipients(MimeMessage.RecipientType.TO, to); // to 보내는 대상
        message.setSubject("요비 회원가입 인증 코드: "); //메일 제목

        // 메일 내용 메일의 subtype을 html로 지정하여 html문법 사용 가능
        String msg="";
        msg += "<h1 style=\"font-size: 30px; padding-right: 30px; padding-left: 30px;\">이메일 주소 확인</h1>";
        msg += "<p style=\"font-size: 17px; padding-right: 30px; padding-left: 30px;\">아래 확인 코드를 회원가입 화면에서 입력해주세요.</p>";
        msg += "<div style=\"padding-right: 30px; padding-left: 30px; margin: 32px 0 40px;\"><table style=\"border-collapse: collapse; border: 0; background-color: #F4F4F4; height: 70px; table-layout: fixed; word-wrap: break-word; border-radius: 6px;\"><tbody><tr><td style=\"text-align: center; vertical-align: middle; font-size: 30px;\">";
        msg += ePw;
        msg += "</td></tr></tbody></table></div>";

        message.setText(msg, "utf-8", "html"); //내용, charset타입, subtype
        message.setFrom(new InternetAddress(id,"요비 매니저")); //보내는 사람의 메일 주소, 보내는 사람 이름

        return message;
    }

    public MimeMessage createPasswordMessage(String to, String ePw)throws MessagingException, UnsupportedEncodingException {

        log.info("보내는 대상 : "+ to);
        log.info("인증 번호 : " + ePw);
        MimeMessage  message = javaMailSender.createMimeMessage();

        message.addRecipients(MimeMessage.RecipientType.TO, to); // to 보내는 대상
        message.setSubject("요비 임시 비밀번호 생성안내: "); //메일 제목

        // 메일 내용 메일의 subtype을 html로 지정하여 html문법 사용 가능
        String msg="";
        msg += "<h1 style=\"font-size: 30px; padding-right: 30px; padding-left: 30px;\">임시 비밀번호 생성</h1>";
        msg += "<p style=\"font-size: 17px; padding-right: 30px; padding-left: 30px;\">새로운 임시 비밀번호 생성 안내를 위해\n" +
                "발송된 메일입니다. 임시 비밀번호는 아래와 같습니다.</p>";
        msg += "<div style=\"padding-right: 30px; padding-left: 30px; margin: 32px 0 40px;\"><table style=\"border-collapse: collapse; border: 0; background-color: #F4F4F4; height: 70px; table-layout: fixed; word-wrap: break-word; border-radius: 6px;\"><tbody><tr><td style=\"text-align: center; vertical-align: middle; font-size: 30px;\">";
        msg += ePw;
        msg += "</td></tr></tbody></table></div>";

        message.setText(msg, "utf-8", "html"); //내용, charset타입, subtype
        message.setFrom(new InternetAddress(id,"요비 매니저")); //보내는 사람의 메일 주소, 보내는 사람 이름

        return message;
    }

    // 인증코드 만들기
    public static String createKey() {
        StringBuffer key = new StringBuffer();
        Random rnd = new Random();

        for (int i = 0; i < 6; i++) { // 인증코드 6자리
            key.append((rnd.nextInt(10)));
        }
        return key.toString();
    }

    /*
        메일 발송
        sendSimpleMessage의 매개변수로 들어온 to는 인증번호를 받을 메일주소
        MimeMessage 객체 안에 내가 전송할 메일의 내용을 담아준다.
        bean으로 등록해둔 javaMailSender 객체를 사용하여 이메일 send
     */
    public String sendSimpleMessage(String target) {

        // 임의의 authKey 생성
        Random random = new Random();
        String ePw = String.valueOf(random.nextInt(888888) + 111111); // 범위 : 111111 ~ 999999

        MimeMessage message = null;
        try {
            message = createMessage(target, ePw);
        } catch (MessagingException | UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
        try {
            redisUtil.setDataExpire(ePw, target, 60 * 5L); // 유효 시간 설정하여 Redis에 저장(유효시간 5분)
            javaMailSender.send(message); // 메일 발송
        } catch (MailException es) {
            es.printStackTrace();
            throw new IllegalArgumentException();
        }
        return ePw; // 메일로 보냈던 인증 코드를 서버로 리턴
    }

```