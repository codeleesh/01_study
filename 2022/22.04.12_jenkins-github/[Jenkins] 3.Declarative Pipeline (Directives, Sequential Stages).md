# [Jenkins] 3.Declarative Pipeline (Directives, Sequential Stages)

프로젝트 진행하면서 내부 CI/CD 도구로 jenkins를 사용하기로 하였습니다. 설치부터 `Pipeline` 을 정리까지 진행을 해보고 관련 내용을 블로그로 정리하려고 합니다.  

해당 블로그에서는 `Jenkinsfile` 작성을 위한 작성 문법에 대해서 알아보도록 하겠습니다.

작성 문법은 내용이 많아서 두 개의 블로그로 나누었으며, 이전 블로그에서는 `sections` 를 다루었으며, 해당 블로그에서는 `Directives`, `Sequential Stages` 에 대해서 알아보겠습니다.

- [Jenkins] 1.설치와 초기 설정
- [Jenkins] 2.Pipeline 개념 및 Declarative Pipeline (sections)
- [Jenkins] 3.Declarative Pipeline (Directives, Sequential Stages)
- [Jenkins] 4.Multiproject Pipeline 적용해보기



지난 블로그부터 선언형 파이프라인의 문법에 대해서 정리하고 있습니다. `Diectives` 부터 알아보도록 하겠습니다.



## Directives

지시문에 대해서 알아보도록 하겠습니다. 지시문은 다음의 목록들이 존재합니다.

* environment
* options
* parameters
* triggers
* Jenkins cron syntax
* stage
* tools
* input
* when



### environment

* `pipeline` 또는 `stage` 에서 사용할 수 있습니다.
* 최상위 `pipeline` 블록에 사용되는 환경 변수는 `pipeline` 내의 모든 단계에 적용됩니다.
* `stage` 내 선언한 환경 변수는 `stage` 내에서만 사용 가능합니다.
* Jenkins 환경에서 미리 정의된 식별자를 사용할 수 있도록 특별한 메소드인 `credentials()` 제공합니다. 
	* Secret Text, Secret File, Username and password, SSH with Private Key 등을 지원합니다.
* 사용할 수 있는 환경 변수는 다음과 같습니다.
	* Jenkins environment variables
	* environment variables
	* environment variables dynamically



#### Jenkins environment variables

* Jenkins에서 기본적으로 제공해주는 환경 변수입니다.
* 목록은 다음과 같습니다.
	* BUILD_ID
		* 현재의 Build id, BUILD_NUMBER의 식별값
	* BUILD_NUMBER
		* 현재의 Build number
	* BUILD_TAG
		* ${JOB_NAME}-${BUILD_NUMBER}로 구성. 리소스 파일 또는 jar 파일 등을 넣어서 쉽게 식별 가능
	* BUILD_URL
		* 빌드의 결과를 나타내는 URL
	* EXECUTOR_NUMBER
		* 이 빌드를 수행하는 동일한 컴퓨터의 실행자 중 현재 실행자를 식별하는 고유 번호
	* JAVA_HOME
		* JDK의 HOME
	* JENKINS_URL
		* Jenkins의 Full URL
	* NODE_NAME
		* 현재 빌드가 실행 중인 노드의 이름
	* WORKSPACE
		* 작업 공간의 절대 경로

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Current ${env.WORKSPACE}"
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                echo "Result ${env.BUILD_URL}""
            }
        }
    }
}
```



#### environment variables

* `pipeline` 내에 `environment` 를 활용하여서 환경 변수를 설정할 수 있습니다.
* `pipeline` 내에 선언하였다면 전역변수로 활용 가능합니다.
```groovy
pipeline {
    agent any
    environment {
        SSH = 'user@localhost'
        GRADLE_HOME = '${GRADLE_HOME}'
        BUILD_TARGET_HOME = '${BUILD_HOME}'
        APP_ADMIN = 'app-admin'
        DOMAIN_MODULE = 'domain-module'
    }
...
```



#### environment variables dynamically

* 환경 변수는 shell script로부터 런타임에 설정할 수 있습니다.



### options

`options` 는 `pipeline` 과 `stage` 로 나뉘어집니다.



### pipeline options

* `pipeline` 에서만 사용할 수 있는 옵션은 다음과 같습니다.
	* buildDiscarder
	* checkoutToSubdirectory
	* disableConcurrentBuilds
	* disableResume
	* newContainerPerStage
	* overrideIndexTriggers
	* preserveStashes
	* quietPeriod
	* retry
	* skipDefaultCheckout
	* skipStagesAfterUnstable
	* timeout
	* timestamp
	* parallelsAlwaysFailFast



#### buildDiscarder

* 빌드 및 아티팩트의 날짜와 수를 기준으로 `logRotator` 를 설정할 수 있습니다. 
* 오래된 빌드 및 아티팩트를 삭제하는데 사용하기도 합니다.
```
pipeline {
    agent any
    options {
        buildDiscarder logRotator(
            artifactDaysToKeepStr: "30",
            artifactNumToKeepStr: "100",
            daysToKeepStr: "60",
            numToKeepStr: "200"
        )
    }
    stages {
        stage("Test") {
            steps {
                echo "Hello World!"
            }
        }
    }
}
```
- 아티팩트를 30일동안 100개를 보관하고, 빌드는 60일동안 200개를 보관



#### checkoutToSubdirectory

* `checkout scm` 명령의 디렉토리를 파이프라인 작업 공간의 하위 디렉토리로 변경하는 작업을 수행합니다.
```
options {
	checkoutToSubdirectory('foo')
}
```



#### disableConcurrentBuilds

* 파이프라인 동시 실행을 허용하지 않습니다.
* 공유 리소스에 대한 동시 액세스를 방지하는데 유용할 수 있습니다.
```groovy
options {
	disableConcurrentBuilds()
}
```



#### disableResume

* 컨트롤러(Jenkins 마스터)가 다시 시작되면 빌드 재개 기능을 비활성화합니다.
```groovy
options {
	disableResume()
}
```



#### newContainerPerStage

* 도커 또는 도커 파일 최상위 에이전트에 사용됩니다. 
* 작업 단계를 실행하는 일반적인 동작은 모든 단계가 하나의 컨테이너 내에서 실행됩니다.
* 이 옵션을 지정하면 각 단계가 동일한 노드의 별도 컨테이너에서 실행됩니다.



#### overrideIndexTriggers

* 분기 인덱싱 트리거의 기본 처리를 재정의할 수 있습니다. 다중 분기 또는 조직 레이블에서 분기 인덱싱 트리거를 사용하지 않도록 설정한 경우 옵션 {overrideIndexTrigger(true)}은(는) 이 작업에 대해서만 사용하도록 설정합니다. 그렇지 않은 경우 옵션 {overrideIndexTrigger(false)}은(는) 이 작업에 대해서만 분기 인덱싱 트리거를 사용하지 않도록 설정합니다.
```
options { 
	overrideIndexTriggers(true) 
}
```



#### preserveStashes

* 숨김 파일을 보존하는데 사용합니다.
* 작업을 빌드하는 동안 일부 파일이 생성될 수 있으며 생성된 파일은 빌드가 끝나면 제거됩니다.
* 일부 시나리오에서는 특정 단계에서 빌드를 다시 시작해야 하는 경우도 있습니다.
* 이러한 경우 이전 빌드 중에 생성된 임시 파일이 필요할수 있어서 해당 기능을 사용합니다.
```groovy
options { 
  preserveStashes(buildCount: 5) 
}
```



#### quietPeriod

* 특정 기간동안 빌드를 완료하는데 사용합니다.
* 새 빌드가 대기열에 배치되고 이 옵션으로 설정된 기간동안 시작하지 않습니다.
```groovy
options { 
  quietPeriod(30) 
}
```



#### retry

* 실패 시 전체 파이프라인을 지정된 횟수만큼 재시도합니다.
```
options {
	retry(3)
}
```



#### skipDefaultCheckout

* 에이전트 기본적으로 소스 코드 체크 아웃을 건너뜁니다.
* `checkout scm` 명령을 사용하여 수동으로 체크아웃해야 합니다.
```
options {
	skipDefaultCheckout()
}
```



#### skipStagesAfterUnstable

* 빌드 상태가 `UNSTABLE` 이 되면 추가 단계를 건너뜁니다.
* 전체 파이프라인을 중지하는 데 사용됩니다.
```
options {
	skipStagesAfterUnstable()
}
```



#### timeout

* 파이프라인 실행에 대한 시간 초과 기간을 설정합니다.
* 기존적으로 파이프라인 실행에는 시간 제한이 없습니다.
* 이 기간이 지나면 Jenkins는 `FloxInterruptedException` 을 발생하고 파이프라인을 중단합니다.
* 파이프라인 중단 설정은 다음과 같이 할 수 있습니다.
	* 전역 실행 시간
	* stage 실행 시간
```
options {
	timeout(time: 1, unit: 'HOURS')
}
```



#### timestamps

- 파이프라인 실행에 의해 생성된 모든 콘솔 출력에 대해서 타임스탬프 시간을 추가합니다.
```groovy
options { 
	timestamps() 
}
```



#### parallelsAlwaysFailFast

- 병렬이 포함된 단계에 failtFast true를 추가하여 병렬 단계 중 하나가 실패하면 병렬 단게가 모두 중단되도록 강제할 수 있는 옵션입니다.
```groovy
options { 
	parallelsAlwaysFailFast() 
}
```



### stage options

* `stage` 에서 사용할 수 있는 옵션은 다음과 같습니다.
* pipeline options과 내용은 동일합니다.
	* skipDefaultCheckout
	* timeout
	* retry
	* timestamps



### parameter

* 매개변수 지시문은 사용자가 파이프라인을 트리거할 때 제공해야 하는 매개변수 목록을 제공합니다. 이러한 사용자 지정 매개변수의 값은 매개변수 개체를 통해 파이프라인 단계에서 사용할 수 있습니다.특정 용도는 매개변수, 선언 파이프라인을 참조하십시오.
* 이용할 수 있는 `parameter` 는 다음과 같습니다.
	* string
	* text
	* booleanParam
	* choice
	* password
- 아래는 모든 파라미터에 대한 예제를 나타냅니다.
```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```



### triggers

* 트리거 지시문은 파이프라인을 다시 트리거하는 자동화된 방법을 정의합니다. GitHub 또는 BitBucket과 같은 소스와 통합된 파이프라인의 경우 웹 후크 기반 통합이 이미 존재할 가능성이 높기 때문에 트리거가 필요하지 않을 수 있습니다. 현재 사용할 수 있는 트리거는 `cron`, `pollSCM` 및 `upstream` 입니다.
* 다음을 이용할 수 있습니다.
	* cron
	* pollSCM
	* upstream



#### cron

* 크론 스타일 문자열을 사용하여 파이프라인을 다시 트리거할 정규 간격을 정의합니다.
```groovy
triggers { 
  cron('H */4 * * * 1-5')
}
```



#### pollSCM

* 크론 스타일 문자열을 사용하여 Jenkins가 새 소스 변경을 확인해야 하는 일정한 간격을 정의합니다. 새 변경사항이 있으면 파이프라인이 다시 트리거됩니다.
```groovy
triggers { 
  pollSCM('H */4 * 1-5') 
}
```
- `pollSCM` 트리거 기능은 Jenkins 2.22 버전 이후부터 사용 가능합니다.



#### upstream

* 쉼표로 구분된 작업 문자열과 임계값을 허용합니다. 문자열의 작업이 최소 임계값으로 완료되면 파이프라인이 다시 트리거됩니다.
```groovy
triggers { 
  upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) 
}
```



### Jenkins cron syntax

* 젠킨스 크론 구문은 크론 유틸리티의 구문을 따릅니다. 특히, 각 행은 TAB 또는 공백으로 구분된 5개의 필드로 구성됩니다.

| MINUTE                         | HOUR                       | DOM                         | MONTH            | DAT                                                 |
| ------------------------------ | -------------------------- | --------------------------- | ---------------- | --------------------------------------------------- |
| Minutes within the hour (0–59) | The hour of the day (0–23) | The day of the month (1–31) | The month (1–12) | The day of the week (0–7) where 0 and 7 are Sunday. |

한 필드에 여러 값을 지정하려면 다음 연산자를 사용할 수 있습니다. 우선 순서에 따라,
* `*` 모든 유효한 값을 지정합니다.
* `M-N` 값의 범위를 지정합니다.
* `M-N/X` or `*/X` 지정된 범위 또는 전체 유효 범위를 통해 'X' 간격으로 단계
* `A,B,…,Z` 여러 값을 열거합니다.



### String 보간

* 애플리케이션의 이름이 반복된다면 상수로 관리하여서 재사용이 가능합니다. 그 외 상수가 필요하다면 비슷한 내용으로 작성이 가능합니다.
* 사용은 다음과 같이할 수 있습니다.
```groovy
def APPLICATION = 'api-app'
echo 'Build Start ${APPLICATION}'
echo "Build End ${APPLICATION}"
```
* 결과
```shell
Build Start ${APPLICATION}
Build End api-app
```

* 상수를 사용하기 위해서는 `' '`가 아니라 `" "` 를 사용해야 합니다.



### stage

* `stages` 안에서 사용해야 합니다.
* `stage` 에는 하나 이상의 DSL  `step` 이 들어 있습니다.
	* `step` 구간은 필수로 존재해야 합니다.

```groovy
stage('Build') {
  steps {
    // 실제 수행하는 스크립트 또는 명령어로 구성
  }
}
stage('Test') {
  steps {
    // 실제 수행하는 스크립트 또는 명령어로 구성
  }
}
stage('Deploy') {
  steps {
    // 실제 수행하는 스크립트 또는 명령어로 구성
  }
}
```



### tools

* `path` 에 자동 설치 및 설정하기 위한 도구를 정의할 수 있습니다.
* `agent none` 으로 설정했다면 tools를 활성화 시킬 노드나 에이전트가 없기 때문에 동작하지 않습니다.
* 해당 옵션을 사용하기 위해서는 `Manage Jenkins -> Global Tool Configuration` 에서 환경변수를 지정해야 합니다.
* 지원하는 tools는 다음과 같습니다.
	* maven, jdk, gradle, git, node 등이 존재합니다.
```groovy
tools {
  maven 'apache-maven-3.0.1'
}
```



### input

* 스테이지의 입력 지시문을 사용하면 입력 단계를 사용하여 입력 프롬프트를 표시할 수 있습니다. 
* 옵션이 적용된 후 해당 단계에 대한 에이전트 블록에 들어가거나 단계의 when 조건을 평가하기 전에 일시 중지됩니다. 입력이 승인되면 단계가 계속됩니다. 입력 제출의 일부로 제공된 모든 매개변수는 나머지 단계의 환경에서 사용할 수 있습니다.
* 사용할 수 있는 옵션은 다음과 같습니다.
	* message
		* 필수값. 사용자가 입력을 제출할 때 이 정보가 표시됩니다.
	* id
		* 이 입력에 대한 선택적 식별자입니다. 기본값은 스테이지 이름입니다.
	* ok
		* 입력 양식의 "확인" 단추에 대한 선택적 텍스트입니다.
	* submitter
		* 이 입력을 제출할 수 있는 사용자 또는 외부 그룹 이름의 쉼표로 구분된 목록입니다. 기본적으로 모든 사용자를 허용합니다.
	* submitterParameter
		* 제출자 이름(있는 경우)과 함께 설정할 환경 변수의 선택적 이름입니다.
	* parameters
		* 제출자에게 제공하라는 메시지를 표시하는 매개변수의 선택적 목록입니다. 자세한 내용은 매개 변수를 참조하십시오.



### when

* 주어진 조건에 따라 `stage` 실행해야 하는지 여부를 결정할 수 있습니다.
* 조건이 하나 이상 포함되어야 합니다.
* 조건문의 상태는 다음과 같습니다.
	* `allOf`
		* 지시문이 둘 이상의 조건을 포함하는 경우 단계가 실행되기 위해서는 모든 조건이 참인 경우 실행합니다.
	* `anyOf`
		* 지시문이 둘 이상의 조건을 포함하는 경우 단계가 실행되기 위해서는 하나의 조건이 참인 경우 실행합니다.
	* `not`
		* 조건문이 참이 아닌 경우 실행합니다.
* `when` 을 이용해서 사용할 수 있는 기본 옵션들은 다음과 같습니다.
	* branch
	* buildingTag
	* changelog
	* changset
	* changeRequest
	* environment
	* equals
	* expression
	* tag
	* not
	* allOf
	* anyOf
	* triggeredBy 등



#### branch

* `master` 브랜치일때 단계가 실행됩니다.
```groovy
when {
  branch 'master'
}
```
* 빌드 중인 브랜치가 `branch` 패턴과 일치하는 경우에 단계를 실행합니다.
```groovy
when {
  branch pattern: "release-\\d+", comparator: "REGEXP"
}
```
- 단순한 문자열 비교 및 정규 표현식 활용하여 추가가 가능합니다.



#### buildingTag

* 빌드가 태그를 빌드해야할 때 해당 옵션을 이용합니다.
```groovy
when {
  buildingTag()
} 
```



#### changelog

* 커밋 로그 중 지정된 정규식 패턴에 해당되는 경우 단계를 실행합니다.
```groovy
when {
  changelog '.*^\\[DEPENDENCY\\].+$'
}
```



#### changeset

* *실제 프로젝트에서 자주 사용할 수 있는 옵션입니다.*
* 변경된 파일이 변경 집합에 지정된 패턴과 일치하는 경우 단계를 실행합니다.
```groovy
when {
  changeset "app-api/**/*"
}
```
- app-api 하위 모듈에 변경이 일어나면 단계를 실행합니다.

```groovy
when {
  changeset pattern: ".TEST\\.java", comparator: "REGEXP"
}
```
- 문자열 `TEST` 를 포함하는 java 파일에 대해서 변경이 일어나면 단계를 실행합니다.



#### changeRequest

* 빌드가 `PR`(Pull Request)인 경우 단계를 실행합니다.
```groovy
when {
  changeRequest()
}
```



#### environment

* 지정된 환경 변수가 주어진 값으로 설정된 경우 단계를 실행합니다.
```groovy
when {
  environment name : "DEPLOY_TO", value : "production"
}
```



#### equals

* 예상 값이 실제 값과 같을 때 단계를 실행합니다.
```groovy
when {
  equals expected: 2, actual: currentBuild.number
}
```



#### expression

* 지정된 조건식 true일 때 단계를 실행합니다.
```groovy
when {
  expression {
    return params.DEBUG_BUILD
  }
}
```
* 문자열을 반환할 때는 boolean으로 변환하거나 null을 반환하여 false로 처리해야 합니다. 단순히 문자열을 "0" 또는 "false"로 반환한다면 true로 인식합니다.



#### tag

* TAG_NAME 변수가 지정된 패턴과 일치하는 경우 단계를 실행합니다.
```groovy
when {
  tag "release-*"
}
```



#### not

* 조건문이 참이 아닌 경우 실행합니다.
```groovy
when { 
  not { 
    branch 'master' 
  } 
}
```



#### allOf

* 지시문이 둘 이상의 조건을 포함하는 경우 단계가 실행되기 위해서는 모든 조건이 참인 경우 실행합니다.
```groovy
when { 
	allOf { 
		branch 'master'; environment name: 'DEPLOY_TO', value: 'production' 
	} 
}
```
- 브랜치가 `master` 이면서 디플로이 환경이 `production` 인 경우 단계 실행합니다.



#### anyOf

* 지시문이 둘 이상의 조건을 포함하는 경우 단계가 실행되기 위해서는 하나의 조건이 참인 경우 실행합니다.
```groovy
when { 
	anyOf { 
		branch 'master'; branch 'staging' 
	} 
}
```
- 브랜치가 `master` 또는 `staging` 인 경우 단계 실행합니다.



#### triggeredBy

* 주어진 매개 변수에 의해 현재 빌드가 트리거되면 단계를 실행합니다.
```groovy
when { triggeredBy 'SCMTrigger' }
when { triggeredBy 'TimerTrigger' }
when { triggeredBy 'BuildUpstreamCause' }
when { triggeredBy cause: "UserIdCause", detail: "vlinde" }
```



### Evaluating `when` before entering `agent` in a `stage`

- 기본적으로 단계의 when 조건은 정의된 경우 해당 단계의 에이전트를 입력한 후 평가됩니다. 그러나 when 블록 내에서 beforeAgent 옵션을 지정하여 변경할 수 있습니다. beforeAgent가 true로 설정되면 when 조건이 먼저 평가되고 when 조건이 true로 평가될 때만 에이전트가 입력됩니다.



### Evaluating `when` before the `input` directive

- 기본적으로 스테이지의 when 조건은 정의된 경우 입력 전에 평가되지 않습니다. 그러나 when 블록 내에서 beforeInput 옵션을 지정하여 변경할 수 있습니다. beforeInput이 true로 설정되면 when 조건이 먼저 평가되고 when 조건이 true로 평가되는 경우에만 입력이 입력됩니다.
- beforeInput true는 beforeAgent true보다 우선합니다.



### Evaluating `when` before the `options` directive

- 기본적으로 단계의 when 조건은 정의된 경우 해당 단계에 대한 옵션을 입력한 후 평가됩니다. 그러나 when 블록 내에서 beforeOptions 옵션을 지정하여 변경할 수 있습니다. beforeOptions가 true로 설정되면 when 조건이 먼저 평가되고 when 조건이 true로 평가될 때만 옵션이 입력됩니다.

- beforeOptions true는 beforeInput true 및 beforeAgent true보다 우선합니다.



## Sequential Stages

- 선언적 파이프라인의 `stages` 에서는 순차적으로 실행할 중첩된 단계 목록이 포함될 수 있습니다.
- 하나의  `stage` 에는 `steps`, `stages`, `parallel`, `matrix` 중 하나만 있어야 합니다.



### parallel

- 선언적 파이프라인의 `stages` 에서는 병렬로 실행할 중첩된 단계 목록이 포함된 병렬 섹션이 있을 수 있습니다.
- 하나의  `stage` 에는 `steps`, `stages`, `parallel`, `matrix` 중 하나만 있어야 합니다.
* 병렬이 포함된 단계에 failFast true를 추가하여 병렬 단계 중 하나가 실패하면 병렬 단계가 모두 중단되도록 강제할 수 있습니다. 
* failfast를 추가하는 또 다른 옵션은 파이프라인 정의에 parallelsAlwaysFailFast() 옵션을 추가하는 것입니다.
```groovy
pipeline {
    agent any
    options {
        parallelsAlwaysFailFast()
    }
stages {
...
}
```
* 병렬로 `stage` 를 실행하는 예시입니다.
```groovy
parallel {
  stage('build-service1') {
    steps {
      //
    }
  }
  stage('build-service2') {
    steps {
      //
    }
  }
  stage('build-service3') {
    steps {
      //
    }
  }
}
```
- service1, service2, service3에 대해서 순차적으로 실행하는 것이 아닌, 병렬로 수행하여서 실행시간을 단축할 수 있습니다.



## 정리

- `jenkins` 는 환경변수를 여러가지 방법으로 설정할 수 있도록 제공합니다.
- `step` 구간은 필수로 존재해야 합니다.
- `when` 은 주어진 조건에 따라 `stage` 실행해야 하는지 여부를 결정할 수 있습니다. 특히 `changeset` 은 자주 사용하는 옵션입니다. 
- `parallel` 을 이용하여 병렬적으로 단계를 실행할 수 있습니다.



다음 블로그에서는 Gradle Multiproject에서 Jenkinsfile을 이용하여서 pipeline 설정을 진행해보도록 하겠습니다.



## 참고
- [Jenkins Tutorial — Part 6 — Pipeline Options | by Saeid Bostandoust | ITNEXT](https://itnext.io/jenkins-tutorial-part-6-pipeline-options-5ccd05035aaf)
- [Pipeline](https://www.jenkins.io/doc/book/pipeline/)
