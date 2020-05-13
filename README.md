This is a repository where I execute all my demos.

# Instructions:

First of all we are going to create our environment, to that we need install: Docker, Jenkins and Microk8s.

## Docker install

With this script the docker install is automated.

~~~
curl -fsSL https://get.docker.com | sudo bash
~~~

If you are using Ubuntu 20.04:

~~~
apt update && apt install -y docker.io 
~~~

Add your user to docker group

~~~
sudo usermod -aG docker $USER
~~~

## Jenkins Deploy

We go to use docker to deploy Jenkins:

~~~
docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
~~~

Go inside localhost:8080 and do this command in shell:

~~~
docker exec <docker_container> cat /var/jenkins_home/secrets/initialAdminPassword
~~~

Insert the password, click in install with suggest plugins and configure user.


## Minikube Deploy

Now we are going to download the minikube binary and we have to give execution permissions.

~~~
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && mv minikube /usr/local/bin/
~~~

Now we proceed to deploy minikube:

~~~
minikube start
~~~

And install kubectl to operate the cluster:

~~~
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
~~~

## Create shared library

Now we are going to create a shared library, for thiws we need a repository named `jenkinsfile-shared-library` with the next structure:

~~~
--src/com/mfranco/
--vars
--resources
~~~

Nex step is fork my repository:

[Repository](https://github.com/manu756/demo_repository)

This is a example app with resources to deploy it in k8s and build docker image.

> Disclaimer!
>  
> Explain Jenkinsfile

## Map our repository to Jenkins

Now we are going to map our repository to Jenkins to build it.

To do that you need to go to:

* New Job
* Multibranch pipeline (Dont forget the name!)
* Click on add source -> Github
* Add our credentials
* Add repository url

Two ways of add shared library.

* In job folder
* General option

* And Save

That will scan our repository looking for Jenkinsfile, if it found one, that branch will be execute.

# Kubernetes agents

Now we are going to create jenkins agents in k8s:

For this we need to install the plugin `kubernetes plugin` for this we must to go to plugins install new and install and reboot.

Now in system configuration at the bottom we click on cloud configuration.

To do that we need create a ServiceAccount to jenkins, this is in `manifests/serviceAccount.yaml` and apply it with: 

`kubectl apply -f manifests/serviceAccount.yaml`

Now, need the token and certificated created for that execute the next commands:

For Token:


`SECRET=$(kubectl get secrets | grep jenkins | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.token}' | base64 -d`

For Certificate:

`SECRET=$(kubectl get secrets | grep jenkins | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.ca\.crt}' | base64 -d`

Insert Certificate in the kubernetes plugin and token in secret text to use later.

Now we are going to create a kubeconfig file to can access to our cluster with the agents to deploy pods:

Here we need the template:

`manifests/kubeconfig-template.yaml`

Before in the serviceAccount.yaml we creat another SA named deployer, so, now we are going to take the token and the certificate-authority-data:

`SECRET=$(kubectl get secrets | grep deployer | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.token}' | base64 -d`

`SECRET=$(kubectl get secrets | grep deployer | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.ca\.crt}'`

Now we need to create a secret with the kubeconfig.

Finally we are going to create podTemplates:

Create resource library and put in podTemplates.

## For Demo

In this case we are going to use docker containers like agents:

First we have to create a agent in our localhost.

This will made with ssh key and a jenkins user.
