
# Small maven CI/CD pipeline using minikube/Sonarqube/Nexus and Jenkins

## Background

This project uses the following MVN project [https://github.com/zivkashtan/course] to demonstrate Jenkins pipeline that compiles the code,runnning SonarQube test and upload war to Nexus, create docker image and upload it to Nexus registry.
All will run locally using minikube and HELM for deployemnt.

## Getting Started

### Prequisites:

1. Install a virtualization software (I used VirtualBox - https://www.virtualbox.org/wiki/Downloads).
2. install kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/
3. Install Docker - use apt-get or snap
4. Install minikube - https://minikube.sigs.k8s.io/docs/start/linux/
5. Install Helm - https://helm.sh/docs/intro/install/


### Next we will need the Charts for Helm:

* Jenkins: https://hub.helm.sh/charts/codecentric/jenkins
* Nexus: https://hub.helm.sh/charts/choerodon/nexus3
* SonarQube:


### Let's start deploying

It is recommended to change the default config of minikube to better suit the needs of this assignment. This can be done by updating the minikube config file or by running the following commands:
```
$ minikube config set memory 6144
$ minikube config set cpus 6
$ minikube config set disk-size 8000MB
```
run minikube 
```
$ minikube start
```
check that all is well
```
$ kubectl get nodes
```
The output should show you the basic cluster info consists with one node.

 #### Execute the following command to route the connection for Nexus service:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" svc nexus-nexus3)
  export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/
  
  #### Jenkins
  #### Use the following command to retrieve admin password:

export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=jenkins,app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")
kubectl exec --namespace default "$POD_NAME" cat /var/jenkins_home/secrets/initialAdminPassword


#### Accessing your Jenkins server:

Create port forwarding to access Jenkins at http://127.0.0.1:8080

export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=jenkins,app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace default "$POD_NAME" 8080:8080

