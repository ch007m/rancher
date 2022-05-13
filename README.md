Table of Contents
=================
* [Rancher](#rancher)
* [References](#references)
* [Architecture](#architecture)
* [Install Rancher](#install-rancher)
* [Tips](#tips)
* [Cleanup](#cleanup)

# Rancher

See official documentation: https://rancher.com/products/rancher

# References

https://blog.bitforger.dev/installing-rancher-on-k3s-without-public-internet-access
https://www.liquidweb.com/kb/how-to-install-and-configure-rancher-k3s/

# Architecture

The Rancher solution is composed of 2 systems: a rancher or k3s server running a supservisor top of akubernetes cluster
and a k3s agent which is deployed/installed next to each node cluster

![](https://k3s.io/img/how-it-works-k3s-revised.svg)

# Install Rancher

To install Rancher within a VM, execute the following command:

```bash
curl -sfL https://get.k3s.io | sh -s - server
sudo chmod -R 644 /etc/rancher/k3s/k3s.yaml
k3s kubectl get node
NAME           STATUS   ROLES                  AGE     VERSION
k122-centos7   Ready    control-plane,master   3m26s   v1.23.6+k3s1
```
It will install a linux service to start/stop the k3s server like different tools or scripts:
- k3s client
- kubectl
- crictl
- ctr
- k3s-killall.sh
- k3s-uninstall.sh

**Note**: The kubectl app installed by K3s will automatically use a newly created kubeconfig file written to `/etc/rancher/k3s/k3s.yaml`.

Next, change the permissions to access the k3s kubeconfig file and copy it to `~~.kube/config`
```bash
sudo chmod -R 644 /etc/rancher/k3s/k3s.yaml
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```
Install now different packages such as: Certificate manager, Rancher UI, fleet, etc:

```bash
IP=65.108.212.158
RANCHER_ADMIN_PASSWORD=k3s-admin

kubectl create ns cattle-system

kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.5.1

helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=$IP.sslip.io \
  --set replicas=1 \
  --set bootstrapPassword=$RANCHER_ADMIN_PASSWORD
```

You can now open the URL of the dashboard to create new clusters using the address: https://<IP>.sslip.io/
and password passed as parameter to the Helm command.

![](image/dashboard.png)

# Tips

To get the kubeconfig and access it from another machine
```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

# Cleanup

To uninstall
```bash
/usr/local/bin/k3s-killall.sh
/usr/local/bin/k3s-uninstall.sh
```
