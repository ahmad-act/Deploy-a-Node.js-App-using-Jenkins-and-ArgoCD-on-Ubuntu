# Deploy a Node.js App using Jenkins and ArgoCD on Ubuntu

## Install docker

1. Run the commands:
```bash
sudo apt update
sudo apt upgrade
sudo apt-get install -y apt-utils
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
sudo usermod -aG sudo jenkins
```
2. Log Out and Log Back In:
- Apply the changes to your user group by logging out and logging back in.
3. Check Docker Version:
```bash
docker –version
```
4. Check Docker Service Status:
```bash
sudo systemctl status docker
```
5. Start Docker Service (if needed):
```bash
sudo systemctl start docker
```
6. Enable Docker Service to Start on Boot:
```bash
sudo systemctl enable docker
```
7. Run a Test Container:
```bash
sudo docker run hello-world
```

## Install kubectl

1. Download the latest release of kubectl:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
2. Make the kubectl binary executable:
```bash
chmod +x kubectl
```
3. Move the binary in to your PATH:
```bash
sudo mv kubectl /usr/local/bin/
```
4. Test to ensure the version you installed is up-to-date:
```bash
kubectl version –client
```

## Install minikube

1. Download the latest release of minikube:
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
2. Install minikube:
```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
3. Verify minikube Installation:
```bash
minikube start --driver=docker
```
4. Start minikube:
```bash
minikube start --driver=docker
```
5. Verify minikube image:
```bash
docker images
```

## Verify the Installations

1. docker:
```bash
docker –version
```
2. kubectl:
```bash
kubectl version --client
```
3. minikube:
```bash
minikube version
```
4. Check if kubectl can connect to your minikube cluster:
```bash
kubectl get nodes
```
- This should output details about your minikube node, indicating that both kubectl and minikube are installed and working correctly.

## Troubleshooting Tips

- Permissions: Ensure you run the commands with appropriate permissions (sudo where necessary).
- Dependencies: Make sure you have the required dependencies installed, such as curl and virtualbox for minikube (if you are using VirtualBox as the driver).
- Environment Variables: If kubectl or minikube commands are not found, make sure /usr/local/bin is in your PATH.

## Install Git

### First, install Git to manage your source code repositories:

```bash
sudo apt-get install git
```

## Install Jenkins

### 1. Install Java

#### Jenkins requires Java to run:
```bash
sudo apt install fontconfig openjdk-17-jre
```

### 2. Add Jenkins Repository and Install Jenkins

<https://www.jenkins.io/doc/book/installing/linux/>
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
```

### 3. Obtain Jenkins IP Address

#### To determine the IP address Jenkins is using, run:

```bash
ip addr show
```
#### Identify the IP address associated with your primary network interface. For example, 192.168.1.249.

Output:

```bash
“1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:2e:5c:b8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.249/24 metric 100 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 84118sec preferred_lft 84118sec
    inet6 fe80::a00:27ff:fe2e:5cb8/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:1f:e9:a8:07 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-8b7fc4737f90: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:d5:8b:e1:e1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.49.1/24 brd 192.168.49.255 scope global br-8b7fc4737f90
       valid_lft forever preferred_lft forever”
```

#### Explanation:

- lo (loopback): 127.0.0.1 (not accessible from outside the VM)
- enp0s3: 192.168.1.249 (likely the primary interface, accessible from the network)
- docker0: 172.17.0.1 (Docker bridge interface)
- br-8b7fc4737f90: 192.168.49.1 (Docker bridge network)

#### Based on the typical use case, Jenkins usually binds to the primary network interface of the VM. In the case, it is most likely 192.168.1.249.


### 4. Configure Firewall

#### Ensure the firewall allows traffic on Jenkins' default port (8080):

```bash
sudo ufw allow 8080/tcp
sudo ufw enable
```

### 5. Retrieve Initial Admin Password

#### To get the initial admin password for Jenkins:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 6. Access Jenkins
#### Open Jenkins in your browser at http://Jenkins-IP:8080 and follow the setup wizard.

### 7. Install Docker Pipeline Plugin on Jenkins UI
#### Install the Docker Pipeline plugin via the Jenkins plugin manager.

### 8. Restart Jenkins

#### To restart Jenkins, use the following URL:

```text
http://<Jenkins-IP>:8080/restart
```

## Install ArgoCD

### 1. Install ArgoCD

#### Create the ArgoCD namespace and install ArgoCD:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Verify Installation

####  Check the status of ArgoCD pods:

```bash
kubectl get pods -n argocd
kubectl get all -n argocd
```

#### Wait for all pods to be in a running state.

### 3. Run ArgoCD UI

#### To find the ArgoCD server's port:

```bash
kubectl get svc -n argocd
```

Output:
```test
argocd-server   ClusterIP   10.106.2.123    <none>  80/TCP,443/TCP
```

#### Forward the port to make the ArgoCD UI accessible:

#### kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8088:443


### 4. Access ArgoCD UI

#### Open the ArgoCD UI in your browser at https://Jenkins-IP:8088.

### 5. Initial Login

#### Default username: 

```text
admin
```

#### Retrieve the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

#### Output:

```text
apiVersion: v1
data:
  password: anZlUU9hVGN5VExEVW9xSA==
kind: Secret
metadata:
  creationTimestamp: "2024-07-08T17:06:16Z"
  name: argocd-initial-admin-secret
  namespace: argocd
  resourceVersion: "37829"
  uid: 8f4f3456-b811-4066-9590-a0b21aabd8b8
type: Opaque
```

#### Decode the password:

```bash
echo anZlUU9hVGN5VExEVW9xSA== | base64 –decode
```

Output:
```text
jveQOaTcyTLDUoqH
```

#### Use the username admin and the decoded password to log in.

## Apply ArgoCD Application Configuration

### 1. Clone Application Manifest Repository

### Clone the repository containing ArgoCD manifests for the Node.js application:

```bash
git clone https://github.com/awsaf-utm/ArgoCD-Manifests-For-Node.js-App.git
cd ArgoCD-Manifests-For-Node.js-App
```

### 2. Apply Application Configuration
 
#### To apply ArgoCD YAML (application.yaml):

```bash
kubectl apply -n argocd -f application.yaml
```

### 3. Now check on ArgoCD UI

#### https://Jenkins-IP:8088
