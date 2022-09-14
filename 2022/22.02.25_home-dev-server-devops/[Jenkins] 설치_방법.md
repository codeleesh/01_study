# Jenkins

- Java로 빌드된 오픈소스로, `CI`(Continuos Integration) 및 `CD`(Continuos Delivery) 도구
- 소프트웨어 프로젝트를 빌드, 테스트 및 배포허기 위해서 사용
- DevOps 개발 도구의 가장 기본적인 도구



참고한 URL 공식 문서를 확인하였습니다.

[Jenkins 공식 문서](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu)



## 설치 환경

- OS : Ubuntu 20.04.3 LTS
  

## 설치 요구 사항

최소 하드웨어 요구사항 : 

- 메모리 256MB

- 1GB의 드라이브 공간(Jenkins를 Docker 컨테이너로 실행하는 경우 최소 10GB가 권장됨)

소규모 팀에 권장되는 하드웨어 구성:

- 메모리 4 GB 이상
- 50 GB 이상 드라이브 공간

공통적인 하드웨어 권장 사항 : 

- 하드웨어 : 상세 페이지 참고([하드웨어 요구사항](https://www.jenkins.io/doc/book/scaling/hardware-recommendations))

소프트웨어 요구 사항 : 

- Java : 상세 페이지 참고([자바 요구사항](https://www.jenkins.io/doc/administration/requirements/java))
- Web browser : 상세 페이지 참고([웹 브라우저 요구사항](https://www.jenkins.io/doc/administration/requirements/web-browsers))
- 윈도우 OS : 상세 페이지 참고([윈도우 OS 요구사항](https://www.jenkins.io/doc/administration/requirements/windows))



## 설치 방법

### Linux

#### curl 설치

```
sudo apt  install curl
```



```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```



```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```



```
sudo apt-get update
```



```
sudo apt-get install jenkins
```



#### 접속

Jenkins의 기본 포트는 8080에서 수신하도록 설정되어 있습니다. 

기본 설정을 하기 위해서는 화면 접속을 통해서 진행할 수 있습니다.

URL : `http://localhost:8080`



#### 초기 패스워드 확인

```
lovethefeel@lovethefeel-H310H5-M7:/var/lib/jenkins$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
c26ed7b8083d4dd9819598e3b98a35e1
```



### Docker

#### Jenkins 다운로드 및 실행

- 사용할 권장 Docker 이미지는 공식 jenkins/jenkins 이미지 : https://hub.docker.com/r/jenkins/jenkins/

- Jenkins와 Docker의 모든 기능을 사용하려면 아래에 설명된 설치 프로세스를 진행하는 것을 추천

