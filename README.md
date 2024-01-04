# Deploying-an-application-to-Kubernetes

# About Kubernetes:
Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers. It groups containers that make up an application into logical units for easy management and discovery. Kubernetes was originally designed by Google and is now maintained by the Cloud Native Computing Foundation. Here are some key aspects of Kubernetes:

# Container Orchestration:
Kubernetes manages containers, usually Docker containers, but it also supports other container runtimes. It handles the deployment and management of containers on a cluster of machines.
# Cluster Management: 
It operates across a cluster of machines, providing high availability and scalability. A cluster consists of at least one master machine and multiple worker nodes.
# Pods: 
The basic scheduling unit in Kubernetes is a pod. A pod is a group of one or more containers, with shared storage/network, and a specification for how to run the containers.
# Services and Labels:
Kubernetes uses labels and selectors to organize and group pods. Services in Kubernetes are an abstraction which defines a logical set of pods and a policy by which to access them.
# Self-healing:
Kubernetes can restart containers that fail, replace and reschedule containers when nodes die, and kills containers that don’t respond to user-defined health checks.
# Scalability:
Kubernetes allows you to automatically scale your application up and down based on demand.
# Automated rollouts and rollbacks: 
You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate.
# Load Balancing: 
Kubernetes can distribute network traffic so that the deployment is stable.
# Storage Orchestration: 
Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
Secret and Configuration Management: Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys.

We are going to need 2 EC2 instances. Please go over to the EC2 console and create 2 instances with the following specifications:

Leave name blank as we will be creating 2 instances.

t2.large Ubuntu 20.04 LTS

20 Gib of storage

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/478675e7-fb26-4d50-a9d7-c22ce8ca05b0)

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/0bbb5479-ac41-482e-bcf4-a9cf3bad3c7f)

Click on Launch instance. You can now provide names for the instances. I will be naming mine K8-Master Node and K8-Slave Node.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/ed3c9f7f-b7e2-4ae9-a73b-5fce38b4b57b)

Add port 6443 to the security group we have been using.

Installing Kubernetes:
Step 1: SSH into the master node and slave nodes and run the following commands as sudo one by one in both nodes:

apt-get update -y

apt-get install docker.io -y

service docker restart

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >/etc/apt/sources.list.d/kubernetes.list

apt-get update

apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y

Step 2: On the Master Node, run:

kubeadm init --pod-network-cidr=192.168.0.0/16

Make sure you copy the kubeadm join command after running Step 2 and paste it into the Slave Node SSH session.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/89d067d1-8a6f-42cb-a977-e8d5e395be28)

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/c00823ce-312f-40e4-a742-71c98620bcf7)

Step 3: Run as the root user in Master Node:

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

Step 4: Run the following as the root user on the Master Node:

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml

Now to view our nodes we can type:

kubectl get nodes

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/ce734510-4879-4417-bf1d-c34f36baf869)

Let’s take a look at what namespaces exist at the moment:

kubectl get namespace

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/a9e09a09-d9df-42a9-8a41-645d15e8a4eb)

Deploying an application:
In the master node, create a directory where we can store the Manifest YAML files needed to deploy the application, then CD into it. EX:

mkdir sam

Let’s create the Manifest by typing vi deploymentservice.yaml

Copy and paste the code below into the file and save it.

apiVersion: apps/v1
kind: Deployment # Kubernetes resource kind we are creating
metadata:
  name: spring-boot-k8s-deployment
spec:
  selector:
    matchLabels:
      app: spring-boot-k8s
  replicas: 2 # Number of replicas that will be created for this deployment
  template:
    metadata:
      labels:
        app: spring-boot-k8s
    spec:
      containers:
        - name: spring-boot-k8s
          image: adijaiswal/shopping-cart:latest # Image that will be used to containers in the cluster
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8070 # The port that the container is running on in the cluster


---

apiVersion: v1 # Kubernetes API version
kind: Service # Kubernetes resource kind we are creating
metadata: # Metadata of the resource kind we are creating
  name: springboot-k8ssvc
spec:
  selector:
    app: spring-boot-k8s
  ports:
    - protocol: "TCP"
      port: 8070 # The port that the service is running on in the cluster
      targetPort: 8070 # The port exposed by the service
  type: NodePort # the type of service.
Now we are going to run the deployment command (make sure you are in the same directory where the YAML file is located):

kubectl apply -f deploymentservice.yaml
To view the pods we just created, run:

kubectl get pods

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/55a088b2-6e86-4a76-aa21-4afcb88b4556)

We can see the two pods that we defined inside of the deployment YAML file.

Before testing the application, we need to get the port number. We can achieve this by running the following command:

kubectl get svc
Look for the NodePort. In my case it was 30769/TCP. Let’s test the application by pasting in the Slave Node IP:30769.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/a2a92845-d336-4ce6-a67e-032185d1c6a5)

The credentials to log in are:

Username: admin

Password: admin

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/8c14ba1b-1638-4271-99a0-67a8de3bda3e)

Kubernetes + Jenkins:
Let’s install Jenkins on the master node.

sudo apt install openjdk-11-jre -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y 

sudo apt-get install jenkins -y

sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins
To locate the Jenkins admin password let’s type:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Now log into Jenkins and create a new pipeline.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/2d073a93-c718-40c3-9230-39f873d96c09)

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/dc95f047-b180-4ef6-b5c8-13ab9353c681)

Copy and paste the pipeline script below:

pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        
         stage('Kubernetes Deploy') {
            steps {
                sh "echo password12345 | sudo -S kubectl get nodes"
            }
        }
    }
}
In the pipeline script we are going to echo the password for the Jenkins user we need to create next. Please be advised that this is not secure at all and in a production environment, it is best to use Jenkins credentials or any other method that does not display passwords.

To create the user type visudo and add jenkins ALL=(ALL:ALL) ALL to the file.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/f78ba14e-91b6-4c96-8ccf-816929348a03)

Save the file then type passwd jenkins to set a password. This password needs to be added into the pipeline script.

After that, run the pipeline build and it should succeed. Also, all the nodes should display at the end.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/51757021-6701-43ae-8382-ad621cba462b)

Go back to the pipeline script and overwrite it with the below:

pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        
         stage('Kubernetes Deploy') {
            steps {
                sh "echo password12345 | sudo -S kubectl apply -f deploymentservice.yml"
                sh "echo password12345 | sudo -S kubectl get pods"
                sh "echo password12345 | sudo -S kubectl get svc"
            }
        }
    }
}
Run the build again.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/8e18c1dc-2d86-4dd4-9219-797ae0c290ae)

We can view the application using the same Slave node IP:and the NodePort.

![image](https://github.com/fazalUllah-Khan/Deploying-an-application-to-Kubernetes/assets/148821704/4d102c76-9a1c-444c-91c4-fe7f7c758e8d)



reference:
https://towardsdev.com/devops-lab-8-deploying-an-application-to-kubernetes-c9e3411b4a0 




