# Git Command



### Branch 생성

```
git branch step1
```



### Branch 이동

```
git switch step1
```



### 소스 커밋

```
git commit
```



### 특정 브랜치로 소스 Clone 하기

```
git clone -b codeleesh --single-branch https://github.com/codeleesh/infra-subway-monitoring.git
```



### 원격저장소 등록

```
git remote -v 
```



### 미션저장소 등록

```
git remote add upstream https://github.com/codeleesh/java-racingcar-playground.git
```



### 최신 코드 정보

```
git fetch upstream codeleesh
```



### 

```
git checkout -b step1
```



###

```
git branch -D 삭제한브랜치
```



###

```
git rebase upstream/codeleesh
```

