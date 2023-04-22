>프로젝트를 진행하고 배포를 진행하고 나면 프로그램이 변경될 때마다 배포된 컴퓨터에 들어가 파일을 업데이트하고 다시 서버를 실행시키는 수고를 다들 한번쯤 겪었을 것이다.
>프로젝트가 끝나게 되더라도 버그가 발견되거나 고도화 작업등을 하게 된다면 직접 배포를 해야하는데 여간 귀찮은 작업이 아니다. 
>프로그래머는 게을러야 한다. 게으르기 위해선 모든 과정이 자동으로 진행 되야 하고 그렇게 등장하게 된것이 CI/CD이다. 오늘은 CI/CD란 무엇인지와 CI/CD툴중 하나인 젠킨스에 대해 알아보고자 한다.

## CI/CD란?
CI/CD (Continuous Integration/Continuous Deployment)란 지속적 통합 / 지속적 배포 라는 의미를 가지고 있으며 애플리케이션 개발과 배포 단계를 자동화하여 유저에게 더욱 짧은 주기로 애플리케이션을 제공하는 방법을 뜻한다.


### 지속적 통합(Continuous Integration)
CI는 개발자를 위한 프로세스다. 여럿이 협업을 하게 되면 동시간대에 코드를 작성하게 될 것이고, 작성한 코드를 합치게 될 것이다. 코드를 합치다보면 다른 개발자가 변경한 코드와 충돌이 일어날 수 있게 된다.
CI와 테스트코드를 이용하면 통합을 할때 마다 자동으로 테스트코드를 실행시킨다. 테스트에 문제가 있을 경우, 해당 통합은 이뤄지지 않는다. 테스트를 통해 개발자는 더 빠른 피드백을 받을 수 있고, 버그가 적은 코드를 만들수 있다.

### ## 지속적 배포(Continuous Deployment)
CD는 CI가 모두 진행된 후 이어지는 프로세스다. 지속적 통합이 완료되었다면, 작성된 코드를 배포 서버에 올려 배포된 코드를 신규 코드로 변경시켜준다.

이 모든 작업은 자동으로 이뤄지며, CI/CD 과정중 문제가 생겼을 경우, 현재 배포되어있는 파일에는 영향을 끼치지 않아 안정적인 배포 환경이 갖춰지게 된다.

[![CI/CD Flow](https://www.redhat.com/rhdc/managed-files/styles/wysiwyg_full_width/private/ci-cd-flow-desktop.png?itok=NNRD1Zj0 "CI/CD Flow")](https://www.redhat.com/rhdc/managed-files/ci-cd-flow-desktop.png?cicd=32h281b)
(출처: [Red Hat](https://www.redhat.com/ko/topics/devops/what-is-ci-cd))

### CI/CD 툴
CI/CD에 대해 알아봤으니 CI/CD를 이용하기 위한 대표적인 3가지 툴과 장단점을 살펴보자

### Jenkins
[![파일:Jenkins logo with title.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Jenkins_logo_with_title.svg/799px-Jenkins_logo_with_title.svg.png)](https://upload.wikimedia.org/wikipedia/commons/e/e3/Jenkins_logo_with_title.svg)
Jenkins는 Java로 만들어진 오픈소스 CI/CD툴 이다. 별도의 서버가 없어 초기 구성과 서버를 직접 구성해야한다.

#### 장점
- 오픈소스라 무료다.
- 사용자가 많아 커뮤니티가 활성화 되어있다.

#### 단점
- 별도의 서버를 직접 구축해서 사용해야 한다.
- 다른 CI/CD툴에 비해 설정이 복잡하며, Jenkins프로그램 또한 관리해 줘야한다.

### Github Action
![](https://velog.velcdn.com/images/kisuk623/post/075c2f8d-2cca-4641-a5a5-9e9cea453d83/image.png)

GitHub Action은 수많은 개발자들이 사용하는 GitHub를 기반으로 CI/CD를 지원하는 툴이다.
저장소에 특정 이벤트가 발생할 때 마다 원하는 Workflow를 만들어 빌드, 테스트, 배포등을 진행할 수 있다.

#### 장점
- public 저장소에 대해선 무료다.
- 별도의 서버를 설치할 필요가 없다.

#### 단점
- private 저장소는 일정 용량/시간을 초과해 사용한다면 요금이 부과된다.

### AWS CodePipeline
![The AWS Codepipeline logo](https://uploads-ssl.webflow.com/5c1bedd2f3c4a4332cfd911c/5c50d7157f77d56fef8f31cd_AWS-codepipeline_1-2-e1535049169402_7d48b3a0910a00c8296552126f151d02_1000.png)
AWS CodePipeline은 AWS에서 제공하는 CI/CD 서비스이다. AWS에서 제공하는 CodeCommit, CodeDeploy, CodeBuild등을 하나의 파이프라인처럼 연결시켜 소스코드의 업로드부터 배포까지의 과정을 담당한다.

#### 장점
- 하나의 플랫폼에서 모든 작업이 가능하다.

#### 단점
- 하나의 플랫폼에 종속적이다.
- 프리티어 기간에는 몇몇개의 서비스가 무료로 사용가능하지만 프리티어 종료시 요금이 발생한다.


## Jenkins 설치

AWS상에서 기존 프로젝트가 배포되어있는 인스턴스가 존재하고 있다는 가정하에 진행한다.

![](https://velog.velcdn.com/images/kisuk623/post/8405c82f-9a10-434b-aa7f-e0817db77e25/image.png)



인스턴스유형은 t3.micro로 키 페어는 새로 생성해도 되지만 기존에 사용하던 키페어를 그대로 사용 해도 무방하다.
보안그룹 또한 기존에 배포된 인스턴스의 보안그룹을 그대로 따라가도 무방하며, 세가지 설정을 마친 뒤 인스턴스를 생성하면 된다.

### Docker 설치

Jenkins는 자바로 개발된 프로그램이다. Jenkins를 직접 설치한다면 설치하고 설정해 줄것이 많지만, Jenkins를 Docker로 설치한다면 아주 간단하게 Jenkins를 설치할 수 있다. 
[Docker 홈페이지 Document](https://docs.docker.com/engine/install/ubuntu/)

#### 저장소 셋업

```shell
$ sudo apt-get update
```

```shell
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

#### Docker GPG 키 추가

```shell
$ sudo mkdir -p /etc/apt/keyrings
```

```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### 저장소 설정

```shell
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Docker 엔진 설치

```shell
$ sudo apt-get update
```

```shell
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### docker-compose 설치

```shell
$ sudo apt install docker-compose
```

위 과정을 모두 마친뒤 `docker -v` 명령어를 입력하면 docker의 버전이 출력되어 docker가 설치된 것을 알수있다.
그 후 docker-compose를 통해 jenkins를 다운로드, 실행 해줄 것이다.

```yaml
version: "3"
services:
  jenkins:
    image: jenkins/jenkins:lts
    user: root
    volumes:
      - ./jenkins:/var/jenkins_home
    ports:
      - 8080:8080
```

`docker-compose.yml`파일을 만들어 위 내용을 붙여넣기 한다.

#### jenkins 설치

```shell
$ sudo docker-compose up -d
```

그 뒤 위 명령어를 실행하면 Docker가 jenkins이미지를 다운받아 실행시켜 줄것이다.

Jenkins는 초기 비밀번호가 존재한다. jenkins가 실행되면서 비밀번호를 출력해주며, 해당 로그를 보지 못한채 종료 했다면 파일로도 남아있으니 걱정하지 않아도 된다.

```shell
$ sudo docker logs jenkins:lts
```

![](https://velog.velcdn.com/images/kisuk623/post/a5163b1a-04a1-429b-8c52-72281265017a/image.png)


위 암호는 필시 적어두고 다음 단계로 넘어가자

혹 해당 화면이 출력되지 않는다면 Docker로 직접 들어가 비밀번호를 가져올 수 있다.

```shell
$ sudo docker exec -it jenkins:lts /bin/bash
```

```shell
$ cat /var/jenkins_home/secrets/initialAdminPassword
```

![](https://velog.velcdn.com/images/kisuk623/post/b043adce-9a2e-4da2-b92f-2e7178998cdc/image.png)


## Jenkins 설정

이제 브라우저를 열어 EC2 퍼블릭 IP주소에 `docker-compose.yml`에 입력한 포트로 접속을 진행한다.
접속이 되지않는다면, 보안설정에서 위에서 입력한 포트를 열어주면 접속이 될것이다.

![](https://velog.velcdn.com/images/kisuk623/post/c1b8b004-a0ed-44d0-815b-caef8cd4ce9c/image.png)


비밀번호는 위에서 초기 생성된 비밀번호를 넣으면 다음단계로 진입할수 있다.

![](https://velog.velcdn.com/images/kisuk623/post/a5b7dd8e-1887-46ff-a761-0f00304887f8/image.png)


그 후 Install suggested plugins를 클릭해 추천 플러그인을 설치한다.

설치가 끝난후 Jenkins 메인화면이 등장하게 된다.

### Node.js 설치

Jenkins는 Node.js를 기본적으로 탑재하고 있지 않는다. Jenkins에서 Node.js 프로젝트를 사용하기 위해선 Node.js 플러그인을 설치해야 한다.

메인화면 > Jenkins 관리 > 플러그인 관리에 들어가 NodeJS 플러그인을 설치해 준다.

![](https://velog.velcdn.com/images/kisuk623/post/4d8728ef-e61b-4636-a4f1-bb153d54ebe2/image.png)


### Project 생성

다시 메인화면으로 나와 좌측상단에 새로운item 아이콘을 클릭하면 아래와 같은 화면이 나오게 된다. Jenkins에서 CI/CD는 주로 Freestyle과 Pipeline을 통해 이루어 지는데 오늘은 Freestyle 프로젝트로 진행한다.
![](https://velog.velcdn.com/images/kisuk623/post/1236284c-4386-4d6a-b030-560b185b1c7a/image.png)


그 후 GitHub project와 소스코드 관리에 GitHub의 저장소를 각각 입력해 준다.
저장소가 public이라면 상관이 없지만 private 저장소라면 해당 저장소에 접근 가능한 계정이 존재해야한다.
Credential 하단에 Add를 클릭해 GitHub 계정을 추가한다. 
GitHub은 현재 평문 비밀번호로 외부 엑세스가 불가능하다. 그렇기 때문에 password대신 GitHub 토큰을 발급받아 입력 해주면 된다.

![](https://velog.velcdn.com/images/kisuk623/post/0b0f899d-de82-4f4f-9c28-cd8671a382a8/image.png)


그 후 변경된 것을 감지할 branch를 하단에 입력 해주면 된다. branch는 여러개도 가능하다.
![](https://velog.velcdn.com/images/kisuk623/post/879f9b3b-149c-4ba2-b217-4cfb82d33aa8/image.png)


그리고 빌드 유발 설정을 해준다. 빌드 유발은 언제 CI/CD를 작동해 빌드할 것인가 결정하는 항목인데 Github에서는 Webhook이라는 것을 지원해 준다.
Webhook은 GitHub저장소에 PR, merge, push등 코드의 변화가 일어났을 때 특정 주소로 알림을 해주는 기능이 있다. 이 Webhook을 통해 GitHub 저장소의 코드가 변경된것을 감지한 Jenkins가 저장소에 있는 코드를 다운받아 빌드작업을 시작하게 된다.
그리고 Node버전은 개발에 사용된 버전을 세팅 해주면 된다.

![](https://velog.velcdn.com/images/kisuk623/post/e53f9ff1-fff6-4265-8b8d-18e8cd5ee7e3/image.png)


마지막으로 Build Steps을 추가해준다. ssh방식을 이용하려면 Execute shell을 build step으로 사용해야한다.

![](https://velog.velcdn.com/images/kisuk623/post/119a6b7b-ba66-46cb-a0cf-5ff22444e916/image.png)



Jenkins는 기본적으로 몇가지의 환경변수를 가지고 있다. 해당 환경변수는 shell명령어에서 사용할수 있으며, 환경변수를 직접 선언해 사용할 수도 있다.
명령어는 위에서부터 아래로 한줄씩 실행되고 실행에 실패한 순간 해당 빌드는 정지되고 실패로 처리된다.

위 명령어를 기준으로 순서를 설명하면
1. 기존 빌드된 파일 삭제
2. 환경 변수 파일 붙여넣기
3. npm 설치, 빌드
4. E2E 테스트 진행
5. ec2 원격 빌드 위치에 `$JOB_NAME` 폴더 생성 (JOB_NAME은 Jenkins에서 만들었던 프로젝트 명)
6. rsync를 통해 Jenkins에 있는 폴더와 원격 위치에 있는 폴더 동기화 (WORKSPACE는 Jenkins내부에 있는 프로젝트 폴더 명)
7. 원격 폴더로 이동해 기존 pm2 종료
8. 프로젝트 재실행

순서로 진행된다. 테스트가 실패하게 되면 그대로 빌드가 끝나게 되어 현재 배포되어있는 파일에는 전혀 영향이 가지 않게 된다.

## 끝나지 않았다.

위 작업을 모두 끝내고 GitHub에 코드를 push해도 Jenkins는 아무런 변화가 없을 것이다. 왜냐하면 GitHub의 Webhook설정도 하지 않았고, 기존 EC2에 원격 접속 허용또한 하지 않았다. 차근차근 하나씩 마무리 해보자.

### GitHub Webhook 설정

![](https://velog.velcdn.com/images/kisuk623/post/5618e576-ece0-43c6-8166-9e687d49c4dc/image.png)


Jenkins에서 설정한 Github저장소에서 Settings > Webhooks에 들어가면 Webhook을 추가할 수 있다.
Add webhook을 클릭해 Jenkins가 있는 EC2의  `public-ip:8080/github-webhook`를 추가해 준다.
밖으로 나오면 방금 추가한 주소 앞에 X표시가 떠있을 것이다. 이는 EC2인스턴스에서 GitHub의 주소를 허용해주지 않아 X가 표시된 것으로 EC2가 포함된 보안그룹에 들어가 GitHub의 주소를 추가 허용해 준다.

![](https://velog.velcdn.com/images/kisuk623/post/7f1498da-49d6-40eb-a4fe-663d3f627d5d/image.png)


Github주소는 총 4개이고 4개 모두 보안그룹 허용에 추가해줘야한다.
- 140.82.112.0
- 192.30.252.0
- 185.199.108.0
- 143.55.64.0

4개를 추가한 뒤 시간이 지나거나 webhook을 삭제 후 다시 추가하면 체크 표시로 변경되었을 것이다.

### AWS 보안설정 추가

jenkins에서 원격 EC2서버로 접속하기 위해선 보안규칙이 허용되야 한다. public ip보다 private ip를 규칙에 추가하는것이 보안에 더 좋으니 해당 ip를 보안규칙에 추가해 준다.

![](https://velog.velcdn.com/images/kisuk623/post/4f1d563a-cebd-431c-8f89-b53b21967255/image.png)


### Shell 설정

마지막 한 단계만 남았다.
우리가 `ssh -i`명령어를 입력해 EC2의 shell에 들어갈 때 AWS에서 발급받은 Keypair를 입력해 접속 했던것을 떠올려 보자
원격 ssh에 접속하기 위해선 ssh-key가 필요하다. jenkins의 ssh-key를 발급받아 EC2에서 해당 ssh-key를 허용해 주면 jenkins에서 원격EC2로 접근이 가능하다.

Jenkins를 설치했던 EC2로 들어가 Docker상의 Jenkins로 접속해 ssh-key를 발급받자
```shell
$ sudo docker exec -it jenkins:lts /bin/bash
```

```shell
$ ssh-keygen
```

```shell
$ cd .ssh
```

`.ssh` 폴더에 들어가면 `id_rsa`,  `id_rsa.pub` 두개의 파일이 생성되었을 것이다. 두 파일은 keypair로 두개가 한쌍으로 이뤄져있다 `id_rsa` 는 private key, `id_ras.pub` 는 public key이다.

public key를 EC2에 등록하려면 우선 원격 EC2에 접속해 `.ssh` 폴더로 들어간다.
폴더 내부엔 `authorized_keys` 파일이 있을 것이다 해당 파일에 jenkins에서 발급받은 public key를 추가해 준다.
기존에 있던 문자를 삭제해 버릴경우 EC2에 접속할 keypair가 유효하지 않게 된다 주의해서 추가해 준다.

마지막으로 Jenkins로 돌아와 EC2에 대해 원격 접속을 해줘야한다.

```shell
$ ssh ubuntu@private-ip ls -al
```

명령어는 아무거나 상관없으며 명령어를 입력할 경우 최초 접속으로 핑거프린트를 받게 될것이다 yes를 입력해 핑거프린트를 등록하면 Shell에 대한 모든설정이 종료된다.

이제 다시 Github에 프로젝트를 업로드 해보면 Jenkins가 제대로 작동되는것을 볼 수있으며 진행사항을 콘솔로 지켜볼 수 있다.
![](https://velog.velcdn.com/images/kisuk623/post/981500de-7f26-4eed-b1f5-6339ddf5368d/image.png)


## 마치며

오늘은 CI/CD와 CI/CD의 툴중 하나인 Jenkins에 대해 알아 보았다.
CI/CD는 매우 매력적인 요소다. 초기에 세팅에는 시간이 걸리지만 세팅이 완료되고 난 뒤에는 개발자는 개발에만 신경을 쓸 수 있다. 빌드나 테스트 과정중 실패가 된다면 Slack이나 이메일을 통해 알림등을 받을수도 있어 즉각 대응이 가능하며, 매번 번거롭게 배포 시간을 가지지 않아도 된다.
CI/CD를 한번 써보게 되면 앞으로는 CI/CD가 구축되어 있지 않은 프로젝트는 진행하기 싫어질 것이다.


## Reference
 https://www.redhat.com/ko/topics/devops/what-is-ci-cd
 https://docs.docker.com/engine/install/ubuntu/
