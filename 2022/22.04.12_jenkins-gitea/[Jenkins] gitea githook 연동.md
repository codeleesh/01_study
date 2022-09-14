# [Jenkins] gitea githook 연동

회사에서 진행하고 있는 프로젝트 중에서 Jenkins와 Gitea를 연동하여 CI/CD를 도입하는 부분이 있어서 해당 내용을 정리하면서 기록하기 위해서 남깁니다.



## Githook이란?

Git 저장소에 push된 commit에 의해 jenkins에서 빌드를 자동으로 트리거하는 메커니즘입니다.

변경이 감지 될 때만 새로운 빌드를 시도하도록 지시합니다.



## 필요 환경

- Gitea 설치
- Jenkins 설치
  - Gitea Plugin 설치



## 환경 설정

### Jenkins 

Manage Jenkins -> Configure System -> Gitea Servers

- Gitea 서버 정보 입력

- Manage hooks는 선택하지 않을 것



### Gitea

- 계정 생성
- 해당 Repo의 추가한 계정을 공동 작업자로 등록
- 해당 Repo의 쓰기 권한 부여



## 빌드 프로젝트 설정

Item -> Gitea Organization

- Gitea Organization
  - Owner
  - Behaviours
    - Discover branches
      - Only branches that are not also filed as PRs
    - Discover pull reuqests from origin
      - Merging the pull request with the current target branch revision
    - Discover pull requests from forks
      - Merging the pull request with the current target branch revision
      - Contributors



Item -> Source Code Mangement

- Git
- URL 입력
- credentials 설정
- branch 설정



Build Triggers 

- check Poll SCM
  - 이 설정은 기본적으로 Jenkins에게 webhook을 통해 요청한 경우에만 Gitea repo를 폴링하도록 지시



## Webhook

Gitea 

repo -> Settings -> Webhooks

- Add Webhook
  - Gitea
- URL
  - ex) https://jenkins.d4science.org/gitea-webhook/post?job=project
- POST Content Type
  - `application/json`

- Trigger On

  - `Push Events` 

- 주의

  - 대소문자 주의
  - POST 요청을 위해선 HTTPS 필요

  ```
  curl -vvv -H "Content-Type: application/json" -H "X-Gitea-Event: push" -X POST http://jenkins.d4science.org/gitea-webhook/post -d "{}"
  ```

  