



권한 설정

```bash
chmod 755 ./bin/h2.sh
```



jdk 경로 설정

build.sh

```bash
#!/bin/sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```



실행

```
./h2.sh
```



접속

```
http://localhost:8082
```

