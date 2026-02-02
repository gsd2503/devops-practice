# Maven - Java - Docker Swarm

Maven+Java+Docker swarm (Manual/On-demand Pipeline)  : Design and implement a CI/CD pipeline for a java application built uisng Maven. Use Git/Github for version control, Jenkins for pipeline execution, and Docker for containerization. Deploy the containerized application on Docker Swarm and explain each stage of the pipeline during execution.
 
# Install Java 17

### Update Package List and Install Java

    sudo apt update
    sudo apt install openjdk-17-jdk -y
    java -version
 
 ### If multiple Java versions exist - set java 17 as default:
Go to:

    sudo update-alternatives --config java
Select:

    /usr/lib/jvm/java-17-openjdk-amd64/bin/java
To set Javac:

    sudo update-alternatives --config javac

### Set Java Home (Compulsory):
Open:

    nano ~/.bashrc

At the end add:

    export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
    export PATH=$JAVA_HOME/bin:$PATH

Apply changes:

    source ~/.bashrc

Verify:

    echo $JAVA_HOME


# How to Uninstall Java from Your System

### 1. Check Installed Java Versions

    java --version
    dpkg -l | grep -i java

### 2. Remove OpenJDK (most common) 

    sudo apt remove --purge openjdk-\* -y

if you know the exact version:

    sudo apt remove --purge openjdk-17-jdk -y

### 3. Remove Oracle Java (if installed)

    sudo apt remove --purge oracle-java\* -y

### 4. Remove Left over java files:

    sudo rm -rf /usr/lib/jvm
(Optional but not recomended)

    sudo apt autoremove -y
    sudo apt autoclean

### 5. Verify Java is removed

    java --version
  
  You should see: `command not found`

# Install Jenkins

    sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
    echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee 
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt update
    sudo apt install jenkins

**Start Jenkins**

    sudo systemctl enable jenkins

**Start Jenkins service**

    sudo systemctl start jenkins
  
  **Check Status**
  
    sudo systemctl status jenkins

## Change Jenkins Port

    YOURPORT=8080
    PERM="--permanent"
    SERV="$PERM --service=jenkins"
    
    firewall-cmd $PERM --new-service=jenkins
    firewall-cmd $SERV --set-short="Jenkins ports"
    firewall-cmd $SERV --set-description="Jenkins port exceptions"
    firewall-cmd $SERV --add-port=$YOURPORT/tcp
    firewall-cmd $PERM --add-service=jenkins
    firewall-cmd --zone=public --add-service=http --permanent
    firewall-cmd --reload

## By Pass the Jenkins Default Login

    sudo systemctl stop jenkins
**Edit the config file:**

    sudo nano /var/lib/jenkins/config.xml
**Find the line:**

    <useSecurity>true</useSecurity>
**Change it to:**

    <useSecurity>false</useSecurity>
 **If you see authorization blocks, remove:**

    <authorizationStrategy>
    <securityRealm>
**Restart Jenkins:**

    sudo systemctl start jenkins
**Open Local Host:**

    http://localhost:8080
**Note:** Directly press login, without giving any credentials.

**Now Navigate to:**

    Manage Jenkins -> Security
   * Under Authentication section: Change **Security Realm** from **None** --> **Jenkin's own user database**
   * Click the check box True to **Allow users to sign up**
   * Under Authorization section: Change **Anyone can do anything** --> **Logged-in User can do anything**
 **Now Save and reload**
 -> Jenkins will reload and you will see the page to regiter a new user.
 -> Create a new user and login as that user.
 **Note:** uncheck the check box from True to False for **Allow users to sign up**

## Install Maven

    sudo apt update
    sudo apt install maven -y
    mvn --version
 
## Step 1: Create Maven Project

    java --version
    mvn --version

**Generate Maven Project**

    mvn archetype:generate \
    -DgroupId=com.example \
    -DartifactId=my-app \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false
 **Open pom.xml file and edit:**

    cd my-app
    nano pom.xml
  **Add this inside pom.xml file before </ project> and after </ dependencies>** 

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <release>17</release>
                </configuration>
            </plugin>
            
        </plugins>
    </build>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

### Next:
**Compile:**

    mvn clean compile
**Create Package:**

    mvn package

**Run:**

    java -cp target/my-app-1.0-SNAPSHOT.jar com.example.App
    java -jar target/my-app-1.0-SNAPSHOT.jar
**Output:**

    Hello World
**Test:**

    mvn test
   
# Step 2: Push Porject on to GitHub
### Create a Public Access Tocken (Mandatory)
* Click on the profile --> Settings --> Left side at last find **Developer Settings**
* Developer Settings --> Personal Access Tokens --> **Tokens (Classic)**
* Tokens (Classic) --> Generate New Token (Classic)
	Give Note / Name: **Devops-Test**
	Select Scopes --> **repo** (Mandatory)
	Generate Token --> Copy and keep it safe in a notepad
### Go to Project folder

    cd java17-maven-app
### Verify

    ls
**Output should be:**

    pom.xml
    src/
### Initialize Git Repository

    git init
### Add Git ignore (Its Important)

    nano .gitignore
**Inside Git ignore paste the below code:**

    target/
    *.log
    *.class
    .idea/
    .vscode/
    *.iml
   Save and Exit
### Commit the files to git hub

    git add .
    git commit -m "Initial commit: Java 17 Maven project"

### Create a repository on the github
**Note: Do not add any readme or .gitignore files on the repo**

### Connect remote repo to local repo

    git remote add origin https://github.com/<your-username>/java17-maven-app.git
    git branch -M main
    git push -u origin main

# Step 3: Jenkins for Automation

## Configure Java and Maven in Jenkins
* Go to **Manage Jenkins** --> Tools
* Under JDK installations --> click add jdk

    Name: Java17
    JAVA_HOME: /usr/lib/jvm/java-17-openjdk-amd64

* Under Maven Installations --> click add maven
	Name: Maven3
	Click Install Automaitcally
	Select the latest version
* Save and go to Jenkins Dashboard

## Create a pipeline project

* Go to Dashboard
* Click new Item
*  Give Name: **Maven-Java-Docker-Swarm**
* Select Item Type as **Pipeline**
* Save

## Configure Pipeline project
* Pipeline --> Definition --> **Pipeline Script from SCM**
* SCM --> **Git**
* Repository URl --> **< your-github-repo-url >**
* Credentials --> add credentials
* **kind** --> **username with password**
* username --> **< github-username>**
* password --> **previously-copied-github-PAT** (Personal access token)
* ID --> **Devops-test**
* Save and Exit

## Add Jenkins File 
**create jenkinsfile**

    nano Jenkinsfile
   **Content**

    pipeline {
        agent any
    
        tools {
            jdk 'Java17'
            maven 'Maven3'
        }
    
        environment {
            DOCKERHUB_USER = "<docker-hub-id"
            IMAGE_NAME     = "<image-name>"
            IMAGE_TAG      = "latest"
            FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
    
        stages {
    
            stage('Checkout') {
                steps {
                    git branch: 'main',
                        url: '<git-repo-name>',
                        credentialsId: '<git-credentials-in-jenkins>'
                }
            }
    
            stage('Build JAR') {
                steps {
                    sh 'mvn clean package'
                }
            }
    
            stage('Verify JAR') {
                steps {
                    sh 'java -jar target/*.jar'
                }
            }
    
            stage('Docker Build') {
                steps {
                    sh 'docker build -t $FULL_IMAGE .'
                }
            }
    
            stage('Docker Login') {
                steps {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        '''
                    }
                }
            }
    
            stage('Docker Push') {
                steps {
                    sh 'docker push $FULL_IMAGE'
                }
            }
        }
    
        post {
            success {
                echo "✅ Image pushed to Docker Hub: $FULL_IMAGE"
            }
            always {
                sh 'docker logout || true'
            }
        }
    }


# Step 4: Docker Setup:
### Install Docker
**Check the version**

    docker --version
 **If docker not available:**

    sudo apt update
    sudo apt install docker.io -y
    sudo systemctl start docker
    sudo systemctl enable docker

**Do this mandatorily**

    sudo usermod -aG docker $USER
    newgrp docker
    sudo usermod -aG docker jenkins
    sudo systemctl restart jenkins

### Create the Dockerfile

    nano Dockerfile
   **Insert the Content below into the File**

    FROM maven:3.9.9-eclipse-temurin-17 AS build
    WORKDIR /app
    COPY pom.xml .
    COPY src ./src
    RUN mvn clean package -DskipTests
    
    FROM eclipse-temurin:17-jre
    WORKDIR /app
    COPY --from=build /app/target/*jar app.jar
    CMD ["java", "-jar", "app.jar"]
 
  ### Push Dockerfile and Jenkinsfile on to github
  

    git add .
    git commit -m "Dockerfile and jenkinsfile"
    git push -u origin main
  
 ### Build Image Localy
**Build** 

    docker build -t my-app .
**Run**

    docker run --rm my-app
  **Output**

    Hello World!

  



