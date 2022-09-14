# [Jenkins] 1.설치와 초기 설정

프로젝트 진행하면서 내부 CI/CD 도구로 jenkins를 사용하기로 하였습니다. 설치부터 `Pipeline`  정리까지 진행을 해보고 관련 내용을 블로그로 정리하려고 합니다.  

`Jenkins` 설치과 초기 설정을 시작으로 아래 내용을 진행하도록 하겠습니다.

- [Jenkins] 1.설치와 초기 설정
- [Jenkins] 2.Pipeline 개념 및 Declarative Pipeline (sections)
- [Jenkins] 3.Declarative Pipeline (Directives, Sequential Stages)
- [Jenkins] 4.Multiproject Pipeline 적용해보기



## jenkins란

* 소프트웨어 개발시 지속적 통합(continuous integration) 서비스를 제공하는 도구입니다.
* 다수의 개발자들이 하나의 프로그램을 개발할 때 버전 충돌을 방지하기 위해 각자 작업한 내용을 공유 영역에 있는 Git, Svn 등의 저장소에 빈번히 업로드함으로써 지속적 통합이 가능하도록 지원합니다.
* 비슷한 CI/CD 도구로는 `Travis CI`, `CircleCI`, `TeamCity`, `Buildkite`, `GitLab Runner` 가 있습니다.
* MIT 라이선스입니다.

> 지속적 통합이란?
> 지속적으로 품질 관리(Quality Control)를 적용하는 프로세스를 실행하는 것입니다. 지속적인 통합은 모든 개발을 완료한 뒤에 품질 관리를 적용하는 고전적인 방법을 대체하는 방법으로서 소프트웨어의 질적 향상과 소프트웨어를 배포하는데 걸리는 시간을 줄이는 데 초점이 맞추어져 있습니다.

> MIT 라이선스
> 매사추세츠 공과대학교(MIT)를 기원으로 하는 소프트웨어 라이선스 중 가장 대표적인 것입니다. 오픈 소스 여부에 관계없이 재사용을 인정합니다.

그외 자세한 내용은 공식 문서를 확인하였습니다. 



## 설치 요구 사항

- 최소 하드웨어 요구사항 : 
  - 메모리 256MB
  - 1GB의 드라이브 공간(Jenkins를 Docker 컨테이너로 실행하는 경우 최소 10GB가 권장됨)

- 소규모 팀에 권장되는 하드웨어 구성:
  - 메모리 4 GB 이상
  - 50 GB 이상 드라이브 공간

- 공통적인 하드웨어 권장 사항 : 
  - 하드웨어 : 상세 페이지 참고([하드웨어 요구사항](https://www.jenkins.io/doc/book/scaling/hardware-recommendations))

- 소프트웨어 요구 사항 : 
  - Java : 상세 페이지 참고([자바 요구사항](https://www.jenkins.io/doc/administration/requirements/java))
  - Web browser : 상세 페이지 참고([웹 브라우저 요구사항](https://www.jenkins.io/doc/administration/requirements/web-browsers))
  - 윈도우 OS : 상세 페이지 참고([윈도우 OS 요구사항](https://www.jenkins.io/doc/administration/requirements/windows))



## 설치

Jenkins 설치를 하는 방법은 다운로드 받아서 설치하는 방법과 docker를 이용하여 설치를 진행할 수 있습니다.

여기서는 docker를 이용하여 설치를 빠르게 해보도록 하겠습니다.

docker 이미지는 여기서 확인할 수 있습니다. [공식 Jenkins 이미지 확인](https://hub.docker.com/r/jenkins/jenkins/) 



docker pull command를 이용하여 설치를 진행해보도록 하겠습니다.

```
docker pull jenkins/jenkins:jdk11
```



## 실행

다음 명령어를 이용하여서 실행을 해볼 수 있습니다.

```bash
docker run -d 
          --name jenkins 
          -p 8080:8080 -p 50000:50000 
          -v /home/docker_home/config/jenkins:/var/jenkins_home jenkins/jenkins:jdk11
```

옵션에 대한 내용을 살펴보도록 하겠습니다.

- `-d` : 컨테이너를 백그라운드로 실행
- `--name jenkins` : 컨테이너 이름 ex) jenkins
  - 설정한 이름을 통해 `docker start`, `docker stop` 시에 편리하게 이용할 수 있습니다.
- `-p 8080:8080 -p 50000:50000` :  컨테이너의 포트를 포워딩 하는 옵션으로 아래와 같이 옵션을 적용하면 컨테이너의 8080 포트를 로컬의 8080 포트로, 컨테이너의 50000 포트를 로컬의 50000 포르토 포워딩한다. ex) 도커 호스트의 포트:컨테이너 내부 포트
- `--restart always` : 서버에서 죽는 경우 재시작할 수 있도록 설정, 로컬에서 개발 시 docker desktop 실행하면 container 실행
- `-v /home/docker_home/config/jenkins:/var/jenkins:z` : 로컬과 컨테이너 간의 볼륨을 설정하기 위해 사용, 로컬 파일 시스템의 특정 경로를 컨테이너의 파일 시스템에 특정 경로로 마운트 시킨다. ex) 보여질 호스트 폴더:컨테이너의 폴더
- `jenkins/jenkins:jdk11` : 해당 도커 이미지 실행



### 결과

위의 명령어를 `jenkins-start.sh` 파일로 만들어서 실행하였습니다. 아래는 그 결과입니다.

```bash
./jenkins-start.sh
Unable to find image 'jenkins/jenkins:jdk11' locally
jdk11: Pulling from jenkins/jenkins
e756f3fdd6a3: Pull complete
74f7dac6952f: Pull complete
9a2ebc31ba4f: Pull complete
2c459ac7d1c4: Pull complete
4b6d6e008a50: Pull complete
a84a9101eb10: Pull complete
471b62b48b08: Pull complete
b20f897fbce6: Pull complete
001db2c42b2b: Pull complete
ededef3056bd: Pull complete
5c14529b2ede: Pull complete
779c1c3cbe3e: Pull complete
dd4cd22988ac: Pull complete
3e32c6899267: Pull complete
Digest: sha256:49b0dad346f2dabdd2c8caadd6197e584530a4e742d33c2d0448d3ae16622eb6
Status: Downloaded newer image for jenkins/jenkins:jdk11
d510f621eab945c8c48fb49424f04f466d89be17bd7a58bee84f309421d8e4a0
```

- 해당 docker image 로컬에 없기 때문에 새로 다운을 받습니다.



## 접속

- Jenkins의 기본 포트는 8080에서 수신하도록 설정되어 있습니다. 
- 기본 설정을 하기 위해서는 화면 접속을 통해서 진행할 수 있습니다.
  - URL : `http://localhost:8080`



### 초기 패스워드 확인

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
17d626a6789747479ec8c1e1901bbfcb
```

- 초기 패스워드를 이용해서 로그인 진행합니다.



### 플러그인 설치 

- 필요한 플러그인만 설치하기 위해서 플러그인은 모두 선택을 하지 않았습니다.



### 계정생성

- 계정 ID는 `admin` 으로 해서 생성해야 합니다.



## 정리

- jenkins는 오픈소스로 소프트웨어 개발시 지속적 통합(continuous integration) 서비스를 제공하는 도구입니다.
- docker를 이용하여 쉽게 설치할 수 있습니다.
- 초기 패스워드는 서버에서 확인해야 합니다.



다음 블로그에서는 `Pipeline` 의 개념과 `Jenkinsfile` 작성을 위한 작성 문법에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Jenkins 공식 문서](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu)