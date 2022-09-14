# [Kubernetes] minikube install

![minikube](./images/minikube.png)



이전 회사에서 MSA 방향성을 토대로 프로그램을 개발하였습니다. 기존 Monolithic 아키텍처에 익숙한 입장에서 불편한 느낌은 지울수가 없었습니다. 하지만 MSA 환경 구축을 위해서 k8s 환경에서 개발을 진행하다보니 장점이 많았습니다. 물론 조직의 규모가 작거나 서비스가 작다면 MSA는 맞지 않을 수 있습니다. 

집에서 MSA 환경을 구축 및 실습하기 위해서 Kubernetes 를 설치하려고 하였습니다. 하지만 Kubernetes 환경을 구축하기 위해서는 마스터와 각 노드의 구조를 갖춰야 하지만 이는 로컬 환경에서 갖추기 위해서는 과하다는 생각이 많이 들었습니다. 그래서 찾아보니 `Minikube` 라는 것이 있었습니다.  `Minikube` 는 로컬 환경에서 Kubernetes 환경을 쉽게 설치가 가능하며 테스트하기에도 좋습니다. 집에서 클러스터 구성을 연습하기 좋아보여서 설치를 진행하게 되었습니다.

설치를 하면서 한번에 되지는 않았습니다. 문제가 된 부분들에 대해서도 설치할때 참고를 위해서 모두 다 작성하였습니다.



## 사전 확인

- CPU 2개 이상

- 2GB의 여유 메모리

- 20GB의 디스크 여유 공간

- 인터넷 연결

- 컨테이너 또는 가상 머신 관리자(예: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox 또는 VMware Fusion/Workstation)
- Docker 설치
  - minikube에서는 minikube를 설치 및 사용하기 위한 환경으로 Docker를 가장 추천합니다.



## Docker

먼저 설치가능한 리스트를 최신으로 업데이트합니다.

````bash
$ apt-get update -y
````



Docker 설치를 위해서는 다음 항목이 필수로 설치되어 있어야합니다.

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common -y
```

- `apt-transport-https` : 패키지 관리자가 https를 통해 데이터 및 패키지에 접근할 수 있도록 합니다.
- `ca-certificates` : ca-certificate는 certificate authority에서 발행되는 디지털 서명으로, SSL 인증서의 PEM 파일이 포함되어 있어 SSL 기반 앱이 SSL 연결이 되어있는지 확인할 수 있습니다.
- `curl` : 특정 웹사이트에서 데이터를 다운로드 받을 때 사용합니다.
- `software-properties-common` : 개인 패키지 저장소를 추가하거나 제거할 때 사용합니다.



Docker의 공식 구글 클라우드의 공개 키(`GPG key`)를 추가합니다.

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

- `f` : HTTP 요청 헤더의 contentType을 multipart/form-data로 전송합니다.
- `s` : 진행 과정이나 에러 정보를 보여주지 않는다.(–silent)
- `S` : SSL 인증과 관련있다고 들었는데, 정확히 아시는 분 있다면 댓글 부탁!
- `L` : 서버에서 301, 302 응답이 오면 redirection URL로 이동합니다.
- `apt-key` : apt가 패키지를 인증할 때 사용하는 키 리스트를 관리하며, 이 키를 사용해 인증된 패키지는 신뢰할 수 있는 것으로 간주합니다. add 명령어는 키 리스트에 새로운 키를 추가하겠다는 의미입니다.



Docker를 설치하기 위한 레포지토리를 추가합니다.

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```



Docker 엔진과 containerd 를 설치합니다.

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```



Docker 설치가 잘되었는지 확인을 위해서 version을 조회해봅니다.

```bash
$ docker --version
```

Docker version은 다음과 같습니다.

```
Docker version 20.10.17, build 100c701
```



## Minikube

`Minikube` 설치 파일 바이너리를 다운로드 받습니다.

```
$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```



다운로드 받은 바이너리 파일의 경로를 지정해줘서 설치를 마무리합니다.

```
$ cp minikube-linux-amd64 /usr/local/bin/minikube
```



설치한 `Minikube` 가 정상적으로 설치되었는지 버전을 확인해봅니다.

```
$ minikube version
```

`Minikube` version은 다음과 같습니다.

```bash
minikube version: v1.26.0
commit: f4b412861bb746be73053c9f6d2895f12cf78565
```



## Kubectl

`Kubernetes` 는 커맨드 라인 도구인 `kubectl` 을 사용하면 쿠버네티스 클러스터에 대해 명령을 실행할 수 있습니다.

apt 패키지 인덱스 업데이트 및 `Kubernetes` 저장소를 사용하는 데 필요한 패키지 설치합니다.

```bash
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
```



구글 클라우드의 공개 키(`GPG key`)를 추가합니다.

```bash
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```



`Kubernetes` apt 저장소를 추가합니다.

```bash
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```



apt 패키지 인덱스 업데이트 및 kubectl 설치합니다.

```
$ sudo apt-get update && sudo apt-get install -y kubectl
```



## Troubleshooting

- kubectl 명령어 정상 실행하지 않는 경우
- Minikuce start 되지 않는 경우 - `crictl: 명령이 없습니다`
- Minikuce start 되지 않는 경우 - `Exiting due to GUEST_START`



### kubectl 명령어 정상 실행하지 않는 경우

#### 원인 

kubectl 실행시 다음과 같은 에러를 만난다면 아래 내용을 참고해서 해결할 수 있습니다

```bash
The connection to the server 192.168.0.39:8443 was refused - did you specify the right host or port?
```

#### 해결방법

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



#### 원인

kubectl 실행시 다음과 같은 에러를 만난다면 아래 내용을 참고해서 해결할 수 있습니다

```bash
error: error loading config file "/home/lovethefeel/.kube/config": read /home/lovethefeel/.kube/config: is a directory
```

#### 해결방법

```bash
$ chmod 666 admin.conf
```



### Minikuce start 되지 않는 경우 - crictl

minikube 설치시 다음과 같은 에러를 만나다면 아래 내용을 참고해서 해결할 수 있습니다.

```
😄  Ubuntu 20.04 의 minikube v1.26.0
✨  기존 프로필에 기반하여 none 드라이버를 사용하는 중
👍  minikube 클러스터의 minikube 컨트롤 플레인 노드를 시작하는 중
🔄  Restarting existing none bare metal machine for "minikube" ...
ℹ️  OS release is Ubuntu 20.04.3 LTS

❌  Exiting due to RUNTIME_ENABLE: Temporary Error: sudo crictl version: exit status 1
stdout:

stderr:
sudo: crictl: 명령이 없습니다


╭───────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                           │
│    😿  If the above advice does not help, please let us know:                             │
│    👉  https://github.com/kubernetes/minikube/issues/new/choose                           │
│                                                                                           │
│    Please run `minikube logs --file=logs.txt` and attach logs.txt to the GitHub issue.    │
│                                                                                           │
╰───────────────────────────────────────────────────────────────────────────────────────────╯
```

Kubernetes가 컨테이너 런타임으로써의 docker 지원 중단을 발표하였고 1.20 버전부터 docker를 런타임으로 사용 할 수 없다는 경고가 표시되고 있습니다. 지원 중단을 발표한 버전 1.20 버전부터는 deprecation되며 1.22 버전부터는 deprecated 된다고 발표 되었습니다. 그래서 docker 대신 crictl를 사용할 수 있습니다.



#### 해결책

`crictl` 설치 파일 다운로드 및 설치를 진행합니다.

```bash
$ VERSION="v1.24.1"
$ wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
$ sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
```

설치한 `crictl` 가 정상적으로 설치되었는지 버전을 확인해봅니다.

```bash
$ crictl --version
```

`crictl` 의 version 정보는 다음과 같습니다.

```bash
Version:  0.1.0
RuntimeName:  docker
RuntimeVersion:  20.10.17
RuntimeApiVersion:  1.41.0
```



### Minikuce start 되지 않는 경우 - docker group

`Minikube` 설치시 다음과 같은 에러를 만난다면 docker 그룹을 추가하면 됩니다.

```
😄  Ubuntu 20.04 의 minikube v1.26.0
✨  기존 프로필에 기반하여 none 드라이버를 사용하는 중
👍  minikube 클러스터의 minikube 컨트롤 플레인 노드를 시작하는 중
🔄  Restarting existing none bare metal machine for "minikube" ...
ℹ️  OS release is Ubuntu 20.04.3 LTS

❌  Exiting due to GUEST_START: Failed to check container runtime version: docker version --format : exit status 1
stdout:


stderr:
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied


╭───────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                           │
│    😿  If the above advice does not help, please let us know:                             │
│    👉  https://github.com/kubernetes/minikube/issues/new/choose                           │
│                                                                                           │
│    Please run `minikube logs --file=logs.txt` and attach logs.txt to the GitHub issue.    │
│                                                                                           │
╰───────────────────────────────────────────────────────────────────────────────────────────╯
```

root가 아닌 새로운 유저를 만들고(아니면 기존 유저에다가) 도커가 그 유저에서 관리되도록 설정해주어야 합니다.



#### 해결책

Docker 그룹의 사용자를 추가합니다.

```bash
$ sudo usermod -a -G docker $USER
```



서버 재기동을 합니다.

```bash
$ reboot
```



현재 사용자가 Docker 그룹의 포함되었는지 확인해봅니다.

```bash
$ id
uid=1000(lovethefeel) gid=1000(lovethefeel) 그룹들=1000(lovethefeel),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),132(lxd),133(sambashare),136(docker)
```



## minikube start

`Minikube` 설치를 해보도록 하겠습니다.

```bash
$ minikube start
```



`Minikube` 정상적으로 설치가 되었으면 다음과 같은 로그가 나오게 됩니다.

```bash
😄  Ubuntu 20.04 의 minikube v1.26.0
✨  기존 프로필에 기반하여 none 드라이버를 사용하는 중
👍  minikube 클러스터의 minikube 컨트롤 플레인 노드를 시작하는 중
🔄  Restarting existing none bare metal machine for "minikube" ...
ℹ️  OS release is Ubuntu 20.04.3 LTS
🐳  쿠버네티스 v1.24.1 을 Docker 20.10.17 런타임으로 설치하는 중
    ▪ kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
    > kubectl.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubelet.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 2.16 MiB / 42.31 MiB [->_________________________] 5.09%     > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 4.37 MiB / 42.31 MiB [-->_______________________] 10.34%     > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 6.06 MiB / 42.31 MiB [--->______________________] 14.33%     > kubectl: 990.79 KiB / 43.59 MiB [>________________________] 2.22%     > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 7.25 MiB / 42.31 MiB [-->___________] 17.13% 8.49 MiB p/s    > kubectl: 2.00 MiB / 43.59 MiB [->_________________________] 4.59%     > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 8.44 MiB / 42.31 MiB [-->___________] 19.94% 8.49 MiB p/s    > kubectl: 2.97 MiB / 43.59 MiB [->_________________________] 6.81%     > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 9.66 MiB / 42.31 MiB [--->__________] 22.82% 8.49 MiB p/s    > kubectl: 3.87 MiB / 43.59 MiB [->_____________] 8.89% 4.84 MiB p/s    > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 11.00 MiB / 42.31 MiB [--->_________] 25.99% 8.34 MiB p/s    > kubectl: 4.81 MiB / 43.59 MiB [->____________] 11.04% 4.84 MiB p/s    > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 12.22 MiB / 42.31 MiB [--->_________] 28.87% 8.34 MiB p/s    > kubectl: 5.78 MiB / 43.59 MiB [->____________] 13.26% 4.84 MiB p/s    > kubelet: 46.80 KiB / 110.96 MiB [>________________________] 0.04%     > kubeadm: 13.47 MiB / 42.31 MiB [---->________] 31.83%     > kubeadm: 42.31 MiB / 42.31 MiB [--------------] 100.00% 6.69 MiB p/s 6.5sB p/s ETA 7s
    > kubectl: 43.59 MiB / 43.59 MiB [--------------] 100.00% 5.07 MiB p/s 8.8s
    > kubelet: 110.96 MiB / 110.96 MiB [-------------] 100.00% 6.16 MiB p/s 18s
```



`Minikube` 가 정상적으로 설치가 되었는지 status를 통해서 확인할 수 있습니다.

```bash
$ minikube status
```

결과는 다음과 같습니다.

```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```



## 정리

- k8s 환경을 minikube 를 이용하여 로컬 환경에서 쉽게 구축할 수 있습니다.
- k8s 에서는 docker 를 컨테이너 런타임으로 사용하지 않습니다.



## 참고

- [Kubernetes 공식 홈페이지](https://kubernetes.io/ko/)

- [minikube github](https://github.com/kubernetes/minikube)

- [minikube install](https://minikube.sigs.k8s.io/docs/start/)

- [kubectl docs](https://kubernetes.io/ko/docs/reference/kubectl/kubectl/)

- [kubernetes-stop-docker](https://kubernetes.io/ko/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

- [Removing dockershim from kubelet](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2221-remove-dockershim)
