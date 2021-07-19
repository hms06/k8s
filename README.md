## Kubernetes 설치 및 Cluster 생성하여 배포해보기

	# kubeadm, kubelet 및 kubectl 설치
 	sudo apt-get update
 	sudo apt-get install -y apt-transport-https ca-certificates curl
 	sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
 	echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
 	sudo apt-get update
 	sudo apt-get install -y kubelet kubeadm kubectl
 	sudo apt-mark hold kubelet kubeadm kubectl            -> 버전 고정해주기. 패키지가 자동으로 업데이트 하지 못하게 방지함.


	# cluster 생성
	* k-control
	sudo kubeadm init --control-plane-endpoint 192.168.200.50 --pod-network-cidr 192.168.0.0/16 --apiserver-advertise-address 192.168.200.50
	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config    -> 권한주기


## node 추가

	kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
	kubectl get nodes     -> ready 상태인지 확인
	kubeadm token list    -> 토큰 값 확인
	TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
	bkoyxl.3aht0jk9ws0d4nfj   23h         2021-07-07T06:42:57Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   	system:bootstrappers:kubeadm:default-node-token

	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
	>    openssl dgst -sha256 -hex | sed 's/^.* //'
	015ac96083dbcdeece03ff30344df57cf26c0785570975dbeece3c90f0eff824               -> 해쉬값 확인


	# k-node1, 2, 3 가서 토큰과 해쉬값 붙여넣어 명령어 입력
	(ubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>)
	sudo kubeadm join 192.168.200.50:6443 --token bkoyxl.3aht0jk9ws0d4nfj --discovery-token-ca-cert-hash sha256:015ac96083dbcdeece03ff30344df57cf26c0785570975dbeece3c90f0eff824 

	# 다시 k-control 가서 노드 확인
	kubectl get nodes
 
  
  
## 버전 업그레이드
>	apt-get 버전 1.1부터 다음 방법을 사용할 수도 있다.

	apt-get update && \
	apt-get install -y --allow-change-held-packages kubeadm=1.21.x-00  -> 홀드된 패키지를 업그레이드 하겠다는 의미

## 1.18.19 -> 1.18.20 으로 패치만 업그레이드 해보기
>	 컨트롤 플레인 업그레이드
>	 
	# kubeadm 업그레이드
	sudo apt-cache madison kubeadm
	sudo apt-get update && \
	sudo apt-get install -y --allow-change-held-packages kubeadm=1.18.20-00
	kubeadm upgrade plan
	sudo kubeadm upgrade apply v1.18.20

	#kubelet과 kubectl 업그레이드
	sudo apt-get update && \
	sudo apt-get install -y --allow-change-held-packages kubelet=1.18.20-00 kubectl=1.18.20-00
	sudo systemctl daemon-reload
	sudo systemctl restart kubelet


>	노드 업그레이드
>	
	# kubeadm 업그레이드
	sudo apt-get update && \
	sudo apt-get install -y --allow-change-held-packages kubeadm=1.18.20-00
	sudo kubeadm upgrade node

	# kubelet과 kubectl 업그레이드
	sudo apt-get update && \
	sudo apt-get install -y --allow-change-held-packages kubelet=1.18.20-00 kubectl=1.18.20-00
	sudo systemctl daemon-reload
	sudo systemctl restart kubelet


	# 컨트롤 플레인에 가서 버전 제대로 올라왔는지 확인
	kubectl get nodes
	kubectl get pods -A인        -> 러닝되는지 확인

  

 ## Metal-LB
 
  	# Manifest를 이용하여 설치해주기
  	kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
  	kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
	
  	# configmap.yaml 파일 작성
	apiVersion: v1
	kind: ConfigMap
	metadata:
  	  namespace: metallb-system
  	  name: config
	data:
  	  config: |
   		address-pools:
    	- name: default
      	  protocol: layer2
      	  addresses:
      	  - 192.168.200.200-192.168.200.210
 
 ## ingress
 

 ## rook
 
 ## metrics-server
	wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml

	vi components.yaml   ->파일에 들어가서
	- --kubelet-insecure-tls  -> Deployment.spec.template.spec에 추가해줘야함
	
	kubectl create -f components.yaml

	# 확인
	kubectl get po -n kube-system
	kubectl describe po -n kube-system metrics-server-766c9b8df-nmwlj

	# 노드&파드 확인해보기
	kubectl top nodes
	kubectl top pods


