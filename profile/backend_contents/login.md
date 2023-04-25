# 0. Contents
1. 로그인(일반&소셜)
2. 토큰 갱신

<br>

# 1. 로그인(일반&소셜)

```java
// user/service/UserService.java

// 일반 로그인

public ResponseCreateUserDto login (LoginUserDto loginUserDto){

        String token = JwtUtil.createJwt(loginUserDto.getEmail(), secretkey, expiredMs);//토큰생성

        String refreshToken = JwtUtil.createRefreshToken(expiredMsRe, secretkey);

        ResponseCreateUserDto ruser = new ResponseCreateUserDto();
        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(loginUserDto.getEmail()));

        if (optionalUser.isEmpty()) {
            return null;
        }
        else {
            UserDto joinedUser = new UserDto();
            BeanUtils.copyProperties(optionalUser.get(),joinedUser);//이미 db에 있는 값들 복사해주고
            joinedUser.setRefreshToken(refreshToken);//재로그인 했을때 refresh 토큰 새로 저장
            joinedUser.setFcmToken(loginUserDto.getFcmToken());//fcm 토큰 새로 저장해주고
            joinedUser.setType(0);  //일반 로그인은 타입0
            userRepository.save(joinedUser.toEntity());//db 업데이트해주고

            ruser.setAccessToken(token); //엑세스 토큰 응답값에 넣어주고
            ruser.setRefreshToken(joinedUser.getRefreshToken());
            ruser.setNickname(joinedUser.getNickName());
            ruser.setProfileImage(joinedUser.getProfileImage());
            ruser.setUserId(joinedUser.getUserId());
            ruser.setEmail(joinedUser.getEmail());
            ruser.setPassword(joinedUser.getPassword());
            ruser.setType(joinedUser.getType());
            ruser.setFcmToken(joinedUser.getFcmToken());
            ruser.setLevel(joinedUser.getLevel());

            return ruser;
        }

    }

// 소셜 로그인

    public ResponseCreateUserDto socialLogin (SocialLoginUserDto socialLoginUserDto){

        String token = JwtUtil.createJwt(socialLoginUserDto.getEmail(), secretkey,expiredMs);//토큰생성

        System.out.println(token);

        String refreshToken = JwtUtil.createRefreshToken(expiredMsRe,secretkey);

        ResponseCreateUserDto ruser = new ResponseCreateUserDto();
        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByEmail(socialLoginUserDto.getEmail()));

        if (optionalUser.isEmpty()) {
            return null;
        }
        else {
            UserDto joinedUser = new UserDto();
            BeanUtils.copyProperties(optionalUser.get(),joinedUser);//이미 db에 있는 값들 복사해주고
            joinedUser.setRefreshToken(refreshToken);//재로그인 했을때 refresh 토큰 새로 저장
            joinedUser.setFcmToken(socialLoginUserDto.getFcmToken());//fcm 토큰 새로 저장해주고
            joinedUser.setType(socialLoginUserDto.getType()); // 카카오는 2 구글은 1
            userRepository.save(joinedUser.toEntity());//db 업데이트해주고

            ruser.setAccessToken(token); //엑세스 토큰 응답값에 넣어주고
            ruser.setRefreshToken(joinedUser.getRefreshToken());
            ruser.setNickname(joinedUser.getNickName());
            ruser.setProfileImage(joinedUser.getProfileImage());
            ruser.setUserId(joinedUser.getUserId());
            ruser.setEmail(joinedUser.getEmail());
            ruser.setPassword(joinedUser.getPassword());
            ruser.setType(joinedUser.getType());
            ruser.setFcmToken(joinedUser.getFcmToken());
            ruser.setLevel(joinedUser.getLevel());

            return ruser;
        }

    }
```

<br>

# 2. 토큰갱신

```java
// user/service/UserService.java

// 토큰 유효성 검사

public boolean tokenCheck(String token) { // valid: true / expired: false
        boolean result = true;
        try {
            result = !JwtUtil.isExpired(token,secretkey);
        }
        catch (ExpiredJwtException e) {
            System.out.println(e);
            result = false;
        }
        catch (Exception e) {
            System.out.println(e);
        }

        return result;
    }

// 토큰 리프레시

public ResponseValidateTokenDto refresh(ValidateTokenDto validateTokenDto){// 리프레시 토큰 오면 보내는거

        String accessToken = validateTokenDto.getAccessToken();
        String refreshToken = validateTokenDto.getRefreshToken();

        Optional<User> optionalUser = Optional.ofNullable(userRepository.findByRefreshToken(refreshToken));
        if (!optionalUser.isPresent()){
            throw new EntityNotFoundException("User not present in the database");
        }
        User user = optionalUser.get();

        String email = user.getEmail();

        ResponseValidateTokenDto responseValidateTokenDto = new ResponseValidateTokenDto();

        if (!tokenCheck(accessToken) && tokenCheck(refreshToken)) {
            //String email = JwtUtil.getUserName(refreshToken,secretkey);
            //System.out.println(email);
            System.out.println("User by refreshToken : " + email);
            String newAccessToken = JwtUtil.createJwt(email, secretkey,expiredMs);
            System.out.println("User by newAccessToken : " + JwtUtil.getUserName(newAccessToken,secretkey));

            responseValidateTokenDto.setAccessToken(newAccessToken);
            responseValidateTokenDto.setRefreshToken(refreshToken);

        } else if (!tokenCheck(accessToken) && !tokenCheck(refreshToken)) {
            responseValidateTokenDto.setAccessToken(null);
            responseValidateTokenDto.setRefreshToken(null);
        }
        else {
            responseValidateTokenDto.setAccessToken(accessToken);
            responseValidateTokenDto.setRefreshToken(refreshToken);
        }

        return responseValidateTokenDto;
    }

```