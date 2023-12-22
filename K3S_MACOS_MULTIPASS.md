# lists_k3s

## k3s on MacOS
- Uses Canoncial Multipass

## Multipass

### Install Multipass
```
brew install --cask multipass
```

### Create k3s nodes with Mulitpass
```
multipass launch --name k3s01 --mem 1G --disk 5G --network en0
multipass launch --name k3s02 --mem 1G --disk 5G --network en0
multipass launch --name k3s03 --mem 1G --disk 5G --network en0
```

## Setup k3s

### Configure master & worker nodes
```
multipass exec k3s01 -- bash -c "curl -sfL https://get.k3s.io | sh -"
URL_IP=`multipass info k3s01 | grep ^IP | awk '{print $2}'`
NODE_TOKEN=`multipass exec k3s01 -- bash -c "sudo cat /var/lib/rancher/k3s/server/node-token"`
multipass exec k3s02 -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=https://${URL_IP}:6443 K3S_TOKEN=${NODE_TOKEN} sh -"
multipass exec k3s03 -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=https://${URL_IP}:6443 K3S_TOKEN=${NODE_TOKEN} sh -"
multipass exec k3s01 -- bash -c "sudo kubectl get nodes"
```

### Add k3s cluster to kubectl config
Assumes kubecm is installed

```
multipass exec k3s01 -- bash -c "sudo cat /etc/rancher/k3s/k3s.yaml" > k3s-kubeconfig.conf
sed -i '' "s/127.0.0.1/${URL_IP}/g" k3s-kubeconfig.conf
kubecm add -f k3s-kubeconfig.conf
kubecm list
```

