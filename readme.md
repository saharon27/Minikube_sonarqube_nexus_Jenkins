
# Small maven CI/CD pipeline using minikube/Sonarqube/Nexus and Jenkins

## Background

This project uses the following MVN project [https://github.com/zivkashtan/course] to demonstrate Jenkins pipeline that compiles the code,runnning SonarQube test and upload war to Nexus, create docker image and upload it to Nexus registry.
All will run locally using minikube and HELM for deployemnt.
The Jenkins deployment will utilize K8s plugin to create dynamic slave-nodes that are using Docker.sock mount to overcome Docker in Docker problem.

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
* Nexus: https://hub.helm.sh/charts/stable/sonatype-nexus
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
helm install nexus3 ./sonatype-nexus-1.21.1/sonatype-nexus/
```

Nexus is deployed as stateful set over Ingress so you can reach it by going to http://nexus.minikube/

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

### Some comments on working with the deployment:

* It is recommended to change passwords once deployed.
* It is important to supply Jenkin k8s plugin with a .pfx file to allow it to create pods in Minikube cluster.use this      instructions:
```
The client certificate needs to be converted to PKCS, will need a password

openssl pkcs12 -export -out ~/.minikube/minikube.pfx -inkey ~/.minikube/apiserver.key -in ~/.minikube/apiserver.crt -certfile ~/.minikube/ca.crt -passout pass:secret

Validate that the certificates work

curl --cacert ~/.minikube/ca.crt --cert ~/.minikube/minikube.pfx:secret --cert-type P12 https://$(minikube ip):8443

Add a Jenkins credential of type certificate, upload it from ~/.minikube/minikube.pfx, password secret

Fill Kubernetes server certificate key with the contents of ~/.minikube/ca.crt
```
* For Jenkins to work with SonarQube:
  * You need to define token in SonarQube and apply it as secret text cred in Jenkins.
  * If using Port-Forwarding with the SonarQube deployment than in Jenkins Configure system you need to put ClusterIP or         Service name.
  * It is recommended to create web hook to Jenkins in SonarQube server.
* For Jenkins to work with Nexus:
  * You need to congiure the nexus server in Jenkins system and apply creds.
