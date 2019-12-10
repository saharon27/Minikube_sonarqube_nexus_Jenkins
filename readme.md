
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

All the needed charts are in this repo. They were changed to fit the need of this project.
Here is the list of the originals:

* Jenkins: https://github.com/helm/charts/tree/master/stable/jenkins
* Nexus: https://hub.helm.sh/charts/choerodon/nexus3
* SonarQube: https://hub.helm.sh/charts/stable/sonarqube


### Let's start deploying

It is recommended to change the default config of minikube to better suit the needs of this assignment. This can be done by updating the minikube config file or by running the following commands:
```
$ minikube config set memory 6144
$ minikube config set cpus 6
$ minikube config set disk-size 16000MB
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

Let's deploy our charts:
#### Nexus:
```
helm install nexus3 ./nexus3-0.2.0/nexus3/
```

Execute the following command to get address of Nexus service:
```
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" svc nexus-nexus3)
export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT/
```
To get initial admin password run the following command:
```
kubectl exec --namespace default "{POD-NAME}" cat /nexus-data/admin.password
```

#### Jenkins
```
helm install jenkins ./jenkins-1.9.7/jenkins/
```

Use the following command to retrieve admin password:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=jenkins,app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")
kubectl exec --namespace default "$POD_NAME" cat /var/jenkins_home/secrets/initialAdminPassword
```
Accessing your Jenkins server:
```
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" svc jenkins)
export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT/
```
#### Sonarqube
```
helm install sonarqube ./sonarqube-3.2.5/sonarqube/
```
Accessing your SonarQube server:
```
export POD_NAME=$(kubectl get pods --namespace default -l "app=sonarqube,release=sonarqube" -o jsonpath="{.items[0].metadata.name}")
echo "Visit http://127.0.0.1:9000 to use your application"
kubectl port-forward $POD_NAME 9000:9000
```
The default user is sonaUser and pass is sonarPass
