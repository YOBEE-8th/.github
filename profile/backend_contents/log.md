# 로그 집중화

개발을 하다보면 여러 개의 서버에서 로그가 쏟아져 나옵니다.

- 에러 로그
- 지표 측정을 위해 출력한 로그

등 여러 가지 로그들을 마주하게 되는데, 문제는 이 로그를 확인하기 위해서는 EC2에 접속해서 별도의 명령어를 입력해야하는 번거로운 과정을 거쳐야 한다는 것입니다.

프론트와 백 팀원들이 쉽고 간편하게 로그를 확인할 수 있는 방법을 몰색중에 logspout + papertrail 조합으로 아래 그림과 같이 웹사이트에서 로그를 체크할 수 있는 방법을 발견하게 되었습니다.

![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/log1.png)

---

# 설치 방법

1. papertrail ([https://www.papertrail.com/](https://www.papertrail.com/))
   - 회원가입
   - 이메일 인증
   - 빨간 버튼(Add your first system) 클릭
     ![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/log2.png)
2. 도커로 logspout 컨테이너 실행하기

   - logspout 이미지 pull 받기

   ```python
   docker pull gliderlabs/logspout:latest
   ```

   - 빨간 박스에 해당하는 부분을 복사하여 EC2에 붙여넣기 후 실행
     ![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/log3.png)
   - 아래 빨간 네모박스 (logsN.papertrailapp.com:port) 부분을 복사
     ![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/log4.png)
   - logspout (위에서 복사한 내용으로 {logsN.papertrailapp.com:port} 부분 대체) 컨테이너 실행
     ```python
     docker run --name logspout --restart=always -d -v=/var/run/docker.sock:/var/run/docker.sock gliderlabs/logspout:latest syslog+tls://{logsN.papertrailapp.com:port}
     ```

3. 서비스 할 컨테이너 실행하기

   ```python
   # 예시
   sudo docker run --rm -d -p 8090:8090\
   --log-driver=syslog\
   --log-opt syslog-address=udp://{logsN.papertrailapp.com:port}\
   --log-opt tag={tag}\
   {도커 이미지}
   ```

   - {tag} - 로그에서 식별자로 쓰일 이름 ex) Flask, Springboot 등
   - {도커 이미지} - 컨테이너로 실행할 이미지의 ID 혹은 이름

---

위 과정을 거치면

Events탭([https://my.papertrailapp.com/events](https://my.papertrailapp.com/events))에서 로그를 확인할 수 있습니다.

설정을 통해 글씨체, 테마 등을 변경할 수 있습니다.

![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/log5.png)
