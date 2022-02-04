# node-kubernetes

![img](https://blog.kakaocdn.net/dn/rzTKn/btrsfWXst7B/EAzuMe8r1WRG9dKw5ooeyK/img.png)![img](https://blog.kakaocdn.net/dn/nD6k5/btrsaTf8XBp/zKkgWKw3br3ZXrP2MKUby0/img.png)

### 목차
1. [들어가며](#들어가며)
2. [사전준비](#사전-준비)   
  2.1. [lima + docker + docker-compose 설치](#lima--docker--docker-compose-설치)   
  2.2. [Lima, minikube 로 kubernetes 개발 환경 구축](#lima-minikube-로-kubernetes-개발-환경-구축) - 작성 중
  
### 들어가며

Node.js Application 을 실제 production 용으로 배포할 시 생각해야 할 것들이 많습니다.

특히 많은 사용자들의 접속으로 인해 서버가 언제 터질지 모르는 상황에서 1대의 서버만 운영하는 것은 위험한 생각일겁니다. 

Auto-Scalable 하다는 것은 요청에 비례하여 서버를 자동으로 Scaling 해준다는 의미입니다. 즉, 요청이 특정 기준을 넘으면 replica를 만들어 분산시키는 것이죠. 

google cloud, aws 에서도 kubernetes cluster를 만들 수 있지만, 여기서는 Local 에서 직접 minicube를 통해 kubernetes cluster를 구축하고, sample node.js application을 배포하는 연습을 해보겠습니다.

kubernetes cluster에 올리기 위해 docker 로 컨테이너를 만들어 올리는 것도 연습해보겠습니다.

### 사전 준비

#### lima + docker + docker-compose 설치

```
brew install lima docker docker-compose
```

저는 Apple Silicon Mac 을 사용 중이므로 lima (Linux VM 을 만들어주는 오픈소스) 를 설치하여 개발환경을 구축하려고 합니다.

중간에 

![img](https://blog.kakaocdn.net/dn/YdPVy/btrscOFghXu/QQgM6mphpdkKSwOYFSVEQ0/img.png)

이런 에러와 함께 lima가 설치 되지 않아서 따로 xcode-select --install 커맨드를 실행시켜 설치 후 다시 진행하니 되었습니다.

![img](https://blog.kakaocdn.net/dn/bxFjOv/btrr9hayscg/whFgHljCwuSNayxxs88Fs0/img.png)

설치 후 limactl 명령어가 정상적으로 작동하면 됩니다.

![img](https://blog.kakaocdn.net/dn/bph1KE/btrsfXhQkX0/pHFDkp2KE8gZqhV7ke4iKk/img.png)

가상 머신을 하나 만들어보겠습니다.

```
limactl start
```

default 로 진행하면 기본 ubuntu 머신이 생깁니다.

![img](https://blog.kakaocdn.net/dn/blthni/btrsfXvmbJO/kiBmD58wM1akTaTk1hK2vK/img.png)

```
lima
```

명령어를 실행해 보면 Ubuntu 쉘로 진입하는 걸 볼 수 있습니다.

제대로 인스턴스가 실행되는 걸 확인하고 나면 기본 vm 인스턴스는 삭제해줍니다.

우리는 lima repository에서 제공해주는 docker 설정을 받아서 생성할겁니다.

```
limactl stop default # 인스턴스 중지
limactl remove defulat # 인스턴스 삭제
```

![img](https://blog.kakaocdn.net/dn/l9CXG/btrr9zCQBHb/ydWUiwdxj30Xowzq7ZZGsK/img.png)

기본 docker config 파일로 vm을 생성하기 위해 docker yaml 파일을 받아옵니다.

```
curl -o default.yaml https://raw.githubusercontent.com/lima-vm/lima/master/examples/docker.yaml
```

![img](https://blog.kakaocdn.net/dn/cGDXzB/btrssCdfD3N/G8Kfejt5kVKPDaYudAesak/img.png)

파일이 제대로 들어온 것을 확인하고 vm을 시작합니다.

```
limactl start default.yaml
# 설치 완료 후
lima docker ps -a  # docker 확인
```

![img](https://blog.kakaocdn.net/dn/MiNir/btrsktvqs0t/xqKjWTL0oat0lTxWHKnKcK/img.png)

docker vm이 제대로 실행된 것을 볼 수 있습니다.

#### Lima, minikube 로 kubernetes 개발 환경 구축
