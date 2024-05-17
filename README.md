# Yelp Camp Web Application

This web application allows users to add, view, access, and rate campgrounds by location. It is based on "The Web Developer Bootcamp" by Colt Steele, but includes several modifications and bug fixes. The application leverages a variety of technologies and packages, such as:

- **Node.js with Express**: Used for the web server.
- **Bootstrap**: For front-end design.
- **Mapbox**: Provides a fancy cluster map.
- **MongoDB Atlas**: Serves as the database.
- **Passport package with local strategy**: For authentication and authorization.
- **Cloudinary**: Used for cloud-based image storage.
- **Helmet**: Enhances application security.
- ...

## Setup Instructions

To get this application up and running, you'll need to set up accounts with Cloudinary, Mapbox, and MongoDB Atlas. Once these are set up, create a `.env` file in the same folder as `app.js`. This file should contain the following configurations:

```sh
CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
CLOUDINARY_KEY=[Your Cloudinary Key]
CLOUDINARY_SECRET=[Your Cloudinary Secret]
MAPBOX_TOKEN=[Your Mapbox Token]
DB_URL=[Your MongoDB Atlas Connection URL]
SECRET=[Your Chosen Secret Key] # This can be any value you prefer
```

After configuring the .env file, you can start the project by running:
```sh
docker compose up
```

## Application Screenshots
![](./images/home.jpg)
![](./images/campgrounds.jpg)
![](./images/register.jpg)


## CI IMPLEMENTATION


#### To Install Jenkins With Bash Script

- create a file ".sh" eg jen.sh

- paste the command needed in the file

```sh
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

- make the file executable

```sh
sudo chmod +x jen.sh
```

- execute with this command

```sh
./jen.sh
```

![](https://github.com/Emmylong1/3-tier-app-project/assets/137091610/7775d984-18ab-4e1c-afe8-5cb250bab874)


#### Jenkins Configuration

- tap on suggested plugins, always.

![](https://github.com/Emmylong1/3-tier-app-project/assets/137091610/ae3bff8c-e975-4d23-a2fb-4f8a6a24b451)


- go to manage jenkins > available plugins to install [ docker, docker pipeline,kubernetes, kubernetes cli, nodejs,sonarqube scanner ]


![](https://github.com/Emmylong1/3-tier-app-project/assets/137091610/b569bcb7-56d2-4384-8e60-99ed67fa8396)


- next we need to install sonarqube and docker on sonarqube server check image below for command


![](https://github.com/Emmylong1/3-tier-app-project/assets/137091610/b1aadebc-01f8-496e-8e00-31cc9d445654)


- on your jenkins server install docker too and trivy


##### Sonarqube Configuration

- go to your aws and copy the public IP of the sonarqube server and attach the port :9000 and paste on your webapp


![](https://github.com/Emmylong1/3-tier-app-project/assets/137091610/61f62e2c-6fb5-44fc-9c5b-28f94947f79e)


- to access it make use of " admin " as username and password thus you could change it after.


- go to administration > user > and generate a token, copy and save it .

![](https://github.com/UzonduEgbombah/3-tier-app-project/assets/137091610/3b185b56-8542-4980-bfa8-890e056161be)


- on your jenkins webapps go to credentials and <global> create a credential,select kind and choose secret and in secret paste the sonarqube token you generated and use 
  sonar-token as ID SAVE.


![](https://github.com/UzonduEgbombah/3-tier-app-project/assets/Screenshot 2024-05-16 165628.png)

  
- next go to systems and search for sonarqube add and use the name sonar > copy your sonarqube url and paste without the "/"  > now under server authentication select the 
  sonar-token we created earlier.


![](https://github.com/UzonduEgbombah/3-tier-app-project/assets/137091610/a0486103-19c9-45aa-8145-9f06ae2d3f66)


- also in systems search for nodejs add and save the name with "node21" 


- now create a new pipeline with any name of your choice and scroll down to where you see pipeline syntax, please tap and select xxxx generator > in steps select and scroll 
  down find "with Docker Registry" and select > you would see where to add username and password please use your dockerhub username and password, also ID should be
  "Docker- cred" then apply and save. go back > under registry credential select the docker-cred you just created.


![](https://github.com/UzonduEgbombah/3-tier-app-project/assets/137091610/06247332-16b8-475b-bd16-607fa8dd6de4)


- after saving generate the pipeline syntax you should expect something similar to this below 
  " withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { "
  > you should copy it and apply to your script watchout for where it should be from my script below.

- in a boxy space right above [generate pipeline syntax] paste your script and build !


```sh
pipeline {
    agent any

    tools {
        nodejs 'node21'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: ''
            }
        }
        
        stage('Install package Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
    
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
            }
        }
        
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t xxxx:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format=json -o fs-report.json xxxx:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push xxxx:latest"
                    }
                }
            }    
        }
        
        stage('Docker Deploy To Dev') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 xxxx:latest"
                    }
                }
            }    
        }        
    }
}
```

## My Pipeline

![](https://github.com/UzonduEgbombah/3-tier-app-project/assets/137091610/db52e8f9-84e4-4d85-b4d7-0c4aac51c3fc)




## COMMANDS YOU MIGHT NEED

#### command to install nodejs

installs NVM (Node Version Manager)

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
#### download and install Node.js

```sh
nvm install 22
```

#### verifies the right Node.js version is in the environment

```sh
node -v # should print `v22.1.0`
```

#### link to install jenkins

```sh
jenkins.io/doc/book/installing/
```

```sh          
import {v2 as cloudinary} from 'cloudinary';
          
cloudinary.config({ 
  cloud_name: '', 
  api_key: '', 
  api_secret: '' 
});
```




1. Install AWS CLI:
2. 
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
aws --version
``` 

2. Install kubectl:

```sh
curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.22.2/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --short --client
```   

3. Install eksctl:

```sh
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin/
eksctl version
```  


Creating EKS Cluster

```sh
eksctl create cluster --name=my-eks2 \
 --region=us-east-1 \
 --zones=us-east-1a,us-east-1b \
 --without-nodegroup

eksctl utils associate-iam-oidc-provider \
 --region us-east-1 \
 --cluster my-eks2 \
 --approve

eksctl create nodegroup --cluster=my-eks2 \
 --region=us-east-1 \
 --name=node2 \
 --node-type=t3.medium \
 --nodes=3 \
 --nodes-min=2 \
 --nodes-max=4 \
 --node-volume-size=20 \
 --ssh-access \
 --ssh-public-key=devops \
 --managed \
 --asg-access \
 --external-dns-access \
 --full-ecr-access \
 --appmesh-access \
 --alb-ingress-access
```

Create Service Account, Role & Assign that role, And create a secret for Service Account and geenrate a Token
Creating Service Account

```sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```


Create Role

```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - secrets
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```


#### Bind the role to service account

```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: jenkins
``` 


Generate token using service account in the namespace

Create Token

[Non-expiring Service Account Tokens](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)


  



















