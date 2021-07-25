# Creation a Helm Chart for Grafana

Welcome to my article. You will see all about the integration of Dockerfile, Helm, Grafana, etc, in this article. So let's get started without delay.
## Pre-requisite
To perform this scenario you will need mentioned platform.
* [AWS Account](https://aws.amazon.com/console/)


## Kubernetes Setup
To demonstrate this scenario, first of all, we have to install the Kubernetes setup then we can move ahead for further part. So I am installing the Kubernetes cluster on the top of AWS. let's launch the instance with mentioned configuration.
* Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
* t2.xlarge Instance type
* Minimum Storage 20 GiB

After launching AWS Instance, connect it with the help of any remote software eg. putty, etc, or ssh protocol and then follow the following steps.

:small_orange_diamond: login with root power.

```
sudo su -
```

:small_orange_diamond: Install kubectl.

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

:small_orange_diamond: Update the instance and install docker.

```
sudo apt-get update -y
sudo apt-get install docker.io -y
```
:small_orange_diamond: Install curl software to install Minikube.

```
sudo apt-get install curl -y
```
### What's Minikube ?
Minikube is a utility you can use to run Kubernetes (k8s) on your local machine. It creates a single node cluster contained in a virtual machine (VM). This cluster lets you demo Kubernetes operations without requiring the time and resource-consuming installation of full-blown K8s.

:small_orange_diamond: So let's install Minikube.

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo chmod +x minikube
sudo mv minikube /usr/local/bin/
sudo apt install conntrack
```
:warning: *Note*: Now go into sudo if not gone.

```
sudo -i
```
:small_orange_diamond: Start the Minikube

```
minikube start --vm-driver=none
```
:small_orange_diamond: Our single node cluster is ready to use so run the below command to check the minikube status.
```
minikube status
```
:diamond_shape_with_a_dot_inside: you will get output such as:

```
root@ip-172-31-39-130:~# minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
Hopefully, your installation will had been completed. So now time is to move towards creating a Container image for Grafana. But before creating it, let's try to understand Grafana and Dockerfile.

### What's Grafana ?
Grafana is a [multi-platform open source](https://en.wikipedia.org/wiki/Multi-platform) analytics and [interactive visualization](https://en.wikipedia.org/wiki/Interactive_visualization) web application. It provides charts, graphs, and alerts for the web when connected to supported data sources and End users can create complex monitoring dashboards using interactive query builders. Grafana is divided into a front end and back end, written in TypeScript and Go, respectively.

### What's Dockerfile ?
Dockerfile is a simple text file that consists of instructions to build Docker images. Mentioned below is the syntax of a Dockerfile to creating Grafana Docker image.


:small_orange_diamond: So create a file as Dockerfile and Ensure D should be capital in Dockerfile.

```
vim Dockerfile
```
and write the below code inside Dockerfile.

```
FROM centos:7
RUN yum install wget -y
RUN wget https://dl.grafana.com/oss/release/grafana-7.0.3-1.x86_64.rpm
RUN yum install grafana-7.0.3-1.x86_64.rpm -y
WORKDIR /usr/share/grafana
CMD /usr/sbin/grafana-server start && /usr/sbin/grafana-server enable && /bin/bash
```
:small_orange_diamond: After writing the code, build the docker image with the help of following command.

```
docker build -t username/imagename:version .
```
> ``eg. # docker build -t hackcoderr/grafana:v1 .
``

:small_orange_diamond:  Now login to your DockerHub Account. But if you haven't DockerHub Account then first of all create it then move ahead with below command.

```
docker login
```
:small_orange_diamond: After it, just push your image to DockerHub so that you can use it in the future.

```
docker push username/imagename:version 
```
:diamond_shape_with_a_dot_inside: After running above command, the output should be similar to the following:
```
root@ip-172-31-39-130:~# docker push hackcoderr/grafana:v1
The push refers to repository [docker.io/hackcoderr/grafana]
4b603ec3a2e0: Pushed
a1be9f0c6dee: Pushed
9d1af48bd5b4: Pushed
174f56854903: Mounted from library/centos
v1: digest: sha256:6bd02f99b6e905582286b344980b2f83c75348876a58eb15786fd5baab04ce0b size: 1166
```

:diamond_shape_with_a_dot_inside: But if you don't want to create this container image then you can simply pull my pre-created image from the *Dockerhub* with help of ``docker pull`` command.

```
docker pull hackcoderr/grafana:v1
```
:warning: We will use this image in the upcoming steps when we will create ``deployment.yaml`` file in the Helm chart. But before it, let's know about Helm.

### What's Helm ?
So let's try to understand what helm is?
* Helm is package manager for Kubernetes
* Helm packages are called Charts.
* Helm Charts help define, install and upgrade complex Kubernetes application.
* Helm Charts can be versioned, shared, and published.
* Helm Charts can accept input parameter.
    * Kubectl need template engine to do this (Kubernetes, jinja etc)
* Popular packages already available.

Now let's see how we can install Helm in the cluster.

:small_orange_diamond: Run the below commands to install Helm.

```
wget https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz
tar -xvzf helm-v3.5.2-linux-amd64.tar.gz

```
:diamond_shape_with_a_dot_inside: The output should be similar to the following, after running the ``tar`` command.

```
root@ip-172-31-39-130:~# tar -xvzf helm-v3.5.2-linux-amd64.tar.gz
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md
```

:small_orange_diamond: After this, copy ``linux-amd64/helm`` file in the ``/usr/bin/``.

```
cp linux-amd64/helm /usr/bin/
```
:diamond_shape_with_a_dot_inside: Also, Ensure that Helm is installed or not with the help of the ``helm version`` and the output should be similar to the following:

```
root@ip-172-31-39-130:~# helm version
version.BuildInfo{Version:"v3.5.2", GitCommit:"167aac70832d3a384f65f9745335e9fb40169dc2", GitTreeState:"dirty", GoVersion:"go1.15.7"}
```

#### Create a Helm Chart

let's create a new Helm Chart from the scratch. Helm created a bunch of files for you that are usually important for a production-ready service in Kubernetes. To concentrate on the most important parts, we can remove a lot of the created files. Let’s go through the only required files for this example.

![Helm file structure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8bg2vbgdbu7687novaki.jpg)


:small_orange_diamond: Create a Helm Chart for Grafana.

```
mkdir grafana
cd grafana
```
:diamond_shape_with_a_dot_inside: But here we need a project file that is called Chart.yaml and contains all the metadata information. 
:small_orange_diamond: So, create this file. Also, C should be capital in the ``Chart.yaml``.

```
vim Chart.yaml
```
:small_orange_diamond: Write the below code inside ``Chart.yaml``.

```
apiVersion: v2
name: Grafana
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: 1.16.0
```
:small_orange_diamond: Make a ``templates`` folder inside ``grafana`` and go inside it.

```
mkdir templates
cd templates
```

:small_orange_diamond: Use this command to create a code of the ``deployment.yaml`` file.

```
kubectl create deployment grafana --image=hackcoderr/grafana:v1 --dry-run -o yaml > deployment.yaml
```
:diamond_shape_with_a_dot_inside: The output should be similar to the following:

```
root@ip-172-31-39-130:~/grafana/templates# kubectl create deployment grafana --image=hackcoderr/grafana:v1 --dry-run -o yaml > deployment.yaml
W0420 18:01:34.362388   20835 helpers.go:557] --dry-run is deprecated and can be replaced with --dry-run=client.
```
:small_orange_diamond: Go outside the ``grafana`` directory and install the helm chart.
```
cd 
helm install grafana grafana/
```
:warning: Here ``grafana`` is the name of Helm Chart and ``grafana/`` is the path of the chart. 

:diamond_shape_with_a_dot_inside: After running above command, the output should be similar to the following:

```
root@ip-172-31-44-81:~# helm install grafana grafana/
NAME: grafana
LAST DEPLOYED: Tue Apr 20 18:10:44 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

:small_orange_diamond: Now again go inside `` grafana/templates`` and use the below command to create a code of the ``service.yaml`` file.

```
cd grafana/templates/
kubectl expose deployment grafana --port=3000 --type=NodePort --dry-run -o yaml > service.yaml
```
:diamond_shape_with_a_dot_inside: The output should be similar to the following:

```
root@ip-172-31-39-130:~/grafana/templates# kubectl expose deployment grafana --port=3000 --type=NodePort --dry-run -o yaml > service.yaml
W0420 18:12:55.289972   23635 helpers.go:557] --dry-run is deprecated and can be replaced with --dry-run=client.
```

:small_orange_diamond: After it, run the below command for exposing the ``grafana`` pod.

```
kubectl apply -f service.yaml
```
:diamond_shape_with_a_dot_inside: To ensure your pod is working well with the below commands.

```
kubectl get pods
kubectl get deployment
kubectl get svc
```
:diamond_shape_with_a_dot_inside: After running these commands, the output should be similar to the following:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y7h5wbhkvswrdlw7hi5a.jpg)

You can check the list of the helm you have using the ``helm list`` command. Now we can check the pods in which slaves are running in the Kubernetes cluster.

```
root@ip-172-31-39-130:~# helm list
NAME    NAMESPACE       REVISION        UPDATED                                STATUS   CHART           APP VERSION
grafana default         1               2021-04-20 18:10:44.189498387 +0000 UTCdeployed Grafana-0.1.0   1.16.0
```

Hopefully, Everything is working well till yet. So let's check that Grafana is working fine or not. For this, you have to take the ``public_ip_of_instance`` and ``port no.`` of your pod.
> ``eg.  15.207.72.25:31130``

Where ``13.233.237.202`` is the public IP of my instance that contains all the setup which I have done till yet and ``31130`` is the port no. of ``grafana`` pod which you can see in the above screenshot after running ``kubectl get svc`` command. So browse it.

:diamond_shape_with_a_dot_inside: After browsing it, login page will pop up. So login with by-default username and password ``admin``.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ipxwk7fnmej4yuxvqb74.jpg)
:warning: After logging you will get the page for changing the username and password, So you can if you want.



![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g24oqkaa44grybrl5w5n.jpg)

Now Grafana is ready to use. So let's enjoy :relaxed:

#### Packing resources inside the Helm Chart.

So helm chart ready inside the ``grafana/`` directory, but we can’t publish it as it is. Firstly, we have to create a package for this helm chart.

:small_orange_diamond: Create one directory named ``charts``. Make sure this directory should be inside ``grafana`` directory.

```
mkdir charts/
```
:small_orange_diamond: Now, run the following command to packages the chart and store it inside the charts/ directory.
```
helm package /root/grafana -d charts/
```

:diamond_shape_with_a_dot_inside: you will get output something like this:

```
root@ip-172-31-44-81:~/grafana# helm package /root/grafana -d charts/
Successfully packaged chart and saved it to: charts/Grafana-0.1.0.tgz
```
#### Creating an index.yaml file.

For every Helm repository, we must require an index.yaml file. The index.yaml file contains the information about the chart that is present inside the current repository/directory.

:small_orange_diamond: For generating ``index.yaml`` file inside ``charts/`` directory, run following command.

```
helm repo index charts/
```
:diamond_shape_with_a_dot_inside: You can see an ``index.yaml`` file generated with the details of the chart.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dhociga8qk4shp2xyr0h.jpg)

#### Hosting the chart with GitHub pages
Now everything is fine in our helm chart we can publish this to [ArtifcatHub](https://artifacthub.io/) so that we can use it in the future when we require it. 

But before we have to host this chart anywhere. So that we can publish it to ArtifactHub. So I am going to host it with the GitHub page. but before doing this, We have to push the chart to Github. So let's follow the following steps.

:small_orange_diamond: Install git and config cluster with your GitHub account.

```
sudo apt-get install git -y
git config --global user.name 'username'
git config --global user.email 'usermail@gmail.com'
```
:small_orange_diamond: After installing ``git`` go inside ``grafana`` directory and then initialize it with the help of below command.

```
git init
```

:small_orange_diamond: Now add, commit then push this directory to GitHub.

```
git add .
git commit -m "give any msg according to you"
git branch -M main
git remote add origin URL
git push -u origin main
```
:small_orange_diamond: After pushing it, go to the Github repository and click on ``setting`` then ``GitHub Pages``.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nl14kiypwvrosfiadxc2.jpg)

:warning: To activate the GitHub page, you have to select ``branch`` first which you want to activate then ``save`` it.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cpx7mydtxe9lkamurju0.jpg)

#### Publishing the Helm chart to ArtifactHub

Artifact Hub includes a fine-grained authorization mechanism that allows organizations to define what actions can be performed by their members. It is based on customizable authorization policies that are enforced by the Open Policy Agent. So go to your [ArtifactHub Account](https://artifacthub.io/) and login to it.

:small_orange_diamond: Now click on ``profile icon > control Panel> Add repository``.

:warning: Make sure your repository's url should be like ``https://username.github.io/repository_name/chart/`` when you're adding repository in ArtifactHub. Otherwise, it can create some issues related to ``url``. 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7q5837mznaju9etiwsxm.jpg)

:small_orange_diamond: After giving the required information to your Helm repository, click on ``Add``. if you had provided the right information then it will create a Helm repository.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/172jmtzuwhb4l3zyla3n.jpg)

So, we had successfully published our Helm chart to the [ArtifactHub](https://artifacthub.io/).

### Conclusion

The power of a great templating engine and the possibility of executing releases, upgrades, and rollbacks makes Helm great. On top of that comes the publicly available Helm Chart Hub that contains thousands of production-ready templates. This makes Helm a must-have tool in your toolbox if you work with Kubernetes on a bigger scale!
