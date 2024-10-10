# Configurando cluster kubernetes v1.29.1 Ubuntu Server v24.04

### Configurações mínimas das VMs
2 VMs
- Uma será o control plane e as outras serão worker nodes
1. 8GB de Ram
1. 2 Núcleos
1. 48GB de HD

```
sudo su
```

```
printf "\n192.168.4.220 k8sappsmaster01\n192.168.4.224 k8sappsworker01\n\n" >> /etc/hosts
```

```
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
```

```
modprobe overlay
modprobe br_netfilter
```

```
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf
sysctl --system
```

```
cp containerd-1.7.13-linux-amd64.tar.gz /tmp/
tar Cxzvf /usr/local /tmp/containerd-1.7.13-linux-amd64.tar.gz
```

```
cp containerd.service /etc/systemd/system/
```
```
systemctl daemon-reload
systemctl enable --now containerd
```

```
cp runc.amd64 /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
```

```
cp cni-plugins-linux-amd64-v1.4.0.tgz -P /tmp/
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.4.0.tgz
```

```
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml   <<<<<<<<<<< manually edit and change SystemdCgroup to true (not systemd_cgroup)

vi /etc/containerd/config.toml
systemctl restart containerd
```

```
swapoff -a  <<<<<<<< just disable it in /etc/fstab instead
```

```
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
```

```
mkdir -p -m 755 /etc/apt/keyrings
```
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```
apt-get update
reboot
```

```
sudo su

apt-get install -y kubelet=1.29.1-1.1 kubeadm=1.29.1-1.1 kubectl=1.29.1-1.1
apt-mark hold kubelet kubeadm kubectl
```

# Verificar se o swap config está em 0

```
free -m
```

# Configurar node control plane
Somente no node que será o control plane do cluster executar os comandos:
```
kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.29.1 --node-name k8s-apps-master-01
```
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```
# Adicionar Calico 3.27.2 CNI 
```
kubectl create -f tigera-operator.yaml
```
```
vi custom-resources.yaml <<<<<< edit the CIDR for pods if its custom
kubectl apply -f custom-resources.yaml
```
# Adicionar NODE no cluster
Executar no control plane para gerar o token para adicionar novos NODES
```
kubeadm token create --print-join-command
```
Executar o token nos NODES que serão adicionados no cluster
