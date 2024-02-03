---
title: "쿠버플로우 설치"
description: "쿠버플로우 설치법과 간단한 서술의 내용입니다."
date: 2022-10-20T04:46:35+09:00
lastmod: 2022-12-06T12:29:59+09:00
categories: ["도커", "리눅스", "서버"]
tags: ["docker", "kubeflow", "kubernetes", "ML-Ops"]
image: "thum.png"
draft: true
---

## 서론

### 개요

쿠버플로우, 큐베플로우, 쿠브플로우.. 이것을 배포해야 하는 일이 있었습니다.  
하지만 추가 패키지를 이용하지 않고 K8s로 직접 설치하는 자료는 적었으며 따라해도 오류가 발생했습니다.

그도 그럴것이 쿠버플로우는 계속해서 업데이트가 되고 있으며, 운영체제, 쿠버네티스, 도커를 포함한 여러 패키지의 업데이트에도 민감하기 때문에 문제가 계속 생기는 것 같습니다.

이번 포스트에선 쿠버플로우에 대한 설명을 따로 하진 않을 것이며,  
Clone이 가능한 가상머신을 기준, GPU가 없는 것을 예제로 설명해 보도록 하겠습니다.  
해당 포스트를 이해하셨다면 서버에 직접 설치하는건 문제 없을거라 예상합니다.

### 고려사항

다음은 이유는 파악 못했으나, 문제 원인을 찾아낸 것들의 내용입니다. **2022-12** 기준입니다.

 - 우분투 22.04 환경 쿠버네티스 설치에서 인증 메니저 노드가 설치 안되는 것을 확인했습니다. 20.04는 문제가 없었습니다.
 - Readme에서는 쿠버플로우 1.20 이상 버전을 권장했으나 1.21.5-00 버전이 아니면 설치 문제가 발생했습니다.
 - 마스터, 워커노드 모두 사양을 크게 잡아먹었습니다. 각 노드 모두 16GB이상의 메모리가 아니면 문제가 생겼습니다.
 - 종종 kubeflow manifests의Install with a single command 방법이 종료되지 않는 문제를 겪었습니다. 


## 설치

config등을 수정, 추가하는 부분은 위와 아래의 글을 참고해주세요

### 공통 사전 설치

```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
도커, 도커 컴포즈를 설치합니다. 버전이 문제가 없다면 다른 방식으로 설치해도 무방할 것 같습니다.


su--
```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```
suend--

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo usermod -aG docker ${USER}
newgrp docker
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
sudo systemctl daemon-reload
sudo systemctl enable docker

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

sudo apt-get install -y kubelet=1.21.5-00 kubeadm=1.21.5-00 kubectl=1.21.5-00 --allow-downgrades --allow-change-held-packages
sudo apt-mark hold kubelet kubeadm kubectl
```
도커 권한 설정을 하고 쿠버네티스를 설치합니다.

su--
```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
suend--

```bash
sudo sysctl --system
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

이 이후로는 해당 가상 머신을 Clone 1회 또는 2회하여 마스터 노드와 워커노드로 나눕니다.  
호스트 네임과 아이피 주소에 주의하세요, 호스트 네임같은 동일할 경우 문제가 발생합니다.


### 마스터 노드 설치-조인 전

```bash
sudo kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.6/install/kubernetes/quick-install.yaml
kubectl get pods -n kube-system --selector=k8s-app=cilium
```
쿠버네티스를 초기화합니다. 이후 출력되는 명령어를 이용하여 워커노드를 조인 시킵니다. (sudo)

### 마스터 노드 설치-조인 후

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
sudo apt install nfs-common
sudo apt install nfs-kernel-server
sudo mkdir /nfsroot
sudo chown nobody:nogroup /nfsroot
sudo chmod 777 /nfsroot
```

해당 과정은 NFS Client Provisioner를쿠버네티스가 사용할 수 있게 구축하고  
사용할 NFS Server도 마스터 노드에 올리는 과정을 설명하고 있습니다.

sudo nano /etc/exports 로 아래 내용 추가

```bash
/nfsroot 192.168.80.110(rw,insecure,sync,no_root_squash,no_subtree_check)
/nfsroot 192.168.80.111(rw,insecure,sync,no_root_squash,no_subtree_check)
```

해당 과정을 통하여, 존재하는 모든 노드에 NFS 공간을 할당하고 있습니다.  
쿠버플로우에서 NFS는 의무가 아니지만 없을경우 가변적으로, 쿠버플로우가 볼륨을 잡기 때문에  
비추천 하고 있습니다. 

```bash
sudo service nfs-kernel-server restart
sudo systemctl enable rpcbind
sudo systemctl enable nfs-server
sudo systemctl start rpcbind
sudo systemctl start nfs-server
sudo exportfs -a

curl https://raw.githubusercontent.com/helm/helm/release-2.16/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
helm repo add raphael https://raphaelmonrouzeau.github.io/charts/repository/
helm repo update
helm install nfs-provisioner \
--set nfs.server=192.168.80.110 \
--set nfs.path=/nfsroot \
--set storageClass.defaultClass=true \
--set storageClass.name=nfs-provisioner \
raphael/nfs-server-provisioner

kubectl apply -f https://raw.githubusercontent.com/mojokb/handson-kubeflow/master/registry/kubeflow-registry-deploy.yaml
kubectl apply -f https://raw.githubusercontent.com/mojokb/handson-kubeflow/master/registry/kubeflow-registry-svc.yaml
```
**아이피 주소 192.168.80.110, 192.168.80.111은 각각 마스터, 워커의 주소입니다.** 환경에 맞춰서 사용해 주셔야 합니다.  
쿠버플로우의 경우 nfs를 사용하지 않을 시 가변용량 저장소를 이용하기 때문에 사양문제가 발생한다고 합니다.

### 마스터, 워커 공통 설치 진행

*/etc/docker/daemon.json*에 추가
```bash
"insecure-registries": [
    "kubeflow-registry.default.svc.cluster.local:30000"
  ]
```

그리고 재시작합니다.

```bash
sudo systemctl restart docker
```

*/etc/hosts*에 추가(이 부분은 미리 추가해도 됨)

```bash
192.168.80.110 kubeflow-registry.default.svc.cluster.local
```

### 워커노드에서 nfs 확인

```bash
curl kubeflow-registry.default.svc.cluster.local:30000/v2/_catalog
```

### 마스터 노드에서의 쿠버플로우 설치

```bash
wget https://github.com/kubernetes-sigs/kustomize/releases/download/v3.2.0/kustomize_3.2.0_linux_amd64
mv kustomize_3.2.0_linux_amd64 kustomize
chmod 777 kustomize
sudo mv kustomize /usr/local/bin

git clone https://github.com/kubeflow/manifests.git
cd manifests
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

후에 
```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```
로 user@example.com/12341234을 사용하여 로컬상의 테스트가 가능합니다.  
또는 
```bash
kubectl port-forward --address 0.0.0.0 svc/istio-ingressgateway -n istio-system 8080:80
```
로 외부노출을 허용시킬 수 있습니다.

## 참고 자료들

만약에 정상작동하지 않을 경우 아래의 자료를 확인해주세요.

### 설치법

https://github.com/kubeflow/manifests  
https://learning-sarah.tistory.com/entry/Kubeflow-13-%EC%84%A4%EC%B9%98  
https://jwher.github.io/posts/install-kubeflow/  
https://lsjsj92.tistory.com/580  
https://yooloo.tistory.com/229  
https://velog.io/@ehddnr/kubernetes-kubeflow-%EC%84%A4%EC%B9%98-feat.-GCP  
https://losskatsu.github.io/it-infra/mlops01/#  
https://velog.io/@csk6124/Kubeflow-1.4-%EC%84%A4%EC%B9%98  
https://velog.io/@dev_halo/%EC%95%84%EB%AC%B4%EB%8F%84-%EC%95%88%EC%95%8C%EB%A0%A4%EC%A3%BC%EB%8A%94-on-premise-Kubeflow-%EA%B5%AC%EC%B6%95%ED%95%B4%EB%B3%B4%EC%9E%90

### 트러블 슈팅

https://hackmd.io/@maelvls/debug-cert-manager-webhook  
https://github.com/calebhailey/homelab/issues/3#issuecomment-569543391  
https://github.com/kubeflow/manifests/issues/2086  
https://mlops-for-all.github.io/docs/setup-components/install-components-kf/
