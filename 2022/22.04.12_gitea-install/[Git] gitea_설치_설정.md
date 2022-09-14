# [Git] gitea 설치



gitea 설치는 다음 문서를 참고하여 설치하였습니다.

https://docs.gitea.io/en-us/install-with-docker/



docker-compose를 이용하여 설치

```yaml
version: "3"

networks:
  gitea:
    external: false

volumes:
  gitea:
    driver: local

services:
  server:
    image: gitea/gitea:1.16.6
    container_name: gitea
    restart: always
    networks:
      - gitea
    volumes:
      - gitea_home:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "18081:3000"
      - "222:22"
```

