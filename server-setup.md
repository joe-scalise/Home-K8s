Setup a single-node Kubernets cluster with MicroK8s on spare hardware.

For this guide, I was using a Dell inspiron Laptop


Install kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl'
kubectl version --client
```

Install git

```
sudo apt install git-all
```

Install Helm

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

Tell Ubuntu to do nothing when the lid closes:
sudo gedit /etc/systemd/logind.conf
HandleLidSwitch=ignore – do nothing
systemctl restart systemd-logind.service
