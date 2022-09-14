# Docker



## 설치 환경

- OS : Ubuntu 20.04.3 LTS



## 설치 요구 사항

>  참고 - 서버 OS 요구사항
>
> https://docs.docker.com/engine/install/#server

Ubuntu 를 통해서 설치를 진행해보도록 하겠습니다.

Ubuntu에서 도커 엔진을 설치하기 위해서는 64 bit 버전의 Ubuntu이어야 합니다.

- Ubuntu Impish 21.10
- Ubuntu Hirsute 21.04
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

도커 엔진은 다음을 지원합니다.

- x86_64
- amd64
- armhf
- arm64
- s390x



## 오래된 버전 삭제

오래된 버전의 도커 삭제

```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```



## 저장 드라이버 지원

Ubuntu의 Docker 엔진은 스토리지 드라이버를 지원

- `overlay2`

- `aufs` 

- `btrfs` 

Docker 엔진은 기본적으로 'overlay2' 스토리지 드라이버를 사용합니다. 대신 `aufs`를 사용해야 하는 경우 수동으로 구성

[use the AUFS storage driver](https://docs.docker.com/storage/storagedriver/aufs-driver/)



## 설치 방법

도커 엔진을 설치하는 여러가지 방법 제공

- 대부분의 사용자는 도커 레파지토리를 설정하여서 설치하며, 추천하는 접근 방법
  - [set up Docker’s repositories](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

- 약간의 사용자는 DEB package 다운로드하여 설치하며, 인터넷에 액세스할 수 없는 상황에서 설치
  - [install it manually](https://docs.docker.com/engine/install/ubuntu/#install-from-a-package)
- 일부 사용자는 테스트 및 개발 환경에서 자동화된 편의성 스크립트를 사용하여 Docker를 설치
  - [convenience scripts](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)



여기서는 추천하는 방법인 도커 레파지토리를 설정하여 설치하는 방법을 추천

### Install using the repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.



#### Set up the repository

1. Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:

   ```
   $ sudo apt-get update
   
   $ sudo apt-get install \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

2. Add Docker’s official GPG key:

   ```
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

3. Use the following command to set up the **stable** repository. To add the **nightly** or **test** repository, add the word `nightly` or `test` (or both) after the word `stable` in the commands below. [Learn about **nightly** and **test** channels](https://docs.docker.com/engine/install/).

   ```
   $ echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```



#### Install Docker Engine

1. Update the `apt` package index, and install the *latest version* of Docker Engine and containerd, or go to the next step to install a specific version:

   ```
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

   > Got multiple Docker repositories?
   >
   > If you have multiple Docker repositories enabled, installing or updating without specifying a version in the `apt-get install` or `apt-get update` command always installs the highest possible version, which may not be appropriate for your stability needs.

2. To install a *specific version* of Docker Engine, list the available versions in the repo, then select and install:

   a. List the versions available in your repo:

   ```
   $ apt-cache madison docker-ce
   
     docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
     docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
   ```

   b. Install a specific version using the version string from the second column, for example, `5:18.09.1~3-0~ubuntu-xenial`.

   ```
   $ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
   ```

3. Verify that Docker Engine is installed correctly by running the `hello-world` image.

   ```
   $ sudo docker run hello-world
   ```

   This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The `docker` group is created but no users are added to it. You need to use `sudo` to run Docker commands. Continue to [Linux postinstall](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands and for other optional configuration steps.

#### Upgrade Docker Engine

To upgrade Docker Engine, first run `sudo apt-get update`, then follow the [installation instructions](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository), choosing the new version you want to install.



### Install from a package

If you cannot use Docker’s repository to install Docker Engine, you can download the `.deb` file for your release and install it manually. You need to download a new file each time you want to upgrade Docker.

1. Go to [`https://download.docker.com/linux/ubuntu/dists/`](https://download.docker.com/linux/ubuntu/dists/), choose your Ubuntu version, then browse to `pool/stable/`, choose `amd64`, `armhf`, `arm64`, or `s390x`, and download the `.deb` file for the Docker Engine version you want to install.

   > **Note**
   >
   > To install a **nightly** or **test** (pre-release) package, change the word `stable` in the above URL to `nightly` or `test`. [Learn about **nightly** and **test** channels](https://docs.docker.com/engine/install/).

2. Install Docker Engine, changing the path below to the path where you downloaded the Docker package.

   ```
   $ sudo dpkg -i /path/to/package.deb
   ```

   The Docker daemon starts automatically.

3. Verify that Docker Engine is installed correctly by running the `hello-world` image.

   ```
   $ sudo docker run hello-world
   ```

   This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The `docker` group is created but no users are added to it. You need to use `sudo` to run Docker commands. Continue to [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-privileged users to run Docker commands and for other optional configuration steps.



