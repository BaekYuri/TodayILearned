# Jenkins로 nodejs 자동배포

AWS EC2 환경(Ubuntu 18.04)에서 GitLab에 Push 할 때 마다, 자동으로 배포해주는 시스템을 만들었다.

## jenkins 설치

1. 패키지 업데이트

   ```
   > apt-get update -y && apt-get upgrade -y
   ```

   

2. Java 설치

   ```
   > sudo apt install openjdk-8-jdk
   ```

3. 패키지에 jenkins 추가

   ```
   > wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   > sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   ```

4. jenkins 설치

   ```
   > sudo apt update
   > sudo apt install jenkins
   ```

5. jenkins가 오류없이 설치되었다면, 상태를 확인할 수 있다.

   ```
   sudo systemctl status jenkins.service
   ```

6. 이제 시스템이 시작될 때 젠킨스가 자동으로 실행된다.

## jenkins 웹에서 설정

1. http://[ip]:8080/에 접속한다.

2. 초기 비밀번호를 Administrator password에 입력한다. 아래 명령어를 입력하면 초기비밀번호를 알 수 있다.

```
> sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```



![image-20211022223901309](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022223901309.png)

3. Install suggested plugins를 선택하고 기다린다. 후에 admin 계정 설정 및 주소를 정할 수 있다.

## 플러그인 설치

4. Jenkins 관리 > 플러그인 관리에 들어간다.

5. gitlab 계정과 연동하여 nodejs 환경을 배포할 것이기 때문에 ''설치가능'' 탭에서 nodejs, gitlab을 검색해 관련 플러그인을 모두 체크한다.

6. 체크한 후, Install without restart를 누른다.

![image-20211022224403519](C:\Users\multicampus\AppData\Roaming\Typora\typora-user-images\image-20211022224403519.png)

## Nodejs 설정

7. Jenkins 관리 > Global Tool Configuration에 들어가서 nodejs 설정을 해놓아야 pipeline에서 툴을 사용할 수 있다.

8. 아래로 쭉 내리면 NodeJS가 있고, Add NodeJS를 눌러 Name, Version을 설정한다. 우분투의 NodeJS 버전을 확인하는 것이 좋다. Apply, Save를 눌러 저장한다.

   ```
   > node -v
   ```

   ![image-20211022224925231](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022224925231.png)

## GitLab 설정

9. Jenkins 관리 > 시스템 설정 탭에 들어가서 Gitlab을 탭을 찾는다.

   ![image-20211022225109931](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022225109931.png)

10. Connection name을 적당히 설정하고, 깃랩 URL을 입력한다.

11. Credential은 아래의 Add>Jenkins를 이용해 설정할 수 있다.

    ![image-20211022225234558](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022225234558.png)

## GitLab API Token 발급

12. GitLab에 로그인 후 , User Setting > Access Tokens에 들어간다.

13. 이름 및 만료일자를 정하고, Scopes에 api를 체크한 후 Create personal access token을 누른다.

![image-20211022225635346](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022225635346.png)

14. 발급된 토큰은 복사해두고 까먹지 않게 잘 보관해둔다.
15. 다시 Jenkins에 돌아와서 Credentials를 등록한다.

![image-20211022225814785](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022225814785.png)

16. 등록한 후, Test Connection을 누르고 Success가 나온다면 정상인 상태이다. Apply > 저장을 누르고 설정 페이지를 벗어난다.
17. 메인 페이지에서 "새로운 Item"을 누른다.
18. 제목을 입력하고, Pipeline을 선택한다. 

19. Build Triggers > Build when a change is pushed to GitLab. GitLab webhook URL: xxxxxxxx를 체크하고, 고급 버튼을 눌러 Secret token 탭을 찾는다.

![image-20211022230251280](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022230251280.png)

![image-20211022230405729](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022230405729.png)

20. Generate를 눌러 Secret token을 발급받고, 이를 복사해둔다.

21. Pipeline 탭에서 원하는 배포과정을 작성한다.

![image-20211022230814745](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022230814745.png)

22. Apply > 저장을 누르고 나온다.

## GitLab Repository에 Webhook 설정

특정 레포지토리에 Push Event가 생길 때 마다, api 요청을 보내는 Webhook을 만들어야, 레포지토리가 업데이트 될 때 마다 젠킨스가 정상동작한다.

1. Git Repository > Settings > Webhooks에 들어간다.

2. URL과 Secret token은 Jenkins에 생성해놓은 파이프라인의 "구성"에서 확인할 수 있다.

   ![image-20211022231347375](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022231347375.png)

   ![image-20211022231436909](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022231436909.png)

   ![image-20211022231547116](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022231547116.png)

3. Add webhook을 한 후, Project Hooks 에서 Test > Push Event를 누르면 아래처럼 된다면 정상이다.

   ![image-20211022231709947](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022231709947.png)

4. Jenkins의 파이프라인에서 확인해보면 정상적으로 실행되었다는 것을 알 수 있다.

![image-20211022231850646](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022231850646.png)

## 부록 : Pipeline 실행 시 서버가 터진다면?

내가 사용하는 서버는 AWS 프리티어였기 때문에, t2.micro의 램이 1GB밖에 되지 않았다. 그래서 조금이라도 무거운 작업을 실행하면 서버가 갑자기 멈추는 경우가 많이 생겼다.

돈을 더 내는것 보다, 최대한 프리티어 한도 내에서 해결해보려는 와중에 Ubuntu에서 Swap 메모리를 설정하는 방법을 찾을 수 있었다.

### Swap 메모리란?

-  RAM이 부족할 때, HDD의 일정 공간을 RAM처럼 사용하는 것이다.

### Swap 메모리 설정하기

- 먼저, 현재 우분투의 용량을 확인하기 위해 df로 확인해준다.

  ```
  > df
  ```

  ![image-20211022232749828](https://raw.githubusercontent.com/BaekYuri/images/main/img/image-20211022232749828.png)

- /dev/xvdal에서 Available 크기에 따라, 적당한 Swap 메모리를 생각해 놓는다. 나는 512MB 정도를 Swap 메모리로 설정했다.

- 원하는 정도의 swap 메모리를 할당한다. 64M * 8 = 512MB

  ```
  sudo dd if=/dev/zero of=/swapfile bs=64M count=8
  ```

- 스왑 파일에 읽기,쓰기 권한을 준다.

  ```
  sudo chmod 600 /swapfile
  ```

- Linux 스왑 영역 설정

  ```
  sudo mkswap /swapfile
  ```

- 스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 만든다.

  ```
  sudo swapon /swapfile
  ```

- 절차가 성공했는지 확인한다.

  ```
  sudo swapon -s
  ```

- /etc/fstab 파일을 편집해, 부팅 시 스왑 파일을 활성화한다. 먼저 편집기를 연다.

  ```
  sudo vi /etc/fstab
  ```

- 맨 끝에 아래의 내용을 추가하고, 저장 후 종료한다.

  ```
  /swapfile swap swap defaults 0 0
  ```

- 스왑 메모리가 추가되었는지 free를 이용해 확인한다.

  ```
  > free
                total        used        free      shared  buff/cache   available
  Mem:        1002072      629300      117044         144      255728      223524
  Swap:        524284      252768      271516
  ```

  

