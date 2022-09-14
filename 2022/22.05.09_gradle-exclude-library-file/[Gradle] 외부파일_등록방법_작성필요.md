# 외부파일(*.jar) 등록 방법



프로젝트안에 폴더 생성

- 해당 폴더 : project/libs
- 해당 폴더에 등록할 jar 파일 이동



build.gradle 설정

- 특정 jar 추가하기

```groovy
implementation files('libs/jar')
```

- 특정 폴더의 모든 jar 추가하기

```groovy
implementation fileTree(dir, 'libs', include: '*.jar')
```

