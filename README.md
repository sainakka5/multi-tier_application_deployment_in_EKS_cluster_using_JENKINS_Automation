
 ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/ef89ba00-12c2-4b61-96d9-f19ec233fb2c)
 
PROJECT -5 : 
KUBERNETES CLUSTER OPTIMIZATION AND SCALING FOR MICROSERVICES ARCHITECTURE
STEPS TO CREATE - Kubernetes Cluster Optimization and Scaling for Microservices Architecture and making pods :
1. Create an EC2 instance in AWS console, I create an instance with name “Jenkins_project” and   the instance resource must be min t2.medium with 20gb of volume storage.

   ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/b60a66d1-efa9-4820-a862-a4988e631435)

3. Install the Jenkins and its requirements in that instance using Jenkins installation code in Jenkins documentary for centos 7.
 
        sudo yum install wget -y
        sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
        sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
        yum install fontconfig java-11-openjdk -y
        yum install jenkins -y
        service jenkins start
        systemctl enable jenkins
        service jenkins status 

3.  After execution of the above script, enable the http (8080) port in your Jenkins server security group then login into Jenkins with your ip address like, give your <public_ip:8080> on your browser and give the authentication password, which you can find it in path :  < /var/jenkins_home/secrets/initialAdminPassword > of Jenkins server give that password and login to the Jenkins webserver.

Create a repository in GitHub and give the all files of microservice application in that repo and copy the URL of that repository. I had saved in my GitHub repository https://github.com/sainakka5/multi-tier_application_deployment_files

   ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/43f2db4f-c00d-4cd8-9ec5-a02b2d502ef2)
     
5.	Login into the putty terminal and install the required packages to create EKS cluster , the requirements are docker ,kubectl , eksctl and awscli into the instance. Create a IAM ROLE and attach it to this instance, in that IAM ROLE give full administration access to that role.
	Docker installation script, it automatically install the packages of docker.

        sudo yum install -y yum-utils &&
        sudo yum-config-manager \
           --add-repo \
           https://download.docker.com/linux/centos/docker-ce.repo &&
        sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y &&
        sudo service docker start &&
        sudo systemctl enable docker.service &&
        sudo service docker status

	KUBECTL, AWSCLI AND EKSCTL installation script.

      yum install zip unzip wget git -y
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      export PATH="/usr/local/bin:$PATH"
      echo export PATH="/usr/local/bin:$PATH" >> ~/.bashrc
      
      curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.7/2022-10-31/bin/linux/amd64/kubectl
      chmod +x ./kubectl
      mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
      echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
      
      curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
      sudo mv /tmp/eksctl /usr/local/bin
      export PATH="/usr/local/bin:$PATH"
      echo PATH="/usr/local/bin:$PATH" >> ~/.bashrc
      eksctl version
      kubectl version 

5.	Create dockerfiles for multi-tier application like frontend, backend and database files and build images from that dockerfile (docker build -t <imagename> -f <filename> .) and push them to the dockerhub repository with the commands given below. Created images are shown in figures below.

docker login
docker tag <local-image>:tag <yourusername/yourrepository:tag>
docker push <yourusername/yourrepository:tag>

    docker tag frontend:latest sainakka/appdeploy:frontend-tag1 
    docker tag backend:latest sainakka/appdeploy:backend-tag2 
    docker tag database:latest sainakka/appdeploy:database-tag4
    docker push sainakka/appdeploy:frontend-tag1
    docker push sainakka/appdeploy:backend-tag2
    docker push sainakka/appdeploy:database-tag4
    
  ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/e46dee88-0c5a-4ce7-833d-59bf1d2f292a)
  ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/ce17c2f1-55fb-4957-aeac-bfc44d39cb6b)    
  
6.	Now create the deployment and service files from this images with having load balancer in these files and also create StatefulSets, and DaemonSets files for making high availability cluster.here are my files that created for front,back,database,statefulset and deamonset deployment.
 ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/c9aa5e4a-3c4e-4003-ba74-b1e9b0af403f)

7.	Here is the explaination of all files given in this directory to deploy an application into the cluster
Statefulset : StatefulSets are used for managing stateful applications that require stable network identities, stable storage, and ordered pod creation and deletion. StatefulSets often use PersistentVolumeClaims (PVCs) to provide persistent storage for each pod. StatefulSets allow you to scale pods up and down while maintaining the ordinal number and identity of pods.

DaemonSet: DaemonSets are used to ensure that one instance of a pod runs on every node in the cluster, and it's commonly used for cluster-level services or agents. DaemonSets are useful for running node-level services like log collectors, monitoring agents, or networking services. Pods managed by DaemonSets are automatically placed on nodes as they join the cluster and removed when nodes are decommissioned. While DaemonSets automatically scale up, they do not automatically scale down when nodes are removed from the cluster.

Prometheu: Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It is primarily used for collecting time-series data from various sources, including applications, systems, and services.it simply monitors the application /cluster and give the alerts to the kubernetes.

Grafana: Grafana is an open-source, feature-rich metrics dashboard and visualization tool. It provides a flexible and user-friendly interface for querying, visualizing, and alerting on data from various sources, including Prometheus.it simply shows charts, tabular form of the prometheus metrics.

ELK STACK :
Elasticsearch : Elasticsearch is a powerful, distributed search and analytics engine. It is designed to handle large volumes of data and provides fast and flexible search capabilities. Elasticsearch stores log and event data in a structured way, making it easy to search, filter, and analyze logs. It can ingest and index log data in real-time, allowing you to perform near-instant searches and analysis.

Logstash: Logstash is a data collection and log ingestion tool. It can collect log data from various sources, such as log files, syslog, or databases, and normalize it for further processing. Logstash allows you to transform and enrich log data, making it more structured and standardized before sending it to Elasticsearch. : Logstash can send the processed log data to Elasticsearch for indexing and storage.

Kibana: Kibana is a web-based visualization and dashboarding tool. It allows you to create interactive and customizable dashboards to visualize log data from Elasticsearch. You can use Kibana to search and query log data stored in Elasticsearch using a user-friendly interface. Kibana also provides alerting features, allowing you to set up notifications based on log data patterns or thresholds.

9.	You can see the scripts of all files in my github url : https://github.com/sainakka5/multi-tier_application_deployment_files
I had given the script according to the images from dockerhub, these scripts make the application more high availability , monitoring and centralized logging using tools like ELK Stack.
  ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/778b2805-7c8e-4a5b-94b0-899c9b54b11d)
       
10.	After saving all deployment and service files, now create a cluster with autoscaling and high availability make workernode type t2.medium which is a minimum requirement cluster.

        ## cluster creation script ##
        apiVersion: eksctl.io/v1alpha5
        kind: ClusterConfig
        metadata:
          name: sai-cluster
          region: ap-south-1
        
        availabilityZones: ["ap-south-1a", "ap-south-1b"]  # Define your desired availability zones here
        
        nodeGroups:
          - name: my-nodes
            instanceType: t2.medium ## medium##
            desiredCapacity: 2  # Increase desired capacity for high availability
            minSize: 2  # Ensure minimum size is at least 2 for high availability
            maxSize: 3
            privateNetworking: true
            iam:
              withAddonPolicies:
                autoScaler: true
            labels:
              instance-type: onDemand
            ssh:
        publicKeyName: key
  	
	By saving the script in cluster.yaml file and give the command to create a cluster in EKS 

    eksctl create cluster -f cluster.yaml
    
this command create a cluster with 2 workernodes (min-2 / max-3) autoscaling.  

 ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/e0fc5341-7ac4-47ec-8443-d974c1b41b92)

We can see the cluster ,workernodes and cloud formation in aws console 
    
  ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/2299d60b-b953-4a9b-9fd0-e03b17d379ab)
   
   ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/43792096-0c82-4629-96e7-49fb8a8fdcea)

             
10.	After creation of cluster deploy all files of application into the cluster with this command using kubectl
kubectl apply -f front.yaml -f frontserv.yaml -f back.yaml -f backserv.yaml -f database.yaml -f databaseserv.yaml -f prometheus.yaml -f grafana.yaml -f statefulset.yaml -f deamonset.yaml -f elastic.yaml -f log.yaml -f kibana.yaml
11.	After deploying, it creates pods and replicas in the cluster by giving commands
   kubectl get pods 
kubectl get all
it will show the running status of all pods and services if one pod gets crashed it will automatically create pods again without errors or discontinuity of application
we can see the pods status and services in the below figure mentioned

  ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/e3c2f973-0d0a-4c8a-b1bf-0717c97990d9)
                 
13.	We can check the application live running in load balancer through aws console, we get the DNS for front, back and database of the application. By searching the DNS in internet browser with ports declaration and suitable security group to that load balancer.
    
 ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/56c6b8be-e532-4a31-939e-712133c44552)

![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/c4807b7a-79de-4b64-b142-ebdcdcbb03f3)

![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/2a0bd443-afa9-47fb-8336-8f4833b7f709)
     
It was successfully running the application the cluster without any problem by seeing the above figures

STEPS TO CREATE CLUSTER AND DEPLOYMENT OF APPLICATIONS WITH JENKINS PIPELINE AUTOMATION
1.	In this project we are giving automation of above process in Jenkins pipeline, for this create a repository and push the all files from instance to GITHUB repository. By git action commands

        git clone <github repo url >
        git add .
        git commit -m “message”
        git push -u origin 

![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/61cfdde4-6fdf-44f7-ac57-fef3bea8fbe2)

 
2.	By this repository we can call the files to Jenkins pipeline and create cluster and deployment of application with groovy script. For this create a pipeline and give the below script
   
        ## jenkins automation script for cluster app deployment 
        pipeline {
            agent any
            environment {
                DOCKERHUB_USERNAME = 'sainakka'
                DOCKERHUB_PASSWORD = credentials('sainakka')
                PATH = "/root/bin:$PATH"
            }
        
            stages {
                stage('git') {
                    steps {
                        // Check out your GitHub repository
                        git branch: 'main', url: 'https://github.com/sainakka5/multi-tier_application_deployment_files.git'
                    }
                }
        
                stage('Build and Push Image') {
                    steps {
                        script {
                            
                                sh 'docker build -t front-image -f front .'
                                sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                                sh 'docker tag front-image:latest sainakka/appdeploy:frontend-tag1'
                                sh 'docker push sainakka/appdeploy:frontend-tag1'
                                
                                sh 'docker build -t back-image -f backend .'
                                sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                                sh 'docker tag back-image:latest sainakka/appdeploy:backend-tag4'
                                sh 'docker push sainakka/appdeploy:backend-tag4'
                                
                                sh 'docker build -t database-image -f database .'
                                sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                                sh 'docker tag database-image:latest sainakka/appdeploy:database-tag3'
                                sh 'docker push sainakka/appdeploy:database-tag3'  
                                }
                           }
                    }
        
                stage('Create EKS Cluster') {
                    steps {
                        script {
                            sh 'eksctl create cluster -f cluster.yaml'
                        }
                    }
                }
        
                stage('Create deployment') {
                    steps {
                        script {
                            env.PATH = "/root/bin:$env.PATH"
                            sh '/root/bin/kubectl apply -f front.yaml -f frontserv.yaml -f back.yaml -f backserv.yaml -f database.yaml -f databaseserv.yaml -f prometheus.yaml -f grafana.yaml -f statefulset.yaml -f deamonset.yaml -f elastic.yaml -f log.yaml -f kibana.yaml'
                        }
                    }
                }
        
            }
          }

3.	By saving the script and build the pipeline, it will create docker images from docker file , and pushes them to dockerhub repo , and deploy the application with that docker images in the deployment file.

   ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/aa72964d-395b-4f20-90a3-124a433dff02)
   ![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/fcbc1c5a-0df4-4689-aae2-3cbe11efd447)
   
  By this figures it shows the status of pipeline stages that were build successfully in the EKS cluster 

4.	We can check this by seeing load balancer in the aws console and search the respective DNS in web browser
I had checked the loaadbalancer and DNS in web browser these are the results that occurred in my project
Below figures are the results of my full automation jenkins pipeline

![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/be1b5235-b231-4900-b974-8210572fdbbe)
  
6.	We can see monitoring, health check of the server ,loadbalancing, security groups of loadbalancer,
Status of nodes in the loadbalancer dashboard in AWS console.
Below figures are the running status of application in servers
        
![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/346eff97-7acc-40cf-a931-f2a7d5372292)
![image](https://github.com/sainakka5/multi-tier_application_deployment_in_EKS_cluster_using_JENKINS_Automation/assets/136338958/be448207-8d8c-4dd5-9a2e-18d4da0e1759)
      

	By this project I gained experience on designing, deploying, and optimizing a Kubernetes cluster for a complex microservices-based application. 
	The goal is to handle real-world challenges in Kubernetes. And make the whole process into automation in jenkins pipeline.
