
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

Jenkins: https://hub.helm.sh/charts/codecentric/jenkins
Nexus:
SonarQube:


### Let's start deploying

Make sure that VirtualBox is running.
run minikube 
```
$ minikube start
```

check that all is well
```
$ kubectl get nodes
```
The output should show you the basic cluster info consists with one node.
