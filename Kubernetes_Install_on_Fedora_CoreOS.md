## Install Kubernetes on Fedora CoreOS

If IPTables is active, a br_netfilter Kernel module is required for Kubernetes installation.
With this module, packages traveling over the bridge can be processed by iptables for filtering and port forwarding.kubernetes pords will be able to communicate with each other in the cluster.
For this reason, the module is activated as follows;

    modprobe overlay
    modprobe br_netfilter

As a requirement for your Linux Node’s iptables to correctly see bridged traffic, 
you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

```bash
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

SELinux is turned off on all servers;

```bash
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```
Rebbot Machine

    reboot

**Install Container linux**  with [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Install CNI plugins (required for most pod network):
```bash
CNI_VERSION="v0.8.2"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```
Install crictl (required for kubeadm / Kubelet Container Runtime Interface (CRI))
```bash
Interface (CRI))
CRICTL_VERSION="v1.16.0"
mkdir -p /opt/bin
curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" | tar -C /opt/bin -xz
```

Install kubeadm, kubelet, kubectl and add a kubelet systemd service:
```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"
cd /usr/local/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}
```
```bash
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/usr/local/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/usr/local/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
Enable and start kubelet:
```bash
systemctl enable --now kubelet
```

Configure cgroup driver used by kubelet on control-plane node

    KUBELET_EXTRA_ARGS=--cgroup-driver=systemd

Restarting the kubelet is required:
```bash
systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet
```

the kubelet logs are checked
```bash
journalctl -xeu kubelet | tail -20f
```
Docker service restarts
```bash
systemctl enable docker.service
systemctl start docker.service
systemctl status docker.service
```
The following method is recommended because of the error received on "lex-volume-plugin-dir" volume on Fedora CoreOS.

    vi kubeadm-custom.yaml
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.17.4
controllerManager:
  extraArgs:
    flex-volume-plugin-dir: "/etc/kubernetes/kubelet-plugins/volume/exec"
networking:
  podSubnet: 192.168.0.0/16
  ```

With the command below, the kubernetes cluster is initialized;
```bash
sudo kubeadm init --config kubeadm-custom.yaml
```
Following these operations, the following commands are executed in order to be able to use Kubernetes commands;
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
After this process, we are ready for calico network configuration. For this, we can use the following command;

```bash
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

## SSH
**Master Server**

 1. Create rsa key
 
`ssh-keygen -t rsa`

`cat ~/.ssh/id_rsa.pub`

`ssh-rsa  AAAAB3NzaC1yc2U0= root@master01.local`

 2. add hostname and ip on hosts file
 
 `vi /etc/hosts`
 
 `10.10.10.10 master01.local`
 
 `10.10.10.11 worker01.local`

**Worker Server**

3.  add master `id_rsa.pub` to worker `authorized_keys`

    `vi ~/.ssh/authorized_keys`
    
    `ssh-rsa AAAAB3NzaC1yc2U0= root@master01.local`
    
4. Test SSH

`sudo -i ssh root@10.10.10`
