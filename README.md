
# 🚀 CI/CD Pipeline using Jenkins, SonarQube, Nexus, and Tomcat on AWS (Ubuntu)

## 📘 Overview

This project demonstrates a **complete CI/CD pipeline** setup using **Jenkins** as the orchestrator, integrating **SonarQube** for code quality analysis, **Nexus** as an artifact repository, and **Tomcat** for deployment — all hosted on **AWS EC2 Ubuntu servers**.

The pipeline automates the process of:

* Building a Java web application with Maven
* Analyzing code quality using SonarQube
* Storing build artifacts in Nexus
* Deploying automatically to a Tomcat server

---

## 🏗️ Architecture Diagram

```
       ┌──────────────┐
       │  Developer   │
       │ (Git Commit) │
       └──────┬───────┘
              │
              ▼
       ┌──────────────┐
       │   Jenkins     │
       │ (CI Orchestrator)
       └──────┬────────┘
              │
 ┌────────────┼────────────┐
 │            │            │
 ▼            ▼            ▼
SonarQube   Nexus       Tomcat
(Code QL)  (Artifacts) (Deployment)
```

---

## 🧰 Tech Stack / Tools Used

| Tool                 | Purpose                                         |
| -------------------- | ----------------------------------------------- |
| **AWS EC2 (Ubuntu)** | Host servers                                    |
| **Jenkins**          | Continuous Integration (Pipeline Orchestration) |
| **Maven**            | Build automation                                |
| **SonarQube**        | Code Quality & Static Analysis                  |
| **Nexus**            | Artifact Repository Manager                     |
| **Tomcat**           | Application Deployment                          |
| **Java 17 / 21**     | Runtime environment                             |

---

## ⚙️ Setup and Configuration Steps

### 🖥️ 1. Create EC2 Instances

| Server    | Instance Type | Ports Opened | Description            |
| --------- | ------------- | ------------ | ---------------------- |
| Jenkins   | t2.micro      | 22, 8080     | Main CI server         |
| SonarQube | t2.large      | 22, 9000     | Code analysis          |
| Nexus     | t2.medium     | 22, 8081     | Artifact storage       |
| Tomcat    | t2.micro      | 22, 8080     | Application deployment |

Each server uses Ubuntu as the OS and a unique key pair for SSH.

---

### 🧩 2. SSH into Each Server & Set Hostnames

```bash
ssh -i <your-key.pem> ubuntu@<public-ip>
sudo hostnamectl set-hostname <server-name>
sudo init 6  # Reboot to apply hostname
```

Example:

```
Jenkins-lab
SonarQube-lab
Nexus-lab
Tomcat-lab
```

---

### ☕ 3. Install Java and Required Tools

#### Jenkins Server:

```bash
sudo apt update -y
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

#### Tomcat Server:

```bash
sudo apt-get update -y
sudo apt-get install -y openjdk-17-jdk
java -version
```

#### SonarQube Server:

```bash
sudo apt update -y
sudo apt install openjdk-17-jre-headless maven -y
java -version
mvn -v
```

#### Nexus Server:

```bash
sudo apt update -y
sudo apt install openjdk-17-jre-headless -y
java -version
```

---

## 🧱 4. Jenkins Installation and Configuration

### Install Jenkins:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
```

### Start and Enable Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### Access Jenkins:

Open in browser:

```
http://<JENKINS_PUBLIC_IP>:8080
```

Unlock Jenkins:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Paste the password → Install suggested plugins → Create admin user → Jenkins dashboard ready ✅

---

## 🧩 5. Configure SonarQube

```bash
cd /home/ubuntu
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.6.0.92116.zip
unzip sonarqube-10.6.0.92116.zip
mv sonarqube-10.6.0.92116 sonarqube
cd sonarqube/bin/linux-x86-64
./sonar.sh start
```

Access:

```
http://<SONAR_PUBLIC_IP>:9000
```

Default credentials:

```
admin / admin
```

Change password when prompted.

---

## 📦 6. Configure Nexus Repository

```bash
cd /home/ubuntu
wget https://download.sonatype.com/nexus/3/nexus-3.85.0-03-linux-x86_64.tar.gz
tar -xvzf nexus-3.85.0-03-linux-x86_64.tar.gz
cd nexus-3.85.0-03
bin/nexus start
bin/nexus status
```

Access:

```
http://<NEXUS_PUBLIC_IP>:8081
```

Default login:

```
Username: admin
Password: (found in /home/ubuntu/sonatype-work/nexus3/admin.password)
```

Change password and configure hosted repository.

---

## 🌐 7. Configure Tomcat Server

```bash
cd /home/ubuntu
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.109/bin/apache-tomcat-9.0.109.tar.gz
tar -xvzf apache-tomcat-9.0.109.tar.gz
cd apache-tomcat-9.0.109/bin
./startup.sh
```

Verify:

```
http://<TOMCAT_PUBLIC_IP>:8080
```

If default Tomcat page appears — ✅ Tomcat is running.

---

### Optional: Add Tomcat Manager User

```bash
sudo vi /home/ubuntu/apache-tomcat-9.0.109/conf/tomcat-users.xml
```

Add:

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="admin" roles="manager-gui,manager-script"/>
```

Restart Tomcat:

```bash
./shutdown.sh
./startup.sh
```

---

## 🔑 8. Jenkins Integrations and Credentials

Install Jenkins plugins:

* GitHub Integration
* GitHub Authentication
* Maven Integration
* SonarQube Scanner
* Quality Gates
* Nexus Artifact Uploader
* Publish over SSH
* SSH Agent / SSH Pipeline Steps

---

### Configure Jenkins Global Tools

* **Java** → `/usr/lib/jvm/java-17-openjdk-amd64/bin/java`
* **Maven** → `/usr/share/maven`

```bash
readlink -f $(which java)
```

Add both in Jenkins → Manage Jenkins → Global Tool Configuration.

---

### SSH Setup Between Jenkins and Other Nodes

On each worker server:

```bash
mkdir Jenkins
```

On Jenkins master:

```bash
sudo su jenkins
ssh-keygen -t rsa
```

Copy public key (`/var/lib/jenkins/.ssh/id_rsa.pub`) to all worker servers in:

```
/home/ubuntu/.ssh/authorized_keys
```

Verify SSH access:

```bash
ssh ubuntu@<worker-server-public-ip>
```

---

### Add Jenkins Credentials

1. **Sonar Token** → Manage Jenkins → Credentials → Add Secret Text
2. **Tomcat & Nexus** → Add username/password
3. **SSH Key** → Add “SSH Username with private key”

Then, in **Manage Jenkins → Configure System**, add:

* SonarQube server name + URL + token.

---

## ⚙️ 9. Configure Jenkins Nodes

In Jenkins dashboard:

* Go to **Manage Jenkins → Nodes**
* Add:

  * Sonar-node
  * Nexus-node
  * Tomcat-node

Assign labels for each and connect successfully.

---

## 🧪 10. Create Pipeline (Pipeline-as-Code)

Define the stages:

* **Build** → Compile with Maven
* **Test** → Unit testing
* **Code Quality** → SonarQube Analysis
* **Artifact Upload** → Push WAR to Nexus
* **Deploy** → Transfer WAR to Tomcat
* **Monitor** → Log success

Each stage runs sequentially across configured nodes.

---

## 📸 Screenshots

*(Replace these paths with your actual image links or files)*

* ![Jenkins Dashboard](screenshots/jenkins_dashboard.png)
* ![SonarQube Analysis](screenshots/sonarqube_analysis.png)
* ![Nexus Repository](screenshots/nexus_repo.png)
* ![Tomcat Deployment](screenshots/tomcat_deploy.png)

---

## ✅ Results

* Jenkins automated build and deployment across all servers.
* Code scanned by SonarQube with quality gate validation.
* Artifact stored in Nexus for version control.
* WAR deployed automatically to Tomcat server.
* Fully functional CI/CD pipeline built on AWS EC2.

---

## 🧾 Conclusion

This project demonstrates a **real-world CI/CD pipeline** integrating all major DevOps components — Jenkins, SonarQube, Nexus, and Tomcat.
It provides automation from **code commit to deployment**, ensuring quality, consistency, and speed in the software delivery process.

---

## 👤 Author

**Developed by Sai**
💼 DevOps Enthusiast | AWS | Jenkins | CI/CD Automation

---

Would you like me to:

* 🪶 Add **badges** (like build status, tech stack, etc.)
* 💡 Or include a **sample `Jenkinsfile`** snippet for your pipeline section?

That’ll make your README even more impressive for GitHub or interview sharing.
