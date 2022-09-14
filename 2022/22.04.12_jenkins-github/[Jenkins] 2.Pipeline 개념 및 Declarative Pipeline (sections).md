# [Jenkins] 2.Pipeline 개념 및 Declarative Pipeline (~sections)

프로젝트 진행하면서 내부 CI/CD 도구로 jenkins를 사용하기로 하였습니다. 설치부터 `Pipeline` 을 정리까지 진행을 해보고 관련 내용을 블로그로 정리하려고 합니다.  

해당 블로그에서는  `Pipeline` 의 개념과 `Jenkinsfile` 작성을 위한 작성 문법에 대해서 알아보도록 하겠습니다.

작성 문법은 내용이 많아서 두 개의 블로그로 나누었으며, 해당 블로그에서는 `sections` 를 다루며, 다음 블로그에서는 `Directives`, `Sequential Stages` 에 대해서 알아보겠습니다.

- [Jenkins] 1.설치와 초기 설정
- [Jenkins] 2.Pipeline 개념 및 Declarative Pipeline (sections)
- [Jenkins] 3.Declarative Pipeline (Directives, Sequential Stages)
- [Jenkins] 4.Multiproject Pipeline 적용해보기



## pipeline이란

* 소프트웨어의 대한 모든 변경은 릴리즈되는 과정에서 프로세스를 거쳐야 합니다. 프로세스는 다음으로 가정을 합니다.
	* 단위/인수/통합 테스트 수행
	* 빌드 수행
	* 배포 수행(feat. 재시작)
* 위 프로세스를 수동으로 한다면 처음 몇 번은 어렵지 않게 해낼 수 있습니다. 하지만 소프트웨어 변경 작업은 더욱 많아질 것이고 그럴때마다 반복적인 수행을 해야 합니다.
* 반복적인 수행을 수동으로 한다면 휴먼에러가 발생할 여지가 충분히 있습니다.
* `pipeline` 을 통해 여러 단계의 테스트 및 빌드, 그리고 배포를 진행할 수 있으며, 안정적인 반복 작업이 가능합니다.



## 왜 필요할까요?

* 모든 브랜치의 대한 `PR` (Pull Request)의 대해서 `pipeline` 빌드 프로세스를 생성하기 때문에 쉽게 배포할 수 있습니다.
* `pipeline` 프로세스 실행 시 로그 추적이 가능하여서 문제가 생겼을때 확인이 수월합니다.



## 사용 방법

* `pipeline` 을 사용하기 위해서는 프로젝트의 저장소(feat. Git, Svn)의 `Jenkinsfile` 파일이 작성되어야 합니다.
* `Jenkinsfile` 을 작성하기 위해서 선언형과 스크립트형 파이프라인으로 2가지를 사용할 수 있습니다.
* 여기서는 선언형 파이프라인을 기준으로 설명하겠습니다.



## Pipeline Concepts

Jenkins Pipeline의 주요 항목은 다음과 같습니다.

* Pipeline
* Node
* Stage
* Step

각 항목에 대한 상세 내용을 알아보도록 하겠습니다.



### pipeline

* 애플리케이션의 빌드, 테스트 및 배포 단계를 포함하는 전체 빌드 프로세스를 정의합니다.
* `pipeline` 의 블록은 선언형 파이프라인 구문의 핵심 부분입니다.
* `Jenkinsfile` 의 시작 지점에 선언해야 합니다.

```groovy
pipeline {
  // 세부 작성 필요
}
```



### node

* `node` 의 블록은 스크립트형 파이프라인 구문의 핵심 부분입니다.
* `Jenkinsfile` 의 시작 지점에 선언해야 합니다.
* 스크립트형 파이프라인 구문은 여기서는 알아보지 않도록 하겠습니다.

```groovy
node {
	// 세부 작성 필요
}
```



### stage

* `stage` 블록은 전체 파이프라인 단계를 통해 수행되는 작업의 하위 집합을 나타냅니다.
	* 예를 들어, jenkins를 통해서 테스트 -> 빌드 -> 배포 -> 서버재시작을 수행해야 한다고 가정을 해보겠습니다.
	* 그렇다면, 테스트, 빌드, 배포, 서버재시작이 각각 하나의 `stage` 가 됩니다.
* 전체 파이프라인의 흐름에서 특정한 시점에 실행이 필요한 것들을 묶을 수 있습니다.
* 아래 `Step` 예시와 같이 설명하도록 하겠습니다.



### step

* 단일 작업을 나타내며, 특정 시점에서 수행할 작업을 알려줍니다.

```groovy
stage('build-application-admin') {
  steps {
    echo 'Build Start "${APP_ADMIN}"'
    sh './gradlew --gradle-user-home ${GRADLE_HOME} ${APP_ADMIN}:clea ${APP_ADMIN}:build -x test'
    echo 'Build End "${APP_ADMIN}"'
  }
}
```

- admin 어플리케이션의 빌드를 담당하는 하나의 `stage` 입니다.

* 하나의 `stage` 안에 배포하는 작업을 담당하고 있는 `step` 에서 수행할 작업을 알려줍니다.



## 선언형 파이프라인(Declarative Pipeline) 문법

* 선언형 파이프라인을 이용해서 `Jenkinsfile` 을 작성하기 위한 문법에 대해서 알아보겠습니다.
* `Groovy` 구문과 동일한 규칙을 따르지만 몇 가지 예외 사항이 있습니다.
	* 시작은 다음과 같이 해야 합니다. : `pipeline { }`
	* 명령문 구분 기호로 세미콜론이 없습니다.
	* 블록은 `sections`,  `directives`,  `stpes` 등 할당문으로 구성되어야 합니다.
	* 속성 참조문은 인자값이 없습니다. : `input()`



## sections

* `sections` 은 `directives` 와  `steps` 를 1개 이상 포함하는 코드로 구성되어 있습니다.
  * `directives` 의 

* `sections` 의 포함된 항목은 다음과 같습니다.
  * agent
  * post
  * stages
  * steps



### agent

* `pipeline` 또는 `stage`에서 조건부로 사용할 수 있는 하나의 구간입니다.
* 선언형 파이프라인 구문에서 사용 가능합니다.
* 사용할 수 있는 종류는 다음과 같습니다.
	* `any`
	* `none`
	* `label`
	* `docker`
	* `kubernetes`



#### any

* 사용 가능한 agent에서 `pipeline` 또는 `stage` 에 상관없이 실행할 수 있습니다.
```groovy
agent any
```



#### none

* `pipeline` 블록의 최상위 레벨에 적용되는 경우 전체 `pipeline` 의 실행에 전역 `agent` 가 할당되지 않으면, 각 단계마다 `agent`를 포함하고 있어야 합니다.
* 활용해볼 수 있는 부분으로는 여러(ex: linux, windows) 서버에서 `pipeline` 또는 `stage` 단계를 진행해야 되는 경우 해당 기능을 사용하면 됩니다.
```groovy
agent none
```



#### label

* 제공된 label이 있는 환경에서 사용 가능합니다.
```groovy
agent { label 'my-defined-label' }
```
- `pipeline` 또는 `stage` 가 'my-defined-label' 이라는 레이블을 가진 에이전트에서 실행될 수 있다는 의미입니다.



#### docker

* Docker Registry 사용 및 Dockerfile 사용 가능합니다.
```groovy
agent {
  docker {     
    image 'maven:3.8.1-adoptopenjdk-11'
    label 'my-defined-label'
    args '-v /tmp:/tmp'
  } 
}
```



#### kubernetes

* k8s 클러스터에 배포된 파드 내부에서 pipeline을 실행합니다.
```groovy
agent {
  kubernetes {
    defaultContainer ''
    yaml '''
    // k8s 내용 작성
         '''
  }
}
```



### post

* `pipeline` 또는 `stage` 에서 조건부로 사용할 수 있는 하나의 구간입니다.
* Job의 빌드 후처리 동작에 대해서 상세 설정을 할 수 있습니다.
* `post`에서 제공하는 옵션들은 다음과 같습니다.
	* always
	* changed
	* fixed
	* regression
	* aborted
	* failure
	* success
	* unstable
	* unsuccessful
	* cleanup



#### always

* `pipreline` 또는 `stage` 의 실행 완료 상태에 상관없이 실행 후 무조건 사후 단계를 실행합니다.
* 다음과 같이 작성하여서 테스트해볼 수 있습니다.
```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
    }
}
```



#### changed

* 현재 `pipreline` 또는 `stage` 의 실행이 이전 실행과 다른 완료 상태를 가진 경우에만 단계를 사후 단계를 실행합니다.



#### fixed

* 현재 `pipreline` 또는 `stage` 의 실행이 성공하였고 이전 실행이 실패했거나 불안정한 상태인 경우 사후 단계를 실행합니다.



#### regression

* 현재 `pipreline` 또는 `stage` 의 실행이 실패 또는 불안정 또는 중단이고 이전 실행이 성공한 경우 사후 단계를 실행합니다.



#### aborted

* 현재 `pipreline` 또는 `stage` 의 실행이 `aborted` 상태인 경우 사후 단계를 실행합니다.



#### failure

* 현재 `pipreline` 또는 `stage` 의 실행이 `failed` 상태인 경우 사후 단계를 실행합니다.



#### success

* 현재 `pipreline` 또는 `stage` 의 실행이 `success`  상태인 경우 사후 단계를 실행합니다.



#### unstable

* 현재 `pipreline` 또는 `stage` 의 실행이 `unstable` 상태인 경우 사후 단계를 실행합니다.



#### unsuccessful

* 현재 `pipreline` 또는 `stage` 의 실행이 성공 상태가 아닌 경우 사후 단계를 실행합니다.



#### cleanup

* 현재 `pipreline` 또는 `stage` 의 실행이 상태에 관계없이 다른 모든 post 조건이 진행된 후 단계를 실행합니다.



### stages

* `pipeline` 또는 `stage` 에서 조건부로 사용할 수 있는 하나의 구간입니다.
* 하나 이상의 `stage`를 포함합니다.
* 선언형과 스크립트 파이프라인 모두에서  `step` 를 정의합니다.
* 예를 들어, order, member, produce 어플리케이션이 있다면, 하나의 stage의 세부 stage로 Build, Backup & Copy, Deploy 등을 수행할 수 있습니다.



### steps

* `pipeline` 또는 `stage` 에서 조건부로 사용할 수 있는 하나의 구간입니다.
* 지정된 `stage` 에서 실행할 하나 이상의 `step` 를 정의합니다.
* 단일 작업을 나타냅니다.
* 예를 들어, 세부 stage 중  order 어플리케이션 빌드 수행 또는 디플로이 등을 수행할 수 있습니다.



## 정리

- `Pipeline` 을 이용하면 단위/인수/통합 테스트 수행, 빌드, 배포 프로세스를 쉽게 구성할 수 있습니다.
- 선언형 파이프라인인 `pipeline` 은 선언형 파이프라인 구문의 핵심 부분이며, 시작 지점에 반드시 선언해야 합니다.
- `stage` 블록은 전체 파이프라인 단계를 통해 수행되는 작업의 하위 집합이며, 전체 파이프라인의 흐름에서 특정한 시점에 실행이 필요한 것들을 묶을 수 있습니다.
- `step`  블록은 단일 작업을 수행하며, 하나의 `stage` 안에 배포하는 작업을 담당하고 있는 `step` 에서 수행할 작업을 알려줍니다.
- `post` 는 빌드 후처리 동작에 대해서 상세 설정을 할 수 있습니다.



다음 블로그에서는  `Jenkinsfile` 작성을 위한 작성 문법(`Directives`, `Sequential Stages`) 대해서 알아보도록 하겠습니다.



## 참고
- [Jenkins Tutorial — Part 6 — Pipeline Options | by Saeid Bostandoust | ITNEXT](https://itnext.io/jenkins-tutorial-part-6-pipeline-options-5ccd05035aaf)
- [Pipeline](https://www.jenkins.io/doc/book/pipeline/)
