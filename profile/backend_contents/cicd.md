# CI/CD

소스 코드 레포지토리에 대한 지속적인 통합과 지속적인 배포 환경을 구축하기 위한 툴로 `Jenkins`를 사용하였습니다. 빌드, 배포 프로세스를 자동화하여 개발 생산성을 높일 수 있습니다.

![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/cicd1.png)

**준비물: GitLab, AWS EC2(Jenkins-Server, Deploy-Server), Docker Hub**

# 1. Jenkins-Server

1. **Jdk 설치**

   openjdk-11-jdk 설치

   ```
   sudo apt-get update
   ```

   ```
   sudo apt-get install openjdk-11-jdk
   ```

2. **Jenkins 설치**

   jenkins 설치에 필요한 패키지 업데이트

   ```
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   ```

   ```
   echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
   ```

   ```
   sudo apt update
   ```

   jenkins 설치 및 실행 확인

   ```
   sudo apt install jenkins
   ```

   ```
   sudo systemctl status jenkins
   ```

   jenkins 접속

   ```
   http://{ip 주소}:8080
   ```

   비밀번호 입력 (아래 명령어 입력해서 초기비밀번호 get)

   ```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

   `Install suggested plugins` 클릭해서 설치

   `타임존 설정` (빌드 시간을 한국시간으로 확인하려면 수행)

   jenkins 관리 > Script Console(맨 아래쪽에 있음) > 아래코드 복붙 > 실행

   ```
   System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Seoul')
   ```

3. **Docker 설치**

   ```bash
   sudo apt update
   ```

   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
   ```

   ```bash
   sudo apt update
   ```

   도커 설치

   ```
   sudo apt install docker-ce
   ```

   도커 실행 확인

   ```
   sudo systemctl status docker
   ```

   `sudo` 없이 도커사용하기

   도커 그룹에 사용자 추가하기

   ```
   sudo usermod -aG docker ${USER}
   ```

   새 그룹 구성원 자격을 적용하기 위해서 다음 명령어를 입력합니다.

   ```
   sudo su - ${USER}
   ```

   도커 그룹 확인하기

   ```
   id -nG
   ```

   <<응답>>

   ```
   {USER} ... sudo ... docker
   ```

   docker.sock 권한 변경

   ```
   sudo chmod 666 /var/run/docker.sock
   ```

4. **Docker login**

   ```bash
   sudo su - jenkins
   docker login
   ```

# 2. Jenkins x GitLab 연동

1. **Jenkins 플러그인 설치**

   jenkins 관리 > 플러그인 관리 > Available plugins > "gitlab" `검색`

   gitlab 설치

   Generic Webhook Trigger Plugin 설치

2. **New Job 생성 (Freestyle)**

   소스 코드 관리

   - Git
     - Repo URL : gitlab repo 주소
       - ex) [https://lab.ssafy.com/iamseongmin/jenkins-prac.git](https://lab.ssafy.com/iamseongmin/jenkins-prac.git)
     - Credentials > add > jenkins
       - Kind를 'Username with password'로 변경
       - **UserNamegitlab 계정 아이디Passwordgitlab 계정 비밀번호idCredential을 식별하는 아이디DescriptionCredential에 대한 설명**
     - Branch Specifier 설정
       - /main 혹은 /master 혹은 /develop 등

   빌드 유발

   - Build when a change is pushed to GitLab. GitLab webhook ... `클릭`
     - Push Events 체크
     - Opened Merge Request Events 체크

   Build Steps

   - Add build step > Execute shell
   - 아래는 테스트 코드임 (나중에 도커빌드 등 코드 입력)
     ```
     echo 'jenkins build started..'
     ```
     `저장`

3. **GitLab Webhooks 설정**

   gitlab 접속

   연동할 repo > Settings > Webhooks

   ![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/cicd2.png)

   gitlab에서 빨간 네모부분 작성 후 아래로 스크롤하면

   Project Hooks가 생성되어 있음.

   Test > push events 클릭 -> 젠킨스에서 자동 빌드 되는지 확인

# 3. Jenkins x Deploy-Server 연동

1. **Jenkins-Server에서 ssh키 생성**

   ```bash
   sudo mkdir /var/lib/jenkins/.ssh
   ```

   ```bash
   ssh-keygen -t rsa -C "키명칭" -m PEM -P "" -f /var/lib/jenkins/.ssh/"키명칭"

   ex) sudo ssh-keygen -t rsa -C "jenkins-deploy" -m PEM -P "" -f /var/lib/jenkins/.ssh/jenkins-deploy
   ```

   공개키를 검색해서 복사

   ```bash
   sudo cat /var/lib/jenkins/.ssh/"키명칭".pub

   ex) sudo cat /var/lib/jenkins/.ssh/jenkins-deploy.pub
   ```

2. **Deploy-Server에 public키 등록**

   ```bash
   sudo nano .ssh/authorized_keys
   ```

   1 에서 복사한 공개키를 맨 아랫줄에 추가한 후 저장

3. **Jenkins 플러그인 설치**

   Jenkins 관리 > 플러그인 관리 > 설치가능 탭에서 Publish over ssh 플러그인 설치

4. **Jenkins 시스템 관리**

   jenkins 관리 > 시스템 관리 > publish over ssh 영역의 SSH servers 추가버튼 클릭

   Path to Key (Private key의 경로를 작성)

   ```
   /var/lib/jenkins/.ssh/"키명칭"

   ex) /var/lib/jenkins/.ssh/jenkins-deploy
   ```

   Key (Private key 파일안의 내용을 복사 붙여넣기)

   ```
   sudo cat /var/lib/jenkins/.ssh/jenkins-deploy
   ```

   ```
   ex)
   -----BEGIN RSA PRIVATE KEY-----
   .
   .
   .
   -----END RSA PRIVATE KEY-----
   ```

   Name (접속할 ssh 서버의 이름을 작성)

   ```
   ex) deploy-server
   ```

   hostname (접속할 인스턴스의 주소 `배포서버 ip`)

   ```
   ex) xx.xx.xx.xx
   ```

   username은 접속할 유저명

   ```
   ex) ubuntu
   ```

5. **Jenkins Job Configure 수정**

   - 빌드 환경 (publish over ssh 플러그인 설치를 미리해야함)

     - Send files or execute commands over SSH after the build runs

       - Name : 배포 서버 이름 입력
       - Exec command : 배포 서버에서 실행할 명령어 입력

         ```
         # 예시

         sudo docker ps -q --filter name=test | grep -q . && sudo docker rm -f $(sudo docker ps -aq --filter name=test)
         sudo docker rmi hsm2358/test
         sudo docker pull hsm2358/test
         sudo docker run --rm -d --name test -p 8090:8090 hsm2358/test
         ```

   - Build Steps

   **Execute shell**

   ```
   # 예시
   cd BE/yobee
   chmod +x gradlew
   ./gradlew clean build --exclude-task test
   ```

   ```
   # 예시
   cd BE/yobee
   docker build -t hsm2358/test .
   docker push hsm2358/test
   ```

# [오류 및 해결]

E1. EC2 프리티어(t2.micro)를 사용할 경우, jar 빌드시 메모리 부족현상 발생 (Jenkins 무한 로딩 현상)

S1. Swap 메모리를 할당하여 문제를 해결할 수 있음

(참고: [https://sundries-in-myidea.tistory.com/102](https://sundries-in-myidea.tistory.com/102))

dd 명령어를 통해 swap 메모리를 할당. 128M x 16 = 2048 (2GB 할당)

```bash
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
```

스왑 파일에 대한 읽기 및 쓰기 권한을 업데이트

```
sudo chmod 600 /swapfile
```

Linux 스왑 영역을 설정

```
sudo mkswap /swapfile
```

스왑 공간에 스왑 파일을 추가

```
sudo swapon /swapfile
```

절차가 성공했는지 확인

```
sudo swapon -s
```

**/etc/fstab** 파일을 편집하여 부팅 시 스왑 파일을 활성화합니다.

편집기로 파일 열기

```
sudo nano /etc/fstab
```

파일 끝에 다음 줄을 새로 추가하고 파일을 저장한 다음 종료

```
/swapfile swap swap defaults 0 0
```

스왑메모리가 적용됐는지 확인

```
free
```

![Untitled](https://github.com/YOBEE-8th/.github/blob/main/profile/backend_contents/img/cicd3.png)
